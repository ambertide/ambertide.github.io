---
layout:	post
title:	"Introduction to Fluxion Language"
date:	2020-09-19 14:00:00 +0300
categories: maths
---

When it comes to scientific computing using specially prepared languages and packages, there are many options from native Matlab to Python and Julia and even some more calculator-like options that you can use in your phones like Wolfram Alpha, yet, one has to wonder, why not both.

Wouldn't it be cool to have a single free and open source language that can run on a phone and a computer alike? A symbolic computation library and language that can be embedded to almost any application on any platform?

As a thought experiment, I tried my best to design such a language, and have started work on implementing it, keep in mind this is just a hobby project, it may never end, but still...

Fluxion is a symbolic computation language, it can be used to define functions, take their derivatives and integrals, simplify expressions and calculate solutions in a similar mathematical notation. Here is a small example:

```
> f(x) := sin(x)
> g(x) := e^x + 2x
> g'(x) 
| e^x + 2
> r(x) := (f(x)g(x))'
: f'(x)g(x) + f(x)g'(x)
| (cos(x))(e^x ^ 2x) + (sin(x))(e^x + 2) 
```
# Basic Fluxion

Without going into the interesting parts of Fluxion, let's consider the easiest and most fundamental parts of the language.

## Comments

Single line comments start with `;;`. Multiline comments start with `;*` and end with `*;`

## Expressions

In Fluxion, data is carried through expressions, Expressions come in various shapes and forms.
 
### Variables and Constants

Variables and Constants are two most fundamental building blocks in Fluxion. Constants are numeric values, whereas variables can be single letters, or more complex alphanumeric tokens, with only limitation being that their first character is an english letter.

Variables themselves are pretty standard, they can be bound to constants, both in functions and by themselves, they can be differentiated and integrated, as functions themselves are variables.

```
> x := 12
> x
| 12
> r(x) := x
> r'(x)
| 1
```

They can also be substituted.

```
> r(x) := x^2
> r(y) + y^2
| 2y^2
```

A small note here, you probably already realized this, but the assign operator is `:=`. If you come from a languages inspired by the C family, this may look a bit out of place.


### Imaginary Numerals

Fluxion, without any additions, supports the first imaginary number `i`. It is used similar to math.

```
> f(x) := x + 2i
> sqrt(-1)
| i
```

### Operations

Of course, mathematics is hardly mathematics without operations, Operations are defined as an expression consisting of one or two more expressions tied together by an operator. Operations can evaluate into simpler expressions.

Basic _algebraic_ binary operators are `+`, `-`, `*`, `/` and `^`, the two unary operators are `-` (negation) and `!` (factorial). As the order of operations go, they follow classic rules, first `^` then (`*` and `/`) and finally (`+` and `-`). Parenthesis' can be used to change this as usual, but they are not considered operators. 

Algebraic operators act on imaginary numbers, real numbers and matrices/vectors.

What do we mean by operations may evaluate to simpler forms, well, there are some obvious examples, such as:

```
> 2 + 3
| 5
> x + x
| 2*x
```

But there are also more a bit more robust examples

```
> 3 * (x^2 - 1) * 5 * (x^2 - 1)
| 15 * (x^2 - 1)^2
```

With this being said, there are some caveats to this evaluation, for one, there are sometimes conditions to these simplifications, for instance:

```
> (x^2 - 1) / (x^2 - 1)
| 1 
; x \= 0
```
Here, lines starting with `;` are called conditionals. This is easy to see, but this could be a bit more complex, such as within roots.

In functions, evaluation does not occur until the function is invoked and its variable is bound. This guarantees functions not coming with a large list of conditionals.

### Sets

Sets are types that carry unordered elements, they can be finite or infinite, and can not carry more than one of the same element. Sets are defined using the `{` and `}`. Empty set is defined thusly:

```
> S := {}
```

Set builder notation can also be used to build sets, with the set builder notation, sets can also be infinite.

```
> S := {x | x in dN}
> T := {x | x in dN && 12 <= x < 34}
```

The special `dN` character here is a domain, and implies `x` is a natural number, using this set builder notation, `S` is a set of natural numbers and is infinite. Whereas, `T` is a finite set of natural numbers between twelve and thirty four.

