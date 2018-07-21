# Designing a programming language

Having worked out algebraic theories in [previous lectures](lectures-1-2-3.md),
let us turn the equational theories into a small programming language.

What we have to do:

1. Change mathematical terminology to one that is familiar to programmers.
2. Reuse existing concepts (generators, operations, trees) to set up the overall
   structure of the language.
3. Add missing features, such as primitive types and recursion, and generally
   rearrange things a bit to make everything look nicer.
4. Provide operational semantics.
5. Provide typing rules.


## Reading material

There are many possible ways and choices of designing a programming language
around algebraic operations and handlers, but we shall mostly rely on Matija
Pretnar's tutorial [An Introduction to Algebraic Effects and Handlers. Invited
tutorial paper](http://www.eff-lang.org/handlers-tutorial.pdf). A more advanced
treatment is available in [An effect system for algebraic effects and
handlers](https://arxiv.org/abs/1306.6316).


## Change of terminology

* The elements of `Free Σ V` are  are **computations** (instead of trees)
* The elements of `V` are **values** (instead of generators)
* We speak of **value types** (instead of sets of generators)
* We speak of **computation type** (instead of free models)

Henceforth we ignore equations.


## Abstract syntax

We add only one primitive type, namely `bool`. Other constructs (integers,
products, sums) are left as exercises.

Value:

    v ::= x              (variable)
        | false          (boolean constants)
        | true
        | h              (handler)
        | λ x . c        (function)

Handler:

    h ::= handler { return x ↦ c_ret, ... opᵢ(x, κ) ↦ c_i, ... }

Computation:

    c ::= return v               (pure computation)
        | if v then c₁ else c₂   (conditional)
        | v₁ v₂                  (application)
        | with v handle c        (handling)
        | do x ← c₁ in c₂        (sequencing)
        | op (v, λ x . c)        (operation call)
        | fix x . c              (fixed point)

We introduce **generic operations** as syntactic abbreviation and let
`op v` stand for `op(v, λ x . return x)`.

## Operational semantics

We provide small-step semantics, but big step semantics can also be given (see
reading material). In the rules below `h` stands for

    handler { return x ↦ c_ret, ... opᵢ(x,y) ↦ cᵢ, ... }

We write `e₁[e₂/x]` for `e₁` with `e₂` substituted for `x`. The operational rules are:

    ________________________________
    (if true then c₁ else c₂)  ↦  c₁
    
    
    _________________________________
    (if false then c₁ else c₂)  ↦  c₂
    
    
    ______________________
    (λ x . c) v  ↦  c[v/x]
    
    
    _____________________________________
    with h handle return v  ↦  c_ret[v/x]
    
    
    _____________________________________________________________
    with h handle opᵢ(v,κ)  ↦  cᵢ[v/x, (λ x . with h handle κ x)/y]
    
    
    _________________________________
    do x ← return v in c₂  ↦  c₂[v/x]
    
    
    _______________________________________________________
    do x ← op(v, κ) in c₂  ↦  op(v, λ y . do x ← κ y in c₂)
    
    
    ______________________________
    fix x . c  ↦  c[(fix x . c)/x]

## Effect system

### Value and computation types

Value type:

    A, B := bool | A → C | C ⇒ D

Computation type:

    C, D := A!Δ

Dirt:

    Δ ::= {op₁, …, opⱼ}

The idea is that a computation which returns values of type `A` and *may*
perform operations `op₁, …, opⱼ` has the computation type `A!{op₁, …, opⱼ}`.

### Signature

We presume that some way of declaring operations is given, i.e., that we have
a signature `Σ` which lists operations with their parameters and arities:

    Σ = { …, opᵢ : Aᵢ ↝ Bᵢ, … }

Note that the the parameter and the arity types `Aᵢ` and `Bᵢ` are both value types.

### Typing rules

A typing context assigns value types to free variables:

    Γ ::= x₁:A₁, …, xᵢ:Aᵢ

We think of `Γ` as a map which takes variables to their types.

There are two forms of typing judgement:

* `Γ ⊢ v : A` – value `v` has value type `A` in context `Γ`
* `Γ ⊢ c : C` – computation `c` has computation type `C` in context `Γ`

Rules for value typing:

    Γ(x) = A
    _________
    Γ ⊢ x : A
    
    
    ________________
    Γ ⊢ false : bool
    
    
    ________________
    Γ ⊢ true : bool
    
    
    Γ, x : A ⊢ c_ret : B!Θ
    Γ, x : Pᵢ, κ : Aᵢ → B!Θ ⊢ c_i : B!Θ  (for each opᵢ : Pᵢ ↝ Aᵢ in Δ)
    _______________________________________________________________________
    Γ ⊢ (handler { return x ↦ c_ret, ... opᵢ(x) κ ↦ c_i, ... }) : A!Δ ⇒ B!Θ
    
    
       Γ, x:A ⊢ c : C
    _____________________
    Γ ⊢ (λ x . c) : A → C

Rules for computation typing:

        Γ ⊢ v : A
    __________________
    Γ ⊢ return v : A!Δ
    
    
    Γ ⊢ v : bool    Γ ⊢ c₁ : C    Γ ⊢ c₂ : C
    ________________________________________
        Γ ⊢ (if v then c₁ else c₂) : C
    
    
    Γ ⊢ v₁ : A → C    Γ ⊢ v₂ : A
    ____________________________
       Γ ⊢ v₁ v₂ : C
    
    
    Γ ⊢ v : C ⇒ D     Γ ⊢ c : C
    ___________________________
     Γ ⊢ (with v handle c) : D
    
    
    Γ ⊢ c₁ : A!Δ    Γ, x:A ⊢ c₂ : B!Δ
    _________________________________
      Γ ⊢ (do x ← c₁ in c₂) : B!Δ
    
    
    Γ ⊢ v : Aᵢ    opᵢ ∈ Δ    opᵢ : Aᵢ ↝ Bᵢ
    ___________________________________
            Γ ⊢ op v : Bᵢ!Δ
    
    
     Γ, x:A ⊢ c : A!Δ
    _____________________
    Γ ⊢ (fix x . c) : A!Δ


## Safety theorem

If `⊢ c : A!Δ` then:

1. `c = return v` for some `⊢ v : A` **or**
2. `c = op(v, κ)` for some `op ∈ Δ` and some value `v` and continuation `κ`, **or**
3. `c ↦ c'` for some `⊢ c' : A!Δ`.

**Proof.** See [An effect system for algebraic effects and handlers](https://arxiv.org/abs/1306.6316)
for a mechanised proof.


## Other considerations

1. The effect system suffers from the so-called *poisoning*, which can be resolved if we introduce
**effect subtyping**.

2. Recursion requires that we use domain-theoretic denotational semantics. Such
   a semantics turns out to be adequate (but not fully abstract for the same reasons
   that domain theory is not fully abstract for PCF).

See [An effect system for algebraic effects and handlers](https://arxiv.org/abs/1306.6316) where the above points are treated
carefully.


## Problems

### Problem: products

Add simple products `A × B` to the core language:

1. Extend the syntax of values with pairs.
2. Extend the syntax of computations with an elimination of pairs, e.g., `do (x,y) ← c₁ in c₂`.
3. Extend the operational semantics.
4. Extend the typing rules.


### Problem: sums

Add simple sums `A + B` to the core language:

1. Extend the syntax of values with injections.
2. Extend the syntax of computations with an elimination of sums (a suitable `match` statement).
3. Extend the operational semantics.
4. Extend the typing rules.


### Problem: `empty` and `unit` types

Add the `empty` and `unit` types to the core language. Follow the same steps as
in the previous exercises.


### Problem: non-terminating program

Define a program which prints infinitely many booleans. You may assume that the
`print : bool → unit` operation is handled appropriately by the runtime
environment. For extra credit, make it "funny".


### Problem: implement a language with operations and handlers

Implement the core language from Matija Pretnar's
[tutorial](http://www.eff-lang.org/handlers-tutorial.pdf). To make it
interesting, augment it with recursive function definitions, integers, and
product types. Consider implementing the language as part of the [Programming
Languages Zoo](http://plzoo.andrej.com)
