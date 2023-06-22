- Types of homomorphic encryptions: PHE, SHE, FHE (https://www.youtube.com/watch?v=7IUS-ixypos)
- Homomorphic: A structure-preserving map between two algebraic structures of the same type e.g. (Group-like, Ring-like, Lattice-like)
    - an algebraic structure consists of a nonempty set A (called the underlying set, carrier set or domain), a collection of operations on A (typically binary operations such as addition and multiplication), and a finite set of identities, known as axioms, that these operations must satisfy.
    - An algebraic structure may be based on other algebraic structures with operations and axioms involving several structures. For instance, a vector space involves a second structure called a field, and an operation called scalar multiplication between elements of the field (called scalars), and elements of the vector space (called vectors).

## Intro to Homomorphic Encryption (https://www.youtube.com/watch?v=umqz7kKWxyw) - Pascal Paillier

What does fully homomorphic encryption require?

In a symmetric encryption setting (e.g. AES-like), the same secret key $sk$ is used both to encrypt the message as well as to decrypt it. We call plaintext message $m$ and encrypted version of it $m'$. 

Even in symmetric setting, there exists a public key $pk$ called "public evaluation key" to operate on the encrypted messages.

Hence a symmetric setting requires tuple $(sk, pk_{eval})$ and asymmetric requires us having tuple $(sk_{enc}, pk_{enc}, pk_{eval})$.

**Amazing property**: Asymmetric FHE is equivalent to Symmetric FHE and you can build one from the other. Unlike in traditional setting where (AES, Elgamal) is different from (RSA, ECDSA) QTBA

FHE allows for data to remain protected
- in-transit: via encryption
- in-computation: via computational possibility

## Timeline for ~40yrs
- 1978: Privacy Homomorphisms: Can we preserve an operation ($+$ or $\times$) over encrypted data?
- For 30 years, we had PHE, either some in addition, some in multiplication but never in both.
- First Generation FHE
    - 2009: Craig Gentry brings an example of a scheme where both operations coexisted homomorphically
    - 2010: DGHV: Doesn't require lattices, just integer field, but very inefficient
- Second Generation FHE
    - 2011: BGV
    - 2012: BFV (leveled schemes)
    - 2012: LTV
    - 2013: BLLN (NTRU branch)
- Third Generation FHE
    - 2013: GSW
    - 2014: FHEW
- Fourth Generation FHE
    - 2016: TFHE
    - 2016: CKKS (leveled schemes)