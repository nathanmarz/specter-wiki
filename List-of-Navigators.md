# List of Navigators with Example(s)

## The All Caps Ones

### ALL

The `ALL` navigator navigates to every element in a collection. If the collection is a map, it will navigate to each key-value pair `[key value]`. The resulting elements will be reconstructed as a vector.

```clojure
=> (select [ALL] [0 1 2 3])
[0 1 2 3]
=> (select [ALL] (list 0 1 2 3))
[0 1 2 3]
=> (select [ALL] {:a :b, :c :d, :e :f})
[[:a :b] [:c :d] [:e :f]]
```