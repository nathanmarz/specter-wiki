Specter is useful for navigating through nested data structures, and then returning (selecting) or transforming what it finds. Specter can also work its magic **recursively**. Many of Specter's most important and powerful use cases in your codebase will require you to use Specter's recursive features. However, just as recursion can be difficult to grok at first, using Specter recursively can be challenging. This guide is designed to help you feel at home using Specter recursively.

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
**Table of Contents**

- [A Review of Recursion](#a-review-of-recursion)
- [Using Specter Recursively](#using-specter-recursively)
- [Applications](#applications)

<!-- markdown-toc end -->

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

We can see that there are up to three elements to basic recursion:
  * a name or symbol for the recursion point or function (defn function-name)
  * optional input arguments
  * the body of the recursion - what you **do** recursively - and, at some point, calling the function by its name

# Using Specter Recursively

The core way that Specter enables recursion is through the parameterized macro, `recursive-path`. `recursive-path` takes the following form:

`(recursive-path params self-sym path)`

You can see that there are three arguments, which match onto our list above. (They are, however, in a different order.) The `self-sym` is the name; the `params` are the optional input arguments; and the `path` is the body of the recursion. This path can be recursive, referencing itself by the `self-sym`.

(NOTE: QUESTION: Is there a reason why the self-sym is the second argument?)

# Applications
