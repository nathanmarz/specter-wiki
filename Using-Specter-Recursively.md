Specter is useful for navigating through nested data structures, and then returning (selecting) or transforming what it finds. Specter can also work its magic **recursively**. Many of Specter's most important and powerful use cases in your codebase will require you to use Specter's recursive features. However, just as recursion can be difficult to grok at first, using Specter recursively can be challenging. This guide is designed to help you feel at home using Specter recursively.

# A Review of Recursion

Most simple functions do not call themselves. For example:

```clojure
(defn square [x] (* x x))
```

Instead of calling `square`, itself, `square` calls another function, `*` or multiply. This, of course, serves the purpose of squaring the input. You don't always need recursion. But sometimes you need a function to call itself - to recur. To take another example from mathematics, you need recursion to implement factorials. Factorials are equal to the product of every positive number equal to or less than the number. For example, the factorial of 5 ("5!") is `5 * 4 * 3 * 2 * 1 = 120`.

Here is a simple implementation of factorials in Clojure:

```clojure
=> (defn factorial [x]
  (if (< x 2)
	  1
	  (* x (factorial (dec x)))))
#'playground.specter/factorial
=> (factorial 5)
120
```

Here, the function definition *calls itself* with the symbol `factorial`. You still supply input - the value x - but producing the function's return value requires calling the function itself.

So there are two elements to basic recursion: a name or symbol for the recursion point or function, and input arguments.

# Using Specter Recursively

Using Specter recursively uses these two elements: a name (`self-sym` or self-symbol), and arguments (`params` or parameters). But it adds a third element: the `path` that the

This is actually similar to the function body.

requires **three elements**: a name (self-sym), arguments

# Applications