One can check if an element is inside a set, using the `in` keyword.

```
> 2 in S
| True
```

There are special sets, reserved in the Fluxion, called the Domain Sets, these are `dN` (natural numbers), `dZ` (integers), `dQ` (rationals), `dIQ` (irrationals), `dR` (reals) and `dC` (imaginary numbers). They faction as infinite sets.

Set Cardinality, (the size of) a set, can be checked with the `|` operator.

```
> |T|
| 23
```

In set operations, `+` acts as join, `\` acts as complement, `'` also acts as complement but on the universal set, `&` acts as intersection and `*` acts as cartesian product.

When checking for equality and alike with lists, `=` checks if the list element are the same, `<` and `>` checks for subsets and containing.

### Sequences

Sequences are ordered groups of elements sequences. Sets that are orderable, that is, finite or countably infinite are automatically sequences. This means `dN` and `dZ` are sequences, (so is `dQ`, but this fact may not be as helpful as you think it is.), members of sequences can be reached via the `_` operand.

```
> dN[0]
| 0
> T := {1, 3, 4}
> T[0]
| 1
```

Since sequences are lists, `=` does not check for one to one correspondence, this could be checked as follows:

```
> T := {0, 1, 2}
> R := {2, 1, 0}
> T = R
| True
> T[x | x in |T|] = R[x| x in |R|]
| False
```

This also implies sequences can be used to put limits on a variable inside an expression or a function, which they can.

### Two Dimensional Matrices and Vectors

Two dimensional matrices and vectors are a basic type of the Fluxion language, a three by three square matrix can be defined thusly:

```
> M := [1 2 3 | 3 2 1 | 3 4 2]
```

Here, `|` works to divide rows, and space is used to divide columns. A vector is just one dimensional matrix.

```
> Vhorizontal := [1 2 3]
> Vvertical := [1 | 2 | 3]
```

As for operations, binary `+`, `-` works as matrix summation on elements, binary `*` between a numerical value and a matrix works as scalar multiplication, whereas, `*` between two vectors is the dot-product, binary `&` acts as the matrix multiplication and unary `'` works as the transposition of a matrix, finally `|V|` gives the norm of a matrix or a vector.

### Functions

Functions are special variables, they take one or more variables and act on them, functions are defined similar to normal variables:

```
> f(x) := x
> g(x, y) := x + y
```

Functions support all of the normal algebraic operator set, and act in accordance to their rules, however, they also have operators of their own, two of which are unary `'`, `\` and one binary `&`. Keep in mind, all of these operators are also used in other contexts, they may have slightly (or completely) different meanings, however.

Functions can be differentiated using the `'` unary operator. This operator is also used with expressions of all kinds, but the functional version can be put right after the function name.

```
> f'(x)
| 1
> (f(x))'
| 1
```

Functions can also be reversed, using the `\' unary operator.

```
> g(x) := x + 2
> ~g(x)
| x - 2
```

Finally, the `&` operator is the function composition operator, and can be used thusly:

```
> r(x) := (f&g)(x)
```

This would set the `r(x)` to be equal to `f(g(x))`.

### Logic

As we see above, Fluxion supports logical expressions, these can be checks, constraints, etc. There are logical operators for comparing: `=` for equals, `\=` for not equals, `>` and `<` for less than and bigger than, as well as `<=` and  `>=` for the _or equal than_ counterparts.

But you can also chain conditional statements together using `&` and `|`, which are used as _and_ and _or_, for conditionals, `\` is used as not.

Logical variables can be defined, but they cannot be used inside a normal function outside one, very specific role. A logical variable is defined just like a normal variable.

```
> x := 12
> P := x <= 12 & x >= 12
> P
| True
```

Here, `P` is a logical variable. There are two logical constants, `True` and `False`.

### Piecewise Functions

Piecewise functions can be easily defined in Fluxion, where the variable declaration of the function is replaced with a conditional bound to a variable. It can be done as follows:

```
> f(x >= 0) := x
> f(x < 0) := 0
> f(23)
| 23
> f(-1)
| 0
```

Piecewise functions can also _sometimes_ be differentiated. The above function can be differentiated, since it is a continuous function:

```
> f'(x)
| 1
; x >= 0
| 0
; x < 0
```


### Limits and Differentiation

We have already seen that differentiation is supported through the unary `'` operator, limits are also supported using the `->` operator as well as `-> +` and `->-` operators as follows:

