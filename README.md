# What can DepTyCheck derive?

## What is going on here?

Since [DepTyCheck](https://github.com/buzden/deptycheck) derivation facilities were developed with practical applications in mind, there is no well-defined class of types that support derivation. This makes it hard to reason about the expressive power of specifications.

This project is an ongoing attempt to show that, even limited to currently supported types, it is possible to express a rich family of properties in a way that DepTyCheck can theoretically work with.

## What exactly are we proving

We will work with a certain subclass of types called *closed inductive families*. Roughly, they correspond to possibly-indexed non-polymorphic types that are defined inductively and only allow applications of constructors to the indices. They are known to be supported by DepTyCheck derivation facilities. However, sometimes, when writing specifications, one might wish to wander out of this cozy realm and, for instance, apply a function to an index, like in this type of (some) arithmetic expressions indexed with their values:

```idris
data ArithmeticExpr : Nat -> Type
    Number : (n : Nat) -> ArithmeticExpr n
    Add : (n : Nat) -> (m : Nat) ->
          ArithmeticExpr n -> ArithmeticExpr m ->
          ArithmeticExpr (n + m)

    Mul : (n : Nat) -> (m : Nat) ->
          ArithmeticExpr n -> ArithmeticExpr m ->
          ArithmeticExpr (n * m)

    Pow : (n : Nat) -> (m : Nat) ->
          ArithmeticExpr n -> ArithmeticExpr m ->
          ArithmeticExpr (n ^ m)
```

In that case, derivation of a generator where index of `ArithmeticExpr` is a given value will fail. In this particular case, one can "lift" the functions to type level:

```idris
data Add : Nat -> Nat -> Nat -> Type where
    ZAddM : Add Z m m
    SnAddM : Add n m k -> Add (S n) m (S k)

data Mul : Nat -> Nat -> Nat -> Type where
    ZMulM : Mul Z m Z
    SnMulM : Mul n m k -> Add m k l -> Mul (S n) m l

data Pow : Nat -> Nat -> Nat -> Type where
    NPowZ : Pow n Z 1
    NPowSm : Pow n m k -> Mul n k l -> Pow n (S m) l
```

These are all inductive families. A wonderful property of these definitions is that `Add n m k`, `Mul n m k` and `Pow n m k` are inhabited precisely when `n + m = k`, `n * m = k` or `n ^ m = k` hold respectively. Now one can redefine `ArithmeticExpr` as an inductive family:

```idris
data ArithmeticExpr : Nat -> Type
    Number : (n : Nat) -> ArithmeticExpr n

    Add : (n : Nat) -> (m : Nat) -> (k : Nat) ->
          ArithmeticExpr n -> ArithmeticExpr m -> 
          Add n m k ->
          ArithmeticExpr k

    Mul : (n : Nat) -> (m : Nat) -> (k : Nat) ->
          ArithmeticExpr n -> ArithmeticExpr m ->
          Mul n m k ->
          ArithmeticExpr k

    Pow : (n : Nat) -> (m : Nat) -> (k : Nat) ->
          ArithmeticExpr n -> ArithmeticExpr m ->
          Pow n m k ->
          ArithmeticExpr (n ^ m)

```

Now, all the possible generators would be perfectly derivable by DepTyCheck. The same approach can be applied to predicates, resulting in a type that is inhabited iff the predicate is satisfied.

However, can we be sure that such procedure would exist for any possible function we might encounter in our life? This is precisely the question I'm trying to answer: given a function or a predicate on closed inductive families, can we be sure that there exists a closed inductive family that is inhabited iff the function is evaluated to a particular value or a predicate is satisfied? It seems that an answer is **yes** for a very large families of functions and predicates: I am going to demonstrate that such procedure exists for any *semidecidable* predicate or any *general recursive function* on closed inductive families.

## Plan of work

We will define a class of types called *closed inductive families* that is known to be support by DepTyCheck derivation facilities. After that, we will

1. Show that there is an injective computable encoding of closed inductive families as natural numbers, and this encoding can itself be expressed as a closed inductive family,
2. Show that arbitrary partial computable functions along with the process of their evaluation can be expressed as closed inductive families,
3. Unify the two approaches, concluding that arbitrary semidecidable properties as partial computable functions over closed inductive families can be expressed in a DepTyCheck-supported way.

## Progress

All the parts of the project will be added here.
