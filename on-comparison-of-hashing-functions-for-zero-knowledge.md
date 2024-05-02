# On comparison of hashing functions for zero-knowledge
Lately I have found myself reading the paper on Poseidon hash function  available [here](https://eprint.iacr.org/2019/458.pdf). In essence, this came from the need to understand why this particular hash function is used so widely in the domain of zero-knowledge - what makes it special, and why others just don't cut it (or maybe they do)?

What happened to all the well known hashing algorithms such as MD5, SHA256, Keccak256 (the ethereum one)? Somewhere I also heard of something called MiMC. What really is that?

The text here is basically my findings while I was struggling with the above question. And if you are trying to find some info on the same, maybe just read on.

## What really is a hash function?
A cryptographic hash function is a mathematical algorithm that takes an input "message" (of variable length possibly) and produces a fixed-size string of characters, which is typically a sequence of numbers and letters. The output, often referred to as the hash value or hash code, is unique to the input data. Even a small change in the input data should result in a significantly different hash value.

In a very general sense, we want the cryptographic hash functions to be practically one-way. This means that given some message $m$ known such that after application of hashing function $H$, the hash generated $h_m$ is such that finding $m$ via reversing the function is practically impossible.

When we say something is "practically impossible", we mean that there is a significant amount of effort that needs to go into finding the pre-image $m$ from $h_m$ in terms of computational resources that we are "practically safe". We can measure this for different hashing systems by their defined "bits of security". If some hash function $H$ advertises $X$ bits of security, it means that on average to find some $m$ (or even some other $m'$, not necessarity $m$) that gives value $h_m$ after application of $H$ as $H(m)$ takes $O(2^X)$ effort in terms of compute. This hardness of not being able to find $m$ is called **pre-image resistance** and the hardness of finding $m'$ such that $m' \not= m$ is called the **collision-resistance** property of a hash function.

## The tale of "Message Digest 5" (MD5)
Wikipedia says:
> MD5 is one in a series of message digest algorithms designed by Professor Ronald Rivest of MIT (Rivest, 1992). When analytic work indicated that MD5's predecessor MD4 was likely to be insecure, Rivest designed MD5 in 1991 as a secure replacement. (Hans Dobbertin did indeed later find weaknesses in MD4.)

Further, the same article says:
> In 1996, a flaw was found in the design of MD5. While it was not deemed a fatal weakness at the time, cryptographers began recommending the use of other algorithms, such as SHA-1, which has since been found to be vulnerable as well. In 2004 it was shown that MD5 is not collision-resistant.

Clearly, this is not suitable for use in modern times. But still, this may provide us with insights on how such hashing systems were envisioned in the minds of cryptographers of the day. Furthermore, these would lead us to contrast such constructions with new constructions that we tend to see in use today.

More on MD5 can be found here: https://www.okta.com/identity-101/md5

However, the highlight in terms of security remains that in modern times, MD5 hash can be broken, as in some $m$ or $m'$ can be found from $h_m$ within seconds.

However, we find a nice information from MD5, something called **[Merkle–Damgård construction](https://en.wikipedia.org/wiki/Merkle%E2%80%93Damg%C3%A5rd_construction)**.

## The Merkle–Damgård construction
The Merkle–Damgård construction is a general structure of creating a hash function which works as follows:
1. **MD-compliant input padding**: Take the initial message $m$ of some length $l_m$ and extend it with some deterministic values $j$ (often a bunch of zeros *but not necessarily*) to make a message $M$ such that $l_M = l_m + l_j$ such that $l_M$ is divisible by $r$ (rate). $r$ is often chosen to be something like 512 or 1024.
2. **Repetitive feeding through "compressor" $f$**: A function $f$ is envisioned such that $f(a, b) \to c$ such that all $a$, $b$, $c$ are of same length $r$ (rate), and have an "avalance" effect, such that any single bit change in $a$ or $b$ is enough to change $c$ such that finding $a$ and $b$ from $c$ is sufficiently hard. Repetedly doing it over chunks of original padded message $M$ can compress arbitrary sized message into a determinstic value of length $r$ at the end (see figure below).
3. **Hardening with initialization vector (IV) and finalization step**: an arbitrary chosen [cryptographic salt](https://en.wikipedia.org/wiki/Salt_(cryptography)) can be used as the first $r$ bits into the first compressor in the series of compressors applied to $M$. Similarly, a one-off finalization process can be added to the end to make hash harder. Infact, the last step can also be used to reduce $r$ bits into some smaller (or even larger, why not) hash of $x$ bits (the final output bits of the hashing function).

With the above in perspective, have a look (shamefully copied from wikipedia) image below:
![image](https://hackmd.io/_uploads/SyXpDQWMA.png)

### Some thoughts on the padding step:



## References
- Octa's reference on MD5: https://www.okta.com/identity-101/md5
- Poseidon paper: https://eprint.iacr.org/2019/458.pdf
- Merkle–Damgård construction: https://en.wikipedia.org/wiki/Merkle%E2%80%93Damg%C3%A5rd_construction
- 