---
layout: document
title:  Control › Monads
---

Use numbered headers: True

 -  **Restrictions:** ECMAScript 5
 -  **API Stability:** Experimental
 -  **Module:** [control.monads](https://npmjs.org/package/control.monads)
 -  **Version:** 0.3.0
 -  **Licence:** MIT
{.summary}

The `control.monads` module provides useful combinators for working with
monadic computations at a higher-level.


# Table of Contents

 *  TOC
{:toc}


## Curried versions of monadic operators

### `concat(a)(b)`

Concatenates two semigroups.

    (Semigroup s) => s(a) → s(a) → s(a)
{:lang=oblige}

{% highlight js %}
concat([1])([2])        // => [1, 2]
{% endhighlight %}


### `empty(a)`

Constructs a new empty monoid.

    (Monoid m) => m → m(a)
{:lang=oblige}


### `map(f)(a)`

Maps over a Functor instance.

    (Functor f) => (a → b) → f(a) → f(b)
{:lang=oblige}

{% highlight js %}
function inc(a) { return a + 1 }

map(inc)(Maybe.Just(1))           // => Just(2)
{% endhighlight %}


### `of(a)(f)`

Constructs a new applicative containing the value `a`.

    (Applicative f) => a → f → f(a)
{:lang=oblige}

Alternatively aliased as `of_` for languages where `of` is a keyword
(LiveScript, CoffeeScript, etc).

{% highlight js %}
of(1)(Maybe)            // => Just(1)
{% endhighlight %}


### `ap(a)(b)`

Applies the function of an Applicative to another Applicative.

    (Applicative f) => f(a → b) → f(a) → f(b)
{:lang=oblige}

{% highlight js %}
function inc(a){ return a + 1 }

ap(Maybe.of(inc))(Maybe.Just(1))        // => Just(2)
{% endhighlight %}


### `chain(f)(a)`

Applies a monadic computation to a monad.

    (Chain c) => (a → c(b)) → c(a) → c(b)
{:lang=oblige}

{% highlight js %}
function inc(a){ return Future.of(a + 1) }

chain(inc)(Future.of(2))        // => Resolved(3)
{% endhighlight %}


## Combinators

### `sequence(m)(ms)`

Evaluates each action in sequence, left to right, collecting the results.

    (Monad m) => m → [m(a)] → m([a])
{:lang=oblige}

{% highlight js %}
sequence(Maybe, [Maybe.of(1), Maybe.of(2)])     // => Just([1, 2])
{% endhighlight %}


### `map-m(m)(f)(ms)`

Equivalent to `compose(sequence, map(f))`.

    (Monad m) => m → (a → m(b)) → [a] → m [b]
{:lang=oblige}

{% highlight js %}
mapM(Maybe, Maybe.of, [1, 2])       // => Just([1, 2])
{% endhighlight %}


### `compose(f)(g)(a)`

Left-to-right composition of monads.

    (Monad m) => (a → m(b)) → (b → m(c)) → a → m(c)
{:lang=oblige}

{% highlight js %}
function inc(a){ return Maybe.Just(a + 1) }
function double(a){ return Maybe.Just(a * 2) }

compose(inc)(double)(3)         // => Just(8)
{% endhighlight %}


### `right-compose(g)(f)(a)`

Right-to-left composition of monads.

    (Monad m) => (b → m(c)) → (a → m(b)) → a → m(c)
{:lang=oblige}

{% highlight js %}
function inc(a){ return Maybe.Just(a + 1) }
function double(a){ return Maybe.Just(a * 2) }

compose(double)(inc)(3)         // => Just(8)
{% endhighlight %}


### `join(m)`

Removes one level of a nested monad.

    (Monad m) => m(m(a)) → m(a)
{:lang=oblige}

{% highlight js %}
join(Maybe.of(Maybe.of(2)))             // => Just(2)
join(Maybe.of(Maybe.of(Maybe.of(2))))   // => Just(Just(2))
{% endhighlight %}


### `lift-m2(f)(m1)(m2)`

Promotes a regular binary function to a function over monads.

    (Monad m) => ((a, b) → c) → m(a) → m(b) → m(c)
{:lang=oblige}

{% highlight js %}
function add(a, b) { return a + b }

liftM2(add)(Maybe.Just(1))(Maybe.Just(2))       // => Just(3)
{% endhighlight %}


### `lift-mN(f)(ms)`

Promotes a regular function of N arity to a function over a a list of monads of
length N.

    (Monad m) => ((a1, a2, ..., aN) → b) → [m(a1), m(a2), ..., m(aN)] → m(b)
{:lang=oblige}

{% highlight js %}
function add3(a, b, c){ return a + b + c }

liftMN(add3)([Maybe.of(1), Maybe.of(2), Maybe.of(3)])   // => Just(6)
{% endhighlight %}
