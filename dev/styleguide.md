---
layout: devdefault
title: Styleguide
permalink: /dev/styleguide/index.html
---


This document is a comprehensive Scala coding styleguide for Storm Enroute
projects.
We ask contributors to comply to these rules.
Repeated offenders will be asked to write Lint checkers for the rules that they
have a habit of breaking.


## Naming guidelines

Every source file can contain symbols of a single package `x.y.z`,
and must be placed into a source code subdirectory `x/y/z`.

- If the file is named with a upper case initial letter,
  then it may only contain a class and its companion object with the same name.
- If the file is named with a lower case initial letter,
  then it may contain any number of related classes.
  In this case, the words in the file name are delimited with dashes (`-`).
- If the file is named `package.scala`,
  then it may only contain the package object for the related package `x.y.z`.

For example:

    package x.y
    
    package object z {
    }

A package object file may additionally declare a package statement for
declarations that must be outside an object, for compiler-specific reasons.

    package z {
      class Z
      extends MustBeOutsideSingletonObject
    }

In specific occasions, a file may declare a different subpackage within its
body, but this is to be avoided.


## Abbreviations

Capitalize only the first letter of the abbreviation.
This is easier to write than having uppercase everywhere.
For example, if we have a debug API class, we call it:

    class DebugApi {
      // ...
    }


## Line length

Line length should be 88 characters or less.
This way, most laptop screens can display 2 screens at a reasonable font size.
Note that legacy code might not comply to this rule, but all new code must.


### Line folding

Lines exceeding 88 characters in length should be folded into the next line, with a
2 space indentation. Prefer breaking the expression in way in which it remains readable.
Here is an example:

    val x = (1 * 5 + 2) +
      7 / 3

In some cases, more aggressive folding increases readability.
It is ok to fold the line like this:

    def veryLongMethodNameWithLongSignature[A: ContextBound, B <: AnyRef, CC[X]]
      (foo: Int, bar: A, baz: CC[B]): String

But this is better:

    def veryLongMethodNameWithLongSignature
      [A: ContextBound, B <: AnyRef, CC[X]]
      (foo: Int, bar: A, baz: CC[B]): String

When type parameters and/or arguments are really long,
consider folding each argument to its own line.
This improves readability and allows easier access to each type parameter or argument.


## Spacing and indentation guidelines

Every source file should have the following structure:

- start with an optional preamble with a license or the project banner
  and 1 empty line after it
- package name, unless in root package
- 3 empty lines
- imports
- 3 empty lines
- a sequence of top level object separated with 2 empty lines
- each indentation level is comprised of 2 spaces
- never use tabs

This is a typical file containing a class declaration:

    /* Optional preamble
     */
    
    package org.styleguide
    
    
    
    import guidelines._
    
    
    
    class Example {
    }
    
    
    object Example {
      def apply() = new Example
    }


### Class declarations

A class declaration keyword and class name should go in one line.
They should be followed by the type parameters, early initialization list,
list of arguments, list of superclasses and the class body.
If the superclasses cannot fit in the same line, they are placed in the next
line:

    class MyMapper[T, S](f: T => S)
    extends LongNamedSuperClass
    with VeryLongNamedMixingTrait

If the argument list cannot fit on the same line,
it should be placed in the next line and indented:

    class MyMapper[T, S]
      (f: T => S, logger: Logger, errorHandler: Exception => S)
    extends MapperBase

Members of a template (class, trait or object) are usually delimited with a
blank line.
The first and the last member does not have to have a blank line after the class
declaration or before the final brace.

In classes and traits, fields come at the beginning of the class,
and methods follow.
Implicit members come before non-implicit members.


### Miscellaneous

Type ascriptions consist of a colon `:` followed by a space and the type:

    def id[T]: T => T = x => x

Operators in expressions should be delimited with a space.
This is bad: `1+1`,
but the following is good: `1 + 1`.
Keywords that are followed by a condition enclosed in parenthesis like `if` and
`while` should be delimited with a space.
By contrast, method calls should not.
Avoid putting blocks of code or method bodies into a new line.
One liner methods are allowed

Here are some examples:

    var x: Int = 1
    val y = x + 1
    def loop() = while (x != y) {
      println(times2(x))
      x += 1
    }
    def times2(x: Int) = {
      x * 2
    }


## Code organization guidelines

This section contains guidelines on how to code organization conventions within
files and class hierarchies, and lists preferred patterns.


### Package statements

A package statement should be on one line.
If the compilation unit makes an extensive use of functionality in its
superpackages, than there can be several `package` statements one after another
after the preamble -- this is preferred to an `import`.
Both this:

    package x.y.z

And this is ok:

    package x.y
    package z

Avoid having `package` statements that are not at the beginning of the file
unless absolutely necessary (typically as a hack).


### Import statements

An import statement should always have the complete package name -- avoid:

    import collection._
    
In favor of:

    import scala.collection._
    
Import statements should be sorted in alphabetical order.

Import statements can be in a class or a method if the imported functionality is
really restricted to that scope:

    class Foo {
      import Foo._
      val b = new Bar
    }
    
    
    object Foo {
      class Bar
    }

Do not import the classes or singleton objects from the same compilation unit at the top-level --
instead, move them to the scope where they are required.


## Language features

Please avoid using the stackable modifications pattern.
Instead, prefer composition with combinator methods.

Also, avoid the cake pattern, as it increases compilation time,
and causes serialization problems.
Instead, prefer using (implicit) *context* objects to refer to other components
in the system.


## ScalaDoc comments

They begin with `/**` and end with `*/`.
In the case they are multiline, each line should start with a `*`.
First line of text should start with the first line of comment, to save vertical
space.
The following descriptive lines should be separate by one blank line.
Parameter, type parameter and return annotations go in separate lines,
and their descriptions are aligned horizontally.
Multiline parameter descriptions are also aligned.
Descriptions start with an uppercase letter and end with a dot `.`.
Inline code and code segments have to be in standard markdown syntax.
Here is an example.

    /** This method is very important.
     *
     *  The long description in this line will not be
     *  shown in the ScalaDoc unless the user clicks on it.
     *
     *  Example:
     *
     *  { { {
     *  mapForMe[Int, Int](x => x + 1)(11)
     *  } } }
     *
     *  @tparam T           A type parameter that we need.
     *  @tparam S           Another type parameter that is hard
     *                      to understand and needs a long explanation.
     *  @param f            A function parameter.
     *  @param elem         This one is hard to explain. The element
     *                      will be used and applied in `f`.
     *  @return             Whatever the function returns.
     */
    def mapForMe[T, S](f: T => S)(elem: T): S = f(elem)

One line comments are also allowed:

    /** This is a very short comment. */
    def println2(msg: AnyRef) {
      println(msg)
      println("")
    }


## Final words

This document addresses some of the typical guidelines when writing Scala code.
When in doubt, use best judgement or consult the official Scala styleguide.
