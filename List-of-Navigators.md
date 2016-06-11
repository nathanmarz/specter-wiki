# List of Navigators with Example(s)

## The All Caps Ones

[to-atom](#atom)

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

