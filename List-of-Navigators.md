**Note:** Many of the descriptions and a couple of the examples are lightly edited from those found on the [Codox documentation](http://nathanmarz.github.io/specter/com.rpl.specter.html).
<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-generate-toc again -->
**Table of Contents**

- [Unparameterized Navigators](#unparameterized-navigators)
    - [AFTER-ELEM](#after-elem)
    - [ALL](#all)
    - [ATOM](#atom)
    - [BEFORE-ELEM](#before-elem)
    - [BEGINNING](#beginning)
    - [DISPENSE](#dispense)
    - [END](#end)
    - [FIRST](#first)
    - [INDEXED-VALS](#indexed-vals)
    - [LAST](#last)
    - [MAP-KEYS](#map-keys)
    - [MAP-VALS](#map-vals)
    - [META](#meta)
    - [NAME](#name)
    - [NAMESPACE](#namespace)
    - [NIL->LIST](#nil-list)
    - [NIL->SET](#nil-set)
    - [NIL->VECTOR](#nil-vector)
    - [NONE-ELEM](#none-elem)
    - [STAY](#stay)
    - [STOP](#stop)
    - [VAL](#val)
- [Parameterized Navigators (and Functions)](#parameterized-navigators-and-functions)
    - [before-index](#before-index)
    - [codewalker](#codewalker)
    - [collect](#collect)
    - [collect-one](#collect-one)
    - [comp-paths](#comp-paths)
    - [compiled-*](#compiled-)
    - [cond-path](#cond-path)
    - [continue-then-stay](#continue-then-stay)
    - [continuous-subseqs](#continuous-subseqs)
    - [eachnav](#eachnav)
    - [filterer](#filterer)
    - [if-path](#if-path)
    - [index-nav](#index-nav)
    - [keypath](#keypath)
    - [map-key](#map-key)
    - [multi-path](#multi-path)
    - [must](#must)
    - [nil->val](#nil-val)
    - [nthpath](#nthpath)
    - [parser](#parser)
    - [pred](#pred)
    - [putval](#putval)
    - [not-selected?](#not-selected)
    - [selected?](#selected)
    - [set-elem](#set-elem)
    - [srange](#srange)
    - [srange-dynamic](#srange-dynamic)
    - [stay-then-continue](#stay-then-continue)
    - [submap](#submap)
    - [subselect](#subselect)
    - [subset](#subset)
    - [terminal](#terminal)
    - [terminal-val](#terminal-val)
    - [transformed](#transformed)
    - [view](#view)
    - [walker](#walker)

<!-- markdown-toc end -->



# Unparameterized Navigators

## AFTER-ELEM

`AFTER-ELEM` navigates to the 'void' element after the sequence. For transformations – if result is not `NONE`, then append that value.

```
=> (setval AFTER-ELEM 3 [1 2])
[1 2 3]
```

## ALL

`ALL` navigates to every element in a collection. If the collection is a map, it will navigate to each key-value pair `[key value]`.

```clojure
=> (select ALL [0 1 2 3])
[0 1 2 3]
=> (select ALL (list 0 1 2 3))
[0 1 2 3]
=> (select ALL {:a :b, :c :d})
[[:a :b] [:c :d]]
=> (transform ALL identity {:a :b, :c :d})
{:a :b, :c :d}
```

`ALL` can transform to `NONE` to remove elements.

```clojure
=> (setval [ALL nil?] NONE [1 2 nil 3 nil])
[1 2 3]
```

## ATOM

`ATOM` navigates to the value of an atom.

```clojure
=> (def a (atom 0))
=> (select-one ATOM a)
0
=> (swap! a inc)
=> (select-one ATOM a)
1
=> (transform ATOM inc a)
=> @a
2
```

## BEFORE-ELEM

`BEFORE-ELEM` navigates to the 'void' element before the sequence. For transformations – if result is not `NONE`, then prepend that value.

```clojure
=> (setval BEFORE-ELEM 3 [1 2])
[3 1 2]
```

## BEGINNING

`BEGINNING` navigates to the empty subsequence before the beginning of a collection. Useful with `setval` to add values onto the beginning of a sequence.

```clojure
=> (setval BEGINNING '(0 1) (range 2 7))
(0 1 2 3 4 5 6)
=> (setval BEGINNING [0 1] (range 2 7))
(0 1 2 3 4 5 6)
=> (setval BEGINNING {0 1} (range 2 7))
([0 1] 2 3 4 5 6)
=> (setval BEGINNING '(0 1) [2 3 4])
[0 1 2 3 4]
=> (setval BEGINNING {:foo :baz} {:foo :bar})
([:foo :baz] [:foo :bar])
```

As of Specter 1.0.0, `BEGINNING` can now work with strings. It navigates to or transforms substrings.

```clojure
=> (select-any BEGINNING "abc")
""
=> (setval BEGINNING "b" "a")
"ba"
```

## DISPENSE

_Added in 0.12.0_

Drops all collected values for subsequent navigation.

```clojure
=> (transform [ALL VAL] + (range 10))
(0 2 4 6 8 10 12 14 16 18)
=> (transform [ALL VAL DISPENSE] + (range 10))
(0 1 2 3 4 5 6 7 8 9)
```

## END

`END` navigates to the empty subsequence after the end of a collection. Useful with `setval` to add values onto the end of a sequence.

```clojure
=> (setval END '(5 6) (range 5))
(0 1 2 3 4 5 6)
=> (setval END [5 6] (range 5))
(0 1 2 3 4 5 6)
=> (setval END {5 6} (range 5))
(0 1 2 3 4 [5 6])
=> (setval END '(5 6) [1 2 3 4])
[1 2 3 4 5 6]
=> (setval END {:foo :baz} {:foo :bar})
([:foo :bar] [:foo :baz])
```

As of Specter 1.0.0, `END` can now work with strings. It navigates to or transforms substrings.

```clojure
=> (select-any END "abc")
""
=> (setval END "b" "a")
"ab"
```

## FIRST

`FIRST` navigates to the first element of a collection. If the collection is a map, returns a key-value pair `[key value]`. If the collection is empty, navigation stops.

```clojure
=> (select-one FIRST (range 5))
0
=> (select-one FIRST (sorted-map 0 :a 1 :b))
[0 :a]
=> (select-one FIRST (sorted-set 0 1 2 3))
0
=> (select-one FIRST '())
nil
=> (select FIRST '())
nil
```

`FIRST` can transform to `NONE` to remove elements.

```clojure
=> (setval FIRST NONE [:a :b :c :d :e])
[:b :c :d :e]
```

As of Specter 1.0.0, `FIRST` can now work with strings. It navigates to or transforms characters.

```clojure
=> (select-any FIRST "abc")
\a
=> (setval FIRST \q "abc")
"qbc"
```

## INDEXED-VALS

`INDEXED-VALS` navigates to [index elem] pairs for each element in a sequence. Transforms of index move element at that index to the new index, shifting other elements in the sequence. Indices seen during transform take into account any shifting from prior sequence elements changing indices.

```clojure
=> (select [INDEXED-VALS] [1 2 3 4 5])
[[0 1] [1 2] [2 3] [3 4] [4 5]]
=> (setval [INDEXED-VALS FIRST] 0 [1 2 3 4 5])
[5 4 3 2 1]
=> (setval [INDEXED-VALS FIRST] 1 [1 2 3 4 5])
[1 5 4 3 2]
```

## LAST

`LAST` navigates to the last element of a collection. If the collection is a map, returns a key-value pair `[key value]`. If the collection is empty, navigation stops.

```clojure
=> (select-one LAST (range 5))
4
=> (select-one LAST (sorted-map 0 :a 1 :b))
[1 :b]
=> (select-one LAST (sorted-set 0 1 2 3))
3
=> (select-one LAST '())
nil
=> (select LAST '())
nil
```

`LAST` can transform to `NONE` to remove elements.

```clojure
=> (setval LAST NONE [:a :b :c :d :e])
[:a :b :c :d]
```

As of Specter 1.0.0, `LAST` can now work with strings. It navigates to or transforms characters.

```clojure
=> (select-any LAST "abc")
\c
=> (setval LAST "q" "abc")
"abq"
```

## MAP-KEYS

`MAP-KEYS` navigates to every key in a map. `MAP-KEYS` is more efficient than `[ALL FIRST]`.

```clojure
=> (select [MAP-KEYS] {:a 3 :b 4})
[:a :b]
```

## MAP-VALS

`MAP-VALS` navigates to every value in a map. `MAP-VALS` is more efficient than `[ALL LAST]`.

```clojure
=> (select MAP-VALS {:a :b, :c :d})
(:b :d)
=> (select [MAP-VALS MAP-VALS] {:a {:b :c}, :d {:e :f}})
(:c :f)
```

`MAP-VALS` can transform to `NONE` to remove elements.

```clojure
=> (setval [MAP-VALS even?] NONE {:a 1 :b 2 :c 3 :d 4})
{:a 1 :c 3}
```

## META

_Added in 0.12.0_

Navigates to the metadata of the structure, or nil if
the structure has no metadata or may not contain metadata.

```clojure
=> (select-one META (with-meta {:a 0} {:meta :data}))
{:meta :data}
=> (meta (transform META #(assoc % :meta :datum)
           (with-meta {:a 0} {:meta :data})))
{:meta :datum}
```

## NAME

`NAME` navigates to the name of a keyword.

```clojure
=> (select [NAME] :key)
["key"]
=> (select [MAP-KEYS NAME] {:a 3 :b 4 :c 5})
["a" "b" "c"]
```

## NAMESPACE

`NAMESPACE` navigates to the namespace of keywords or variables.

```clojure
=> (select [ALL NAMESPACE] [::test ::fun])
["playground.specter" "playground.specter"]
=> (select [ALL NAMESPACE] [::test :fun])
["playground.specter" nil]
```

## NIL->LIST

`NIL->LIST` navigates to the empty list `'()` if the value is nil. Otherwise it stays at the current value.

```clojure
=> (select-one NIL->LIST nil)
()
=> (select-one NIL->LIST :foo)
:foo
```

## NIL->SET

`NIL->SET` navigates to the empty set `#{}` if the value is nil. Otherwise it stays at the current value.

```clojure
=> (select-one NIL->SET nil)
#{}
=> (select-one NIL->SET :foo)
:foo
```

## NIL->VECTOR

`NIL->VECTOR` navigates to the empty vector `[]` if the value is nil. Otherwise it stays at the current value.

```clojure
=> (select-one NIL->VECTOR nil)
[]
=> (select-one NIL->VECTOR :foo)
:foo
```

## NONE-ELEM

`NONE-ELEM` navigates to the 'void' elem in a set. For transformations - if the result is not `NONE`, then add that value to the set.

```
=> (setval NONE-ELEM 3 #{1 2})
#{1 2 3}
=> (setval NONE-ELEM 1 nil)
#{1}
```

## STAY

`STAY` stays in place. It is the no-op navigator.

```clojure
=> (select-one STAY :foo)
:foo
```

## STOP

`STOP` stops navigation. For transformation, returns the structure unchanged.

```clojure
=> (select-one STOP :foo)
nil
=> (select [ALL STOP] (range 5))
[]
=> (transform [ALL STOP] inc (range 5))
(0 1 2 3 4)
```

## VAL

`VAL` collects the current structure.

See also [collect](#collect), [collect-one](#collect-one), and [putval](#putval)

```clojure
=> (select [VAL ALL] (range 3))
[[(0 1 2) 0] [(0 1 2) 1] [(0 1 2) 2]]
;; Collected values are passed as initial arguments to the update fn.
=> (transform [VAL ALL] (fn [coll x] (+ x (count coll))) (range 5))
(5 6 7 8 9)
```

# Parameterized Navigators (and Functions)

## before-index

`(before-index index)`

Navigates to the empty space between the index and the prior index. Selects navigate to NONE.

```clojure
=> (select-any (before-index 0) [1 2 3])
:com.rpl.specter.impl/NONE
```

Transforms to non-NONE insert at the index position.

```
=> (setval (before-index 0) :a [1 2 3])
[:a 1 2 3]
=> (setval (before-index 1) NONE [1 2 3])
[1 2 3]
=> (setval (before-index 1) :a [1 2 3])
[1 :a 2 3]
=> (setval (before-index 3) :a [1 2 3])
[1 2 3 :a]
```

## codewalker

`(codewalker afn)`

Using clojure.walk, `codewalker` executes a depth-first search for nodes where `afn`
returns a truthy value. When `afn` returns a truthy value, `codewalker` stops
searching that branch of the tree and continues its search of the rest of the
data structure. `codewalker` preserves the metadata of any forms traversed.

See also [walker](#walker).

```clojure
=> (select (codewalker #(and (map? %) (even? (:a %))))
           (list (with-meta {:a 2} {:foo :bar}) (with-meta {:a 1} {:foo :baz})))
({:a 2})
=> (map meta *1)
({:foo :bar})
```

## collect

`(collect & paths)`

`collect` adds the result of running `select` with the given path on the current value to the collected vals. Note that `collect`, like `select`, returns a vector containing its results. If `transform` is called, each collected value will be passed as an argument to the transforming function with the resulting value as the last argument.

See also [VAL](#val), [collect-one](#collect-one), and [putval](#putval)

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

## collect-one

`(collect-one & paths)`

`collect-one` adds the result of running `select-one` with the given path on the current value to the collected vals. Note that `collect-one`, like `select-one`, returns a single result. If there is more than one result, an exception will be thrown. If `transform` is called, each collected value will be passed as an argument to the transforming function with the resulting value as the last argument.

See also [VAL](#val), [collect](#collect), and [putval](#putval)

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

## comp-paths

`(comp-paths & path)`

Returns a compiled version of the given path for use with compiled-{select/transform/setval/etc.} functions.

```clojure
=> (let [my-path (comp-paths :a :b :c)]
     (compiled-select-one my-path {:a {:b {:c 0}}}))
0
```

## compiled-*

These functions operate in the same way as their uncompiled counterparts, but they require their path to be precompiled with [comp-paths](#comp-paths).

## cond-path

`(cond-path & conds)`

Takes as arguments alternating `cond-path1 path1 cond-path2 path2...`
Tests if selecting with cond-path on the current structure returns anything.
If so, it navigates to the corresponding path.
Otherwise, it tries the next cond-path. If nothing matches, then the structure
is not selected.

The input paths may be parameterized, in which case the result of cond-path
will be parameterized in the order of which the parameterized navigators
were declared.

See also [if-path](#if-path)

```clojure
=> (select [ALL (cond-path (must :a) :a (must :b) :c)] [{:a 0} {:b 1 :c 2}])
[0 2]
=> (select [(cond-path (must :a) :b)] {:b 1})
()
```

## continue-then-stay

`(continue-then-stay & path)`

Navigates to the provided path and then to the current element. This can be used
to implement post-order traversal.

See also [stay-then-continue](#stay-then-continue).

```clojure
=> (select (continue-then-stay MAP-VALS) {:a 0 :b 1 :c 2})
(0 1 2 {:a 0, :b 1, :c 2})
```

## continuous-subseqs

`(continuous-subseqs pred)`

Navigates to every continuous subsequence of elements matching `pred`.

```clojure
=> (select (continuous-subseqs #(< % 10)) [5 6 11 11 3 12 2 5])
([5 6] [3] [2 5])
=> (select (continuous-subseqs #(< % 10)) [12 13])
()
=> (setval (continuous-subseqs #(< % 10)) [] [3 2 5 11 12 5 20])
[11 12 20]
```

## eachnav

`(eachnav navigator)`

Turns a navigator that takes one argument into a navigator that takes many arguments and uses the same navigator with each argument. There is no performance cost to using this.

`keypath`, `must`, and `nthpath` are all implemented using eachnav, making multiple arguments possible. See their documentation here, or look at their implementation in Specter core.

## filterer

`(filterer & path)`

Navigates to a view of the current sequence that only contains elements that
match the given path. An element matches the selector path if calling select
on that element with the path yields anything other than an empty sequence. Returns a vector when used in a select.

The input path may be parameterized, in which case the result of filterer
will be parameterized in the order of which the parameterized selectors
were declared. Note that filterer is a function which returns a navigator. It is the arguments to filterer that can be late-bound parameterized, not filterer.

See also [subselect](#subselect).

```clojure
;; Note that clojure functions have been extended to implement the navigator protocol
=> (select-one (filterer even?) (range 10))
[0 2 4 6 8]
=> (select-one (filterer identity) ['() [] #{} {} "" true false nil])
[() [] #{} {} "" true]
```

## if-path

`(if-path cond-path then-path)`
`(if-path cond-path then-path else-path)`

Like [cond-path](#cond-path), but with if semantics. If no else path is supplied and cond-path is not satisfied, stops navigation.

See also [if-path](#if-path)

```clojure
=> (select (if-path (must :d) :a) {:a 0, :d 1})
(0)
=> (select (if-path (must :d) :a :b) {:a 0, :b 1})
(1)
=> (select (if-path (must :d) :a) {:b 0, :d 1})
()
;; is equivalent to
=> (select (if-path (must :d) :a STOP) {:b 0, :d 1})
()
```

## index-nav

Navigates to the index of the sequence if within 0 and size. Transforms move element
at that index to the new index, shifting other elements in the sequence.

`(index-nav index)`

```clojure
=> (select [(index-nav 0)] [1 2 3 4 5])
[0]
=> (select [(index-nav 7)] [1 2 3 4 5])
[]
=> (setval (index-nav 2) 0 [1 2 3 4 5])
[3 1 2 4 5]
```

## keypath

`(keypath & keys)`

Navigates to the specified key, navigating to nil if it does not exist. Note that this is different from stopping navigation if the key does not exist. If you want to stop navigation, use [must](#must).

See also [must](#must)

```clojure
=> (select-one (keypath :a) {:a 0})
0
=> (select-one (keypath :a :b) {:a {:b 1}})
1
=> (select [ALL (keypath :a)] [{:a 0} {:b 1}])
[0 nil]
;; Does not stop navigation
=> (select [ALL (keypath :a) (nil->val :boo)] [{:a 0} {:b 1}])
[0 :boo]
```

`keypath` can now take multiple arguments, for concisely specifying multiple steps. It navigates to each key one after another.

```clojure
=> (select-one (keypath "out") {"out" {"in" 3}})
{"in" 3}
=> (select-one (keypath "out" "in") {"out" {"in" 3}})
3
```

`keypath` can transform to `NONE` to remove elements.

```clojure
=> (setval [(keypath :a)] NONE {:a 3 :b 4})
{:b 4}
```

## map-key

`(map-key key)`

Navigates to the given key in the map (not to the value).

```clojure
=> (select [(map-key :a)] {:a 2 :b 3})
[:a]
=> (setval [(map-key :a)] :c {:a 2 :b 3})
{:b 3, :c 2}
```

Navigates only if the key currently exists in the map.

```clojure
=> (select [(map-key :z)] {:a 2 :b 3})
[]
```

Can transform to NONE to remove the key/value pair from the map.

```clojure
=> (setval [(map-key :a)] NONE {:a 2 :b 3})
{:b 3}
```

## multi-path

`(multi-path & paths)`

A path that branches on multiple paths. For transforms,
applies updates to the paths in order.

```clojure
=> (select (multi-path :a :b) {:a 0, :b 1, :c 2})
(0 1)
=> (select (multi-path (filterer odd?) (filterer even?)) (range 10))
([1 3 5 7 9] [0 2 4 6 8])
=> (transform (multi-path :a :b) (fn [x] (println x) (dec x)) {:a 0, :b 1, :c 2})
0
1
{:a -1, :b 0, :c 2}
```

## must

`(must & keys)`

Navigates to the key only if it exists in the map. Note that must stops navigation if the key does not exist. If you do not want to stop navigation, use [keypath](#keypath).

See also [keypath](#keypath) and [pred](#pred).

```clojure
=> (select-one (must :a) {:a 0})
0
=> (select-one (must :a) {:b 1})
nil
```

`must` can now take multiple arguments, for concisely specifying multiple steps. It navigates to each key, one after another.

```clojure
=> (select-any (must :a) {:a {:b 2} :c 3})
{:b 2}
=> (select-any (must :a :b) {:a {:b 2} :c 3})
2
```

`must` can transform to `NONE` to remove elements.

```clojure
=> (setval (must :a) NONE {:a 1 :b 2})
{:b 2}
```

## nil->val

`(nil->val v)`

Navigates to the provided val if the structure is nil. Otherwise it stays
navigated at the structure.

```clojure
=> (select-one (nil->val :a) nil)
:a
=> (select-one (nil->val :a) :b)
:b
```

## nthpath

`(nthpath & indices)`

Navigate to the specified indices (one after another). Transform to NONE to remove the element from the sequence.

```clojure
=> (select [(nthpath 0)] [1 2 3])
[1]
=> (select [(nthpath 2)] [1 2 3])
[3]
=> (setval [(nthpath 2)] NONE [1 2 3])
[1 2]
```

`nthpath` can now take multiple arguments, for concisely specifying multiple steps. It navigates to each index, one after another.

```clojure
=> (select [(nthpath 0)] [1 2 3])
[1]
=> (select [(nthpath 0)] [[0 1 2] 2 3])
[[0 1 2]]
=> (select [(nthpath 0 0)] [[0 1 2] 2 3])
[0]
```

## parser

`(parser parse-fn unparse-fn)`

Navigate to the result of running `parse-fn` on the value. For
transforms, the transformed value then has `unparse-fn` run on
it to get the final value at this point.

```clojure
=> (defn parse [address] (string/split address #"@"))
=> (defn unparse [address] (string/join "@" address))
=> (select [ALL (parser parse unparse) #(= "gmail.com" (second %))]
           ["test@example.com" "test@gmail.com"])
[["test" "gmail.com"]]
=> (transform [ALL (parser parse unparse) #(= "gmail.com" (second %))]
              (fn [[name domain]] [(str name "+spam") domain])
              ["test@example.com" "test@gmail.com"])
["test@example.com" "test+spam@gmail.com"]
```

## pred

`(pred apred)`

Keeps the element only if it matches the supplied predicate. This is the
late-bound parameterized version of using a function directly in a path.

See also [must](#must).

```clojure
=> (select [ALL (pred even?)] (range 10))
[0 2 4 6 8]
```

## pred=

`(pred= value)`

Keeps elements only if they equal the provided value.

See also [pred](#pred).

```clojure
=> (select [ALL (pred= 2)] [1 2 2 3 4 0])
[2 2]
```

## pred<

`(pred< value)`

Keeps elements only if they are less than the provided value.

```clojure
=> (select [ALL (pred< 3)] [1 2 2 3 4 0])
[1 2 2 0]
```

See also [pred](#pred).

## pred>

`(pred> value)`

Keeps elements only if they are greater than the provided value.

```clojure
=> (select [ALL (pred> 3)] [1 2 2 3 4 0])
[4]
```

See also [pred](#pred).

## pred<=

`(pred<= value)`

Keeps elements only if they are less than or equal to the provided value.

```clojure
=> (select [ALL (pred<= 3)] [1 2 2 3 4 0])
[1 2 2 3 0]
```

See also [pred](#pred).

## pred>=

`(pred>= value)`

Keeps elements only if they are greater than or equal to the provided value.

```clojure
=> (select [ALL (pred>= 3)] [1 2 2 3 4 0])
[3 4]
```

See also [pred](#pred).

## putval

`(putval val)`

Adds an external value to the collected vals. Useful when additional arguments
are required to the transform function that would otherwise require partial
application or a wrapper function.

See also [VAL](#val), [collect](#collect), and [collect-one](#collect-one)

```clojure
;; incrementing val at path [:a :b] by 3
=> (transform [:a :b (putval 3)] + {:a {:b 0}})
{:a {:b 3}}
```

## not-selected?

`(not-selected? & path)`

Stops navigation if the path navigator finds a result. Otherwise continues with the current structure.

The input path may be parameterized, in which case the result of selected?
will be parameterized in the order of which the parameterized navigators
were declared.

See also [selected?](#selected?).

```clojure
=> (select [ALL (not-selected? even?)] (range 10))
[1 3 5 7 9]
=> (select [ALL (not-selected? [(must :a) even?])] [{:a 0} {:a 1} {:a 2} {:a 3}])
[{:a 1} {:a 3}]
;; Path returns [0 2], so navigation stops
=> (select-one (not-selected? [ALL (must :a) even?]) [{:a 0} {:a 1} {:a 2} {:a 3}])
nil
```

## selected?

`(selected? & path)`

Stops navigation if the path navigator fails to find a result. Otherwise continues with the current structure.

The input path may be parameterized, in which case the result of selected?
will be parameterized in the order of which the parameterized navigators
were declared.

See also [not-selected?](#not-selected?).

```clojure
=> (select [ALL (selected? even?)] (range 10))
[0 2 4 6 8]
=> (select [ALL (selected? [(must :a) even?])] [{:a 0} {:a 1} {:a 2} {:a 3}])
[{:a 0} {:a 2}]
;; Path returns [0 2], so selected? returns the entire structure
=> (select-one (selected? [ALL (must :a) even?]) [{:a 0} {:a 1} {:a 2} {:a 3}])
[{:a 0} {:a 1} {:a 2} {:a 3}]
nil
```

## set-elem

`(set-elem element)`

Navigates to the given element in the set only if it exists in the set.
Can transform to NONE to remove the element from the set.

```clojure
=> (select [(set-elem 3)] #{3 4 5})
[3]
=> (select [(set-elem 3)] #{4 5})
[]
=> (setval [(set-elem 3)] NONE #{3 4 5})
#{4 5}
```

## srange

`(srange start end)`

Navigates to the subsequence bound by the indexes start (inclusive)
and end (exclusive).

See also [srange-dynamic](#srange-dynamic).

```clojure
=> (select-one (srange 2 4) (range 5))
[2 3]
=> (select-one (srange 0 10) (range 5))
IndexOutOfBoundsException
=> (setval (srange 2 4) [] (range 5))
(0 1 4)
```

As of Specter 1.0.0, `srange` can now work with strings. It navigates to or transforms substrings.

```clojure
=> (select-any (srange 1 3) "abcd")
"bc"
=> (setval (srange 1 3) "" "abcd")
"ad"
=> (setval [(srange 1 3) END] "x" "abcd")
"abcxd"
```

## srange-dynamic

`(srange-dynamic start-fn end-fn)`

Uses start-fn and end-fn to determine the bounds of the subsequence
to select when navigating. Each function takes in the structure as input.

See also [srange](#srange).

```clojure
=> (select-one (srange-dynamic #(.indexOf % 2) #(.indexOf % 4)) (range 5))
[2 3]
=> (select-one (srange-dynamic (fn [_] 0) #(quot (count %) 2)) (range 10))
[0 1 2 3 4]
```

## stay-then-continue

`(stay-then-continue)`

Navigates to the current element and then navigates via the provided path.
This can be used to implement pre-order traversal.

See also [continue-then-stay](#continue-then-stay).

```clojure
=> (select (stay-then-continue MAP-VALS) {:a 0 :b 1 :c 2})
({:a 0, :b 1, :c 2} 0 1 2)
```

## submap

`(submap m-keys)`

Navigates to the specified submap (using select-keys).
In a transform, that submap in the original map is changed to the new
value of the submap.

```clojure
=> (select-one (submap [:a :b]) {:a 0, :b 1, :c 2})
{:a 0, :b 1}
=> (select-one (submap [:c]) {:a 0})
{}
;; (submap [:a :c]) returns {:a 0} with no :c
=> (transform [(submap [:a :c]) MAP-VALS]
              inc
              {:a 0, :b 1})
{:b 1, :a 1}
;; We replace the empty submap with {:c 2} and merge with the original
;; structure
=> (transform (submap []) #(assoc % :c 2) {:a 0, :b 1})
{:a 0, :b 1, :c 2}
```

## subselect

`(subselect & path)`

Navigates to a sequence that contains the results of (select ...),
but is a view to the original structure that can be transformed.
Without subselect, we could only transform selected values individually.
`subselect` lets us transform them together as a seq, much like `filterer`.

Requires that the input navigators will walk the structure's
children in the same order when executed on "select" and then
"transform".

See also [filterer](#filterer).

```clojure
=> (transform (subselect (walker number?) even?)
     reverse
     [1 [[[2]] 3] 5 [6 [7 8]] 10])
[1 [[[10]] 3] 5 [8 [7 6]] 2]
```

## subset

`(subset aset)`

Navigates to the specified subset (by taking an intersection).
In a transform, that subset in the original set is changed to the
new value of the subset.

```clojure
=> (select-one (subset #{:a :b}) #{:b :c})
#{:b}
;; Replaces the #{:a} subset with #{:a :c} and unions back into
;; the original structure
=> (setval (subset #{:a}) #{:a :c} #{:a :b})
#{:c :b :a}
```

## terminal

`(terminal update-fn)`

_Added in 0.12.0_

For usage with `multi-transform`, defines an endpoint in the navigation that
will have the parameterized transform function run. The transform function works
just like it does in `transform`, with collected values given as the first
arguments.

See also [terminal-val](#terminal-val) and [multi-transform](List-of-Macros#multi-transform).

```clojure
=> (multi-transform [(putval 3) (terminal +)] 1)
4
=> (multi-transform [:a :b (multi-path [:c (terminal inc)]
                                       [:d (putval 3) (terminal +)])]
                    {:a {:b {:c 42 :d 1}}})
{:a {:b {:c 43, :d 4}}}
```

## terminal-val

`(terminal-val val)`

_Added in 0.12.0_

Like `terminal` but specifies a val to set at the location regardless of
the collected values or the value at the location.

```clojure
=> (multi-transform (terminal-val 2) 3)
2
```

See also [terminal](#terminal) and [multi-transform](List-of-Macros#multi-transform).

## transformed

`(transformed path update-fn)`

Navigates to a view of the current value by transforming it with the
specified path and update-fn.

The input path may be parameterized, in which case the result of transformed
will be parameterized in the order of which the parameterized navigators
were declared.

See also [view](#view)

```clojure
=> (select-one (transformed [ALL odd?] #(* % 2)) (range 10))
(0 2 2 6 4 10 6 14 8 18)
=> (transform [(transformed [ALL odd?] #(* % 2)) ALL] #(/ % 2) (range 10))
(0 1 1 3 2 5 3 7 4 9)
```

## traversed

`(traversed path reduce-fn)`

Navigates to a view of the current value by transforming with a reduction over
the specified traversal.

```clojure
=> (select-any (traversed ALL +) [1 2 3 4])
10
```

## view

`(view afn)`

Navigates to result of running `afn` on the currently navigated value.

See also [transformed](#transformed).

```clojure
=> (select-one [FIRST (view inc)] (range 5))
1
```

## walker

`(walker afn)`

Using clojure.walk, `walker` executes a depth-first search for nodes where `afn`
returns a truthy value. When `afn` returns a truthy value, `walker` stops
searching that branch of the tree and continues its search of the rest of the
data structure.

See also [codewalker](#codewalker)

```clojure
=> (select (walker #(and (number? %) (even? %))) '(1 (3 4) 2 (6)))
(4 2 6)
;; Note that (3 4) and (6 7) are not returned because the search halted at
;; (2 (3 4) (5 (6 7))).
=> (select (walker #(and (counted? %) (even? (count %))))
     '(1 (2 (3 4) 5 (6 7)) (8 9)))
((2 (3 4) 5 (6 7)) (8 9))
=> (setval (walker #(and (counted? %) (even? (count %))))
           :double
           '(1 (2 (3 4) 5 (6 7)) (8 9)))
(1 :double :double)
```
