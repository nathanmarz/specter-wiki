# List of Macros with Examples

## Path Macros

### declarepath

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

### defpathedfn

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

The syntax is the same as `defn` (optional docstring, etc.).

```clojure
=> (defpathedfn walk-pred 
     "Walks to integers satisfying apred." 
     [^:notpath apred] 
     (walker #(and (integer? %) (apred %))))
=> (select (walk-pred even?) [1 [2] [[[3 4]]] [5 [[6]] 7 8]])
(2 4 6 8)
=> (transform (walk-pred even?) #(/ % 2) [1 2 [3] [[[4]]] [5 6] [[7 8]]])
[1 1 [3] [[[2]]] [5 3] [[7 4]]]


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

### defprotocolpath

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

### extend-protocolpath

`(extend-protocolpath protpath & extensions)`

Extends a protocol path `protpath` to a list of types. The `extensions` argument has the form `type1 path1 type2 path2...`.

See [defprotocolpath](#defprotocolpath) for an example.

### fixed-pathed-nav

`(fixed-pathed-nav bindings select-impl transform-impl)`

`(fixed-pathed-nav bindings transform-impl select-impl)`

The first form is canonical.

This helper is used to define navigators that take in a fixed number of other
paths as input. Those paths may require late-bound params, so this helper
will create a parameterized navigator if that is the case. If no late-bound params
are required, then the result is executable.

`bindings` must be of the form `[path-binding1 path1 path-binding2 path2...]`.

`select-impl` must be of the form `(select* [this structure next-fn] body)`. It should return the result of calling `next-fn` on whatever transformation the navigator applies to `structure`.

`transform-impl` must be of the form `(transform* [this structure next-fn] body)`. It should find the result of calling `nextfn` on whatever transformation the navigator applies to `structure`. Then it should return the result of reconstructing the original structure using the results of the `nextfn` call.

See [defpathedfn](#defpathedfn) for an example.

### providepath

`(providepath name apath)`

Defines the path that will be associated to the provided name. The name must have been previously declared using [declarepath](#declarepath).

### variable-pathed-nav

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
=> (defpathedfn multi-path
     [& paths]
     (variable-pathed-nav [compiled-paths paths]
       (select* [this structure next-fn]
         (->> compiled-paths
              ;; seq with the results of navigating each passed in path
              (mapcat #(compiled-select % structure))
              ;; pass each result to the following navigator
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

## Collector Macros

### defcollector

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

### paramscollector

`(paramscollector params collect-val-impl)`

Defines a Collector with late bound parameters. This collector can be precompiled
with other selectors without knowing the parameters. When precompiled with other
selectors, the resulting selector takes in parameters for all selectors in the path
that needed parameters (in the order in which they were declared).

Returns an "anonymous Collector." See [defcollector](#defcollector).

### pathed-collector

`(pathed-collector [name path] collect-val-impl)`

This helper is used to define collectors that take in a single selector
paths as input. That path may require late-bound params, so this helper
will create a parameterized selector if that is the case. If no late-bound params
are required, then the result is executable.

Binds the passed in path to `name`.

`collect-val-impl` must be of the form `(collect-val [this structure] body)`. It should return the value to be collected.

The implementation of `collect` is a good example of how `pathed-collector` can be used.

```clojure
=> (defpathedfn
     collect
     [& path]
     (pathed-collector [late path]
       (collect-val [this structure]
         (compiled-select late structure))))
```

## Navigator Macros

### defnav

`(defnav name params select-impl transform-impl)`

`(defnav name params transform-impl select-impl)`

Canonically the first is used.

Defines a navigator with given name and parameters. Note that `params` should be a vector,
as would follow `fn`.

`select-impl` must be of the form `(select* [this structure next-fn] body)`. It should return the result of calling `next-fn` on whatever transformation the navigator applies to `structure`.

`transform-impl` must be of the form `(transform* [this structure next-fn] body)`. It should find the result of calling `nextfn` on whatever transformation the navigator applies to `structure`. Then it should return the result of reconstructing the original structure using the results of the `nextfn` call.

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

### nav

`(nav params select-impl transform-impl)`

`(nav params transform-impl select-impl)`

Returns an "anonymous navigator." See [defnav](#defnav).

### paramsfn

`(paramsfn params [structure-binding] impl)`

Helper macro for defining filter functions with late binding parameters.

```clojure
=> (let [c-path (comp-paths (paramsfn [val] [x] (< x val)))]
     (select [ALL (c-path 5)] [2 7 3 4 10 8]))
[2 3 4]
```
