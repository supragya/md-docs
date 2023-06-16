# Experimenting with elliptic curve implementation in rust

Recently, I have been following the awesome tutorial describing the internal details of zkSNARK system PLONK called [PLONK by hand by Joshua Fitzgerald](https://research.metastate.dev/plonk-by-hand-part-1/) (Metastate team). The rust implementation that closely follows this is given by Adrià Massanet in [PLONK by fingers](https://github.com/adria0/plonk-by-fingers/tree/main).

For brevity, for rest of this document, PLONK by hand and PLONK by fingers will be abbreviated as PBH and PBF respectively. While PBF is a bigger implementation of the zkSNARK protocol described in PBH, what we concentrate on in this document is how in rust implementation of elliptic curves go.

The purpose of this document remains me describing as a non-cryptography expert the interpretations I draw from what I see. Things might be hand-wavy and could be wrong, but the point remains for this to be a good entry point for others who like to venture into similar territories, which may be slightly wrong at first. May this be a quick disclaimer for readers :P .

## What are elliptic curves and how do they feature here

Elliptic curves are an important cryptographic primitive in modern times. What makes them valuable in where they are used are generally two properties (one optional, but used in PLONK).

1. **Discrete Log Problem**: Points on elliptic curves form a group $G$. Further, this group has property of being finite and cyclic. What that entails is that there exists some element $g \in G$ (the generator) that all other elements in $G$ can be constructed by raising $g$ to some power $x$ as $g^x$. However once you have $g$ and $g^x$, there exists no efficient "classical computer" algorithm that could deduce $x$.
2. **EC Pairing**(Optional): There exists some bilinear map (a function that takes in two elements from different groups of same order (for EC) $G_1$ and $G_2$ and presents an element of a third group $G_T$) $e: G_1 \times G_2 \to G_T$ such that following property holds: $$
e(aP, bQ) = e(P, Q)^{ab} = e(aP, Q)^{b} = e(P, aQ)^{b} = e(P, abQ) = e(abP, Q)$$
where $P$ and $Q$ are elements in $G_1$ and $G_2$ respectively, while $a$ and $b$ are integers. Why this is useful can be seen later. 
> See more properties of bilinear maps in appendix [here](#properties-bil-map).

## Onto the first trait: `FiniteField`
For coding up the elliptic curve, we need to describe a field. This should be a trait since we are going to define functions that a concrete implementation should provide.

Setup a new project using `cargo init --lib pbf`, make `lib.rs` under `src/`, add
```rust=
pub mod elliptic_curve
```
Create `elliptic_curve.rs` under `src/` and add:
```rust=

pub trait FiniteField {
    /// Order or cardinality of the field
    fn order() -> u64;

    /// Gives the zero element and one element
    /// the multiplicative and additive identities
    fn one() -> Self;
    fn zero() -> Self;

    /// Check whether element is identity
    /// element
    fn is_one(&self) -> bool;
    fn is_zero(&self) -> bool;
}
```
Running `cargo build` works for now. Good start! Let's start adding some real function that we will use in real life w.r.t. Finite Fields.

Adding
```rust
pub trait FiniteField {
    ...
    /// Calculates the inverse of a non-zeo
    /// element. Returns `None` if is_zero
    fn inverse(&self) -> Option<Self>;
}
```
makes the compiler unhappy and it returns:
```
error[E0277]: the size for values of type `Self` cannot be known at compilation time
   --> src/elliptic_curve.rs:23:26
    |
23  |     fn inverse(&self) -> Option<Self>;
    |                          ^^^^^^^^^^^^ doesn't have a size known at compile-time
    |
note: required by a bound in `Option`
```
What it means is to be able to wrap the type `T` in `Option<T>`, the amount of memory required by `T` should be known at compile time. If we think closely, that is true for any finite field we may implement because finite fields are operated on modulo some large prime $p$ which is constant for the field, making the memory footprint of any field element known at compile time (as at maximum it may take as much memory as required to represent $p-1$).

So, we add:
```rust=
pub trait FiniteField:
    Sized
{
    ...
}
```
telling that whatever implements `FiniteField` has to have the `Sized` trait. This solves current compilation issues. We add additional operations which we will use later such as mathematical operation like negation, addition, multiplication. We also add operations for raising element to some power, finding inverse etc.
```rust=
use std::{
    fmt::{Debug, Display},
    ops::{Add, Div, Mul, Neg, Sub},
};

pub trait FiniteField:
    Sized               // Size known at compile time
    + From<u64>         // Can create an element from u64
    + Debug + Display   // For debugging and printing
    // Mathemetical Operations ----
    + Neg<Output=Self>
    + Add<Self, Output=Self>
    + Sub<Self, Output=Self>
    + Mul<Self, Output=Self>
    + Div<Self, Output=Option<Self>>
    // Mathemetical Operations ----
{
    /// Order or cardinality of the field
    fn order() -> u64;

    /// Gives the zero element and one element
    /// the multiplicative and additive identities
    fn one() -> Self;
    fn zero() -> Self;

    /// Check whether element is identity
    /// element
    fn is_one(&self) -> bool;
    fn is_zero(&self) -> bool;

    /// Calculates the inverse of a non-zeo
    /// element. Returns `None` if is_zero
    fn inverse(&self) -> Option<Self>;

    /// Raise element to some power
    fn pow(&self, exp: u64) -> Self;

    // Export to u64
    fn as_u64(&self) -> u64;
}
```
When defining for traits such as `std::ops::Add`, the binary operation will have a right hand side that you like to add with. Such type is the generic parameter that needs to be given (defaults to `Self`). While generally the other element that needs to be added is of same type, in certain instances such an operation may not make sense. For example, only `Duration` should be added to current time clock; a clock may not be added to a clock. 

During instances where such weird behavior is not seen, `Add<Self, Output=Self>` can just be written as `Add`. The `Output` is an associated type here. What it means is that only one implementation of trait `Add<rhstype>` is expected for some `rhstype` and no two traits `Add<T, Output=K>`, `Add<T, Output=P>` are allowed at the same time.

The trait `From<u64>` is interesting here, it allows creation of the object from some `u64` type, the function `as_u64` is the complementary exporting function here.

**A note on blanket implementation**: If you hover over `From<u64>`, you will see the following:
```
Used to do value-to-value conversions while consuming the input value. It is the reciprocal of Into.

One should always prefer implementing From over Into because implementing From automatically provides one with an implementation of Into thanks to the blanket implementation in the standard library.
...
```
Another trait exists `pub trait Into<T>: Sized` in rust core library. With `From` implemented for a type, the following call is satisfied
```rust=
let a: FiniteField = FiniteField::from(41u64);
```
With `Into<T>` implemented, the following call is satisfied:
```rust=
let a: FiniteField = 41u64.into()
```
However, the default implementation of `Into<T>` is provided by rust if `From<U>` is defined. So while implementing `From<u64>` makes both the above possible, with just `Into<T>`, the former may not be possible.

## Appendix
<a name="properties-bil-map"></a>
### Properties of a bilinear maps (ChatGPT)

Bilinear maps possess several important properties. Let's consider a bilinear map B: V × W → X, where V, W, and X are vector spaces over the same field. The properties of a bilinear map are as follows:

Linearity in the first argument: For any fixed w in W, the function v ↦ B(v, w) is a linear transformation from V to X. This means that it satisfies the following properties:

B(v1 + v2, w) = B(v1, w) + B(v2, w) (additivity)
B(cv, w) = cB(v, w) (homogeneity), where v, v1, v2 are vectors in V, c is a scalar, and w is a fixed vector in W.
Linearity in the second argument: For any fixed v in V, the function w ↦ B(v, w) is a linear transformation from W to X. This implies:

B(v, w1 + w2) = B(v, w1) + B(v, w2) (additivity)
B(v, cw) = cB(v, w) (homogeneity), where w, w1, w2 are vectors in W, c is a scalar, and v is a fixed vector in V.
Compatibility with scalar multiplication: The bilinear map satisfies the property:

B(cv, w) = B(v, cw) = cB(v, w), where v is a vector in V, w is a vector in W, and c is a scalar.
Distributive property: Bilinear maps distribute over vector addition in both arguments:

B(v1 + v2, w1 + w2) = B(v1, w1) + B(v1, w2) + B(v2, w1) + B(v2, w2),
where v1, v2 are vectors in V, w1, w2 are vectors in W.
These properties ensure that the bilinear map behaves linearly with respect to each argument and respects the operations of vector addition and scalar multiplication.

Additionally, there are other properties specific to certain types of bilinear maps, such as symmetric bilinear maps (satisfying B(v, w) = B(w, v)) or alternating bilinear maps (satisfying B(v, v) = 0 for all v). These properties depend on the specific context and requirements of the application.

Bilinear maps are not necessarily one-to-one (injective) or onto (surjective). The properties of injectivity and surjectivity depend on the specific bilinear map and the vector spaces involved.

Injectivity: A bilinear map B: V × W → X is injective if distinct pairs of vectors in V and W always map to distinct elements in X. In other words, if B(v1, w1) = B(v2, w2), then it implies that (v1, w1) = (v2, w2). However, in general, bilinear maps can have non-trivial kernels, meaning that different pairs of vectors may map to the same element in X.

Surjectivity: A bilinear map B: V × W → X is surjective if every element in the target space X can be obtained as the result of applying the map to some pair of vectors from V and W. Surjectivity is also dependent on the specific bilinear map and the vector spaces involved. Not all bilinear maps are surjective onto their entire target space.

It's worth noting that when bilinear maps are used in the context of pairing-based cryptography, the bilinear maps involved in cryptographic pairings are carefully constructed to possess desired properties and security characteristics. These constructions often involve selecting appropriate elliptic curves and groups to ensure the desired properties of bilinearity, non-degeneracy, and computability, while also taking into account security considerations.