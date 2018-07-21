# Programming with algebraic effects and handlers

In the last lecture we shall explore how algebraic operations and handlers can
be used in programming.

## Eff

There are several languages that support algebraic effects and handlers. The
ones most faithful to the theory of algebraic effects are
[Eff](http://www.eff-lang.org) and the [multicore
OCaml](https://github.com/ocamllabs/ocaml-multicore). They have very similar
syntax, and we could use either, but let us use Eff, just because it was the
first language with algebraic effects and handlers.

You can [run Eff in your browser](http://www.eff-lang.org/try/) or [install
it](https://github.com/matijapretnar/eff/#installation--usage) locally. The page
also has a quick overview of the syntax of Eff, which mimics the syntax of
OCaml.


## Reading material

We shall draw on examples from [An introduction to algebraic effects and
handlers](http://www.eff-lang.org/handlers-tutorial.pdf) and [Programming with
algebraic effects and handlers](https://arxiv.org/abs/1203.1539). Some examples
can be seen also at the [Effects Rosetta
Stone](https://github.com/effect-handlers/effects-rosetta-stone).


## Basic examples

* [Exceptions](./eff-examples/exception.eff)
* [State](./eff-examples/state.eff)

Other examples, such as I/O and redirection can be seen at the [try Eff](http://www.eff-lang.org/try/) page.

## Multi-shot handlers

A handler has access to the continuation, and it may do with it whatever it
likes. We may distinguish handlers according to how many times the continuation
is invoked:

* an **exception-like** handler does not invoke the continuation
* a **single-shot** handler invokes the continuation exactly once
* a **multi-shot** handler invokes the continuation more than once

Of course, combinations of these are possible, and there are handlers where it's
difficult to "count" the number of invocations of the continuation, such as
multi-threading below.

An exception-like handler is, well, like an exception handler.

A single-shot handler appears to the programmer as a form of dynamic-dispatch
callbacks: performing the operation is like calling the callback, where the
callback is determined dynamically by the enclosing handlers.

The most interesting (and confusing!) are multi-shot handlers. Let us have a
look at one such handler.

### Ambivalent choice

Ambivalent choice is a computational effect which works as follows. There is an
exception `Fail : unit → empty` which signifies failure to compute successfully,
and an operation `Choose : α list → α`, which returns one of the elements of the
list. It has to do return an element such that the subsequent computation does
*not* fail (if possible).

With ambivalent choice, we may solve the `n`-queens problem (of placing `n`
queens on an `n × n` chess board so they do not attack each other), see [queens.eff](eff-examples/queens.eff).

## Cooperative multi-threading

Operations and handlers have explicit access to continuations. A handler need
not invoke a continue, it may instead store it somewhere and run *another*
(previously stored) continuation. This way we get *threads*. This was worked out
in [thread.eff](eff-examples/thread.eff).

## Tree representation of a functional

Let us do same game semantics! Suppose we have a **functional**

    h : (int → bool) → bool

When we apply it to a function `f : int → bool`, we feel that

    h f

will proceed as follows: `h` will *ask* `f` about the value `f x₀` for some
integer `x₀`. Depending on the result it gets, it will then ask some furter
question `f x₁`, and so on, until it provides an *answer* `a`.

We may therefore represent such a functional `h` as a **tree**:

* the leaves are the answers
* a node is labeled by a question, which has two subtrees representing
  the two possible continuations (depending on the answer)

We may encode this as the datatype:

    type tree =
      | Answer of bool
      | Question of int * tree * tree

Given such a tree, we can recreate the functional `h`:

    let rec tree2fun t f =
      match t with
      | Answer y -> y
      | Question (x, t1, t2) -> tree2fun (if f x then t1 else t2) f

Can we go backwards? Given `h`, how do we get the tree? It turns out this is not
possible in a purely functional setting, but it is with computational effects.
You can see how to do it with handlers in [fun_tree.eff](./eff-examples/fun_tree.eff).

# Problems

## Problem: breadth-first search

Implement the *breadth-first search* strategy for ambivalent choice.

## Problem: Monte Carlo sampling

The [online Eff](http://www.eff-lang.org/try/) page has an example showing a
handler which modifies a probabilistic computation (one that uses randomness) to
one that computes the *distribution* of results. The handler computes the
distribution in an exhaustive way that quickly leads to inefficiency.

Improve it by implement a [Monte
Carlo](https://en.wikipedia.org/wiki/Monte_Carlo_method) handler for estimating
distributions of probabilistic computations.

## Problem: recursive cows

Contemplate the [recursive cows](https://github.com/effect-handlers/effects-rosetta-stone/tree/master/examples/recursive-cow).

