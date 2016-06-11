# List of Navigators with Example(s)

## The All Caps Ones

### ALL

`ALL` navigates to every element in a collection. If the collection is a map, it will navigate to each key-value pair `[key value]`.

```clojure
=> (select [ALL] [0 1 2 3])
[0 1 2 3]
=> (select [ALL] (list 0 1 2 3))
[0 1 2 3]
=> (select [ALL] {:a :b, :c :d, :e :f})
[[:a :b] [:c :d] [:e :f]]
```

### ATOM

`ATOM` navigates to the value of an atom.

```clojure
=> (def a (atom 0))
=> (select-one [ATOM] a)
0
=> (swap! a inc)
=> (select-one [ATOM] a)
1
```

### BEGINNING

`BEGINNING` navigates to the empty subsequence before the beginning of a collection. Useful with `setval` to add values onto the beginning of a sequence. Returns a lazy sequence.

```clojure
=> (setval [BEGINNING] '(0 1) (range 2 7))
(0 1 2 3 4 5 6)
=> (setval [BEGINNING] [0 1] (range 2 7))
(0 1 2 3 4 5 6)
=> (setval [BEGINNING] {0 1} (range 2 7))
([0 1] 2 3 4 5 6)
=> (setval [BEGINNING] {:foo :baz} {:foo :bar})
([:foo :baz] [:foo :bar])
```

### END

`END` navigates to the empty subsequence after the end of a collection. Useful with `setval` to add values onto the end of a sequence. Returns a lazy sequence.

```clojure
=> (setval [END] '(5 6) (range 5))
(0 1 2 3 4 5 6)
=> (setval [END] [5 6] (range 5))
(0 1 2 3 4 5 6)
=> (setval [END] {5 6} (range 5))
(0 1 2 3 4 [5 6])
=> (setval [END] {:foo :baz} {:foo :bar})
([:foo :bar] [:foo :baz])
```

### FIRST

`FIRST` navigates to the first element of a collection. If the collection is a map, returns the key-value pair `[key value]`. If the collection is empty, navigation stops.

```clojure
=> (select-one [FIRST] (range 5))
0
=> (select-one [FIRST] (sorted-map 0 :a 1 :b))
[0 :a]
=> (select-one [FIRST] (sorted-set 0 1 2 3))
0
=> (select-one [FIRST] '())
nil
=> (select [FIRST] '())
nil
```

### LAST

`LAST` navigates to the last element of a collection. If the collection is a map, returns the key-value pair `[key value]`. If the collection is empty, navigation stops.

```clojure
=> (select-one [LAST] (range 5))
4
=> (select-one [LAST] (sorted-map 0 :a 1 :b))
[1 :b]
=> (select-one [LAST] (sorted-set 0 1 2 3))
3
=> (select-one [LAST] '())
nil
=> (select [LAST] '())
nil
```

### MAP-VALS

`MAP-VALS` navigates to every value in a map. `MAP-VALS` is more efficient than `[ALL LAST]`. Note that `MAP-VALS` returns a lazy seq.

```clojure
=> (select [MAP-VALS] {:a :b, :c :d})
(:b :d)
=> (select [MAP-VALS MAP-VALS] {:a {:b :c}, :d {:e :f}})
(:c :f)
```

### NIL->LIST

`NIL->LIST` navigates to the empty list `'()` if the value is nil. Otherwise it stays at the current value.

```clojure
=> (select-one [NIL->LIST] nil)
()
=> (select-one [NIL->LIST] :foo)
:foo
```

### NIL->SET

`NIL->SET` navigates to the empty set `#{}` if the value is nil. Otherwise it stays at the current value.

```clojure
=> (select-one [NIL->LIST] nil)
#{}
=> (select-one [NIL->LIST] :foo)
:foo
```

### NIL->VECTOR

`NIL->VECTOR` navigates to the empty vector `[]` if the value is nil. Otherwise it stays at the current value.

```clojure
=> (select-one [NIL->LIST] nil)
[]
=> (select-one [NIL->LIST] :foo)
:foo
```

