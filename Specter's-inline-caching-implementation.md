The [0.11.0 announcement post](https://github.com/nathanmarz/specter/wiki/Specter-0.11.0:-Performance-without-the-tradeoffs) gave a comprehensive overview of how Specter does inline caching and achieves amazing performance. With the 0.13.0 release, Specter's internals have been redesigned and the information in that post is no longer valid. This post will explain Specter's new inline caching implementation.

The main reason for rewriting Specter's internals was because the old implementation was just too complicated. Having separate codepaths for static paths and dynamic paths meant working on Specter was very tedious. Additionally, the many mechanisms for creating navigators was confusing (`defnav`, `defpathedfn`, `defnavconstructor`, `fixed-pathed-nav`, `variable-pathed-nav`, `richnav`).

The rewrite greatly simplifies the implementation (codebase is 20% smaller) while bringing the following additional benefits:

- Paths can now contain dynamic vars and special forms, e.g. `[*dynamic-nav* (if some-condition? (keypath a) ALL)]`. These paths still undergo inline compilation and caching and will run with great performance.
- Performance with dynamic parameters is greatly improved. A path like `[(keypath a) (keypath b) (keypath c)]` runs more than 40% faster.
- Writing navigators or higher-order navigators is much simpler now. 


## Old inline caching implementation

This example illustrates the challenges in making Specter fast:

```clojure
(defn foo [a data]
  (select [ALL (selected? (keypath a) even?) :b] data))
```

Compiling the path on every invocation is a no-go. Compilation from scratch involves sequence traversal (nested arbitrarily), running a protocol function to coerce implicit navigators (like `:b`) to their corresponding navigator (`:b` => `(keypath :b)`) and a bunch of object allocation to create a composed, nested function that can be executed.

Since compiling on every invocation creates too much overhead, you instead need to cache something that can be re-used. You can't memoize based on the dynamic params (in this example `a`), because that would require an unbounded cache and/or have inconsistent performance.

In prior versions of Specter, the solution was to enable paths to be compiled *without their parameters*. So this example could be rewritten like this:

```clojure
(let [cpath (comp-paths ALL (selected? keypath even?) :b)]
  (defn foo [a data]
    (compiled-select (cpath a) data)))
```

`(cpath a)` would package `a` into an array and thread it through the precompiled path during execution. Until Specter 0.11.0, this factoring would have to be done manually to get performance. Specter 0.11.0 introduced inline factoring + caching which did that factoring for you automatically on the first invocation of that callsite.

At the time this made a lot of sense. The precompilation without parameters design was done before the inline caching technique was even a viable concept. So it was natural to first find an elegant way to get the performance manually, and then to build upon that for the first inline caching implementation.

With all the ins and outs of doing inline caching now being understood, it turns out there a better way for Specter to work by further leveraging the flexibility of inline compilation.

## New inline caching implementation

The goal of inline caching is to do as much work ahead of time so the work to finish compilation at runtime (like parameterizing with local variables) is extremely fast. The prior design reduced the runtime work to a single operation: creating an array and filling it with dynamic params. However, "one runtime operation" is not a hard constraint. It's fine to have more operations as long as they are all fast.

With new new inline caching implementation, the `foo` example from before compiles to this:

```clojure
(defn foo [a data]
  (compiled-select (comp-navs ALL (<ANON1> (comp-navs (keypath a) <ANON2>) <ANON3>) data)))
```

`<ANON*>` refers to objects that are precompiled/cached and re-used for every runtime compilation. `<ANON1>` is a navigator builder for `selected?` that takes in the compiled subpath as input, `<ANON2>` is a `RichNavigator` implementation equivalent to `(pred even?)`, and `<ANON3>` is a `RichNavigator` implementation equivalent to `(keypath :b)`.

Finally, `comp-navs` combines many `RichNavigator` into a single `RichNavigator`. It runs extremely fast since it only entails an object allocation and a few field sets. It's basically the same as `comp`, except instead of composing functions together it composes implementations of `RichNavigator`. It *does not* do the expensive work of `comp-paths` which additionally analyzes nested sequences and converts implicit navs to their `RichNavigator` form.

Here are some more examples of how paths get converted:

```clojure
[(keypath a) :b :c] => (comp-navs (keypath a) <ANON>)

[:a :b :c] => <ANON>

[:a b :c] => (comp-navs <ANON> (coerce-nav b) <ANON>)
```

The first example shows how sequential static navigators get precompiled together to eliminate that work from runtime compilation. The second example shows that a fully static path gets precompiled into a single `RichNavigator`, completely eliminating any runtime work.

The last example shows what Specter does when it has a local variable in the position of a navigator. Since Specter does not know if `b` is an implementation of `RichNavigator`, it cannot put it into a `comp-navs` call directly. `b` could be an implicit navigator like `:some-keyword`, or it could be an uncompiled path like `[ALL even?]`. So it inserts the call to `coerce-nav` to determine that at runtime. If you know for sure a symbol or form will create a `RichNavigator` object, then you can annotate it with metadata like this:

```clojure
[:a ^:direct-nav b :c] => (comp-navs <ANON> b <ANON>)
```

