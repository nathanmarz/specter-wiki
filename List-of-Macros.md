<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-generate-toc again -->
**Table of Contents**

- [List of Macros with Examples](#list-of-macros-with-examples)
    - [Core Macros](#core-macros)
        - [replace-in](#replace-in)
        - [select](#select)
        - [select-first](#select-first)
        - [select-one](#select-one)
        - [select-one!](#select-one)
        - [setval](#setval)
        - [transform](#transform)
    - [Path Macros](#path-macros)
        - [declarepath](#declarepath)
        - [defpathedfn](#defpathedfn)
        - [defprotocolpath](#defprotocolpath)
        - [extend-protocolpath](#extend-protocolpath)
        - [fixed-pathed-nav](#fixed-pathed-nav)
        - [path](#path)
        - [providepath](#providepath)
        - [variable-pathed-nav](#variable-pathed-nav)
    - [Collector Macros](#collector-macros)
        - [defcollector](#defcollector)
        - [paramscollector](#paramscollector)
        - [pathed-collector](#pathed-collector)
    - [Navigator Macros](#navigator-macros)
        - [defnav](#defnav)
        - [defnavconstructor](#defnavconstructor)
        - [nav](#nav)
        - [paramsfn](#paramsfn)

<!-- markdown-toc end -->


# Core Macros

## replace-in

`(replace-in apath transform-fn structure & args)`

Similar to transform, except returns a pair of [transformed-structure sequence-of-user-ret].
The transform-fn in this case is expected to return [ret user-ret]. ret is
what's used to transform the data structure, while user-ret will be added to the user-ret sequence
in the final return. replace-in is useful for situations where you need to know the specific values
of what was transformed in the data structure.
This macro will attempt to do inline factoring and caching of the path, falling
back to compiling the path on every invocation it it's not possible to 
factor/cache the path.

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
This macro will attempt to do inline factoring and caching of the path, falling
back to compiling the path on every invocation it it's not possible to 
factor/cache the path.

```clojure
=> (select [ALL even?] (range 10))
[0 2 4 6 8]
=> (select :a {:a 0 :b 1})
[0]
=> (select ALL {:a 0 :b 1})
[[:a 0] [:b 1]]
```

## select-first

`(select-first apath structure)`

Returns first element found. Not any more efficient than select, just a convenience.
This macro will attempt to do inline factoring and caching of the path, falling
back to compiling the path on every invocation it it's not possible to 
factor/cache the path.

```clojure
=> (select-first ALL (range 10))
0
;; Returns the result itself if the result is not a sequence
=> (select-first FIRST (range 10))
0
```

## select-one

`(select-one apath structure)`

Like select, but returns either one element or nil. Throws exception if multiple elements found.
This macro will attempt to do inline factoring and caching of the path, falling
back to compiling the path on every invocation it it's not possible to 
factor/cache the path.

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
This macro will attempt to do inline factoring and caching of the path, falling
back to compiling the path on every invocation it it's not possible to 
factor/cache the path.

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
This macro will attempt to do inline factoring and caching of the path, falling
back to compiling the path on every invocation it it's not possible to 
factor/cache the path.

```clojure
=> (setval [ALL even?] :even (range 10))
(:even 1 :even 3 :even 5 :even 7 :even 9)
```

## transform

`(transform apath transform-fn structure)`

Navigates to each value specified by the path and replaces it by the result of running
the transform-fn on it.
This macro will attempt to do inline factoring and caching of the path, falling
back to compiling the path on every invocation it it's not possible to 
factor/cache the path.

Note that `transform` takes as its initial arguments any collected values. Its last argument will be the structure navigated to by the passed in path.

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

# Path Macros

## declarepath

`(declarepath name)`

`(declarepath name params)`

Declares a new symbol available to be defined as a path. If the path will require parameters, these must be specified here. The path itself must be defined using [providepath](#providepath).

```clojure
=> (declarepath SECOND)
=> (providepath SECOND [(srange 1 2) FIRST])
=> (select-one SECOND (range 5))
1
=> (transform SECOND dec (range 5))
(0 0 2 3 4)
```


## defpathedfn

`(defpathedfn name & args)`

Defines a higher order navigator (a function that returns a navigator) that itself takes in one or more paths
as input. This macro is generally used in conjunction with [fixed-pathed-nav](#fixed-pathed-nav)
or [variable-pathed-nav](#variable-pathed-nav). When inline factoring is applied to a path containing
one of these higher order navigators, it will automatically interepret all 
arguments as paths, factor them accordingly, and set up the callsite to 
provide the parameters dynamically. Use `^:notpath` metadata on arguments 
to indicate non-path arguments that should not be factored – note that in order
to be inline factorable, these arguments must be statically resolvable (e.g. a 
top level var).

The syntax is the same as `defn` (optional docstring, etc.). Note that `defpathedfn` should take **paths** as input. For a parameterized navigator which takes non-path arguments, use [defnavconstructor](#defnavconstructor) to wrap an existing navigator or [defnav](#defnav) to define your own custom navigator.

```clojure
;; The implementation of transformed
(defpathedfn transformed
  "Navigates to a view of the current value by transforming it with the
   specified path and update-fn.
   The input path may be parameterized, in which case the result of transformed
   will be parameterized in the order of which the parameterized navigators
   were declared."
  [path ^:notpath update-fn]
  ;; Bind the passed in path to late
  ;; Returns a navigator with the given select* and transform* implementations
  (fixed-pathed-nav [late path]
    (select* [this structure next-fn]
      (next-fn (compiled-transform late update-fn structure)))
    (transform* [this structure next-fn]
      (next-fn (compiled-transform late update-fn structure)))))
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

## fixed-pathed-nav

`(fixed-pathed-nav bindings select-impl transform-impl)`

`(fixed-pathed-nav bindings transform-impl select-impl)`

The first form is canonical.

This helper is used to define navigators that take in a fixed number of other
paths as input. Those paths may require late-bound params, so this helper
will create a parameterized navigator if that is the case. If no late-bound params
are required, then the result is executable.

`bindings` must be of the form `[path-binding1 path1 path-binding2 path2...]`.

`select-impl` must be of the form `(select* [this structure next-fn] body)`. It should return the result of calling `next-fn` on whatever subcollection of `structure` this navigator selects.

`transform-impl` must be of the form `(transform* [this structure next-fn] body)`. It should find the result of calling `nextfn` on whatever subcollection of `structure` this navigator selects. Then it should return the result of reconstructing the original structure using the results of the `nextfn` call.

See [defpathedfn](#defpathedfn) for an example.

## path

`(path & path)`

Same as calling comp-paths, except it caches the composition of the static part
of the path for later re-use (when possible). For almost all idiomatic uses
of Specter provides huge speedup. This macro is automatically used by the
select/transform/setval/replace-in/etc. macros.

```clojure
```

## providepath

`(providepath name apath)`

Defines the path that will be associated to the provided name. The name must have been previously declared using [declarepath](#declarepath).

## variable-pathed-nav

`(variable-pathed-nav [paths-binding paths-seq] select-impl transform-impl)`

`(variable-pathed-nav [paths-binding paths-seq] transform-impl select-impl)`

The first form is canonical.

This helper is used to define navigators that take in a variable number of other
paths as input. Those paths may require late-bound params, so this helper
will create a parameterized navigator if that is the case. If no late-bound params
are required, then the result is executable.

Binds the passed in seq of paths to `paths-binding`, which can be used in `select-impl` and `transform-impl`.

The implementation of `multi-path` is a nice example of the use of `variable-pathed-nav`.

```clojure
(defpathedfn multi-path [& paths]
  (variable-pathed-nav [compiled-paths paths]
    (select* [this structure next-fn]
      (->> compiled-paths
           ;; seq with the results of navigating each passed in path
           (mapcat #(compiled-select % structure))
           ;; pass each result to the next navigator
           (mapcat next-fn)
           doall))
    (transform* [this structure next-fn]
      ;; apply the transform to each passed in path in order
      (reduce
        (fn [structure path]
          (compiled-transform path next-fn structure))
        structure
        compiled-paths))))
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

## paramscollector

`(paramscollector params collect-val-impl)`

Defines a Collector with late bound parameters. This collector can be precompiled
with other selectors without knowing the parameters. When precompiled with other
selectors, the resulting selector takes in parameters for all selectors in the path
that needed parameters (in the order in which they were declared).

Returns an "anonymous Collector." See [defcollector](#defcollector).

## pathed-collector

`(pathed-collector [name path] collect-val-impl)`

This helper is used to define collectors that take in a single selector
paths as input. That path may require late-bound params, so this helper
will create a parameterized selector if that is the case. If no late-bound params
are required, then the result is executable.

Binds the passed in path to `name`.

`collect-val-impl` must be of the form `(collect-val [this structure] body)`. It should return the value to be collected.

The implementation of `collect` is a good example of how `pathed-collector` can be used.

```clojure
(defpathedfn collect [& path]
  (pathed-collector [late path]
    (collect-val [this structure]
      (compiled-select late structure))))
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

## defnavconstructor

`(defnavconstructor name nav-binding params & body)`

Defines a constructor for a previously defined navigator. Allows for arbitrary specification of the arguments of the navigator.

Note that `defnavconstructor` takes an optional docstring and metadata in the same form as `clojure.core/defn`.

`nav-binding` should have the form `[binding navigator]`.

`params` should be a vector of parameters that your navigator will take as arguments.

`body` should be a form that returns a navigator.

```clojure
;; A constructor for the walker navigator which adds the requirement that the
;; structure be an integer to walker's afn predicate
=> (defnavconstructor walk-ints
     "Arguments passed to this walker's afn must also be integers to return true."
     [p walker]
     [apred]
     (p #(and (integer? %) (apred %))))
=> (select (walk-ints even?) [1 [[[[2]] 3 4]] 5 [6 7] [[[8]]]])
(2 4 6 8)
=> (select (walk-ints #(< % 5)) (range 7))
(0 1 2 3 4)
;; A constructor for the pred navigator which takes two predicate functions
;; as arguments. If either is satisfied, pred will continue navigation.
=> (defnavconstructor or-pred
     [p pred]
     [pred1 pred2]
     (p #(or (pred1 %) (pred2 %))))
=> (select [ALL (or-pred even? #(> % 6))] (range 10))
[0 2 4 6 7 8 9]
```

## nav

`(nav params select-impl transform-impl)`

`(nav params transform-impl select-impl)`

Returns an "anonymous navigator." See [defnav](#defnav).

## paramsfn

`(paramsfn params [structure-binding] impl)`

Helper macro for defining filter functions with late binding parameters.

```clojure
;; val is the parameter that will be bound at a later time to complete the navigator
;; x represents the current structure for the navigator
=> (def less-than-n-pred (comp-paths (paramsfn [val] [x] (< x val))))
=> (select [ALL (less-than-n-pred 5)] [2 7 3 4 10 8])
[2 3 4]
=> (select [ALL (less-than-n-pred 9)] [2 7 3 4 10 8])
[2 7 3 4 8]
=> (transform [ALL (less-than-n-pred 9)] inc [2 7 3 4 10 8])
[3 8 4 5 10 9]
;; bot and top are late bound parameters
;; x represents the current structure for the navigator
=> (def between (comp-paths (paramsfn [bot top] [x] (and (< bot x) (< x top)))))
=> (select [ALL (between 1 5)] (range 10))
[2 3 4]
```
