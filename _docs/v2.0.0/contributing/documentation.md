---
title: Contributing documentation
prev_doc: v2.0.0/contributing/new-features
next_doc: v2.0.0/contributing/tests
---

Folktale uses several forms of documentation, and they could always use some improvements. This guide shows how you can contribute to different parts of the documentation in Folktale.


## Contents
{:.no_toc}

* TOC
{:toc}


## How is Folktale documented?

Folktale has three forms of documentation:

  - **Entity metadata**: each function, property, and object in Folktale is annotated with their stability, type, and other information directly in the source code.
  - **API reference**: each function, property, and object in Folktale has a piece of documentation describing at least what it does, and providing a usage example.
  - **Guides**: things like release notes, how to migrate from one version to another, how to contribute to Folktale, etc.

Documentation is a mix of [YAML](http://yaml.org/) and [Markdown](http://commonmark.org/), and is defined both in JavaScript and Markdown files in the repository.


## Entity metadata

Each entity in Folktale has metadata associated with it, through [Meta:Magical](https://github.com/origamitower/metamagical). Metadata is provided directly in the JavaScript source files, with a meta-data comment — multi-line comments that start with the `~` character.

For example, a meta-data for `identity` would look like this:

{% highlight js %}
/*~
 * stability: stable
 * type: |
 *   forall a. (a) => a
 */
const identity = (a) => a;
{% endhighlight %}

The metadata itself is described as a YAML object. The following keys are supported:

  - `stability`: Defines how stable the feature is. May be one of `experimental`, `deprecated`, `stable`, or `locked`. Possible values are explained in the [stability index]({% link _docs/v2.0.0/misc/stability-index.md %}) document. E.g.:

        /*~
         * stability: stable
         */

  - `type`: Describes the type of the entity. The [type notation guide]({% link _docs/v2.0.0/misc/type-notation.md %}) explains the type notation used. E.g.:

        /*~
         * type: |
         *   (String) => String
         */

  - `complexity`: When a function has non-trivial algorithmic complexity, its complexity in [Big O notation](https://en.wikipedia.org/wiki/Big_O_notation). E.g.:

        /*~
         * complexity: O(array.length)
         */


## API Reference