### STAY

`STAY` stays in place. It is the no-op navigator.

```clojure
=> (select-one [STAY] :foo)
:foo
```

### STOP

`STOP` stops navigation. For selection, returns nil. For transformation, returns the structure unchanged.

```clojure
=> (select-one [STOP] :foo)
nil
=> (select [ALL STOP] (range 5))
[]
=> (transform [ALL STOP] inc (range 5))
(0 1 2 3 4)
```

## the lower case ones

### bind-params*

Binds late binding params. Write this one later.

### codewalker

Walks code? Let's do this one later.

### collect

`(collect & paths)`

`collect` adds the result of running `collect` with the given path on the current value to the collected vals. Note that `collect`, like `select`, returns a vector containing its results. If `transform` is called, each collected value will be passed in as an argument to the transforming function, with the resulting value as the last argument.

```clojure
=> (select-one [(collect ALL) FIRST] (range 3))
[[0 1 2] 0]
=> (select [(collect ALL) ALL] (range 3))
[[[0 1 2] 0] [[0 1 2] 1] [[0 1 2] 2]]
=> (select [(collect ALL) (collect ALL) ALL] (range 3))
[[[0 1 2] [0 1 2] 0] [[0 1 2] [0 1 2] 1] [[0 1 2] [0 1 2] 2]]
;; Add the sum of the evens to the first element of the seq
=> (transform [(collect ALL even?) FIRST] 
              (fn [evens first] (reduce + first evens))
              (range 5))
(6 1 2 3 4)
;; Replace the first element of the seq with the entire seq
=> (transform [(collect ALL) FIRST] (fn [all _] all) (range 3))
([0 1 2] 1 2)
```

### collect-one

`(collect-one & paths)`

`collect-one` adds the result of running `collect` with the given path on the current value to the collected vals. Note that `collect-one`, like `select-one`, returns a single result. If there is more than one result, an exception will be thrown. If `transform` is called, each collected value will be passed in as an argument to the transforming function, with the resulting value as the last argument.

```clojure
=> (select-one [(collect-one FIRST) LAST] (range 5))
[0 4]
=> (select [(collect-one FIRST) ALL] (range 3))
[[0 0] [0 1] [0 2]]
=> (transform [(collect-one :b) :a] + {:a 2, :b 3})
{:a 5, :b 3}
=> (transform [(collect-one :b) (collect-one :c) :a] * {:a 3, :b 5, :c 7})
{:a 105,, :b 5 :c 7}
```

### comp-paths

`(comp-paths & path)`

Returns a compiled version of the given path for use with compiled-{select/transform/setval/etc.} functions. This can compile navigators (defined with `defnav`) without their parameters, and the resulting compiled
path will require parameters for all such navigators in the order in which they were declared. Provides a speed improvement of about 2-15% over the inline caching introduced with version 0.11.2.

```clojure
=> (let [my-path (comp-paths :a :b :c)]
     (compiled-select-one my-path {:a {:b {:c 0}}}))
0
=> (let [param-path (comp-paths :a :b keypath))]
     (compiled-transform (param-path :c) inc {:a {:b {:c 0, :d 1}}})
{:a {:b {:c 1, :d 1}}}
=> (let [range-path (comp-paths srange)]
     (compiled-select-one (range-path 1 4) (range 4)))
[1 2 3]
```

### compiled-*

