---
layout: document
title: Data › Persistent
---

Use numbered headers: True

<!-- * * * -->

 -  **Restrictions:** ECMAScript 5
 -  **API Stability:** Experimental
 -  **Module:** [data.persistent](https://npmjs.org/package/data.persistent)
 -  **Version:** 0.1.0
 -  **Licence:** MIT
{.summary}

The `data.persistent` module provides fast implementations of persistent data
structures (compiled from ClojureScript by [Mori][]), with the algebraic
instances defined by [Fantasy Land][]. This allows a broad set of generic
operations to be written and shared by all of these structures, and other
algebraic structures.

[Mori]: http://swannodette.github.io/mori/
[Fantasy Land]: https://github.com/fantasyland/fantasy-land

All collections provide instances for `Semigroup`, `Monoid`, `Functor`,
`Applicative`, `Chain`, and `Monad`, from the Fantasy Land specification, and
the `Eq`, `Hashable` and `Show` typeclasses.


# Table of Contents

 *  TOC
{:toc}


## Class: `List(a)`             {#list}

An immutable linked list, with efficient prepending elements to the head of the
list, but linear time random access.

    List(...a) <: Semigroup
                , Monoid
                , Functor
                , Applicative
                , Chain
                , Monad
                , Eq
                , Show
                , Hashable
{:lang=oblige}

Lists provide ordered heterogeneous collections optimised for insertion at the
head. As a linked list, random access has linear time on the size of the list.

To construct a new list, you call the variadic `List` with the elements of the
list:

{% highlight js %}
var numbers = new List(1, 2, 3, 4, 5)
{% endhighlight %}

Unlike native compound data structures, `List` doesn't give you any common
syntactical for accessing the elements. Instead the [first](#first), and
[rest](#rest) operations are commonly used. You can access any element using
[nth](#nth), but as a linked list, the structure is not the most efficient for
these sorts of usage, and you might prefer taking a look at [Vector](#vector),
instead.


### Semigroup

#### `List:concat(b)`

Concatenates this list with another list.

    (@List(a)) => List(a) -> List(a)
{:lang=oblige}

{% highlight js %}
var as = new List(1, 2, 3)
var bs = new List(4, 5, 6)

as.concat(bs)   // => List(1, 2, 3, 4, 5, 6)
bs.concat(as)   // => List(4, 5, 6, 1, 2, 3)
{% endhighlight %}


### Monoid

#### `List:empty()`

Returns a new empty list.

    (@List(a)) => () -> List(a)
{:lang=oblige}

{% highlight js %}
var as = new List(1, 2, 3)

as.empty()      // => List()
as              // => List(1, 2, 3)
{% endhighlight %}


### Functor

#### `List:map(f)`

Returns a new list, where every element is transformed by the unary function
`f`.

    (@List(a)) => (a -> b) -> List(b)
{:lang=oblige}

{% highlight js %}
var as = new List(1, 2, 3)

as.map(function(a){ return a + 1 })     // => List(2, 3, 4)
{% endhighlight %}


### Applicative

#### `List.of(a)`

Constructs a new singleton list with the given value.

    a -> List(a)
{:lang=oblige}

{% highlight js %}
List.of(1)              // => List(1)
List.of(List.of(2))     // => List(List(2))
{% endhighlight %}


#### `List:ap(b)`

Returns a new list for all the combinations of functions in this list applied
to every item in the `b` functor.

    (@List(a -> b)) => List(a) -> List(b)
{:lang=oblige}

{% highlight js %}
var fs = new List(
  function(a) { return a * a }
, function(a) { return a + 1 }
, function(a) { return a - 1 }
)

fs.ap(new List(1, 2, 3))        // => List(1, 4, 9, 2, 3, 4, 0, 1, 2)
{% endhighlight %}


### Chain

#### `List:chain(f)`

Returns a new list, where all of the elements are transformed by the given
function to lists, and spliced in the original position.

    (@List(a)) => (a -> List(b)) -> List(b)
{:lang=oblige}

{% highlight js %}
function siblings(a) {
  return new List(a - 1, a, a + 1)
}

new List(1, 4, 7).chain(siblings)
// => List(0, 1, 2, 3, 4, 5, 6, 7, 8)
{% endhighlight %}


### Show

#### `List:toString()`

Returns a textual representation of the list.

    (@List(a)) => () -> String
{:lang=oblige}

{% highlight js %}
new List(1, 2, 3).toString()    // => "(1 2 3)"
{% endhighlight %}


### Eq

#### `List:isEqual(b)`

Returns whether the list is structurally equal to another.

    (@List(a)) => List(a) -> Boolean
{:lang=object}

{% highlight js %}
var as = new List(1, 2, 3)
var bs = new List(1, 2)
var cs = new List(3)

as.isEqual(bs)                  // => false
as.isEqual(bs.concat(cs))       // => true
bs.concat(cs).isEqual(as)       // => true
{% endhighlight %}


### Hashable

#### `List:hash()`

Returns an unique hash for the list object.

    (@List(a)) => () -> Number
{:lang=oblige}

{% highlight js %}
var as = new List(1)
var bs = new List(2)
var cs = new List(1, 2)

as.hash()               // => 1
bs.hash()               // => 2
cs.hash()               // => -1640531462
as.concat(bs).hash()    // => -1640531462
{% endhighlight %}


## Class: `Vector(a)`           {#vector}

An immutable vector, with efficient appending elements to the end of the
vector, and efficient random access.

    Vector(...a) <: Semigroup
                  , Monoid
                  , Functor
                  , Applicative
                  , Chain
                  , Monad
                  , Eq
                  , Show
                  , Hashable
{:lang=oblige}

Vectors provide ordered heterogeneous collections optimised for insertion at
the end, and with efficient (constant time) random access.

To construct a new vector, you call the variadic `Vector` with the elements:

{% highlight js %}
var numbers = new Vector(1, 2, 3, 4, 5)
{% endhighlight %}

Unlike native compound data structures, `Vector` doesn't give you any common
syntactical for accessing the elements. Instead the [first](#first), and
[rest](#rest), and [nth](#nth) operations are commonly used.


### Semigroup

#### `Vector:concat(b)`

Concatenates this vector with another vector.

    (@Vector(a)) => Vector(a) -> Vector(a)
{:lang=oblige}

{% highlight js %}
var as = new Vector(1, 2, 3)
var bs = new Vector(4, 5, 6)

as.concat(bs)   // => Vector(1, 2, 3, 4, 5, 6)
bs.concat(as)   // => Vector(4, 5, 6, 1, 2, 3)
{% endhighlight %}


### Monoid

#### `Vector:empty()`

Returns a new empty vector.

    (@Vector(a)) => () -> Vector(a)
{:lang=oblige}

{% highlight js %}
var as = new Vector(1, 2, 3)

as.empty()      // => Vector()
as              // => Vector(1, 2, 3)
{% endhighlight %}


### Functor

#### `Vector:map(f)`

Returns a new vector, where every element is transformed by the unary function
`f`.

    (@Vector(a)) => (a -> b) -> Vector(b)
{:lang=oblige}

{% highlight js %}
var as = new Vector(1, 2, 3)

as.map(function(a){ return a + 1 })     // => Vector(2, 3, 4)
{% endhighlight %}


### Applicative

#### `Vector.of(a)`

Constructs a new singleton vector with the given value.

    a -> Vector(a)
{:lang=oblige}

{% highlight js %}
Vector.of(1)                // => Vector(1)
Vector.of(Vector.of(2))     // => Vector(Vector(2))
{% endhighlight %}


#### `Vector:ap(b)`

Returns a new vector for all the combinations of functions in this vector applied
to every item in the `b` functor.

    (@Vector(a -> b)) => Vector(a) -> Vector(b)
{:lang=oblige}

{% highlight js %}
var fs = new Vector(
  function(a) { return a * a }
, function(a) { return a + 1 }
, function(a) { return a - 1 }
)

fs.ap(new Vector(1, 2, 3))        // => Vector(1, 2, 0, 4, 5, 3, 9, 4, 2)
{% endhighlight %}


### Chain

#### `Vector:chain(f)`

Returns a new vector, where all of the elements are transformed by the given
function to vectors, and spliced in the original position.

    (@Vector(a)) => (a -> Vector(b)) -> Vector(b)
{:lang=oblige}

{% highlight js %}
function siblings(a) {
  return new Vector(a - 1, a, a + 1)
}

new Vector(1, 4, 7).chain(siblings)
// => Vector(0, 1, 2, 3, 4, 5, 6, 7, 8)
{% endhighlight %}


### Show

#### `Vector:toString()`

Returns a textual representation of the vector.

    (@Vector(a)) => () -> String
{:lang=oblige}

{% highlight js %}
new Vector(1, 2, 3).toString()    // => "[1 2 3]"
{% endhighlight %}


### Eq

#### `Vector:isEqual(b)`

Returns whether the vector is structurally equal to another.

    (@Vector(a)) => Vector(a) -> Boolean
{:lang=object}

{% highlight js %}
var as = new Vector(1, 2, 3)
var bs = new Vector(1, 2)
var cs = new Vector(3)

as.isEqual(bs)                  // => false
as.isEqual(bs.concat(cs))       // => true
bs.concat(cs).isEqual(as)       // => true
{% endhighlight %}


### Hashable

#### `Vector:hash()`

Returns an unique hash for the vector object.

    (@Vector(a)) => () -> Number
{:lang=oblige}

{% highlight js %}
var as = new Vector(1)
var bs = new Vector(2)
var cs = new Vector(1, 2)

as.hash()               // => 1
bs.hash()               // => 2
cs.hash()               // => -1640531462
as.concat(bs).hash()    // => -1640531462
{% endhighlight %}
