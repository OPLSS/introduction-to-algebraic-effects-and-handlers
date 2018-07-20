# Designing a programming language

In the second lecture we shall focus on properly defining a core programming
language with algebraic effects and handlers, based on the algebraic theories
from [Lecture 1](lecture-1.md).

## Outline

### Core language

Value:

    v ::= x              (variable)
        | false          (boolean constants)
        | true
        | h              (handler)
        | λ x . c        (function)

Handler:

    h ::= handler { return x ↦ c_ret, ... opᵢ(x) κ ↦ c_i, ... }

Computation:

    c ::= return v              (pure computation)
        | if v then c₁ else c₂   (conditional)
        | v₁ v₂                  (application)
        | with v handle c        (handling)
        | do x ← c₁ in c₂        (sequencing)
        | op v                   (operation)
        | fix x . c              (fixed point)

### Operational semantics

In the rules below `h` stands for `handler { return x ↦ c_ret, ... opᵢ(x) κ ↦ c_i, ... }`

    -------------------
    return v ↦ return v
    
    ------------------------------
    (if true then c₁ else c₂) ↦ c₁
    
    -------------------------------
    (if false then c₁ else c₂) ↦ c₂
    
    --------------------
    (λ x . c) v ↦ c[v/x]
    
    -----------------------------------
    with h handle return v ↦ c_ret[V/X]

    
    --------------------------------------------------------
    with h handle opᵢ v ↦ cᵢ[v/x, (λ x . with h handle κ x)/κ]
    
    ----------------------------
    fix x . c ↦ c[(fix x . c)/x]

### Effect system

Value type:

    A, B := bool | A → C | C ⇒ D

Computation type:

    C, D := A!Δ

Dirt:

    Δ ::= {op₁, …, opⱼ}

Value typing:

    Γ(x) = A
    ---------
    Γ ⊢ x : A
    
    ----------------
    Γ ⊢ false : bool
    
    ----------------
    Γ ⊢ true : bool

Computation typing:

        Γ ⊢ v : A
    ------------------
    Γ ⊢ return v : A!Δ
    
    
    Γ ⊢ v : bool    Γ ⊢ c₁ : C    Γ : c₂ : C
    ----------------------------------------
        Γ ⊢ (if v then c₁ else c₂) : C
    
    Γ ⊢ v₁ : A → C    Γ ⊢ v₂ : A
    ----------------------------
       Γ ⊢ v₁ v₂ : C
    
    Γ ⊢ v : C ⇒ D     Γ ⊢ c : C
    ---------------------------
     Γ ⊢ (with v handle c) : D
    
    Γ ⊢ c₁ : A!Δ    Γ,x:A ⊢ c₂ : B!Δ
    --------------------------------
      Γ ⊢ (do x ← c₁ in c₂) : B!Δ
    
    Γ ⊢ v : A    opᵢ ∈ Δ
    --------------------
       Γ ⊢ op v : A!Δ
    
     Γ, x : A ⊢ c : A!Δ
    ---------------------
    Γ ⊢ (fix x . c) : A!Δ


## Safety

If `⊢ c : A!Δ` then:

* **either** `c = return v` for some `⊢ v : A` **or**
* `c = op(v, κ)` for some `op ∈ Δ` and some value `v` and continuation `κ`, **or**
* `c ↦ c'` for some `⊢ c' : A!Δ`.


## Reading material

There are many possible ways and choices of designing a programming language
around algebraic operations and handlers, but we shall mostly rely on Matija
Pretnar's tutorial [An Introduction to Algebraic Effects and Handlers. Invited
tutorial paper](http://www.eff-lang.org/handlers-tutorial.pdf). A more advanced
treatment is available in [An effect system for algebraic effects and
handlers](https://arxiv.org/abs/1306.6316).

## Problems

### Problem: products

Add simple products `A × B` to the core language:

1. Extend the syntax of values with pairs.
2. Extend the syntax of computations with an elimination of pairs, e.g., `let (x,y) = v in c`.
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

### Problem: implement a language with operations and handlers

Implement the core language from Matija Pretnar's
[tutorial](http://www.eff-lang.org/handlers-tutorial.pdf). To make it
interesting, augment it with recursive function definitions, integers, and
product types. Consider implementing the language as part of the [Programming
Languages Zoo](http://plzoo.andrej.com)