```
> f(x) := 1/x^2
> f(x -> 0)
| 1
> g(x) := 1/x
> g(x -> +0)
| 1
> g(x -> -0)
| -1
```

`'` operator can also used with almost any algebraic expression

```
> f(x) := x
> f'(x)
| 1
> x'
| 1
> 1'
| 0
```

It can also be used more than once

```
> f(x) := x^4
> f'(x)
| 4x^3
> f''(x)
| 12x^2
```

## Equation Solving

Conditionals by themselves, if written following a `?`, logical statements evaluates and binds a variable. Take, for instance:

```
> ? x + 1 = 1 - x
> x
| 0
```

`x` is a normal variable, and can be used in the program as normal following this solution.

Equation solving can also be used with imaginary numbers.

```
> ? x + 3i = -1 + yi
> x
| -1
> y
| 3
```

## Keywords

Some keywords names are _reserved_, they can be variables, functions, or sometimes, neither. They can be categorized roughly as follows:

### Constants

Special mathematical constants such as `e`, `pi` and `tau` are predefined.

Mathematical domains, such as natural numbers are predefined `dN`, `dZ`, `dQ`, `dIQ`, `dC` as well as some of their positive and negative counterparts, `dNn` `dNp`, `dZn`, `dZp`, `dQn`, `dIQn`, `dIQp`, `dCn`, `dCp`.

### Functions

Trigonometric functions such as `sin`, `cos`, `tan`, `cot`, `sec`, `csc`, `havsin` are predefined, as is their reverse functions, they are called with an extra `a` before, such as `asin`, `acos` (or with normal reverse function notation).

Moreover, `sqrt` and `square` is also predefined as shorthand, as well as the `root(base, power)` function.

### Statements

Statements are an integral part of the language, they cannot be defined by the users, and are _baked-in_ to the language implementation. As of its current iteration, We have already seen `in` which is used to check if an element is inside a set. There is one we haven't touched upon yet, the `include`. We will talk about include soon.

### Not-a-Numbers

`Undefined` is used for division of any number by zero, a piecewise function's result outside its defined range and differentiation of a piecewise functions that non-continuous around its critical parts.

`Indeterminate` is used for indeterminate forms in mathematics such as `0/0`.

`inf` and `-inf` is used for infinity and negative infinity for derivation and limits, which will be introduced later.

## Reserved Keywords

Some keywords, despite not being used by the language, are reserved for possible future use, these are `const`, `string`, `list`, `goto` and

# Standard Library

Fluxion isn't just what you get right out of the box, there is more to it, to understand it, we need to talk about _Transformations_ and _Contexts_.

## Transformations

You can't do everything using operations. For instance, differentiation and integration aren't really functions. They act on the expressions themselves in very complex ways, they _transform_ them into other expressions, hence the name transformation.

You might have realized, despite them being an integral (hehe) part of maths, I have not introduced integration or complex differentiation. That's because, in many contexts, you don't need them. But sometimes, you do, so how do we reach them?

## Contexts

Fluxion handles these different contexts using something called, well, contexts. Using the `introduce` keyword, you can introduce a new context to your program, following this, functions and variables from the context can be reached using the `::` operator. Let us see an example:

```
> introduce Calculus
> f(x) := x^2 + 3
> Calculus::integrate(f, x)
| (1/3)x^2 + 3x + Calculus::c
```

Within a program, many contexts can be introduced. Standard distribution of Fluxion has many Context available to the user, as of this post, four basic but vital contexts are planned.

### Complex

Complex context includes support for complex numerals up to four dimensions, as well as the operations surrounding them, this context includes `Complex::complex(real, i, j, k)` once a complex number is formed, normal algebraic expressions can act on it. It also includes more functionality for imaginary numbers, such as polar representation, ie: `Complex::polar(imaginary)`.

