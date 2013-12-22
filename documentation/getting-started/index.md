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
using Node, you're all set. If you're not using Node, you'll need to install it
so you can grab the libraries. Don't worry, installing Node is pretty easy:


 1. Go the the [Node.js][] download page.
 2. If you're on Windows, grab the `.msi` installer. If you're on Mac, grab the
    `.pkg` installer.
    
[NPM]: http://npmjs.org/
[Node.js]: http://nodejs.org/download/


> If you're on Linux, the easiest way is to grab the **Linux Binaries**, extract
> them to some folder, and place the `node` and `npm` binaries on your `$PATH`:
>
>     $ mkdir ~/Applications/node-js
>     $ cd ~/Applications/node-js
>     $ wget http://nodejs.org/dist/v0.10.24/node-v0.10.24-linux-x64.tar.gz
>     # or linux-x86.tar.gz, for 32bit architectures
>     $ tar -xzf node-.tar.gz
>     $ cd /usr/local/bin
>     $ sudo ln -s ~/Applications/node-js/node-v0.10.24-linux-x64/bin/node node
>     $ sudo ln -s ~/Applications/node-js/node-v0.10.24-linux-x64/bin/npm npm
>


