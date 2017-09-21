<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-generate-toc again -->
**Table of Contents**

- [Core Macros](#core-macros)
    - [collected?](#collected)
    - [multi-transform](#multi-transform)
    - [replace-in](#replace-in)
    - [select](#select)
    - [select-any](#select-any)
    - [selected-any?](#selected-any)
    - [select-first](#select-first)
    - [select-one](#select-one)
    - [select-one!](#select-one)
    - [setval](#setval)
    - [transform](#transform)
    - [traverse](#traverse)
    - [traverse-all](#traverse)
- [Path Macros](#path-macros)
    - [declarepath](#declarepath)
    - [defprotocolpath](#defprotocolpath)
    - [extend-protocolpath](#extend-protocolpath)
    - [path](#path)
    - [providepath](#providepath)
    - [recursive-path](#recursive-path)
- [Collector Macros](#collector-macros)
    - [defcollector](#defcollector)
- [Navigator Macros](#navigator-macros)
    - [defnav](#defnav)
    - [nav](#nav)

<!-- markdown-toc end -->


# Core Macros

## collected?

`(collected? params & body)`

_Added in 0.12.0_

Creates a filter function navigator that takes in all the collected values
as input. For arguments, can use `(collected? [a b] ...)` syntax to look
at each collected value as individual arguments, or `(collected? v ...)` syntax
to capture all the collected values as a single vector.

`collected?` operates in the same fashion as [pred](List-of-Navigators#pred), but it takes the collected values as its arguments rather than the structure.

```clojure
=> (select [ALL (collect-one FIRST) LAST (collected? [k] (= k :a))] {:a 0 :b 1})
[[:a 0]]
=> (select [ALL (collect-one FIRST) LAST (collected? [k] (< k 2))] 
     (zipmap (range 5) ["a" "b" "c" "d" "e"]))
[[0 "a"] [1 "b"]]
=> (transform [ALL (collect-one FIRST) LAST (collected? [k] (< k 2)) DISPENSE]
              string/upper-case 
              (zipmap (range 5) ["a" "b" "c" "d" "e"]))
{0 "A", 1 "B", 2 "c", 3 "d", 4 "e"}
```

## multi-transform

`(multi-transform path structure)`

_Added in 0.12.0_

Just like `transform` but expects transform functions to be specified inline in
the path using `terminal`. Error is thrown if navigation finishes at a
non-`terminal` navigator. `terminal-val` is a wrapper around `terminal` and is
the `multi-transform` equivalent of `setval`. Much more efficient than doing the
transformations with `transform` one after another when the transformations
share a lot of navigation. This macro will do inline compilation and caching of the path.

```clojure
(multi-transform [:a :b (multi-path [:c (terminal-val :done)]
                                    [:d (terminal inc)]
                                    [:e (putval 3) (terminal +)])]
                 {:a {:b {:c :working :d 0 :e 1.5}}})
{:a {:b {:c :done, :d 1, :e 4.5}}}
```

See also [terminal](List-of-Navigators#terminal) and [terminal-val](List-of-Navigators#terminal-val).

## replace-in

`(replace-in apath transform-fn structure & args)`

Similar to `transform`, except returns a pair of `[transformed-structure sequence-of-user-ret]`.
The transform-fn in this case is expected to return `[ret user-ret]`. ret is
what's used to transform the data structure, while `user-ret` will be added to the `user-ret` sequence
in the final return. replace-in is useful for situations where you need to know the specific values
of what was transformed in the data structure.
This macro will do inline factoring and caching of the path.

Note that the `user-ret` portion of the return value of `transform-fn` must be a sequence in order to be joined onto all previous user-return values.

`replace-in` takes an optional argument `:merge-fn`. `merge-fn` takes two arguments `[curr-user-ret new-user-ret]` and should return a new user-return value. If no user-return values have been added, `user-ret` will be `nil`.

```clojure
;; double and save evens
=> (replace-in [ALL even?] (fn [x] [(* 2 x) [x]]) (range 10))
[(0 1 4 3 8 5 12 7 16 9) (0 2 4 6 8)]
;; double evens and save largest even
=> (replace-in [ALL even?] (fn [x] [(* 2 x) x]) [3 2 8 5 6]
     :merge-fn (fn [curr new] (if (nil? curr) new (max curr new))))
[[3 4 16 5 12] 8]
```

## select

`(select apath structure)`

Navigates to and returns a sequence of all the elements specified by the path.
This macro will do inline compilation and caching of the path.

```clojure
=> (select [ALL even?] (range 10))
[0 2 4 6 8]
=> (select :a {:a 0 :b 1})
[0]
=> (select ALL {:a 0 :b 1})
[[:a 0] [:b 1]]
```

## select-any

`(select-any apath structure)`

_Added in 0.12.0_

Returns any element found or `com.rpl.specter/NONE` if nothing selected. This is the most
efficient of the various selection operations.
This macro will do inline compilation and caching of the path.

```clojure
=> (select-any STAY :a)
:a
=> (select-any even? 3)
:com.rpl.specter.impl/NONE ; Implementation detail
=> (= com.rpl.specter/NONE :com.rpl.specter.impl/NONE)
true
```

## selected-any?

`(selected-any? apath structure)`

_Added in 0.12.0_

Returns true if any element was selected, false otherwise.
This macro will do inline compilation and caching of the path.

```clojure
=> (selected-any? STAY :a)
true
=> (selected-any? even? 3)
false
=> (selected-any? ALL (range 10))
true
=> (selected-any? ALL [])
false
```

## select-first

`(select-first apath structure)`

Returns first element found. Not any more efficient than `select`, just a convenience.
This macro will do inline compilation and caching of the path.

```clojure
=> (select-first ALL (range 10))
0
;; Returns the result itself if the result is not a sequence
=> (select-first FIRST (range 10))
0
```

## select-one

`(select-one apath structure)`

Like `select`, but returns either one element or nil. Throws exception if multiple elements found.
This macro will do inline compilation and caching of the path.

```clojure
;; srange returns one collection
=> (select (srange 2 7) (range 10))
[[2 3 4 5 6]]
=> (select-one (srange 2 7) (range 10))
[2 3 4 5 6]
=> (select-one ALL (range 10))
IllegalArgumentException More than one element found for params
```

## select-one!

`(select-one! apath structure)`

Returns exactly one element, throws exception if zero or multiple elements found.
This macro will do inline compilation and caching of the path.

```clojure
=> (select-one! FIRST (range 5))
0
;; zero results, throws exception
=> (select-one! [ALL even? odd?] (range 10))
IllegalArgumentException Expected exactly one element for params
;; multiple results, throws exception
=> (select-one! [ALL even?] (range 10))
IllegalArgumentException Expected exactly one element for params
```

## setval

`(setval apath aval structure)`

Navigates to each value specified by the path and replaces it by `aval`.
This macro will do inline compilation and caching of the path.

```clojure
=> (setval [ALL even?] :even (range 10))
(:even 1 :even 3 :even 5 :even 7 :even 9)
```

## transform

`(transform apath transform-fn structure)`

Navigates to each value specified by the path and replaces it by the result of running
the transform-fn on it.
This macro will do inline compilation and caching of the path.

```clojure
=> (transform [ALL] #(* % 2) (range 10))
(0 2 4 6 8 10 12 14 16 18)
;; putval collects its argument
=> (transform [(putval 2) ALL] * (range 10))
(0 2 4 6 8 10 12 14 16 18)
=> (transform [(putval 2) (walker #(and (integer? %) (even? %)))] * [[[[1] 2]] 3 4 [5 6] [7 [[8]]]])
[[[[1] 4]] 3 8 [5 12] [7 [[16]]]]
=> (transform [ALL] (fn [[k v]] [k {:key k :val v}]) {:a 0 :b 1})
{:a {:key :a, :val 0}, :b {:key :b, :val 1}}
```

## traverse

`(traverse apath structure)`

_Added in 0.12.0_

Return a reducible object that traverses over `structure` to every element
specified by the path.
This macro will do inline compilation and caching of the path.

`(reduce afn init (traverse apath structure))` will always return the same thing as `(reduce afn init (select apath structure))`, but more efficiently. The return value of `traverse` is only useful as an argument to `reduce`; for all other uses, prefer [select](#select).

```clojure
=> (reduce + 0 (traverse ALL (range 10)))
45
=> (reduce + 0 (traverse (walker integer?) [[[1 2]] 3 [4 [[5 6 7]] 8] 9]))
45
=> (into #{} (traverse (walker integer?) [[1 2] 1 [[3 [4 4 [2]]]]]))
#{1 4 3 2}
=> (traverse (walker integer?) [[[1 2]] 3 [4 [[5 6 7]] 8] 9])
;; returns object implementing clojure.lang.IReduce
```

## traverse-all

_Added in 1.0.0_

`(traverse-all apath)`

Returns a transducer that traverses over each element with the given path.

Many common transducer use cases can be expressed more elegantly with traverse-all:

```clojure
;; Using Vanilla Clojure
(transduce
 (comp (map :a) (mapcat identity) (filter odd?))
 +
 [{:a [1 2]} {:a [3]} {:a [4 5]}])
;; => 9

;; The same logic expressed with Specter
(transduce
 (traverse-all [:a ALL odd?])
 +
 [{:a [1 2]} {:a [3]} {:a [4 5]}])
;; => 9
```

Here are some more examples of using traverse-all:

```clojure
=> (into [] (traverse-all :a) [{:a 1} {:a 2}])
[1 2]
=> (transduce (traverse-all [ALL :a])
		  +
		  0
		  [[{:a 1} {:a 2}] [{:a 3}]])
6
=> (transduce (comp (mapcat identity)
			(traverse-all :a))
		  (completing (fn [r i] (if (= i 4) (reduced r) (+ r i))))
		  0
		  [[{:a 1}] [{:a 2}] [{:a 4}] [{:a 5}]])
3
```

# Path Macros

## declarepath

`(declarepath name)`

`(declarepath name params)`

Declares a new symbol available to be defined as a path. If the path will require parameters, these must be specified here. The path itself must be defined using [providepath](#providepath). `declarepath` and `providepath` are great for defining recursive navigators, as seen in the second example below.

```clojure
=> (declarepath SECOND)
=> (providepath SECOND [(srange 1 2) FIRST])
=> (select-one SECOND (range 5))
1
=> (transform SECOND dec (range 5))
(0 0 2 3 4)
=> (declarepath DEEP-MAP-VALS)
=> (providepath DEEP-MAP-VALS (if-path map? [MAP-VALS DEEP-MAP-VALS] STAY))
=> (select DEEP-MAP-VALS {:a {:b 2} :c {:d 3 :e {:f 4}} :g 5})
[2 3 4 5]
=> (transform DEEP-MAP-VALS inc {:a {:b 2} :c {:d 3 :e {:f 4}} :g 5})
{:a {:b 3}, :c {:d 4, :e {:f 5}}, :g 6}
```

## defprotocolpath

`(defprotocolpath name)`

`(defprotocolpath name params)`

Defines a navigator that chooses the path to take based on the type
of the value at the current point. May be specified with parameters to
specify that all extensions must require that number of parameters.

Currently not available for ClojureScript.

```clojure
=> (defrecord SingleAccount [funds])
=> (defrecord FamilyAccount [single-accounts])

=> (defprotocolpath FundsPath)
=> (extend-protocolpath FundsPath
     SingleAccount :funds
     FamilyAccount [:single-accounts ALL FundsPath])
=> (select [ALL FundsPath] 
     [(->SingleAccount 100) (->SingleAccount 3) 
      (->FamilyAccount [(->SingleAccount 15) (->SingleAccount 12)])])
[100 3 15 12]

=> (defprotocolpath AfterFeePath [fee-fn])
=> (extend-protocolpath AfterFeePath 
     SingleAccount [:funds view] 
     FamilyAccount [:single-accounts ALL AfterFeePath])
=> (select [ALL (AfterFeePath dec)] 
     [(->SingleAccount 100) (->SingleAccount 3) 
      (->FamilyAccount [(->SingleAccount 15) (->SingleAccount 12)])])
[99 2 14 11]
```

## extend-protocolpath

`(extend-protocolpath protpath & extensions)`

Extends a protocol path `protpath` to a list of types. The `extensions` argument has the form `type1 path1 type2 path2...`.

See [defprotocolpath](#defprotocolpath) for an example.

## path

`(path & path)`

Same as calling `comp-paths`, except it caches the composition of the static part
of the path for later re-use (when possible). For almost all idiomatic uses
of Specter provides huge speedup. This macro is automatically used by the
select/transform/setval/replace-in/etc. macros.

Any higher order navigators passed to `path` must include their arguments, even if their arguments will be evaluated at runtime.

```clojure
=> (def p (path even?))
=> (select [ALL p] (range 10))
[0 2 4 6 8]
```

## providepath

`(providepath name apath)`

Defines the path that will be associated to the provided name. The name must have been previously declared using [declarepath](#declarepath).

## recursive-path

`(recursive-path params self-sym path)`

Assists in making recursive paths, both parameterized and unparameterized.

Here is an example of using `recursive-path` without parameters to select and transform:

```clojure
=> (def tree-walker (recursive-path [] p (if-path vector? [ALL p] STAY)))
#'playground.specter/tree-walker
;; Get all of the values nested within vectors
=> (select tree-walker [1 [2 [3 4] 5] [[6]]])
[1 2 3 4 5 6]
;; Transform all of the values within vectors
=> (transform tree-walker inc [1 [2 [3 4] 5] [[6]]])
[2 [3 [4 5] 6] [[7]]]
```

And here is an example of using `recursive-path` with parameters to select and transform:

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

# Collector Macros

## defcollector

`(defcollector name params collect-val-impl)`

Defines a collector with the given name and parameters. Collectors are navigators which add a value to the list of collected values and do not change the current structure.

Note that `params` should be a vector, as would follow `fn`.

`collect-val-impl` must be of the form `(collect-val [this structure] body)`. It should return the value to be collected.

An informative example is the actual implementation of `putval`, which follows.

```clojure
=> (defcollector putval [val]
     (collect-val [this structure]
       val))
=> (transform [ALL (putval 3)] + (range 5))
(3 4 5 6 7)
```

# Navigator Macros

## defnav

`(defnav name params select-impl transform-impl)`

`(defnav name params transform-impl select-impl)`

Canonically the first is used.

Defines a navigator with given name and parameters. Note that `params` should be a vector,
as would follow `fn`.

`select-impl` must be of the form `(select* [this structure next-fn] body)`. It should return the result of calling `next-fn` on whatever subcollection of `structure` this navigator selects.

`transform-impl` must be of the form `(transform* [this structure next-fn] body)`. It should find the result of calling `nextfn` on whatever subcollection of `structure` this navigator selects. Then it should return the result of reconstructing the original structure using the results of the `nextfn` call.

See also [nav](#nav)

```clojure
=> (defnav nth-elt [n] 
     (select* [this structure next-fn] (next-fn (nth structure n)))
     (transform* [this structure next-fn] 
       (let [structurev (vec structure) 
             ret (next-fn (nth structure n))] 
         (if (vector? structure) 
           (assoc structurev n ret)
           (concat (take n structure) (list ret) (drop (inc n) structure))))))
=> (select-one (nth-elt 0) (range 5))
0
=> (select-one (nth-elt 3) (range 5))
3
=> (select-one (nth-elt 3) (range 0 10 2))
6
=> (transform (nth-elt 1) inc (range 5))
(0 2 2 3 4)
=> (transform (nth-elt 1) inc (vec (range 5)))
[0 2 2 3 4]
```

## nav

`(nav params select-impl transform-impl)`

`(nav params transform-impl select-impl)`

Returns an "anonymous navigator." See [defnav](#defnav).
