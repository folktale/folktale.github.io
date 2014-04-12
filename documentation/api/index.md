---
layout: document
title: API Reference
---

Folktale is a suite of libraries for generic functional programming in
JavaScript. It allows the construction of elegant, and robust programs, with
highly reusable abstractions to keep the code base maintainable.

The library is organised by a variety of modules split into logical categories,
with the conventional naming of `<Category>.<Module>`. This page provides
reference documentation for all the modules in the Folktale library, including
usage examples and cross-references for helping you find related concepts that
might map better to a particular problem.


## Core

Provides the most basic and essential building blocks and compositional
operations, which are likely to be used thorough most programs.

 -  [Lambda](/core.lambda): Essential functional combinators and
    higher-order functions derived from Lambda Calculus.
   
   
## Data

Provides functional (persistent and immutable) data structures for representing
program data.

 -  [Maybe](/data.maybe): Safe optional values for modelling computations
    that may fail, or values that might not be available.
   
 -  [Either](/data.either): Right-biased disjunctions, commonly used for
    modelling computations that may fail with additional information about the
    failure.

 -  [Validation](/data.validation): A disjunction for validating inputs and
    aggregating failures, isomorphic to Either.
   
 -  [Future](/data.future): A structure for capturing the effects of
    time-dependent values (asynchronous computations, latency, etc).
    
[Mori]: https://github.com/swannodette/mori


## Control

Provides operations for control-flow.

 -  [Monads](/control.monads.html): Common monad combinators and sequencing
    operations.
