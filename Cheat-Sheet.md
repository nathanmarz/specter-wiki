# Cheat Sheet

Most of Specter's API consists of an operation, a path and the data to operate on.

## Operations

There are 2 types of operations: queries and transforms.

#### Query

`select`, `select-any`, `select-first`, `select-one`, `select-one!`, `selected-any?`, `traverse-all`

#### Transform

`transform`, `multi-transform`, `replace-in`, `setval`

## Paths

A path (often named `apath`) is a navigator or a vector of navigators.

Navigator sometimes operates on specific data structures.

#### Maps

`MAP-KEYS`, `MAP-VALS`, `keypath`, `map-key`, `submap`, `must`

#### Sequences

`ALL`, `ALL-WITH-META`, `AFTER-ELEM`, `BEFORE-ELEM`, `BEGINNING`, `END`, `FIRST`, `INDEXED-VALS`, `LAST`

`before-index`, `continuous-subseqs`, `filterer`, `index-nav`, `nthpath`, `srange`, `srange-dynamic`

#### Sets

`NONE-ELEM`, `set-elem`, `subsets`

#### Keywords/Symbols

`NAME`, `NAMESPACE`

#### Atoms

`ATOM`

#### Strings

`BEGINNING`, `FIRST`, `END`, `LAST`, `regex-nav`, `srange`

#### Metadata

`ALL-WITH-META`, `META` 

#### Views

`NIL->LIST`, `NIL->SET`, `NIL->VECTOR`, `nil->val`, `parser`, `subselect`, `transformed`, `traversed`, `view`

#### Value collection

`DISPENSE`, `VAL`, `collect`, `collect-one`, `collected?`, `putval`, `with-fresh-collected`

#### Don't know yet

`traverse`

#### Control

`STAY`, `STOP`, `comp-path`, `cond-path`, `continue-then-stay`, `if-path`, `multi-path`, `stay-the-continue`

#### Filters

`pred`, `pred=`, `pred<`, `pred>`, `pred<=`, `pred>=`, `not-selected?`,  `selected?`

#### Walking

`codewalker`, `walker`

#### Multi-transform

`terminal`, `terminal-val`

## Extending Specter

#### Custom paths

`declarepath`, `defprotocolpath`, `extend-protocolpath`, `path`, `providepath`, `recursive-path`

#### Custom collectors

`defcollector`

#### Custom navigators

`defdynamicnav`, `defnav`, `each-nav`, `nav`
