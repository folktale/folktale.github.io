---
layout: document
title:  Data › Maybe
---

Use numbered headers: True

 -  **Restrictions:** ECMAScript 5
 -  **API Stability:** Stable
 -  **Module:** [data.maybe](https://npmjs.org/package/data.maybe)
 -  **Version:** 1.0.0
 -  **Licence:** MIT
{.summary}

The library provides a data structure for values that might or might not be
present, as well as computations that may fail. The `Maybe(a)` data structure
models the effects that are implicit in `Nullable` types, thus has none of the
problems often associated with `null` and `undefined` — e.g.:
`NullPointerExceptions`.

The structure models two different cases:

 -  `Just(a)`, represents a `Maybe(a)` that contains the single value `a`,
    which may be any value (including `null` and `undefined`).
 -  `Nothing`, represents a `Maybe(a)` that has no values. Or a failure that
    needs no additional information.

Common use cases for this structure includes modelling values that may or may
not be present in a collection, so instead of needing to invoke a
`collection.has(a)` method before retrieving the value, the user can just call
`collection.get(a)` and retrieve all information they need (if there is an `a`,
and what that `a` is). The same reasoning can be applied to computations that
may fail to provide a value, such as `collection.find(predicate)`. These
computations can safely return a `Maybe(a)` value, even if the collection
contains nullable values.

{% highlight js %}
var Maybe = require('data.maybe')

// We can have a function that can safely find values in an array.
// Where by safe we mean that we can always know whether it succeeded
// or failed, even if the array may contain null or undefined.
//
// find : [a], (a -> Bool) -> Maybe(a)
function find(collection, predicate) {
  for (var i = 0; i < collection.length; ++i) {
    var item = collection[i]
    if (predicate(item))  return Maybe.Just(item)
  }
  return Maybe.Nothing()
}

var numbers = [-2, -1, 0, 1, 2]
var a = find(numbers, function(a){ return a > 5 })
var b = find(numbers, function(a){ return a === 0 })

// Call a function only if both a and b
// have values (sequencing)
a.chain(function(x) {
  return b.chain(function(y) {
    doSomething(x, y)
  })
})

// Transform values only if they're available:
a.map(function(x){ return x + 1 })
// => Maybe.Nothing
b.map(function(x){ return x + 1 })
// => Maybe.Just(1)

// Use a default value if no value is present
a.orElse(function(){ return Maybe.Just(-1) })
// => Maybe.Just(-1)
b.orElse(function(){ return Maybe.Just(-1) })
// => Maybe.Just(0)
{% endhighlight %}

Furthermore, the structure provides instances for the `Functor`, `Applicative`
and `Monad` algebraic structures, allowing it to be combined and manipulated
using expressive and generic monadic computations. This allows safely
sequencing operations that may fail, and safely composing values that you don't
know if are present or not, failing early (returning `Nothing`) if any of the
operations fail.

If one wants to store additional information about failures, the [Either][] and
[Validation][] structures provide such capability, and should be used instead
of the `Maybe(a)` structure.


[Either]: https://github.com/folktale/data.either
[Validation]: https://github.com/folktale/data.validation


# Table of Contents

 *  TOC
{:toc}


## Class: `Maybe(a)`

    class Maybe(a) <: Applicative
                    , Functor
                    , Chain
                    , Monad
                    , Eq
                    , Show
{:lang=oblige}


### Constructors

#### `Maybe:Nothing()`

Constructs a new `Maybe(a)` structure for an absent value.

    Unit → Maybe(a)
{:lang=oblige}

The Nothing case represents a failure.

{% highlight js %}
Maybe.Nothing()         // => Nothingx
{% endhighlight %}


#### `Maybe:Just(a)`

Constructs a new `Maybe(a)` structure that holds the single value `a`.

    a → Maybe(a)
{:lang=oblige}

`a` can be any value, including `null`, `undefined`, or another `Maybe(a)`
monad.

{% highlight js %}
Maybe.Just(Maybe.Just(1))       // => Just(Just(1))
Maybe.Just(Maybe.Nothing())     // => Just(Nothing)
Maybe.Just(undefined)           // => Just(undefined)
{% endhighlight %}


#### `Maybe:from-nullable(a)`

Constructs a new `Maybe(a)` structure from a nullable type.

    (a | null | undefined) → Maybe(a)
{:lang=oblige}

If the value is either `null` or `undefined`, this function returns a
`Nothing`, otherwise the value is wrapped in a `Just(a)`.

{% highlight js %}
Maybe.fromNullable(null)        // => Nothing
Maybe.fromNullable(undefined)   // => Nothing
Maybe.fromNullable(1)           // => Just(1)
{% endhighlight %}


### Predicates

#### `Maybe:is-nothing`

True if the `Maybe(a)` structure contains a failure (i.e.: `Nothing`).

    Boolean
{:lang=oblige}

{% highlight js %}
Maybe.Nothing().isNothing       // => false
Maybe.Just(1).isNothing         // => true
{% endhighlight %}

#### `Maybe:is-just`

True if the `Maybe(a)` structure contains a single value (i.e.: `Just(a)`).

    Boolean
{:lang=oblige}

{% highlight js %}
Maybe.Nothing().isJust          // => false
Maybe.Just(1).isJust            // => true
{% endhighlight %}


### Applicative

#### `Maybe:of(a)`

Creates a new `Maybe(a)` structure holding the single value `a`.

    a → Maybe(a)
{:lang=oblige}

`a` can be any value, including `null`, `undefined` or another `Maybe(a)`
structure.

{% highlight js %}
Maybe.Just(Maybe.Just(1))       // => Just(Just(1))
Maybe.Just(Maybe.Nothing())     // => Just(Nothing)
Maybe.Just(undefined)           // => Just(undefined)
{% endhighlight %}


#### `Maybe:ap(b)`

Applies the function inside the `Maybe(a)` structure to another applicative
type.

    (@Maybe(a → b), f:Applicative) => f(a) → f(b)
{:lang=oblige}

The `Maybe(a)` structure should contain a function value, otherwise a
`TypeError` is thrown.

{% highlight js %}
function inc(a){ return a + 1 }

Maybe.Just(inc).ap(Maybe.Just(2))       // => Just(3)
Maybe.Nothing().ap(Maybe.Just(2))       // => Just(2)
Maybe.Just(inc).ap(Maybe.Nothing())     // => Nothing
Maybe.Nothing().ap(Maybe.Nothing())     // => Nothing
{% endhighlight %}


### Functor

#### `Maybe:map(f)`

Transforms the value of the `Maybe(a)` structure using a regular unary
function.

    @Maybe(a) => (a → b) → Maybe(b)
{:lang=oblige}

{% highlight js %}
function inc(a){ return a + 1 }

Maybe.Just(1).map(inc)          // => Just(2)
Maybe.Nothing().map(inc)        // => Nothing
{% endhighlight %}


### Chain

#### `Maybe:chain(f)`

Transforms the value of the `Maybe(a)` structure using an unary function
yielding a monad of the same type.

    @Maybe(a) => (a → Maybe(b)) → Maybe(b)
{:lang=oblige}

{% highlight js %}
function duplicate(x){ return Maybe.Just([x, x]) }

Maybe.Just(2).chain(duplicate)          // Just([2, 2])
Maybe.Nothing().chain(duplicate)        // Nothing
{% endhighlight %}


### Show

#### `Maybe:to-string()`

Returns a textual representation of the `Maybe(a)` structure.

    @Maybe(a) => Unit → String
{:lang=oblige}


### Eq

#### `Maybe:is-equal(b)`

Tests if two `Maybe(a)` structures are equivalent.

    @Maybe(a) => Maybe(a) → Boolean
{:lang=oblige}

{% highlight js %}
Maybe.Just(1).isEqual(Maybe.Just(1))    // => true
Maybe.Just(1).isEqual(Maybe.Just([1])   // => false
Maybe.Nothing().isEqual(Maybe.Just(1))  // => false
{% endhighlight %}


### Extracting and Recovering

#### `Maybe:get()`

Extracts the value out of the `Maybe(a)` structure, if it exists. Otherwise
throws a `TypeError`.

    @Maybe(a) => Unit → a    :: throws
{:lang=oblige}

If the structure has no value (`Nothing`), a `TypeError` is thrown. See
[get-or-else](#get-or-else) for a getter that can also handle failures.

{% highlight js %}
Maybe.Just(1).get()     // => 1
Maybe.Nothing().get()   // => TypeError: Can't extract the value of a Nothing
{% endhighlight %}


#### `Maybe:get-or-else()`       {#get-or-else}

Extracts the value out of the `Maybe(a)` structure. If there's no value,
returns the given default.

    @Maybe(a) => a → a
{:lang=oblige}

{% highlight js %}
Maybe.Just(1).getOrElse(2)      // => 1
Maybe.Nothing().getOrElse(2)    // => 2
{% endhighlight %}


#### `Maybe:or-else(f)`

Transforms a failure into a new `Maybe(a)` structure.

    @Maybe(a) => (Unit → Maybe(a)) → Maybe(a)
{:lang=oblige}

The analogous of `chain`, but for `Nothing`. Does nothing if the structure
already contains a value.

{% highlight js %}
function k(a) { return function(b) { return a }}

Maybe.Just(1).orElse(k(Maybe.Just(2)))          // Just(1)
Maybe.Nothing().orElse(k(Maybe.Just(2)))        // Just(2)
{% endhighlight %}
