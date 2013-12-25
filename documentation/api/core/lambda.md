---
layout: document
title: Core › Lambda
---

Use numbered headers: True

 -  **Restrictions:** ECMAScript 5
 -  **API Stability:** Experimental
 -  **Module:** [core.lambda](https://npmjs.org/package/core.lambda)
 -  **Version:** 0.2.0
 -  **Licence:** MIT
{.summary}

The `core.lambda` module provides essential combinators and higher-order
functions derived from Lambda Calculus. All functions are curried to make them
easier to compose and abstract over.


# Table of Contents

 *  Table of Contents
{:toc}


## `identity(a)`

The identity function.

    a → a
{:lang=oblige}

When called with any argument, returns the value unchanged.


{% highlight js %}
identity(5)         // => 5
identity([1, 2])    // => [1, 2]
identity({})        // => {}
{% endhighlight %}


## `constant(a)(b)`

The constant function.

    a → b → a
{:lang=oblige}

When called with two arguments, always returns the first one, unchanged. The
constant function is fairly useful for providing constant behaviour, for
example, when mapping over a [Functor][].

{% highlight js %}
constant(5)(6)                                  // => 5
[1, 2, 3].map(constant('!'))                    // => ['!', '!', '!']
({ toString: constant('<#Nil>') }).toString()   // => <#Nil>
{% endhighlight %}


## `flip(f)(a)(b)`

Flips the arguments of a binary function.

    (a → b → c) → b → a → c
{:lang=oblige}

When applied to a binary function, returns a new function that takes the
arguments in an inverted order, such that `flip(f)(1)(2)` is equivalent to
`f(2)(1)`. Useful for partially applying on the right side of a function.

{% highlight js %}
function subtract(a) {
  return function(b) {
    return a - b
  }
}

var subtract3 = flip(subtract)(3)

subtract3(7)        // => 4
{% endhighlight %}


## `compose(f)(g)(a)`

The function composition combinator.

    (b → c) → (a → b) → a → c
{:lang=oblige}

Composes two unary functions together, such that the result of applying the
argument to the first function is passed as an argument to the second
function. `compose(f)(g)(1)` is equivalent to `f(g(1))`, but the composition
operator is useful for easily creating new functionality by gluing functions
together.

{% highlight js %}
function withoutSpaces(a) {
  return a.replace(/\s/, '')
}
function upcase(a) {
  return a.toUpperCase()
}
var asSymbol = compose(upcase, withoutSpaces)

asSymbol('immutable list')      // 'IMMUTABLELIST'
{% endhighlight %}


## `curry(f)(a)(b)`

Transforms a binary function on tuples, into a [curried][] binary function.

    ((a, b) → c) → a → b → c
{:lang=oblige}

Given a binary function that takes all of its arguments at once, returns a
binary function that takes each single argument as a separate
application. Currying is useful for constructing functions that can be easily
partially specialised (by way of partial application).

{% highlight js %}
var filter = curry(function(f, xs) {
  return xs.filter(f)
})

var onlyEvens = filter(function(a){ return a % 2 === 0 })
var onlyOdds  = filter(function(a){ return a % 2 !== 0 })

onlyEvens([1, 2, 3, 4, 5])      // => [2, 4]
onlyOdds([1, 2, 3, 4, 5])       // => [1, 3, 5]
{% endhighlight %}


## `curry3(f)(a)(b)(c)`

Transforms a function with arity of 3 into a [curried][] function with arity
of 3.

    ((a, b, c) → d) → a → b → c → d
{:lang=oblige}

Similar to `curry`, but works with functions with arity of 3, instead of arity
of 2.


## `curryN(n)(f)(a1)(a2)...(aN)`

Transforms a function with arity N into a [curried][] function with arity N.

    Number → ((a1, a2, ..., aN) → b) → a1 → a2 → ... → aN → b
{:lang=oblige}

Due mostly to limitations on JavaScript's reflection mechanisms, `curryN` needs
to be explicitly given the arity of the function that's being transformed. Also
do note that currying variadic functions usually doesn't make much sense.

Similar to `curry` and `curry3`, but works with functions of any arity, by
requiring you to state the arity upfront.


## `partial(f)(...as)`

Specialises some of the arguments of a function, from left to right.

    ((a1, a2, ..., aN, b1, b2, ..., bN) → c)
    → a1, a2, ..., aN
    → b1, b2, ..., bN
    → c
{:lang=oblige}

The `partial` function can be used for specialising part of a non-curried
function, without evaluating the function's code right away. Instead, the
operation gives you a new function that'll take only the remaining arguments.

{% highlight js %}
function slice(start, end, thing) {
  return thing.slice(start, end)
}
var first = partial(slice, 0, 1)
var rest  = partial(slice, 1, undefined)

first('qux')    // => 'q'
rest('qux')     // => 'ux'
{% endhighlight %}


## `partialRight(f)(...as)`

Specialises some of the arguments of a function, from right to left.

    ((b1, b2, ..., bN, a1, a2, ..., aN) → c)
    → b1, b2, ..., bN
    → a1, a2, ..., aN
    → c
{:lang=oblige}

Similar to `partial`, but specialises the arguments from right to left.


## `uncurry(f)(as)`

Transforms a variadic function into one that takes all arguments as an `Array`.

    (a1, a2, ..., aN → b) → #[a1, a2, ..., aN] → b
{:lang=oblige}

Usually useful when composing functions that need to take more than one
argument. This allows one function to return a list of the arguments, and the
subsequent function to be applied to the list, rather than a single argument.

{% highlight js %}
function vector(x, y) {
  return { x: x, y: y }
}
function opposites(x) {
  return [x, -x]
}

compose(uncurry(vector), opposites)(2)  // { x: 2, y: -2 }
{% endhighlight %}


## `uncurryBind(f)(as)`

Transforms a variadic function into one that takes all arguments (including
`this`) as an `Array`.

    (c:Object) => (a1, a2, ..., aN → b) → #[c, a1, a2, ..., aN] → b
{:lang=oblige}

Like `uncurry`, but more useful for converting methods into plain functions.

{% highlight js %}
var slice = uncurryBind(Array.prototype.slice)

slice(['bar', 1])       // => 'ar'
slice(['bar', 1, 2])    // => 'a'
{% endhighlight %}


## `upon(f)(g)(a)(b)`

Applies an unary function to both arguments of a binary function.

    (b → b → c) → (a → b) → a → a → c
{:lang=oblige}

Generally useful for transforming things in higher-order operations that take a
binary function (e.g.: sorting, folding), before applying the function.

{% highlight js %}
function first(a) {
  return a[0]
}
function compare(a, b) {
  return a < b?  -1
  :      a > b?   1
  :      /* _ */  0
}

[[3, 'foo'], [1, 'bar'], [2, 'baz']].sort(upon(compare, first))
// => [[1, 'bar'], [2, 'baz'], [3, 'foo']]
{% endhighlight %}


<!-- Links -->
[Functor]: https://github.com/fantasyland/fantasy-land#functor
[curried]: http://en.wikipedia.org/wiki/Currying
