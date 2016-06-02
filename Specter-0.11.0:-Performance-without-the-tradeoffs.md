While Specter has had great performance for a long time, taking advantage of that performance required writing code slightly less elegantly than one would prefer. With Specter 0.11.0, that is no longer the case. Now Specter feels more like an extension to the language than just a library (though it is just a library). This post will provide a comprehensive overview of what Specter is doing behind the scenes to achieve its amazing performance and what you need to know as a user to enable Specter to do its optimizations.

Let's start by taking a look at two comparative micro-benchmarks that show a taste of what has been accomplished in 0.11.0 (`benchmark` times how long it takes to run the given function that many times):

```clojure
user=> (benchmark 10000000 #(get-in {:a {:b {:c 1}}} [:a :b :c]))
"Elapsed time: 1018.388 msecs"

user=> (benchmark 10000000 #(select [:a :b :c] {:a {:b {:c 1}}}))
"Elapsed time: 739.439 msecs"


user=> (benchmark 10000000 #(update-in {:a {:b {:c 1}}} [:a :b :c] inc))
"Elapsed time: 10986.831 msecs"

user=> (benchmark 10000000 #(transform [:a :b :c] inc {:a {:b {:c 1}}}))
"Elapsed time: 1975.379 msecs"
```

In each case we're comparing a Clojure built-in function against the equivalent functionality in Specter. The code is effectively identical in each case, yet for these examples Specter is 25% faster for querying and 5x faster for transforming. These are impressive performance numbers even without considering the fact that Specter is dramatically more generic than `get-in` and `update-in`, functions which are highly specialized to one particular use case.

