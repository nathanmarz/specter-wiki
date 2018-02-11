# Cheat Sheet

Most of Specter's API consists of an operation, a path and the data to operate on.

## Operations

There are 2 types of operations: queries and transforms.

#### Query

###### Multiple elements (return a vector of values found)

`select`, `select-any`

###### One element (return a single value)

`select-first`, `select-one`, `select-one!`

###### Query test

`selected-any?`

###### Transducer-related

`traverse`, `traverse-all`

#### Transform

`transform`, `multi-transform`, `replace-in`, `setval`

## Paths

A path (often named `apath`) is a navigator or a vector of navigators.

Navigator sometimes operates on specific data structures.


#### Maps

`MAP-KEYS`, `MAP-VALS`, `keypath`, `map-key`, `submap`

#### Sequences

###### All values

`ALL`, `ALL-WITH-META`

###### Specific position

`AFTER-ELEM`, `BEFORE-ELEM`, `BEGINNING`, `END`, `FIRST`, `LAST`

###### Indexed

`INDEXED-VALS`, `before-index`, `index-nav`, `nthpath`, `srange`, `srange-dynamic`

###### Others

`continuous-subseqs`

#### Sets

`NONE-ELEM`, `set-elem`

#### Keywords/Symbols

`NAME`, `NAMESPACE`

#### Atoms

#### Strings

###### Specific position

`BEGINNING`, `FIRST`, `END`, `LAST`

##### Other

`regex-nav`, `srange`

#### Metadata

`ALL-WITH-META`, `META`

#### Views

`NIL->LIST`, `NIL->SET`, `NIL->VECTOR`, `filterer`, `nil->val`, `subselect`, `transform`, `traversed`, `view`

#### Value collection

`VAL`, `collect`, `collect-one`, `collected?`, `putval`, `with-fresh-collected`

#### Don't know yet

`NONE`, `codewalker`, `each-nav`, `multi-path`, `must`, `parser`, `walker`

#### Control

`STAY`, `STOP`, `comp-path`, `cond-path`, `continue-then-stay`, `if-path`, `stay-the-continue`

#### Filters

`pred`, `pred=`, `pred<`, `pred>`, `pred<=`, `pred>=`, `not-selected?`,  `selected?`

#### Multi-transform

`terminal`, `terminal-val`

## Extending Specter

#### Custom paths

`declarepath`, `defprotocolpath`, `extend-protocolpath`, `path`, `providepath`, `recursive-path`

#### Custom collectors

`defcollector`

#### Custom navigators

`defdynamicnav`, `defnav`, `nav`
