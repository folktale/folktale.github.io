---
layout: document
title:  Data › Either
---

Use numbered headers: True

 -  **Restrictions:** ECMAScript 5
 -  **API Stability:** Stable
 -  **Module:** [data.either](https://npmjs.org/package/data.either)
 -  **Version:** 1.0.0
 -  **Licence:** MIT
{.summary}

The `Either(a, b)` structure represents the logical disjunction between `a` and
`b`. In other words, `Either` may contain either a value of type `a` or a value
of type `b`, at any given time. This particular implementation is biased on the
right value (`b`), thus projections will take the right value over the left
one.

This class models two different cases: `Left a` and `Right b`, and can hold one
of the cases at any given time. The projections are, none the less, biased for
the `Right` case, so a common use case for this structure is to hold the results
of computations that may fail, when you want to store additional information on
the failure (instead of throwing an exception).

{% highlight js %}
// Returns the contents of the file at `path`, if it exists.
//
//  read : String -> Either(Error, String)
function read(path) {
  return exists(path)?     Either.Right(fread(path))
  :      /* otherwise */   Either.Left("Non-existing file: " + path)
}

var intro = read('intro.txt')  // => Right(...)
var outro = read('outro.txt')  // => Right(...)
var nope  = read('nope.txt')   // => Left("Non-existing file: nope.txt")

// We can concatenate things without knowing if they exist at all, and
// failures are handled for us
intro.chain(function(a) {
  outro.map(function(b) {
    return a + b
  })
}).orElse(logFailure)
// => Right(...)

intro.chain(function(a) {
  return nope.map(function(b) {
    return a + b
  })
})
// => Left("Non-existing file: nope.txt")
{% endhighlight %}

Furthermore, the values of `Either(a, b)` can be combined and manipulated by
using the expressive monadic operations. This allows safely sequencing
operations that may fail, and safely composing values that you don't know
whether they're present or not, failing early (returning a `Left a`) if any of
the operations fail.
 
While this class can certainly model input validations, the [Validation][]
structure lends itself better to that use case, since it can naturally
aggregate failures — given that monads shortcut on the first failure.
 
[Validation]: https://github.com/folktale/data.validation


# Table of Contents

 *  TOC
{:toc}


## Class: `Either(a, b)`

    class Either(a, b) <: Applicative
                        , Functor
                        , Chain
                        , Monad
                        , Eq
                        , Show
{:lang=oblige}


### Constructors

#### `Either:Left(a)`

Constructs a new `Either(a, b)` structure holding a `Left`-tagged value.

    a → Either(a, b)
{:lang=oblige}

This usually represents a failure, due to the right-bias of this structure.

{% highlight js %}
Either.Left(1)          // => Left(1)
Either.Left(null)       // => Left(null)
{% endhighlight %}


#### `Either:Right(b)`

Constructs a new `Either(a, b)` structure holding a `Right`-tagged value.

    b → Either(a, b)
{:lang=oblige}

This usually represents a successful value, due to the right-bias of this
structure.

{% highlight js %}
Either.Right(1)         // => Right(1)
Either.Right(null)      // => Right(null)
{% endhighlight %}


#### `Either:from-nullable(a)`

Constructs a new `Either(a, b)` structure from a nullable type.

    a? → Either(a, a)
{:lang=oblige}

If the value is either `null` or `undefined`, this function returns a
`Left`-tagged value, otherwise returns a `Right`-tagged value.

{% highlight js %}
Either.fromNullable(null)        // => Left(null)
Either.fromNullable(undefined)   // => Left(undefined)
Either.fromNullable(1)           // => Right(1)
{% endhighlight %}


### Predicates

#### `Either:is-left`

True if the `Either(a, b)` structure contains a `Left`-tagged value.

    Boolean
{:lang=oblige}

{% highlight js %}
Either.Left(1).isLeft           // => true
Either.Right(1).isLeft          // => false
{% endhighlight %}

#### `Either:is-right`

True if the `Either(a, b)` structure contains a `Right`-tagged value.

    Boolean
{:lang=oblige}

{% highlight js %}
Either.Right(1).isRight         // => true
Either.Left(1).isRight          // => false
{% endhighlight %}


### Applicative

#### `Either:of(b)`

Creates a new `Either(a, b)` instance holding a `Right`-tagged value.

    b → Either(a, b)
{:lang=oblige}

`b` can be any value, including `null`, `undefined` or another `Either(a, b)`
structure.

{% highlight js %}
Either.of(1)                    // => Right(1)
Either.of(Either.Left(1))       // => Right(Left(1))
Either.of(null)                 // => Right(null)
{% endhighlight %}


#### `Either:ap(b)`

Applies the function from a `Right`-tagged value to another applicative.

    (@Either(a, b → c), f:Applicative) => f(b) → f(c)
{:lang=oblige}

The `Either(a, b)` structure should contain a function value, otherwise a
`TypeError` is thrown.

{% highlight js %}
function inc(a){ return a + 1 }

Either.Right(inc).ap(Either.Right(2))   // => Right(3)
Either.Left(inc).ap(Either.Right(2))    // => Right(2)
Either.Right(inc).ap(Either.Left(2))    // => Left(2)
Either.Left(inc).ap(Either.Left(2))     // => Left(2)
{% endhighlight %}


### Functor

#### `Either:map(f)`

Transforms the `Right`-tagged value of the `Either(a, b)` structure using a
regular unary function.

    @Either(a, b) => (b → c) → Either(a, c)
{:lang=oblige}

{% highlight js %}
function inc(a){ return a + 1 }

Either.Right(1).map(inc)        // => Right(2)
Either.Left(1).map(inc)         // => Left(1)
{% endhighlight %}


### Chain

#### `Either:chain(f)`

Transforms the `Right`-tagged value of the `Maybe(a)` structure using an unary
function yielding a monad of the same type.

    @Either(a, b) => (b → Either(a, c)) → Either(a, c)
{:lang=oblige}

{% highlight js %}
function duplicate(x){ return Either.Right([x, x]) }

Either.Right(2).chain(duplicate)        // => Right([2, 2])
Either.Left(2).chain(duplicate)         // => Left(2)
{% endhighlight %}


### Show

#### `Either:to-string()`

Returns a textual representation of the `Either(a, b)` structure.

    @Either(a, b) => Unit → String
{:lang=oblige}


### Eq

#### `Either:is-equal(b)`

Tests if two `Either(a, b)` structures are equivalent.

    @Either(a, b) => Either(a, b) → Boolean
{:lang=oblige}

{% highlight js %}
Either.Right(1).isEqual(Either.Right(1))        // => true
Either.Right(1).isEqual(Either.Left(1))         // => false
Either.Left(1).isEqual(Either.Right(1))         // => false
Either.Left(1).isEqual(Either.Left(1))          // => true
{% endhighlight %}


### Extracting and Recovering

#### `Either:get()`

Extracts the `Right`-tagged value out of the `Either(a, b)` structure. THrows
an error if the `Either(a, b)` contains a `Left`-tagged value.

    @Either(a, b) => Unit → b    :: throws
{:lang=oblige}

If the structure has a `Left`-tagged, a `TypeError` is thrown. See
[get-or-else](#get-or-else) for a getter that can also handle failures.

{% highlight js %}
Either.Right(1).get()   // => 1
Either.Left(1).get()    // TypeError: Can't extract the value of a Left(a).
{% endhighlight %}


#### `Either:get-or-else()`       {#get-or-else}

Extracts the `Right`-tagged value out of the `Either(a, b)` structure. If
there's a `Left`-tagged value, returns the given default.

    @Either(a, b) => b → b
{:lang=oblige}

{% highlight js %}
Either.Right(1).getOrElse(2)    // => 1
Either.Left(1).getOrElse(2)     // => 2
{% endhighlight %}


#### `Either:or-else(f)`

Transforms a `Left`-tagged value into a new `Either(a, b)` structure.

    @Either(a, b) => (Unit → Either(a, b)) → Either(a, b)
{:lang=oblige}

The analogous of `chain`, but for `Left`-tagged value. Does nothing if the
structure already contains a `Right`-tagged value.

{% highlight js %}
function k(a) { return function(b) { return a }}

Either.Right(1).orElse(k(Either.Right(2)))      // => Right(1)
Either.Left(1).orElse(k(Either.Right(2)))       // => Right(2)
{% endhighlight %}


#### `Either:merge()`

Returns the value of whichever side of the disjunction.

    @Either(a, a) => Unit → a
{:lang=oblige}

{% highlight js %}
Either.Right(1).merge()         // => 1
Either.Left(1).merge()          // => 1
{% endhighlight %}


### Folds and Extended Transformations

#### `Either:fold(f, g)`

Catamorphism. Applies `f` to the `Left`-tagged value, and `g` to the
`Right`-tagged value, depending on which value is present.

    @Either(a, b) => (a → c) → (b → c) → c
{:lang=oblige}

{% highlight js %}
function inc(a){ return a + 1 }
function dec(a){ return a - 1 }

Either.Right(1).fold(dec, inc)          // => 2
Either.Left(2).fold(dec, inc)           // => 1
{% endhighlight %}


#### `Either:swap()`

Swaps the disjunction values.

    @Either(a, b) => Unit → Either(a, a)
{:lang=oblige}

{% highlight js %}
Either.Right(1).swap()          // => Left(1)
Either.Left(1).swap()           // => Right(1)
{% endhighlight %}


#### `Either:bimap(f, g)`

Transforms both sides of a disjunction.

    @Either(a, b) => (a → c) → (b → d) → Either(c, d)
{:lang=oblige}

{% highlight js %}
function inc(a){ return a + 1 }
function dec(a){ return a - 1 }

Either.Right(2).map(inc, dec)   // => Right(1)
Either.Left(2).map(inc, dec)    // => Left(3)
{% endhighlight %}


#### `Either:left-map(f)`

Transforms a `Left`-tagged value using an unary function.

    @Either(a, b) => (a → c) → Either(c, b)
{:lang=oblige}

{% highlight js %}
function duplicate(a){ return [a, a] }

Either.Right(1).leftMap(duplicate)              // => Right(1)
Either.Left(1).leftMap(duplicate)               // => Left([1, 1])
{% endhighlight %}
