# Dependent types with Gödel vibes

## Closed inductive families

Here, we will define a class of closed inductive families. For the sake of simplicity, when reasoning, I will use Idris2 syntax with additional restrictions:

- No implicit arguments,
- Everything has multiplicity $\omega$,
- When two types are mutually inductive, they are defined as `mutual` block instead of a forward declaration.

Closed inductive families are types which satisfy the following properties:

1. Their type constructor and data constructors have no `Type` arguments;
2. All the type constructor and data constructor arguments are closed inductive families themselves;
3. Only applications of other data constructors and type constructors are allowed.
4. Mutual definitions are allowed.

For example, a definition of natural numbers along with `IsZero` and mutually-defined `IsEven` and `IsOdd` predicates are all closed inductive families:

```idris
data Nat = Z | S Nat

data IsZero : Nat -> Type where
  ZIsZ : IsZero Z

mutual
  data IsEven : Nat -> Type where
    ZIsEven : IsEven Z
    SOddIsEven : (n: Nat) -> IsOdd n -> IsEven (S n)

  data IsOdd : Nat -> Type where
    SEvenIsOdd : (n: Nat) -> IsEven n -> IsOdd (S n)
```

## Gödel numbering

Recall the **fundamental theorem of arithmetics**:

> Every natural number $n > 1$ can be uniquely factored into the product of prime numbers up to permutation of factors.

We will use this to construct the **Gödel numbering** for any closed inductive family.

Before giving a formal definition, I will outline the procedure.

Let's first show how to define a Gödel numbering for `Nat`. We will start by assigning each constructor a prime number: `Z` would have $2$ assigned, and `S` would have $3$ assigned.

We will go through each constructor, compute an encoding for each of its arguments, After that, we will take the series of consecutive prime numbers, and raise each of them to the encoding of the argument at the corresponding position and multiply them together. After that, we will raise then number for each of the constructors to the power of the number resulting from encoding its set of arguments.

For example, we will define the encoding of `Nat` the following way:

```idris
godelNat : Nat -> Nat
godelNat Z = 2
godelNat (S n) = 3 ^ (2 ^ encode n)
```

An encoding of mutually-defined types will have to be mutually-defined:

```idris
mutual
  godelIsEven : IsEven n -> Nat
  godelIsEven ZIsEven = 2
  godelIsEven (SOddIsEven k odd_prf) = 3 ^ ((2 ^ godelNat k) * (3 ^ godelIsOdd odd_prf))
  
  godelIsOdd : IsOdd n -> Nat
  godelIsOdd (SEvenIsOdd k even_prf) = 2 ^ ((2 ^ godelNat k) * (3 ^ godelIsEven even_prf))
```

## Construction of Gödel numbering

The procedure described relies on the fact that encodings for non-mutually defined types exist, and the truthfulness of this might not be obvious. To show that the encoding process could indeed be defined in this way, we will introduce a special hierarchy of types and apply induction over this hierarchy.

A type can appear as a constructor of a data constructor of another type, either as part of mutual or non-mutual definition.

All the types that are defined mutually together will be assigned to the same level in this hierarchy. We will assign a level $n$ to a collection of mutually-defined CIFs if they satisfy the following criteria:

1. For all data constructors of all the types in the collection, each argument not belonging to the type defined in the collection belongs to the type of level $k < n$,
2. $n$ is the smallest possible number such that the previous condition holds.

Now we have to show that for every type, there is an appropriate level in the hierarchy. Suppose that there exists a type with no appropriately assignable level. That means that all of its mutual definitions don't have an assignable level. The only way this is possible is if there is a type $T_0$ within the collection that has a constructor with an argument of non-mutually-defined type $T_1$ that has no appropriately assignable level. By repeating the procedure, we obtain a non-well-founded infinite sequence of types with no appropriate level assignable.  This situation, however, is impossible, as one cannot define series of types of that kind within a finite program. Hence, all types have a properly assignable level in the hierarchy.

Finally, we can properly define an encoding via induction on hierarchy levels.

For level $0$, for each batch of mutually defined types, there are no arguments to their data constructors other than the values of types from that batch. We will mutually define a set of encodings akin to the procedure described in the informal part.

For level $n$, suppose there are appropriate definitions of encodings on all levels $k < n$. Then, for every batch of mutually defined types, we mutually define a set of encodings based on the procedure described in the informal part. Every argument of non-mutually-defined type will belong to a type with a smaller level in the hierarchy, thus the encoding for it is defined, making the entire definition valid.

Thus, we can conclude that for any closed inductive family, there exists an appropriate Gödel numbering.

## Injectivity and computability

Computability of the encoding directly follows from the fact that we have formally defined the procedure of computing an encoding for any possible value of a type.

Injectivity and ability to decode an encoded number follow from the fundamental theorem of arithmetics: To decode a term of a type, we would consider its factorization. By the fundamental theorem of arithmetics, we would get a single prime number, corresponding to a constructor, to the power of the encoding of the arguments of the constructor. Then, we would factor the encoding of constructor arguments, and sort the prime numbers in the factorization -- this would yield us a number of form $2^{k_1} 3^{k_2} \ldots p_n^{k_n}$. We would recursively decode $k_1, \ldots, k_n$ as terms of appropriate types and assemble everything together into a proper term.  

## Practicality of encodings

It should be noted that the defined encoding is not practical, and any attempts to apply it to generated types in practice would fail miserably. For instance, consider applying the encoding to a natural number $2$. It corresponds to the constructors `S (S Z)`. Applying the encoding would yield the number $3 ^ {2 ^ {3 ^ {2 ^ 2}}}$. This is an enormous number that would require approximately $4.5 \cdot 10^{14}$ GB to store in memory! However, this encoding is going to be used only to reason about the CIFs and predicates and functions on them, and more efficient representations of functions and predicates are often possible in practice.

## Future work ideas

- It would be nice to come up with an internal construction of a universe of closed inductive families and generically define the encoding procedure over it.