These functions operate in the same way as their uncompiled counterparts, but they require their path to be precompiled with [comp-paths](#comp-paths).

### cond-path

`(cond-path & conds)`

Takes as arguments alternating `cond-path1 path1 cond-path2 path2...`
Tests if selecting with cond-path on the current structure returns anything.
If so, it navigates to the corresponding path.
Otherwise, it tries the next cond-path. If nothing matches, then the structure
is not selected.

The input paths may be parameterized, in which case the result of cond-path
will be parameterized in the order of which the parameterized navigators
were declared.

```clojure
=> (select [ALL (cond-path (must :a) :a (must :b) :b)] [{:a 0} {:b 1} {:c 2}])
[0 1]
=> (select [cond-path (must :a) :a] {:b 1})
nil
```

### continue-then-stay

`(continue-then-stay & path)`

Navigates to the provided path and then to the current element. This can be used
to implement post-order traversal.

```clojure
=> (select [(continue-then-stay MAP-VALS)] {:a 0 :b 1 :c 2})
(0 1 2 {:a 0, :b 1, :c 2})
```

### continuous-subseqs

`(continuous-subseqs pred)`

Navigates to every continuous subsequence of elements matching `pred`.

```clojure
=> (select [(continuous-subseqs #(< % 10))] [5 6 11 11 3 12 2 5])
([5 6] [3] [2 5])
=> (select [(continuous-subseqs #(< % 10))] [12 13])
()
```

### filterer

`(filterer & path)`

Navigates to a view of the current sequence that only contains elements that
match the given path. An element matches the selector path if calling select
on that element with the path yields anything other than an empty sequence.

The input path may be parameterized, in which case the result of filterer
will be parameterized in the order of which the parameterized selectors
were declared. Note that filterer is a function which returns a navigator. It is the arguments to filterer that can be parametrized, not filterer.

```clojure
;; Note that clojure functions have been extended to implement the navigator protocol
=> (select-one [(filterer even?)] (range 10))
[0 2 4 6 8]
=> (select-one [(filterer identity)] ['() [] #{} {} "" true false nil])
[() [] #{} {} "" true]
=> (let [pred-path (comp-paths (filterer pred))]
     (select-one (pred-path even?) (range 10)))
[0 2 4 6 8]
=> (let [pred-path (comp-paths filterer)]
     (select-one (pred-path even?) (range 10)))
ClassCastException com.rpl.specter.impl.CompiledPath cannot be cast to clojure.lang.IFn
```

### if-path

`(if-path cond-path then-path)`
`(if-path cond-path then-path else-path)`

Like [cond-path](#cond-path), but with if semantics. If no else path is supplied and cond-path is not satisfied, stops navigation.

```clojure
=> (select [(if-path (must :d) :a)] {:a 0, :d 1})
(0)
=> (select [(if-path (must :d) :a :b)] {:a 0, :b 1})
(1)
=> (select [(if-path (must :d) :a)] {:b 0, :d 1})
()
;; is equivalent to
=> (select [(if-path (must :d) :a STOP)] {:b 0, :d 1})
()
```

### keypath

`(keypath key)`

Navigates to the specified key, navigating to nil if it does not exist. Note that this is different from stopping navigation if the key does not exist. If you want to stop navigation, use [must](#must).

```clojure
=> (select-one [(keypath :a)] {:a 0})
0
;; Only one key allowed
=> (select-one [(keypath :a :b)] {:a {:b 1}})
{:b 1}
=> (select [ALL (keypath :a)] [{:a 0} {:b 1}])
[0 nil]
;; Does not stop navigation
=> (select [ALL (keypath :a) (nil->val :boo)] [{:a 0} {:b 1}])
[0 :boo]
```

### multi-path

`(multi-path & paths)`

A path that branches on multiple paths. For updates,
applies updates to the paths in order.

```clojure
=> (select [(multi-path :a :b)] {:a 0, :b 1, :c 2})
(0 1)
=> (select [(multi-path (filterer odd?) (filterer even?))] (range 10))
([1 3 5 7 9] [0 2 4 6 8])
=> (transform [(multi-path :a :b)] (fn [x] (println x) (dec x)) {:a 0, :b 1, :c 2})
0
1
{:a -1, :b 0, :c 2}
```

### must

`(must key)`

Navigates to the key only if it exists in the map. Note that must stops navigation if the key does not exist. If you do not want to stop navigation, use [keypath](#keypath).

```clojure
=> (select-one (must :a) {:a 0})
0
=> (select-one (must :a) {:b 1})
nil
;; Only follows one key
=> (select-one (must :a :b) {:a {:b 1}})
{:b 1}
```