Specter comes with support for Clojure's Zipper data structure, but this support is located in a different namespace, com.rpl.specter.zipper. We'll need to distinguish between Clojure's core zipper namespace, Specter's main namespace (which provides select, transform, etc.) and Specter's Zipper namespace. Accordingly, in this tutorial, we'll use the following namespace declaration:

```clojure
(ns playground.zippers
  (:require [com.rpl.specter :as S]
			[com.rpl.specter.zipper :as SZ]
			[clojure.zip :as zip]))
```

When working with data using zippers and Specter, you always do the following steps:
  * Navigate to a zipper using `SZ/VECTOR-ZIP`, `SZ/SEQ-ZIP`, `SZ/XML-ZIP` or another zipper navigator created using `SZ/zipper`.
  * Navigate with zippers to whatever you want to change.
  * Navigate using `SZ/NODE` or `SZ/NODE-SEQ` to the actual value for updates.

**Note:** Many of the descriptions and a couple of the examples are lightly edited from those found on the [Codox documentation](https://nathanmarz.github.io/specter/com.rpl.specter.zipper.html).

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
**Table of Contents**

- [Unparameterized Zipper Navigators](#unparameterized-zipper-navigators)
	- [DOWN](#down)
	- [INNER-LEFT](#inner-left)
	- [INNER-RIGHT](#inner-right)
	- [LEFT](#left)
	- [LEFTMOST](#leftmost)
	- [NEXT](#next)
	- [NEXT-WALK](#next-walk)
	- [NODE](#node)
	- [NODE-SEQ](#node-seq)
	- [PREV](#prev)
	- [RIGHT](#right)
	- [RIGHTMOST](#rightmost)
	- [SEQ-ZIP](#seq-zip)
	- [UP](#up)
	- [VECTOR-ZIP](#vector-zip)
	- [XML-ZIP](#xml-zip)
- [Parameterized Zipper Navigators (and Functions)](#parameterized-zipper-navigators-and-functions)
	- [find-first](#find-first)
	- [zipper](#zipper)

<!-- markdown-toc end -->

# Unparameterized Zipper Navigators

## DOWN

Equivalent to `clojure.zip/down`.

```clojure
=> (def data [1 [[2 3 4] 5 6] 7 [8 9]])
#'playground.zippers/data
=> (S/select-any [SZ/VECTOR-ZIP SZ/DOWN SZ/NODE] data)
1
```

## INNER-LEFT

Navigate to the empty subsequence directly to the left of this element.

```clojure
=> (def data [1 [[2 3 4] 5 6] 7 [8 9]])
#'playground.zippers/data
=> (S/select-any [SZ/VECTOR-ZIP SZ/DOWN SZ/NODE] data)
1
=> (S/select-any [SZ/VECTOR-ZIP SZ/INNER-LEFT] data)
[]
=> (S/setval [SZ/VECTOR-ZIP SZ/DOWN SZ/INNER-LEFT] [:a] data)
[:a 1 [[2 3 4] 5 6] 7 [8 9]]
```

## INNER-RIGHT

Navigate to the empty subsequence directly to the right of this element.

```clojure
=> (def data [1 [[2 3 4] 5 6] 7 [8 9]])
#'playground.zippers/data
=> (S/select-any [SZ/VECTOR-ZIP SZ/NODE] data)
[1 [[2 3 4] 5 6] 7 [8 9]]
=> (S/select-any [SZ/VECTOR-ZIP SZ/INNER-RIGHT] data)
[]
=> (S/setval [SZ/VECTOR-ZIP SZ/DOWN SZ/INNER-RIGHT] [:a] data)
[1 :a [[2 3 4] 5 6] 7 [8 9]]
```

## LEFT

Navigate to the element to the left. If no element there, works like `STOP`.

```clojure
=> (def data [1 [[2 3 4] 5 6] 7 [8 9]])
#'playground.zippers/data
=> (S/select-any [SZ/VECTOR-ZIP SZ/DOWN SZ/RIGHTMOST SZ/NODE] data)
[8 9]
=> (S/select-any [SZ/VECTOR-ZIP SZ/DOWN SZ/RIGHTMOST SZ/LEFT SZ/NODE] data)
7
```

## LEFTMOST

```clojure
=> (def data [1 [[2 3 4] 5 6] 7 [8 9]])
#'playground.zippers/data
=> (S/select-any [SZ/VECTOR-ZIP SZ/DOWN SZ/LEFTMOST SZ/NODE] data)
1
=> (S/transform [SZ/VECTOR-ZIP SZ/DOWN SZ/LEFTMOST SZ/NODE] inc data)
[2 [[2 3 4] 5 6] 7 [8 9]]
```

## NEXT

Navigate to the next element in the structure. If no next element, works like `STOP`.

```clojure
=> (def data [1 [[2 3 4] 5 6] 7 [8 9]])
#'playground.zippers/data
=> (S/select-any [SZ/VECTOR-ZIP SZ/DOWN SZ/NEXT SZ/NODE] data)
[[2 3 4] 5 6]
```

## NEXT-WALK

Navigate to every element reachable using calls to `NEXT`.

```clojure
=> (S/select [SZ/VECTOR-ZIP SZ/NEXT-WALK SZ/NODE] [1 [2 3]])
[[1 [2 3]] 1 [2 3] 2 3]
```

## NODE

Equivalent to `clojure.zip/node`.

```clojure
=> (def data [1 [[2 3 4] 5 6] 7 [8 9]])
#'playground.zippers/data
=> (S/select-any [SZ/VECTOR-ZIP SZ/DOWN SZ/NODE] data)
1
=> (S/transform [SZ/VECTOR-ZIP SZ/DOWN SZ/NODE] inc data)
[2 [[2 3 4] 5 6] 7 [8 9]]
```

## NODE-SEQ

Navigate to the subsequence containing only the node currently pointed to. This works just like `srange`, and can be used to remove elements from the structure.

The following example highlights the difference between the final navigator being `SZ/NODE`, or being `SZ/NODE-SEQ`, with both queries and transformations.

```clojure
=> (S/select-any [SZ/VECTOR-ZIP SZ/DOWN SZ/RIGHT SZ/RIGHT SZ/NODE] [1 2 3 4 5])
3
=> (S/select-any [SZ/VECTOR-ZIP SZ/DOWN SZ/RIGHT SZ/RIGHT SZ/NODE-SEQ] [1 2 3 4 5])
[3]
=> (S/setval [SZ/VECTOR-ZIP SZ/DOWN SZ/RIGHT SZ/RIGHT SZ/NODE] [:a :b :c] [1 2 3 4 5])
[1 2 [:a :b :c] 4 5]
=> (S/setval [SZ/VECTOR-ZIP SZ/DOWN SZ/RIGHT SZ/RIGHT SZ/NODE-SEQ] [:a :b :c] [1 2 3 4 5])
[1 2 :a :b :c 4 5]
```

## PREV

Navigate to the previous element. If this is the first element, works like `STOP`.

In this example, going down and then back, and then to the node, is identical to going directly to then node of a zipper:

```clojure
=> (def data [1 [[2 3 4] 5 6] 7 [8 9]])
#'playground.zippers/data
=> (S/select-any [SZ/VECTOR-ZIP SZ/NODE] data)
[1 [[2 3 4] 5 6] 7 [8 9]]
=> (S/select-any [SZ/VECTOR-ZIP SZ/DOWN SZ/NODE] data)
1
=> (S/select-any [SZ/VECTOR-ZIP SZ/DOWN SZ/PREV SZ/NODE] data)
[1 [[2 3 4] 5 6] 7 [8 9]]
```

## RIGHT

Navigate to the element to the right. If no element there, works like `STOP`.

```clojure
=> (def data [1 [[2 3 4] 5 6] 7 [8 9]])
#'playground.zippers/data
=> (S/select-any [SZ/VECTOR-ZIP SZ/NODE] data)
[1 [[2 3 4] 5 6] 7 [8 9]]
=> (S/select-any [SZ/VECTOR-ZIP SZ/DOWN SZ/RIGHT SZ/NODE] data)
[[2 3 4] 5 6]
```

## RIGHTMOST

```clojure
=> (def data [1 [[2 3 4] 5 6] 7 [8 9]])
#'playground.zippers/data
=> (S/select-any [SZ/VECTOR-ZIP SZ/DOWN SZ/RIGHTMOST SZ/NODE] data)
[8 9]
```

## SEQ-ZIP

`SEQ-ZIP` treats a structure as a seq-zipper, by calling the constructor clojure.zip/seq-zip on the structure. Accordingly, you do NOT need to use `SEQ-ZIP` in your path if the data structure you are working with is already a seq-zip.

```clojure
=> (def seq-data '(1 2 (3 4) 5 6))
#'playground.zippers/seq-data
=> (S/select-any [SZ/SEQ-ZIP SZ/NODE] seq-data)
(1 2 (3 4) 5 6)
=> (= (zip/seq-zip seq-data) (S/select-any [SZ/SEQ-ZIP] seq-data))
true
```

## UP

Equivalent to `clojure.zip/up`.

```clojure
=> (def data [1 [[2 3 4] 5 6] 7 [8 9]])
#'playground.zippers/data
=> (S/select-any [SZ/VECTOR-ZIP SZ/NODE] data)
[1 [[2 3 4] 5 6] 7 [8 9]]
=> (S/select-any [SZ/VECTOR-ZIP SZ/DOWN SZ/NODE] data)
1
=> (S/select-any [SZ/VECTOR-ZIP SZ/DOWN SZ/UP SZ/NODE] data)
[1 [[2 3 4] 5 6] 7 [8 9]]
```

## VECTOR-ZIP

`VECTOR-ZIP` treats a structure as a vector-zipper, by calling the constructor `clojure.zip/vector-zip` on the structure. Accordingly, you do NOT need to use `VECTOR-ZIP` in your path if the data structure you are working with is already a vector-zip.

```clojure
=> (def data [1 [[2 3 4] 5 6] 7 [8 9]])
#'playground.zippers/data
=> (S/select-any [SZ/VECTOR-ZIP SZ/NODE] data)
[1 [[2 3 4] 5 6] 7 [8 9]]
=> (= (S/select-any [SZ/VECTOR-ZIP] data) (zip/vector-zip data))
true
```

## XML-ZIP

`XML-ZIP` treats a structure as a xml-zipper, by calling the constructor `clojure.zip/xml-zip` on the structure. Accordingly, you do NOT need to use `XML-ZIP` in your path if the data structure you are working with is already a xml-zip.

```clojure
;; The following example make use of an xml-tree
;; borrowed from
;; http://clojuredocs.org/clojure.zip/xml-zip
;; the original xml is
;; <root><any>foo bar</any>bar</root>
;; parsed by clojure.xml/parse

;; Notice that the xml-parse will not produce the exact
;; xml object as the "foo" and "bar" strings are combined.

;; Travel over the zipper in classic lisp style
=> (def parsed-xml {:tag :root :content [{:tag :any :content ["foo" "bar"]} "bar"]})
#'playground.zippers/parsed-xml
=> (S/select [SZ/XML-ZIP SZ/NODE] parsed-xml)
[{:tag :root, :content [{:tag :any, :content ["foo" "bar"]} "bar"]}]
=> (= (zip/xml-zip parsed-xml) (S/select-any [SZ/XML-ZIP] parsed-xml))
true
```

# Parameterized Zipper Navigators (and Functions)

## find-first

`(find-first predfn)`

Navigate the zipper to the first element in the structure matching `predfn`. A linear scan is done using `NEXT` to find the element.

```clojure
=> (def data [1 [[2 3 4] 5 6] 7 [8 9]])
#'playground.zippers/data
=> (S/select [SZ/VECTOR-ZIP SZ/NODE] data)
[[1 [[2 3 4] 5 6] 7 [8 9]]]
=> (S/select [SZ/VECTOR-ZIP (SZ/find-first #(and (number? %) (even? %))) SZ/NODE] data)
[2]
```

## zipper

`(zipper constructor)`

`Zipper` takes a constructor for a zipper (created with `clojure.clojure.zip/zipper`), and returns an unparameterized navigator for that zipper. This is useful if you need to use Specter with your own custom zippers. Here are the implementation of `VECTOR-ZIP`, `SEQ-ZIP`, and `XML-ZIP`, which provide unparameterized navigators for the zippers provided by clojure.zip:

```clojure
(def VECTOR-ZIP (zipper zip/vector-zip))
(def SEQ-ZIP (zipper zip/seq-zip))
(def XML-ZIP (zipper zip/xml-zip))
```
