---
layout: document
title:  Data › Future
---

Use numbered headers: True

 -  **Restrictions:** ECMAScript 5
 -  **API Stability:** Stable
 -  **Module:** [data.either](https://npmjs.org/package/data.either)
 -  **Version:** 1.0.0
 -  **Licence:** MIT
{.summary}

The `Future(a, b)` structure represents values that depend on time. This
allows one to model time-based effects explicitly, such that one can
have full knowledge of when they're dealing with delayed computations,
latency, or anything that can not be computed immediately.
 
A common use for this structure is to replace the usual Continuation-Passing
Style form of programming, in order to be able to compose and sequence
time-dependent effects using the generic and powerful monadic operations.

{% highlight js %}
var Future = require('data.future')
var fs = require('fs')

// read : String -> Future(Error, Buffer)
function read(path) {
  return new Future(reject, resolve) {
    fs.readFile(path, function(error, data) {
      if (error)  reject(error)
      else        resolve(data)
    })
  }
}

// decode : Future(Error, Buffer) -> Future(Error, String)
function decode(buffer) {
  return buffer.map(function(a) {
    return a.toString('utf-8')
  })
}

var intro = decode(read('intro.txt'))
var outro = decode(read('outro.txt'))

// You can use `.chain` to sequence two asynchronous actions, and
// `.map` to perform a synchronous computation with the eventual
// value of the Future.
var concatenated = intro.chain(function(a) {
                     return outro.map(function(b) {
                       return a + b
                     })
                   })

// But the implementation of Future is pure, which means that you'll
// never observe the effects by using `chain` or `map` or any other
// method. The Future just records the sequence of actions that you
// wish to observe, and defers the playing of that sequence of actions
// for your application's entry-point to call.
//
// To observe the effects, you have to call the `fork` method, which
// takes a callback for the rejection, and a callback for the success.
concatenated.fork(
  function(error) { throw error }
, function(data)  { console.log(data) }
)
{% endhighlight %}


# Table of Contents

 *  TOC
{:toc}


## Class: `Future(a, b)`

    class Future(a, b) <: Applicative
                        , Functor
                        , Chain
                        , Monad
                        , Eq
                        , Show
{:lang=oblige}

To construct a new future, you construct a new instance of the `Future` object
passing a long-running computation. This computation will receive two unary
functions as argument, the first one rejects the future with a value, the
second fulfills the future with a value.

{% highlight js %}
new Future(function(reject, resolve) {
  resolve('yay!')
})
{% endhighlight %}

The computation is memoised by default, which means that we will run the
computation at most once, and all subsequent computations that depend on the
value of this future will use the cached result.


### Applicative

#### `Future:of(b)`

Creates a new `Future(a, b)` instance holding a resolved value.

    b → Future(a, b)
{:lang=oblige}

`b` can be any value, including `null`, `undefined` or another `Future(a, b)`
structure.

{% highlight js %}
Future.of(1)                    // => Resolved(1)
Future.of(null)                 // => Resolved(null)
{% endhighlight %}


#### `Future:ap(b)`

Applies the function from a resolved value to another applicative.

    (@Future(a, b → c), f:Applicative) => f(b) → f(c)
{:lang=oblige}

The `Future(a, b)` structure should contain a function value, otherwise a
`TypeError` is thrown.

{% highlight js %}
function inc(a){ return a + 1 }
function rejected(a){ return new Future(reject){ reject(a) }}

Future.of(inc).ap(Future.of(2))         // => Resolved(3)
rejected(inc).ap(Future.of(2))          // => Resolved(2)
Future.of(inc).ap(rejected(2))          // => Rejected(2)
rejected(inc).ap(rejected(2))           // => Rejected(2)
{% endhighlight %}


### Functor

#### `Future:map(f)`

Transforms the resolved value of the `Future(a, b)` structure synchronously.


    @Future(a, b) => (b → c) → Future(a, c)
{:lang=oblige}

{% highlight js %}
function inc(a){ return a + 1 }
function rejected(a){ return new Future(reject){ reject(a) }}

Future.of(1).map(inc)           // => Resolved(2)
rejected(1).map(inc)            // => Rejected(1)
{% endhighlight %}


### Chain

#### `Future:chain(f)`

Sequences two Future-returning asynchronous computations.

    @Future(a, b) => (b → Future(a, c)) → Future(a, c)
{:lang=oblige}

{% highlight js %}
function delayed(x){
  return new Future(reject, resolve) {
    setTimeout(function(){ resolve(x) }, 1000)
  }
}
function rejected(a){ return new Future(reject){ reject(a) }}

Future.of(2).chain(delayed(3))          // => Resolved(3)
rejected(2).chain(delayed(3))           // => Rejected(2)
{% endhighlight %}


### Show

#### `Future:to-string()`

Returns a textual representation of the `Future(a, b)` structure.

    @Future(a, b) => Unit → String
{:lang=oblige}


### Eq

#### `Future:is-equal(b)`

Tests if two `Future(a, b)` structures are equivalent.

    @Future(a, b) => Future(a, b) → Boolean
{:lang=oblige}

{% highlight js %}
function rejected(a){ return new Future(reject){ reject(a) }}

Future.of(1).isEqual(Future.of(1))        // => true
Future.of(1).isEqual(rejected(1))         // => false
rejected(1).isEqual(Future.of(1))         // => false
rejected(1).isEqual(rejected(1))          // => true
{% endhighlight %}


### Extracting and Recovering

#### `Future:or-else(f)`

Transforms a rejected value into a new `Future(a, b)` structure.

    @Future(a, b) => (Unit → Future(a, b)) → Future(a, b)
{:lang=oblige}

The analogous of `chain`, but for rejected values. Does nothing if the
structure already contains a resolved value.

{% highlight js %}
function k(a) { return function(b) { return a }}
function rejected(a){ return new Future(reject){ reject(a) }}

Future.of(1).orElse(k(Future.of(2)))      // => Resolved(1)
rejected(1).orElse(k(Future.of(2)))       // => Resolved(2)
{% endhighlight %}


### Folds and Extended Transformations

#### `Future:fold(f, g)`

Catamorphism. Applies `f` to a rejected value, and `g` to a resolved value,
depending on which value is present.

    @Future(a, b) => (a → c) → (b → c) → c
{:lang=oblige}

{% highlight js %}
function inc(a){ return a + 1 }
function dec(a){ return a - 1 }
function rejected(a){ return new Future(reject){ reject(a) }}

Future.of(1).fold(dec, inc)          // => Resolved(2)
rejected(2).fold(dec, inc)           // => Rejected(1)
{% endhighlight %}


#### `Future:swap()`

Swaps the disjunction values (rejected values become resolved, and
vice-versa).

    @Future(a, b) => Unit → Future(a, a)
{:lang=oblige}

{% highlight js %}
function rejected(a){ return new Future(reject){ reject(a) }}

Future.of(1).swap()          // => Rejected(1)
rejected(1).swap()           // => Resolved(1)
{% endhighlight %}


#### `Future:bimap(f, g)`

Transforms both sides of a disjunction.

    @Future(a, b) => (a → c) → (b → d) → Future(c, d)
{:lang=oblige}

{% highlight js %}
function inc(a){ return a + 1 }
function dec(a){ return a - 1 }
function rejected(a){ return new Future(reject){ reject(a) }}

Future.of(2).map(inc, dec)   // => Resolved(1)
rejected(2).map(inc, dec)    // => Rejected(3)
{% endhighlight %}


#### `Future:rejected-map(f)`

Applies a synchronous transformation to a rejected value (the analogous of
`map` for resolved values).

    @Future(a, b) => (a → c) → Future(c, b)
{:lang=oblige}

{% highlight js %}
function duplicate(a){ return [a, a] }
function rejected(a){ return new Future(reject){ reject(a) }}

Future.of(1).rejectedMap(duplicate)              // => Resolved(1)
rejected(1).rejectedMap(duplicate)               // => Rejected([1, 1])
{% endhighlight %}
