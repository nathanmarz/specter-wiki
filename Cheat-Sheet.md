# Cheat Sheet

Most of Specter's API consists of an operation, a path and the data to operate on.

Generally usage is like this:

```
(Operation Path Data)
where
Operation: Query|Transform
Path: Navigator|Vector of Navigators
Data: User-provided data structure to operate on.
```

Refer to the specific API documentation to check usage.

## Operations

There are 2 types of operations: queries and transforms. Most of them have a compiled version that uses precompiled paths.

#### Query

[`select`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#select), [`select-any`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#select-any), [`select-first`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#select-first), [`select-one`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#select-one), [`select-one!`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#select-one-1), [`selected-any?`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#selected-any), [`traverse`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#traverse), [`traverse-all`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#traverse-all)

#### Transform

[`transform`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#transform), [`multi-transform`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#multi-transform), [`replace-in`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#replace-in), [`setval`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#setval)

#### Compiled

[`compiled-select`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#compiled-), [`compiled-select-any`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#compiled-), [`compiled-select-first`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#compiled-), [`compiled-select-one`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#compiled-), [`compiled-select-one!`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#compiled-), [`compiled-selected-any?`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#compiled-), [`compiled-setval`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#compiled-), [`compiled-transform`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#compiled-), [`compiled-traverse`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#compiled-), [`compiled-traverse-all`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#compiled-)

## Paths

A path (often named `apath`) is a navigator or a vector of navigators.

Navigator sometimes operates on specific data structures.

#### Maps

[`MAP-KEYS`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#map-keys), [`MAP-VALS`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#map-vals), [`keypath`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#keypath), [`map-key`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#map-key), [`submap`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#submap), [`must`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#must)

#### Sequences

[`ALL`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#all), [`ALL-WITH-META`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#all-with-meta), [`AFTER-ELEM`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#after-elem), [`BEFORE-ELEM`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#before-elem), [`BEGINNING`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#beginning), [`END`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#end), [`FIRST`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#first), [`INDEXED-VALS`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#indexed-vals), [`LAST`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#last)

[`before-index`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#before-index), [`continuous-subseqs`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#continuous-subseqs), [`filterer`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#filterer), [`index-nav`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#index-nav), [`nthpath`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#nthpath), [`srange`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#srange), [`srange-dynamic`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#srange-dynamic)

#### Sets

[`NONE-ELEM`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#none-elem), [`set-elem`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#set-elem), [`subset`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#subset)

#### Keywords/Symbols

[`NAME`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#name), [`NAMESPACE`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#namespace)

#### Atoms

[`ATOM`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#atom)

#### Strings

[`BEGINNING`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#beginning), [`END`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#end), [`FIRST`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#first), [`LAST`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#last), [`regex-nav`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#regex-nav), [`srange`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#srange)

#### Metadata

[`ALL-WITH-META`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#all-with-meta), [`META`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#meta) 

#### Views

[`NIL->LIST`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#nil-list), [`NIL->SET`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#nil-set), [`NIL->VECTOR`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#nil-vector), [`nil->val`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#nil-val), [`parser`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#parser), [`subselect`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#subselect), [`transformed`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#transformed), [`traversed`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#traversed), [`view`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#view)

#### Value collection

[`DISPENSE`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#dispense), [`VAL`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#val), [`collect`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#collect), [`collect-one`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#collect-one), [`collected?`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#collected), [`putval`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#putval), [`with-fresh-collected`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#with-fresh-collected)

#### Control

[`STAY`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#stay), [`STOP`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#stop), [`comp-paths`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#comp-paths), [`cond-path`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#cond-path), [`continue-then-stay`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#continue-then-stay), [`if-path`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#if-path), [`multi-path`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#multi-path), [`stay-then-continue`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#stay-then-continue)

#### Filters

[`pred`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#pred), [`pred=`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#pred-1), [`pred<`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#pred-2), [`pred>`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#pred-3), [`pred<=`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#pred-4), [`pred>=`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#pred-5), [`not-selected?`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#not-selected),  [`selected?`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#selected)

#### Walking

[`codewalker`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#codewalker), [`walker`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#walker)

#### Multi-transform

[`terminal`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#terminal), [`terminal-val`](https://github.com/nathanmarz/specter/wiki/List-of-Navigators#terminal-val)

## Extending Specter

#### Custom paths

[`declarepath`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#declarepath), [`defprotocolpath`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#defprotocolpath), [`extend-protocolpath`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#extend-protocolpath), [`path`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#path), [`providepath`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#providepath), [`recursive-path`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#recursive-path)

#### Custom collectors

[`defcollector`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#defcollector)

#### Custom navigators

[`defdynamicnav`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#defdynamicnav), [`defnav`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#defnav), [`eachnav`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#eachnav), [`nav`](https://github.com/nathanmarz/specter/wiki/List-of-Macros#nav)