The same thing works for dynamic var references and special forms:

```clojure
[*a-dynamic-var*] => (coerce-nav *a-dynamic-*var)

[^:direct-nav *a-dynamic-var*] => *a-dynamic-*var

[:a (if c (keypath a) STAY)] => (comp-navs <ANON> (coerce-nav (if c (keypath a) STAY)))

[:a ^:direct-nav (if c (keypath a) STAY)] => (comp-navs <ANON> (if c (keypath a) STAY))
```

## Dynamic navigators

There are two more additional use cases that Specter must handle, both of which are illustated by the `selected?` navigator. Consider these two uses of `selected?`:

```clojure
(selected? (keypath a) even?)

(selected? even? div-by-3?)
```

`selected?` takes in a path and stays navigated at the current location if the path selects at least one thing. So it's a navigator that takes in a subpath as input. That means Specter needs to compile that subpath at runtime just as it compiles the overall path. The question is, how does Specter know to do this:

```clojure
(selected? (keypath a) even?) => (<ANON> (comp-navs (keypath a) <ANON2>))
```

instead of this:

```clojure
(selected? (keypath a) even?) => (selected? (keypath a) even?)
```

There's no way to know whether the arguments to `selected?` are a single path, multiple paths, regular arguments, or a single path and a single regular argument. The definition of `selected?` needs to tell Specter how to interpret its arguments so that any subpaths can be properly compiled.

There's another separate use case to consider. While the generic implementation of `selected?` is to run a `select` on its subpath and only continue navigation if the result set is non-empty, there's a special case where it can be a lot faster. If all the elements in its path are just static functions (like `(selected? even? div-by-3?)`), then it can just test that each function returns true. This is much faster than adding in the additional overhead of doing a full selection. So `selected?` should be able to determine at compile time whether its implementation should be a function test or a selection with a subpath. That decision should be analyzed once and the results baked into the subsequent runtime path.

(Something like `(selected? even? div-by-3?)` is not the best example since that path is the same as `[even? div-by-3?]`. So this extra optimization logic doesn't seem very important. However, the exact same logic is needed for `if-path` for which there is no alternative – and this optimization makes a huge difference for `if-path`. `selected?` is easier to analyze though which is why it's being used as an example.)

Both of these use cases are handled by `defdynamicnav`. Take a look at the implementation for `selected?`:

```clojure
(defdynamicnav selected? [& path]
  (if-let [afn (n/extract-basic-filter-fn path)]
    afn
    (late-bound-nav [late (late-path path)]
      (select* [this structure next-fn]
        (if-not (identical? NONE (compiled-select-any late structure))
          (next-fn structure)))
      (transform* [this structure next-fn]
        (if-not (identical? NONE (compiled-select-any late structure))
          (next-fn structure)
          structure))
```

`selected?` is a function that returns the navigator to use at runtime. The trick here is that it runs while Specter is compiling the overall path, so the `path` argument can contain unknown values. For example, for `(selected? (keypath a) :b)` `path` will look like this:

```clojure
[#DynamicVal{:code (keypath a)} :b]
```

The `dynamic-param?` function can be used to distinguish which values are static (like `:b`) and which are dynamic.

In this case, `selected?` runs `extract-basic-filter-fn` which checks that every argument is static and that every argument is a function. When that condition holds, it returns as its navigator a composed function that tests that each constituent function returns true.

Otherwise, `selected?` produces a navigator to do the selection on the subpath at runtime. Since its current path is potentially dynamic, it uses `late-bound-nav` to produce a navigator that will have `path` appropriately resolved for runtime. It marks the `path` as a subpath using `late-path`, which will cause the `late` symbol to be bound to the parameterized runtime path. `late-path` instructs the Specter inline compiler to treat that (potentially) dynamic value as a path and do any necessary coercion / composition. This is how Specter knows to compile `(selected? (keypath a) :b)` as `(<ANON1> (comp-navs (keypath a) <ANON2>))`. Then the navigator implementation proceeds normally.

`transformed` is another illustrative example of dynamic navigators. `transformed` takes in a subpath and a transform function and navigates to the view of doing that transformation at the current point. Here's the implementation:

```clojure
(defdynamicnav transformed [path update-fn]
  (late-bound-nav [late (late-path path)
                   late-fn update-fn]
    (select* [this structure next-fn]
      (next-fn (compiled-transform late late-fn structure)))
    (transform* [this structure next-fn]
      (next-fn (compiled-transform late late-fn structure)))))
```

In this case it takes in one subpath argument and one regular argument, either of which can be dynamic. By marking `path` with `late-path` and *not* marking `update-fn` with `late-path`, the appropriate compilation logic takes place.

## Conclusion

Specter is very close to 1.0 with these changes. It is lightning fast and more dynamic than ever. There have been a lot of breaking changes the past few releases, but those should now be coming to an end (or be extremely rare). Specter has near optimal performance for selection and transformation for both static and dynamic paths, so there are no major structural changes needed.

Finally, here's [a benchmark](https://gist.github.com/nathanmarz/b7c612b417647db80b9eaab618ff8d83) showing the great performance of Specter 0.13.0.
