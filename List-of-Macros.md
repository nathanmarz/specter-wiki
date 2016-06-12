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

### providepath

`(providepath name apath)`

Defines the path that will be associated to the provided name. The name must have been previously declared using [declarepath](#declarepath).

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
