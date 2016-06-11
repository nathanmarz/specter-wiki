# List of Navigators with Example(s)

## The All Caps Ones

### ALL

`ALL` navigates to every element in a collection. If the collection is a map, it will navigate to each key-value pair `[key value]`. The resulting elements will be reconstructed as a vector.

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

`BEGINNING` navigates to the empty subsequence before the beginning of a collection. Useful with `transform` to add values onto the beginning of a sequence. Returns a lazy sequence.

```clojure
=> (transform [BEGINNING] (fn [_] '(0 1)) (range 2 7))
(0 1 2 3 4 5 6)
=> (transform [BEGINNING] (fn [_] [0 1]) (range 2 7))
(0 1 2 3 4 5 6)
=> (transform [BEGINNING] (fn [_] {0 1}) (range 2 7))
([0 1] 2 3 4 5 6)
=> (transform [BEGINNING] (fn [_] {:foo :baz}) {:foo :bar})
([:foo :baz] [:foo :bar])
```


### END

`END` navigates to the empty subsequence after the end of a collection. Useful with `transform` to add values onto the end of a sequence. Returns a lazy sequence.

```clojure
=> (transform [END] (fn [_] '(5 6)) (range 5))
(0 1 2 3 4 5 6)
=> (transform [END] (fn [_] [5 6]) (range 5))
(0 1 2 3 4 5 6)
=> (transform [END] (fn [_] {5 6}) (range 5))
(0 1 2 3 4 [5 6])
=> (transform [END] (fn [_] {:foo :baz}) {:foo :bar})
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
=> (select [MAP-VALS MAP-VALS] {:a {:b :c} :d {:e :f}})
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
=> (transform [(collect ALL even?) FIRST] (fn [evens first] (reduce + first evens)) (range 5))
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
=> (transform [(collect-one :b) :a] + {:a 2 :b 3})
{:a 5 :b 3}
=> (transform [(collect-one :b) (collect-one :c) :a] * {:a 3 :b 5 :c 7})
{:a 105 :b 5 :c 7}