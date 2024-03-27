# An introduction to `halo2` for engineers
This document intends to give a quick introduction towards halo2 from an engineer's perspective, targetted towards teams and individuals using the `halo2` or `halo2-ce` crate for writing their circuits.

Wherever possible, this document would intend to keep the theoretical aspects of the proof system abstracted away, unless absolutely necessary.

## A circuit, not a program
While seasoned ZK engineers are well aware of the fact that encoding logic in systems such as one we are going to discuss here, one writes a "circuit" and not a "program". 

A circuit in a very crude sense is description of nodes and connections between such nodes to accomplish some logic. The main control board fuse paired with an electical switch on wall for turning on the fan is a "circuit description" of "only turn the fan on if there is no short circuit (fuse protection) and I requested you to be turned on (electrical switch on the wall)". A program on the other hand, is seldom declarative in the same way. Rather, it describes the process a.k.a the logic itself.

As you progress through this document, keep at the back of your head, that what you are writing is essentially connecting one node to another via different wires, more aptly called *constraints*.

## The first lines of code
Setup a basic rust project with following in `Cargo.toml`:
```toml
[package]
edition = "2021"
name = "halo2-trials"
version = "0.1.0"

[dependencies]
halo2 = "0.0.0"

```
Very first thing we need to establish is what do we mean by `halo2` here, as there are different versions of `halo2` out there.
- There is one from ZCash team: https://github.com/zcash/halo2, quite actively maintained
- There is one called `halo2-ce`: https://github.com/halo2-ce/halo2/, relatively less maintained (last commit somewhere in Oct 2022 as seen here: https://github.com/halo2-ce/halo2/graphs/code-frequency)
- There is one from Ethereum-PSE: https://github.com/privacy-scaling-explorations/halo2, quite actively maintained.

The differences lie in the back-end of `halo2`, one which uses "KZG-based" polynomial commitment scheme, and other that uses "inner product arguements" based PCS. It is okay to not know the differences at the moment though!

For now, we are going ahead with ZCash's version of `halo2`. However, before we do that, instead of depending on version `"0.0.0"`, we can lock down the specific commit that we want to depend on, by changing the dependency in `Cargo.toml` as:
```
// TODO
```
## References
- Orochi Network's simple example: https://docs.orochi.network/halo2-for-dummies/simple-example/section.html
- Zcash Halo2 Book: https://zcash.github.io/halo2/
- Awesome Halo2 references: https://github.com/adria0/awesome-halo2