### Calculus

Calculus is the lifeblood of maths, hence, the Calculus context is rich and has many functions. These include `Calculus::integral(function, variable)`, `Calculus::differentiate(function, variable, degree)` (which includes partial derivation), `Calculus::max` and `Calculus::min` for finding the maximum and minimums points of a function.

### LaTeX

LaTeX context allows for painless rendering of expressions, once introduced, `Latex::convert` will convert your expression to its LaTeX representation, which, with proper IDE support can be shown on the IDE itself.

### IO

The IO context allows for reading and writing data to files. IO context provide the functions for this purpose `IO:read`, `IO:append` and `IO:overwrite`. These functions are special, when using `IO:overwrite` and `IO:append`, you will find that multiline comments you have added inside the function call will find their way into the file, this is not a mistake.

# Extending Fluxion

Fluxion supports new context creation, similar to how other languages support package creation. As well as installing packages to your installation.

## Installing User Defined Contexts

This is handled by the `fluent`, a tool that is distributed alongside with Fluxion. For instance, if a Fluxion context distribution exists at example.com, one can use the `fluent` to retrieve and install it thusly:

`fluent -w example.com`

Unlike many other languages, however, Fluxion is not very suitable for writing actual components that interact and extend the language. This creates two different ways of writing contexts:


## Native contexts

These contexts are written in Fluxion language itself, and are stored in *.fluxion files. They may define mathematical functions, constants and expressions that are easily definable in Fluxion.

Native contexts use the `export` keyword to export parts of its code, `identifier` keyword to identify their version and the filename itself to identify the package name. So, a package called `tangent` with its version 3.1.2 would be inside a file called `tangent.fluent` and:

```
identifier {3, 1, 2}
a := 12
tan(x) := sin(x)/cos(x)
export tan
```

If a user tried to introduce this context using the `introduce tangent`, they would be able to access `tangent::tan`, `introduce` can also be used with version names, such as `introduce tangent 3.1.2` to signal that this script needs a specific version.

## C Extensions

The really complex contexts are written in `C` for the reference implementation. This is too much of a complex topic to go into right now, but their installation works the same.

# Embedding Fluxion

Fluxion can be used as a part of a bigger whole. Fluxion, its reference implementation, `FluxionCore` allows it to be embedded to applications using `C` and `C++`, as well as any application that can import `C` headers, plan is to extend this capability by creating packages for two most popular scientific programming languages, Python and Julia.

# Roadmap

I have a roadmap planned for Fluxion, I am not experienced in these things a lot, though I have dabbled on language creation before on small occasions, but here is what I plan to do:

## Design Specification(s)

Fluxion will first have two design specifications, they will be in depth papers and guides, explaining what the language features will be, first specification will be about the syntax and semantics of the general language itself and the second specification will be about the standard library.

Anyone who wants to have a complete Fluxion distribution will be able to pick this specification up and implement it, for ourselves, the development plan continues to...

## Implementation Specification(s)

Following the creation of the design specifications, two implementation specifications will be detailed, these will detail how the implementation of the design will go, data structures that will be used, packages and alike. They may also include references to some standard optimizations such as the tail-end and NaN boxing. This will also include the design for C extension modules as well as the inner workings of the `fluent` and what it expects to see.

## Reference Implementation

Finally, the reference implementation will be written according to the implementation specification, this will occur in four phases, `FluxionCore` is the core of the language that can be embedded in bigger projects, `FluxionStdLib` is the standard library, written with the C extension module, `Fluxion` is the repl and wrapper around the `FluxionCore` and finally `fluent` is the context manager for the Fluxion project.

## Test Suite

Despite being the fourth in this list, a suite of tets and benchmarks will be developed *during* the third phase, these will be used to benchmark and test the language features of the Fluxion language.

## Optimization and Refactoring

Finally, to tie up the lose ends and to fasten the language, the code might be refactored in some places, test suite will be pivotal here as it will make sure we won't break anything.

## Publishing and Continued Development

Fluxion will be free and open source on GitHub. Versioned using Semantic Versioning, each new version will see updates to necessary specifications, tests and of course, the implementation.