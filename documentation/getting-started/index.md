---
layout: document
title: Getting Started
---


This guide will cover everything you need to start using the Folktale project
right away, from giving you a brief overview of the project, to installing it,
to creating a simple example. Once you get the hang of things, the
[Folktale By Example][] guide should help you
understanding the concepts behind the library, and mapping them to real use
cases.

[Folktale By Example]: /documentation/by-example


## So, what's Folktale anyways?

Folktale is a suite of libraries for allowing a particular style of functional
programming in JavaScript. This style uses overtly generic abstractions to
provide the maximum amount of reuse, and composition, such that large
projects can be built in a manageable way by just gluing small projects
together. Since the concepts underneath the library are too generic, however,
one might find it difficult to see the applications of the data structures to
the problems they need to solve (which the [Folktale By Example][] guide tries
to alleviate by using real world examples and use cases to motivate the
concepts). However, once these concepts are understood, you open up a world of
possibilities and abstractive power that is hard to find anywhere else.

The main goal of Folktale is to allow the development of robust, modular, and
reusable JavaScript components easier, by providing generic primitives and
powerful combinators.


## Do I need to know advanced maths?

Short answer is **no**. You absolutely don't need to know any sort of advanced
mathematics to use the Folktale libraries. 

That said, most of the concepts used by the library are derived from Category
Theory, Algebra, and other fields in mathematics, so the if you want to
extrapolate from the common use cases, and create new abstractions, it will
help you tremendously to be familiar with these branches of mathematics.


## Okay, how can I use it?

Good, let's get down to the good parts!

Folktale uses a fairly modular structure, where each library is provided as a
separate package. To manage all of them, we use [NPM][]. If you're already
using Node, you're all set, just skip to the next section.

If you're not using Node, you'll need to install it so you can grab the
libraries. Don't worry, installing Node is pretty easy:

 1. Go the the [Node.js][] download page.
 2. If you're on Windows, grab the `.msi` installer. If you're on Mac, grab the
    `.pkg` installer.
    
[NPM]: http://npmjs.org/
[Node.js]: http://nodejs.org/download/


> If you're on Linux, the easiest way is to grab the **Linux Binaries**, extract
> them to some folder, and place the `node` and `npm` binaries on your `$PATH`:
>
>     ~ mkdir ~/Applications/node-js
>     ~ cd ~/Applications/node-js
>     ~ wget http://nodejs.org/dist/v0.10.24/node-v0.10.24-linux-x64.tar.gz
>     # or linux-x86.tar.gz, for 32bit architectures
>     ~ tar -xzf node-.tar.gz
>     ~ cd /usr/local/bin
>     ~ sudo ln -s ~/Applications/node-js/node-v0.10.24-linux-x64/bin/node node
>     ~ sudo ln -s ~/Applications/node-js/node-v0.10.24-linux-x64/bin/npm npm
>
> On Ubuntu, you can also use
> [Chris Lea's PPA](https://launchpad.net/~chris-lea/+archive/node.js/).


## Hello, world.

Now that you have Node, we can get down to actually using the library. For
this, let's create a new directory where we'll install the library:

{% highlight sh %}
~ mkdir ~/folktale-hello-world
~ cd ~/folktale-hello-world
~ npm install data.persistent
{% endhighlight %}

The `npm install` command will grab the library for you. In this case, the
library is `data.persistent`, which provides fast immutable data structures to
make your life easier. It should only take a few seconds to get everything
installed, and if all goes well, you'll have a `node_modules` folder with all
the stuff.

Now, run `node` to get dropped into a [Read-Eval-Print-Loop][REPL], which will
allow us to play around with the library interactively. Once in the REPL, you
can load the library:

[REPL]: http://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop

{% highlight js %}
// We load the library by "require"-ing it
var persistent = require('data.persistent')

// And create an alias for the `List` object
var List = persistent.List

// Now, we create an immutable List with two elements
var hello = new List('Hello', 'World')

// And an infinite (lazy) list of exclamations
var exclamation = persistent.repeat('!')

// We then provide a function that'll make emphatic remarks
function emphatic(what, howMuch) {
  return what.concat(persistent.take(howMuch, exclamation))
}

// And finally, display an emphatic Hello World
emphatic(hello, 3)
emphatic(hello, 10)
emphatic(hello, 100)
{% endhighlight %}


## What about the Browser?

Running in the browser takes just a little bit more of effort. To do so, you'll
first need the [Browserify][] too, which converts modules using the Node
format, to something that the Browsers can use. Browserify is just an NPM
module, so it's easy to get it:

[Browserify]: http://browserify.org/

{% highlight sh %}
$ npm install browserify
{% endhighlight %}

Since Browserify has quite a bit more of dependencies than our
`data.persistent` library, it'll take a few seconds to fully install it. Once
you've got Browserify installed, you'll want to create your module using the
Node format. So, create a `hello.js` with the following content:

{% highlight js %}
// We load the data.persistent libraries, just like in Node
var persistent = require('data.persistent')

// Create a list of 10 exclamations
var exclamations = persistent.replicate(10, "!")

// And display it on the browser document
document.body.appendChild(document.createTextNode(exclamations.toString()))
{% endhighlight %}

To compile this file with Browserify, you run the Browserify command giving the
file as input:

{% highlight sh %}
~ $(npm bin)/browserify hello.js > bundle.js
{% endhighlight %}

And finally, include the `bundle.js` file in your webpage:

{% highlight html %}
<!DOCTYPE html>
<html>
  <head>
    <title>Hello, World</title>
  </head>
  <body>
    <script src="bundle.js"></script>
  </body>
</html>
{% endhighlight %}

By opening the page on your webbrowser, you should see 10 exclamations being
added to the page.


## What else do I get?

Folktale is a large collection of base libraries, and still largely a work in
progress, but there's a lot that is already done and can be used today!

  - Safe optional value (replaces nullable types): [Maybe monad][].
  - Disjunction type (commonly encodes errors): [Either monad][].
  - Disjunction with failure aggregation: [Validation applicative][].
  - Asynchronous values and computations: [Future monad][].
  - [Fast immutable List, Vector, Set and Map][persistent].
  - [Common and useful combinators from Lambda Calculus][lambda].
  - [Common and useful monadic combinators][monads].

Each of them are fairly broad concepts. The recommended way of getting familiar
with them is working through the [Folktale By Example][by-example] guide, which
will explain each concept through a series of real world use cases.

[Maybe monad]: https://github.com/folktale/monads.maybe
[Either monad]: https://github.com/folktale/monads.either
[Validation applicative]: https://github.com/folktale/applicatives.validation
[Future monad]: https://github.com/folktale/monads.future
[persistent]: https://github.com/folktale/data.persistent
[lambda]: https://github.com/folktale/core.lambda
[monads]: https://github.com/folktale/control.monads
