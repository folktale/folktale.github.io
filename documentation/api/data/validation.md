---
layout: document
title:  Data › Validation
---

Use numbered headers: True

 -  **Restrictions:** ECMAScript 5
 -  **API Stability:** Stable
 -  **Module:** [data.either](https://npmjs.org/package/data.either)
 -  **Version:** 1.0.0
 -  **Licence:** MIT
{.summary}

The `Validation(a, b)` is a disjunction that's more appropriate for
validating inputs, or any use case where you want to aggregate
failures. Not only the `Validation` provides a better terminology for
working with such cases (`Failure` and `Success` versus `Left` and
`Right`), it also allows one to easily aggregate failures and
successes as an Applicative Functor.


# Table of Contents

 *  TOC
{:toc}


## Class: `Validation(a, b)`

    class Validation(a, b) <: Applicative
                            , Functor
                            , Eq
                            , Show
{:lang=oblige}


### Constructors

#### `Validation:Failure(a)`

Constructs a new `Validation(a, b)` structure holding a `Failure`-tagged value.

    a → Validation(a, b)
{:lang=oblige}

{% highlight js %}
function read(path) {
  return exists(path)?     Validation.Success(fread(path))
  :      /* otherwise */   Validation.Failure("Non-existing file: " + path)
}

read("/foo.txt")            // => Success("foo's contents")
read("/non/existent.txt)    // => Failure("Non-existing file: /non/existent.txt")
{% endhighlight %}


#### `Validation:Success(b)`

Constructs a new `Validation(a, b)` structure holding a `Success`-tagged value.

    b → Validation(a, b)
{:lang=oblige}

{% highlight js %}
Validation.Success(1)         // => Success(1)
Validation.Success(null)      // => Success(null)
{% endhighlight %}


#### `Validation:from-nullable(a)`

Constructs a new `Validation(a, b)` structure from a nullable type.

    a? → Validation(a, a)
{:lang=oblige}

If the value is either `null` or `undefined`, this function returns a
`Failure`-tagged value, otherwise returns a `Success`-tagged value.

{% highlight js %}
Validation.fromNullable(null)        // => Failure(null)
Validation.fromNullable(undefined)   // => Failure(undefined)
Validation.fromNullable(1)           // => Success(1)
{% endhighlight %}


### Predicates

#### `Validation:is-failure`

True if the `Validation(a, b)` structure contains a `Failure`-tagged value.

    Boolean
{:lang=oblige}

{% highlight js %}
Validation.Failure(1).isFailure          // => true
Validation.Success(1).isFailure          // => false
{% endhighlight %}

#### `Validation:is-success`

True if the `Validation(a, b)` structure contains a `Success`-tagged value.

    Boolean
{:lang=oblige}

{% highlight js %}
Validation.Success(1).isSuccess         // => true
Validation.Failure(1).isSuccess         // => false
{% endhighlight %}


### Applicative

#### `Validation:of(b)`

Creates a new `Validation(a, b)` instance holding a `Success`-tagged value.

    b → Validation(a, b)
{:lang=oblige}

`b` can be any value, including `null`, `undefined` or another `Validation(a, b)`
structure.

{% highlight js %}
Validation.of(1)                           // => Success(1)
Validation.of(Validation.Failure(1))       // => Success(Failure(1))
Validation.of(null)                        // => Success(null)
{% endhighlight %}


#### `Validation:ap(b)`

If both applicatives contain a `Success`-tagged value, applies the function in
this applicative to the other. Otherwise, aggregates the errors with a
semigroup — both Failures must hold a semigroup.

    (Semigroup s, @Validation(s a, b → c)) => Validation(s a, b) → Validation(s a, c)
{:lang=oblige}



{% highlight js %}
function inc(a){ return a + 1 }

Validation.Success(inc).ap(Validation.Success(2))   // => Success(3)
Validation.Failure([2]).ap(Validation.Success(2))   // => Failure([2])
Validation.Success(inc).ap(Validation.Failure([2])) // => Failure([2])
Validation.Failure([1]).ap(Validation.Failure([2])) // => Failure([1, 2])
{% endhighlight %}


### Functor

#### `Validation:map(f)`

Transforms the `Success`-tagged value of the `Validation(a, b)` structure using a
regular unary function.

    @Validation(a, b) => (b → c) → Validation(a, c)
{:lang=oblige}

{% highlight js %}
function inc(a){ return a + 1 }

Validation.Success(1).map(inc)        // => Success(2)
Validation.Failure(1).map(inc)        // => Failure(1)
{% endhighlight %}


### Show

#### `Validation:to-string()`

Returns a textual representation of the `Validation(a, b)` structure.

    @Validation(a, b) => Unit → String
{:lang=oblige}


### Eq

#### `Validation:is-equal(b)`

Tests if two `Validation(a, b)` structures are equivalent.

    @Validation(a, b) => Validation(a, b) → Boolean
{:lang=oblige}

{% highlight js %}
Validation.Success(1).isEqual(Validation.Success(1))        // => true
Validation.Success(1).isEqual(Validation.Failure(1))        // => false
Validation.Failure(1).isEqual(Validation.Success(1))        // => false
Validation.Failure(1).isEqual(Validation.Failure(1))        // => true
{% endhighlight %}


### Extracting and Recovering

#### `Validation:get()`

Extracts the `Success`-tagged value out of the `Validation(a, b)` structure. THrows
an error if the `Validation(a, b)` contains a `Failure`-tagged value.

    @Validation(a, b) => Unit → b    :: throws
{:lang=oblige}

If the structure has a `Failure`-tagged, a `TypeError` is thrown. See
[get-or-else](#get-or-else) for a getter that can also handle failures.

{% highlight js %}
Validation.Success(1).get()   // => 1
Validation.Failure(1).get()   // TypeError: Can't extract the value of a Failure(a).
{% endhighlight %}


#### `Validation:get-or-else()`       {#get-or-else}

Extracts the `Success`-tagged value out of the `Validation(a, b)` structure. If
there's a `Failure`-tagged value, returns the given default.

    @Validation(a, b) => b → b
{:lang=oblige}

{% highlight js %}
Validation.Success(1).getOrElse(2)    // => 1
Validation.Failure(1).getOrElse(2)    // => 2
{% endhighlight %}


#### `Validation:or-else(f)`

Transforms a `Failure`-tagged value into a new `Validation(a, b)` structure.

    @Validation(a, b) => (Unit → Validation(a, b)) → Validation(a, b)
{:lang=oblige}

The analogous of `chain`, but for `Failure`-tagged value. Does nothing if the
structure already contains a `Success`-tagged value.

{% highlight js %}
function k(a) { return function(b) { return a }}

Validation.Success(1).orElse(k(Validation.Success(2)))      // => Success(1)
Validation.Failure(1).orElse(k(Validation.Success(2)))      // => Success(2)
{% endhighlight %}


#### `Validation:merge()`

Returns the value of whichever side of the disjunction.

    @Validation(a, a) => Unit → a
{:lang=oblige}

{% highlight js %}
Validation.Success(1).merge()         // => 1
Validation.Failure(1).merge()         // => 1
{% endhighlight %}


### Folds and Extended Transformations

#### `Validation:fold(f, g)`

Catamorphism. Applies `f` to the `Failure`-tagged value, and `g` to the
`Success`-tagged value, depending on which value is present.

    @Validation(a, b) => (a → c) → (b → c) → c
{:lang=oblige}

{% highlight js %}
function inc(a){ return a + 1 }
function dec(a){ return a - 1 }

Validation.Success(1).fold(dec, inc)          // => 2
Validation.Failure(2).fold(dec, inc)          // => 1
{% endhighlight %}


#### `Validation:swap()`

Swaps the disjunction values.

    @Validation(a, b) => Unit → Validation(a, a)
{:lang=oblige}

{% highlight js %}
Validation.Success(1).swap()          // => Failure(1)
Validation.Failure(1).swap()          // => Success(1)
{% endhighlight %}


#### `Validation:bimap(f, g)`

Transforms both sides of a disjunction.

    @Validation(a, b) => (a → c) → (b → d) → Validation(c, d)
{:lang=oblige}

{% highlight js %}
function inc(a){ return a + 1 }
function dec(a){ return a - 1 }

Validation.Success(2).map(inc, dec)   // => Success(1)
Validation.Failure(2).map(inc, dec)   // => Failure(3)
{% endhighlight %}


#### `Validation:left-map(f)`

Transforms a `Failure`-tagged value using an unary function.

    @Validation(a, b) => (a → c) → Validation(c, b)
{:lang=oblige}

{% highlight js %}
function duplicate(a){ return [a, a] }

Validation.Success(1).leftMap(duplicate)      // => Success(1)
Validation.Failure(1).leftMap(duplicate)      // => Failure([1, 1])
{% endhighlight %}
