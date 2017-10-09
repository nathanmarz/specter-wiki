Specter is useful for navigating through nested data structures, and then returning (selecting) or transforming what it finds. Specter can also work its magic **recursively**. Many of Specter's most important and powerful use cases in your codebase will require you to use Specter's recursive features. However, just as recursion can be difficult to grok at first, using Specter recursively can be challenging. This guide is designed to help you feel at home using Specter recursively.

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
**Table of Contents**

- [A Review of Basic Recursion](#a-review-of-basic-recursion)
- [Using Specter Recursively](#using-specter-recursively)
- [A Basic Example](#a-basic-example)
- [Advanced Recursion with Specter](#advanced-recursion-with-specter)
	- [The Building Blocks of Recursion in Specter](#the-building-blocks-of-recursion-in-specter)
	- [The Implementation of `recursive-path`](#the-implementation-of-recursive-path)
	- [Mutually Recursive Paths](#mutually-recursive-paths)
- [Applications](#applications)
	- [Navigate to all of the instances of one key in a map](#navigate-to-all-of-the-instances-of-one-key-in-a-map)
	- [Find the "index route" of a value within a data structure](#find-the-index-route-of-a-value-within-a-data-structure)

<!-- markdown-toc end -->

# A Review of Basic Recursion

Before we review how recursion works with Specter, it might be worth a brief refresher on recursion more broadly. If you're familiar with this material, feel free to skip this section. But a brief review will allow us to separate reviewing the concept of recursion from learning how to combine that concept with Specter's functionality.

Most simple functions do not call themselves. For example:

```clojure
(defn square [x] (* x x))
```

Instead of calling `square`, itself, `square` calls another function, `*` or multiply. This, of course, serves the purpose of squaring the input. You don't always need recursion.

But sometimes you need a function to call itself - to recur. To take another example from mathematics, you need recursion to implement factorials. Factorials are equal to the product of every positive number equal to or less than the number. For example, the factorial of 5 ("5!") is `5 * 4 * 3 * 2 * 1 = 120`.

Here is a simple implementation of factorials in Clojure:

```clojure
=> (defn factorial [x]
	(if (< x 2)
	  1
	  (* x (factorial (dec x)))))
#'playground.specter/factorial
=> (factorial 5)
120
```

Here, the function definition *calls itself* with the symbol `factorial`. You still supply input - the value x - but producing the function's return value requires calling the function itself.

We can see that there are up to three elements to basic recursion:
  * a name or symbol for the recursion point or function (defn function-name)
  * optional input arguments
  * the body of the recursion - what you **do** recursively - and, at some point, calling the function by its name

# Using Specter Recursively

The main way that Specter enables recursion is through the parameterized macro, `recursive-path`. `recursive-path` takes the following form:

`(recursive-path params self-sym path)`

You can see that there are three arguments, which match onto our list above. (They are, however, in a different order.) The `self-sym` is the name; the `params` are the optional input arguments; and the `path` is the body of the recursion. This path can be recursive, referencing itself by the `self-sym`.

Rather than providing a recursive function, `recursive-path` provides a recursive path that can be composed with other navigators. Accordingly, it is often useful - but not necessary - to define a recursive-path using `def`, so that it can be called in multiple places, not just the place where you initially need it.

# A Basic Example

Suppose you had a tree represented using vectors:

```clojure
=> (def tree [1 [2 [[3]] 4] [[5] 6] [7] 8 [[9]]])
#'playground.specter/tree
```

You can define a navigator to all the leaves of the tree like this:

```clojure
=> (def TREE-VALUES
	(recursive-path [] p
	  (if-path vector?
		[ALL p]
		STAY)))
```

`recursive-path` allows your path definition to refer to itself, in this case using `p`. This path definition leverages the `if-path` navigator which uses the predicate to determine which path to continue on. If currently navigated to a vector, it recurses navigation at all elements of the vector. Otherwise, it uses `STAY` to stop traversing and finish navigation at the current point. The effect of all this is a depth-first traversal to each leaf node.

Now we can get all of the values nested within vectors:

```clojure
=> (select TREE-VALUES [1 [2 [3 4] 5] [[6]]])
[1 2 3 4 5 6]
```

Or transform all of those values:

```clojure
=> (transform TREE-VALUES inc [1 [2 [3 4] 5] [[6]]])
[2 [3 [4 5] 6] [[7]]]
```

You can also compose `TREE-VALUES` with other navigators to do all sorts of manipulations. For example, you can increment even leaves:

```clojure
=> (transform [TREE-VALUES even?] inc tree)
[1 [3 [[3]] 5] [[5] 7] [7] 9 [[9]]]
```

Or get the odd leaves:

```clojure
=> (select [TREE-VALUES odd?] tree)
[1 3 5 7 9]
```

Or reverse the order of the even leaves (where the order is based on a depth-first search):

```clojure
=> (transform (subselect TREE-VALUES even?) reverse tree)
[1 [8 [[3]] 6] [[5] 4] [7] 2 [[9]]]
```

That you can define how to get to the values you care about once and easily reuse that logic for both querying and transformation is invaluable. And, as always, the performance is near-optimal for both querying and transformation.

# Advanced Recursion with Specter

More advanced recursion is possible with Specter. In particular, it is also possible to create mutually recursive paths with Specter - recursive paths that call each other within their bodies.

To learn how to do so, let's look at Specter's implementation of `recursive-path`. As with much of Specter's functionality, `recursive-path` is implemented with a [macro](https://clojure.org/reference/macros). Accordingly, it uses the [reader symbols](https://clojure.org/guides/weird_characters) syntax quote (`) and unquote (~).

## The Building Blocks of Recursion in Specter

`recursive-path` is built with three basic elements: `local-declarepath`, `declarepath`, and `providepath`:

```clojure
(defmacro declarepath [name]
  `(def ~name (i/local-declarepath)))

(defmacro providepath [name apath]
  `(i/providepath* ~name (path ~apath)))
```

`local-declarepath` comes from the `com.rpl.specter.impl` namespace (although you can also access it from the core Specter namespace). `local-declarepath` lets you declare a path anonymously which can be provided later. But it can also be used in the meantime in other path definitions. As we will see, this is how recursion and mutual recursion are enabled.

`declarepath` simply assigns an anonymous path created by `local-declarepath` to a var with `def`. This works similarly to Clojure's built-in `declare` macro. `declare` calls `def` on "the supplied var names with no bindings"; this is "useful for making forward declarations." (However, unlike Clojure's `declare`, Specter's `declarepath` only works on one name.)

`providepath` supplies the full path, declared by `declarepath`, rather than an empty, anonymous, and therefore useless path. Here is a simple, non-recursive example of `declarepath` and `providepath` being used in combination:

```clojure
=> (declarepath SECOND)
=> (providepath SECOND [(srange 1 2) FIRST])
=> (select-one SECOND (range 5))
1
=> (transform SECOND dec (range 5))
(0 0 2 3 4)
```

## The Implementation of `recursive-path`

Now we are ready to see the implementation of `recursive-path`. Here is the source code:

```clojure
(defmacro recursive-path [params self-sym path]
  (if (empty? params)
	`(let [~self-sym (i/local-declarepath)]
	   (providepath ~self-sym ~path)
	   ~self-sym)
	`(i/direct-nav-obj
	  (fn ~params
		(let [~self-sym (i/local-declarepath)]
		  (providepath ~self-sym ~path)
		  ~self-sym)))))
```

`recursive-path` uses a let-binding to associate an anonymous path with the provided `self-sym` (just like `declare-path`, but not assigned to a var). Then the specified is provided to that anonymous path with `providepath`. Then the `self-sym` is returned. If there are parameters, all of that is wrapped in an anonymous, navigable function that can take the appropriate parameters. In any case, `recursive-path` returns the var associated with a path that calls itself.

## Mutually Recursive Paths

Now that you know how `recursive-path` is built, you can use that knowledge to implement your own recursive paths in Specter, without `recursive-path`. Here is an example of how you can use `declarepath` and `providepath` to implement recursive paths without `recursive-path`.

```clojure
=> (declarepath DEEP-MAP-VALS)
=> (providepath DEEP-MAP-VALS (if-path map? [MAP-VALS DEEP-MAP-VALS] STAY))
=> (select DEEP-MAP-VALS {:a {:b 2} :c {:d 3 :e {:f 4}} :g 5})
[2 3 4 5]
=> (transform DEEP-MAP-VALS inc {:a {:b 2} :c {:d 3 :e {:f 4}} :g 5})
{:a {:b 3}, :c {:d 4, :e {:f 5}}, :g 6}
```

You can also use this pattern for creating mutually recursive paths. Here is an example of what that might look like:

```clojure
=> (declarepath A)
=> (declarepath B)
=> (providepath A B)
=> (providepath B A)
```

This example, is, of course, useless (and non-terminating, and therefore harmful), but it shows how you would implement mutually recursive paths using Specter.

# Applications

Here are some other examples of using Specter recursively with `recursive-path`.

## Navigate to all of the instances of one key in a map

```clojure
=> (def map-key-walker (recursive-path [akey] p [ALL (if-path [FIRST #(= % akey)] LAST [LAST p])]))
#'playground.specter/map-key-walker
;; Get all the vals for key :aaa, regardless of where they are in the structure
=> (select (map-key-walker :aaa) {:a {:aaa 3 :b {:c {:aaa 2} :aaa 1}}})
[3 2 1]
;; Transform all the vals for key :aaa, regardless of where they are in the structure
=> (transform (map-key-walker :aaa) inc {:a {:aaa 3 :b {:c {:aaa 2} :aaa 1}}})
{:a {:aaa 4, :b {:c {:aaa 3}, :aaa 2}}}
```

## Recursively navigate to every map in a map of maps

You have a deeply nested map of maps:

```clojure
(def data
  {:a {:b {:c 1} :d 2}
   :e {:f 3 :g 4}})
```

You want to recursively navigate this data structure and add a key-value pair to every map at every level. This example, `MAP-NODES`, shows you how to do so:

```clojure
=> (def MAP-NODES
	 (recursive-path [] p
	   (if-path map?
		 (continue-then-stay MAP-VALS p))))

=> (setval [MAP-NODES :X] 0 data)
{:a {:b {:c 1, :X 0}, :d 2, :X 0}, :e {:f 3, :g 4, :X 0}, :X 0}
```

`MAP-NODES` illustrates how to combine recursive paths with `[continue-then-stay](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#continue-then-stay)`, which navigates to the provided path and then to the current element. This should also point to how you might use recursive paths with `[stay-then-continue](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#stay-then-continue)`, which navigates to the current element and then to the provided path.

## Find the "index route" of a value within a data structure

This example comes from [a Stack Overflow question](https://stackoverflow.com/questions/45764946/how-to-find-indexes-in-deeply-nested-data-structurevectors-and-lists-in-clojur).

```clojure
=> (defn find-index-route [v data]
	(let [walker (recursive-path [] p
			(if-path sequential?
			  [INDEXED-VALS
			   (if-path [LAST (pred= v)]
				 FIRST
				 [(collect-one FIRST) LAST p])]))
		  ret (select-first walker data)]
	  (if (or (vector? ret) (nil? ret)) ret [ret])))
#'playground.specter/find-index-route
=> (find-index-route :my-key '(1 2 :my-key))
[2]
=> (find-index-route :my-key '(1 2 "a" :my-key "b"))
[3]
=> (find-index-route :my-key '(1 2 [:my-key] "c"))
[2 0]
=> (find-index-route :my-key '(1 2 [3 [:my-key]]))
[2 1 0]
=> (find-index-route :my-key '(1 2 [3 [[] :my-key]]))
[2 1 1]
=> (find-index-route :my-key '(1 2 [3 [4 5 6 (:my-key)]]))
[2 1 3 0]
=> (find-index-route :my-key '(1 2 [3 [[]]]))
nil
```