(If you're already familiar with Specter, you know that prior to 0.11.0 you would need to manually precompile the path to get high performance. That is no longer the case – more details on this later.)

These examples are actually two of the easier cases for Specter to handle with high performance as the path `[:a :b :c]` is completely known at compile-time. Take a look at these more challenging examples that Specter also handles with great performance:

```clojure
(defn dynamic-example1 [k data]
  (transform [ALL (keypath k)] inc data))

(defn dynamic-example2 [akey apred data]
  (select [(keypath (str akey "!"))
           (selected? ALL (pred apred))
           LAST]
    data))
```

In both these cases, the path is dependent on dynamic parameters. In the second case, one of the parameters is for a path that itself is a parameter for the `selected?` navigator. Unlike the `[:a :b :c]` case, these paths cannot be known until they are constructed at runtime – and the paths can change on every invocation. So Specter seemingly has a lot less ability to do work ahead of time or re-use work for future invocations. The key word there is "seemingly", because most of the work that went into the internals of Specter was to make that statement false.

There are three aspects to how Specter achieves its speed:

1. Compilation of paths (introduced in version 0.5.0)
2. Late-bound parameterization (introduced in version 0.7.0)
3. Inline factoring and caching (introduced in version 0.11.0)

Let's take a look at these one by one to gain a complete understanding of Specter's performance.

## Compilation of paths

At the core of Specter is the `comp-paths` function. It takes a path and produces a compiled version of that path that can be executed very quickly. For example:

```clojure
;; compile path on every invocation
user=> (benchmark 1000000 #(compiled-select (comp-paths :a :b :c) {:a {:b {:c 1}}}))
"Elapsed time: 4371.032 msecs"

;; rely on inline caching
user=> (benchmark 1000000 #(select [:a :b :c] {:a {:b {:c 1}}}))
"Elapsed time: 77.552 msecs"

;; manual precompilation
user=> (def p (comp-paths :a :b :c))
user=> (benchmark 1000000 #(compiled-select p {:a {:b {:c 1}}}))
"Elapsed time: 64.097 msecs"
```

Manually precompiling a path and then using it later is a bit faster than specifying the path inline, and over a 60x improvement to compiling the path on every usage.

`comp-paths` performs a number of optimizations. For the `[:a :b :c]` case for transforms, it turns that sequence of keywords into a function similar to this:

```clojure
(fn [m1 transform-fn]
  (update m1 :a
    (fn [m2]
      (update m2 :b
        (fn [m3]
          (update m3 :c transform-fn)
          )))))
```

So instead of having to iterate through a sequence of navigators and execute Specter's [core protocol](https://github.com/nathanmarz/specter/blob/0.11.0/src/clj/com/rpl/specter/protocols.cljx#L3) over and over, the navigation functions are extracted and composed to call each other directly. There is only function invocation – all protocol invocation and sequence operation has been stripped out. There's more that happens during `comp-paths`, but these are the most important optimizations.

`comp-paths` is easy to use and enables very fast code *as long as you know the complete path ahead of time*. This fact seems to preclude you from precompiling a path that cannot be known until runtime, like this previously shown example:

```clojure
(defn dynamic-example1 [k data]
  (transform [ALL (keypath k)] inc data))
```

Since the path is dependent on the runtime value of `k`, it seems impossible to precompile this path ahead of time. However, this is where Specter's late-bound parameterization feature comes into play.

## Late-bound parameterization

The goal of late-bound paramerization is to enable navigators to be precompiled into larger paths *without their parameters*, with the parameters provided at a later time. To understand how this feature could possibly work, let's take a look at the definition of `keypath`:

```clojure
(defnav keypath [key]
  (select* [this structure next-fn]
    (next-fn (get structure key)))
  (transform* [this structure next-fn]
    (assoc structure key (next-fn (get structure key)))
    ))
```

At first glance, `keypath` looks like a function that takes in a key parameter and returns an instance of the `Navigator` protocol. As a user of `keypath`, that is exactly how you should think of it. In reality though, `keypath` works quite differently. The `defnav` macro expands the `keypath` definition into this code (slightly simplified):

```clojure
(def keypath
  (->ParamsNeededPath

    ;; select* function
    (fn [params params-idx structure real-next-fn]
      (let [key (aget params params-idx)
            next-fn (fn [s] (real-next-fn params (+ params-idx 1) s))]
        (next-fn (get structure key))
        ))

    ;; transform* function
    (fn [params params-idx structure real-next-fn]
      (let [key (aget params params-idx)
            next-fn (fn [s] (real-next-fn params (+ params-idx 1) s))]
        (assoc structure key (next-fn (get structure key)))
        ))

    ;; # needed params
    1

    ))
```

It looks intimidating, but what's happening here is a really simple idea. Rather than dynamically create a navigator based on runtime parameters, a static navigator is created that receives its parameters in a different way. The parameters are provided later via an array passed to the `select*/transform*` functions. Then, since the navigator is static and its `select*/transform*` functions are known before any parameters are provided, it can be precompiled normally. An example of usage should help tie this all together:

```clojure
(let [cpath (comp-paths keypath keypath)]
  (defn read-nested-map [m k1 k2]
    (compiled-select (cpath k1 k2) m)))
```

As you can see here, the two `keypath`'s are compiled together without their parameters. Then all the parameters needed by the path are provided in bulk right before using it. `(cpath k1 k2)` puts `k1` and `k2` into an array and Specter threads the array through the already-composed navigation functions.

Late-bound parameterization also works on higher-order navigators like `selected?` which themselves are parameterized with paths that may require further parameters. Here's how to fully precompile `dynamic-example2` from before:


```clojure
;; not precompiled
(defn dynamic-example2 [akey apred data]
  (select [(keypath (str akey "!"))
           (selected? ALL (pred apred))
           LAST]
    data))

->

;; precompiled
(let [cpath (comp-paths keypath (selected? ALL pred) LAST)]
  (defn dynamic-example2 [akey apred data]
    (compiled-select (cpath (str akey "!") apred) data)))
```

With `comp-paths` and the late-bound parameterization feature, any usage of Specter can be made blazing fast with a slight amount of refactoring by the user. And prior to Specter 0.11.0, that is exactly what you would do when you needed performance.

Specter 0.11.0 introduces inline factoring and caching, which enables awesome performance without having to do any of that refactoring.

## Inline factoring and caching

The idea behind this feature is to do the factoring a user would normally do on their own, but automatically and at runtime. The static part of the path is compiled for future invocations, and an efficient means of extracting the dynamic parameters into an array is produced. For example, let's again look at `dynamic-example2` which in Specter 0.11.0 runs with high performance:

```clojure
(defn dynamic-example2 [akey apred data]
  (select [(keypath (str akey "!"))
           (selected? ALL (pred apred))
           LAST]
    data))
```

The first time this code is run, Specter looks at the provided path to determine if it can be factored. This path can be factored, so the static path `[keypath (selected? ALL pred)]` is extracted and compiled.

It also needs to produce an efficient way to produce the dynamic parameters that are spread throughout the path. So during that first runthrough, it generates and evals this function:

```clojure
(fn [akey apred]
  (let [ret (object-array 2)]
    (aset ret 0 (str akey "!"))
    (aset ret 1 apred)
    ret
    ))
```

Then, on each future invocation, the function is invoked with the relevant locals to get the parameters array for this particular usage of the path. Both this function and the compiled path are cached in a global `ConcurrentHashMap` and re-used for all future invocations.

(Note about ClojureScript – since ClojureScript doesn't have `eval`, a different strategy is used for the dynamic parameters. Although not as efficient as the Clojure strategy, it's still very fast.)

There are cases where a static path cannot be factored and the path will need to be compiled on every invocation. Here are examples of such cases:

```clojure
;; Uses a local symbol where a navigator is expected
(fn [a data]
  (select [a :b] data))

;; Uses a special form where a navigator is expected
(fn [data]
  (select [(if true :a :b) :c] data))

;; Uses a dynamic var where a navigator is expected
(fn [data]
  (select [*dynamic-nav* ALL] data))
```

In all these cases, the path is not static so it cannot be cached/re-used for future invocations. Fortunately, in practice if this ever happens it's almost always possible to specify the path in a slightly different way so that it can be factored and cached. For example, if you have a local keyword that you want to use as a navigator:

```clojure
(fn [akey data]
  (select [ALL akey] data))
```

By refactoring this as the following, inline caching will take effect and it will run extremely fast:

```clojure
(fn [akey data]
  (select [ALL (keypath akey)] data))
```

A similar refactoring is possible for local symbols which represent functions. In that case the usage of the local should be wrapped in `pred`. 

It is highly recommended that you invoke `(must-cache-paths!)` somewhere in your code. With this flag set, Specter will throw a runtime exception if it encounters a path that cannot be factored into static and dynamic parts. It will also print detailed reasons why. For example:

```clojure
user=> (must-cache-paths!)
user=> (let [b :b] (select [(if true :a :b) (selected? b nil?)] {}))

Failed to cache path: Special form (if true :a :b) where navigator expected
Failed to cache path: Local symbol b where navigator expected

IllegalArgumentException Failed to cache path  sun.reflect.NativeConstructorAccessorImpl.newInstance0 (NativeConstructorAccessorImpl.java:-2)
```

Inline factoring and caching is not as fast as manual precompilation – you can expect a slowdown of 2% to 15%. Though for the vast majority of use cases, this is more than fast enough. If you have a very hot piece of code, you can do manual precompilation to squeeze out additional performance.

## Clojure improvement suggestions

If Clojure exposed additional capabilities, Specter's inline factoring and caching implementation could run even faster – potentially as fast as manual precompilation. Here are two suggestions for new Clojure functionality.

The logic Specter invokes during inline factoring and caching seems to be a great candidate for `invokedynamic`. The first time the code is run, it knows exactly what code path to take for all future invocations, and what state those code paths will need (the compiled path and dynamic parameters function). It seems that `invokedynamic` could enable Specter to eliminate most of the current overhead of inline caching, like looking up the cached values in the global map and doing if checks to determine the code path to take. Currently Clojure does not expose any way to utilize `invokedynamic`.

A simpler primitive than `invokedynamic` that Clojure could provide to help with this issue is the ability to declare a mutable global value within the context of a function, e.g.:

```clojure
(defn foo []
  (static-field [mycache]
    (if mycache
      ...
      (let [computed-value ...]
        (set! mycache computed-value)
        ...
        ))))
```

In this example, `mycache` would be independent from any particular invocation of `foo` and would persist its value across invocations. This could be implemented at the JVM level with a static field on the class generated for `foo`. For Specter, such a feature would speed up the cached code path by eliminating the need to perform a lookup in a global `ConcurrentHashMap`. A static field access would be much faster.

_Note: After this post was published, a method for creating a var per callsite to serve as the inline cache was developed, largely superseding the need for the suggested static-field feature._

It's unclear if either of these are good ideas for Clojure as a whole, and that determination should be done by the Clojure developers. The point here is that Specter has hit the limits of what can be done with Clojure, but with an additional primitive exposed by Clojure, Specter could be even faster. The ultimate goal is for the inline cached code path to have the same speed as manual precompilation.

## Conclusion

The [changelog](https://github.com/nathanmarz/specter/blob/master/CHANGES.md) contains the full list of changes in 0.11.0. Since all the `select/transform/etc.` functions were changed to macros to enable inline factoring and caching, and since that is a backwards incompatible change, this release also took the opportunity to rename some of the other core macros to clean up the terminology used by the project.

Inline caching and factoring is definitely the highlight of the release though. That this feature is even possible is a testament to the awesome power of the Clojure language.
