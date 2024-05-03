# On comparison of hashing functions for zero-knowledge
Lately I have found myself reading the paper on Poseidon hash function  available [here](https://eprint.iacr.org/2019/458.pdf). In essence, this came from the need to understand why this particular hash function is used so widely in the domain of zero-knowledge - what makes it special, and why others just don't cut it (or maybe they do)?

What happened to all the well known hashing algorithms such as MD5, SHA256, Keccak256 (the ethereum one)? Somewhere I also heard of something called MiMC. What really is that?

The text here is basically my findings while I was struggling with the above question. And if you are trying to find some info on the same, maybe just read on.

**PSA**: While attempts will be made to understand "WHY" such constructions are used, they are not investigated via any deep cryptographic analysis. I am not a cryptographer, nor do I want to sound like one.

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

### The Merkle–Damgård construction
The Merkle–Damgård construction is a general structure of creating a hash function which works as follows:
1. **MD-compliant input padding**: Take the initial message $m$ of some length $l_m$ and extend it with some deterministic values $j$ (often a bunch of zeros *but not necessarily*) to make a message $M$ such that $l_M = l_m + l_j$ such that $l_M$ is divisible by $r$ (rate). $r$ is often chosen to be something like 512 or 1024.
2. **Repetitive feeding through "compressor" $f$**: A function $f$ is envisioned such that $f(a, b) \to c$ such that all $a$, $b$, $c$ are of same length $r$ (rate), and have an "avalance" effect, such that any single bit change in $a$ or $b$ is enough to change $c$ such that finding $a$ and $b$ from $c$ is sufficiently hard. Repetedly doing it over chunks of original padded message $M$ can compress arbitrary sized message into a determinstic value of length $r$ at the end (see figure below).
3. **Hardening with initialization vector (IV) and finalization step**: an arbitrary chosen [cryptographic salt](https://en.wikipedia.org/wiki/Salt_(cryptography)) can be used as the first $r$ bits into the first compressor in the series of compressors applied to $M$. Similarly, a one-off finalization process can be added to the end to make hash harder. Infact, the last step can also be used to reduce $r$ bits into some smaller (or even larger, why not) hash of $x$ bits (the final output bits of the hashing function).

With the above in perspective, have a look (shamefully copied from wikipedia) image below:
![image](https://hackmd.io/_uploads/SyXpDQWMA.png)

### Some thoughts on the padding step:

A very naive approach towards padding may be just adding a bunch of zeros at the end of message such that the length becomes a multiple of $r$. This can be though of as follows:

```
RATE r   => 4 bytes
MSG  m   => 4a d6 77 90 41
PADDED M => 4a d6 77 90 41 00 00 00
PAD                        ^^ ^^ ^^
```

However imagine what happens when we get two messages
```
MSG  m1  => 4a d6 77 90 41
MSG  m2  => 4a d6 77 90 41 00
```
and pad it using the above setup. Both will result in the same $M$ value. In other words, any message having extra zeros at the end makes it indistinguishable from the one without them

How should one fix this? We can simply use a random value (let's just say `0x67`) as the starting value of padding bits. This would result in following:
```
RATE r    => 4 bytes
MSG  m1   => 4a d6 77 90 41
PADDED M1 => 4a d6 77 90 41 67 00 00
MSG  m2   => 4a d6 77 90 41 00
PADDED M2 => 4a d6 77 90 41 00 67 00
```
However, modern implementations also encode the length of original message in final byte (In real life, this maybe too limiting, a single byte can at max accomodate message length not exceeding 255; but, we can always use $\% 2^{bitcount}$ modulo operation ;) ). In our case, if we just accomodate a single byte for such encoding, we get the following:
```
RATE r    => 4 bytes
MSG  m1   => 4a d6 77 90 41
PADDED M1 => 4a d6 77 90 41 67 00 05
MSG  m2   => 4a d6 77 90 41 00
PADDED M2 => 4a d6 77 90 41 00 67 06
```

In essence due to above considerations, there are reasons we moved from padding bytes `00 00 00` to `67 00 05`.

### Parallelizing the Merkle–Damgård construction
Merkle–Damgård construction as described above shows us an algorithm that is inherently **sequential**. We can make it parallel by in turn having a merkle-tree like construction with each leaf being composed into final hash as below:
![image](https://hackmd.io/_uploads/Hkd7x4WM0.png)

This is premise of a SHA-256 based parallel hashing function called PARSHA-256 (Parallel SHA-256). You can read more about it [here](https://link.springer.com/chapter/10.1007/978-3-540-39887-5_25).

### Back to MD5
MD5 processes a variable-length message into a fixed-length output of 128 bits. The rate $r$ used is 512 (bits). The message hence, is padded to make the input length divisible by 512. Similar to how the padding steps are described above, first, a single bit, 1, is appended to the end of the message. This is followed by as many zeros as are required to bring the length of the message up to 64 bits fewer than a multiple of 512. The remaining bits are filled up with 64 bits representing the length of the original message, modulo $2^{64}$.

To understand the setup better, have a look at the following image:
![image](https://hackmd.io/_uploads/SkFwgB-fC.png)

MD5 maintains a "state" of 128-bits at all times. These 128-bits can be broken down into "words" of size 32-bits. Hence, there are 4 of them in state. We call them $A$ through $D$ which you see above.

Messages as we know are padded and split into chunks of sizes $r = 512$ bits. These are essentially 16 32-bit words.

For every single message chunk of size $r$ a 4-staged algorithm is run where message chunk of size $r$ is used to affect "state" of 128-bits.

Each of the four stages use a different permutation of the 16 32-bit words in the padded message, and also a different equation for $F$ box above.

Since there are four stages, and every stage has to consume 16 32-bit words, in total the setup shown in above image is ran 64 times per chunk. $K_i$ are the round constants and hence 64 in numbers. They are calculated using the following equation:

$$
K_i = \lfloor 2^{32} * abs(sin(i + 1)) \rfloor
$$
where $i$ is in radians.

At the very beginning, the following values are used for $A$, $B$, $C$ and $D$ in the spirit of initialization-vector (IV) of Merkle–Damgård construction.

```
a0 := 0x67452301   // A
b0 := 0xefcdab89   // B
c0 := 0x98badcfe   // C
d0 := 0x10325476   // D
```

Afterwards, the algorithm uses the basic setup:
```
state = [a0, b0, c0, d0];
constants = [floor(2^32 * abs(sin(i+1))) for i in 0..64]

shift[ 0..15] := { 7, 12, 17, 22, ... repeated 2 times,  7, 12, 17, 22 }
shift[16..31] := { 5,  9, 14, 20, ... repeated 2 times,  5,  9, 14, 20 }
shift[32..47] := { 4, 11, 16, 23, ... repeated 2 times,  4, 11, 16, 23 }
shift[48..63] := { 6, 10, 15, 21, ... repeated 2 times,  6, 10, 15, 21 }

for each sequential chunk (c) of size r in padded message (M):
    run 16 times {j =  0..16}: round1(j, state, constants, shift, c)
    run 16 times {j = 16..32}: round2(j, state, constants, shift, c)
    run 16 times {j = 32..48}: round3(j, state, constants, shift, c)
    run 16 times {j = 48..64}: round4(j, state, constants, shift, c)
final_hash = state
```

All the different rounds `round1`, `round2`, `round3` and `round4` do the simple shift as follows:
```
(new_A, new_C, new_D) = (prev_D, prev_B, prev_C)
```
This is evident in the figure given above. However, the way they are different is the definition of function $F$ being used and the permutation of message chunk's words.

To explain permutations better, recall that each message chunk was 16 32-bit long words. These could be layed out as follows `c[0]..c[15]`. This sequential layout may be called a "trivial" permutation. Another permutation may look like: `c[0], c[2], c[4], .., c[14], c[1], c[3], .., c[15]`, basically all even indexed elements before any odd indexed ones. This is another permuation.

Given this context, the different rounds use the following:
```
round1(round_id): 
  F = (B and C) or ((not B) and D)
  new_B = F + constants[round_id] + chunk_word[round_id]
  leftrotate(new_B, shift[round_id])
  
round2(round_id): 
  F = (D and B) or ((not D) and C)
  new_B = F + constants[round_id] + chunk_word[(5×i + 1) mod 16]
  leftrotate(new_B, shift[round_id])

round3(round_id): 
  F = B xor C xor D
  new_B = F + constants[round_id] + chunk_word[(3×i + 5) mod 16]
  leftrotate(new_B, shift[round_id])
  
round4(round_id): 
  F = C xor (B or (not D))
  new_B = F + constants[round_id] + chunk_word[(7×i) mod 16]
  leftrotate(new_B, shift[round_id])
```

That's it, that's the whole of MD5 algorithm.

### Let's run it in code

### Malware with fake certifcate
Flame story: https://en.wikipedia.org/wiki/Flame_(malware) counterfeit

### And then, Flickr's API got busted


## References
- Octa's reference on MD5: https://www.okta.com/identity-101/md5
- Poseidon paper: https://eprint.iacr.org/2019/458.pdf
- Merkle–Damgård construction: https://en.wikipedia.org/wiki/Merkle%E2%80%93Damg%C3%A5rd_construction
- Parallel SHA-256: https://link.springer.com/chapter/10.1007/978-3-540-39887-5_25