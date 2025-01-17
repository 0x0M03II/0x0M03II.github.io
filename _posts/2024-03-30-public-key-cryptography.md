---
layout: post
title:  "Public-Key Cryptography: Diffie-Hellman"
summary: "Learn the basics of public-key cryptography and Diffie-Hellman"
author: Maurice
date: '2024-03-30 15:53:00 +0530'
category: ['encryption', 'math', 'security']
tags: encryption
thumbnail: /assets/img/posts/crypt.png
keywords: encryption, hacking, security, public-key, secure, math, discrete, logarithm
usemathjax: false
permalink: /blog/public-key-diffie-rsa/
---

## Public-Key Cryptography

Public-Key cryptography works by using pairs of mathematically related keys generated by cryptographic algorithms.  These cryptographic algorithms are based on difficult ( one-way ) mathematical problems, such as the discrete logarithm assumption.  In other words, the security of many public-key cryptographic algorithms depends on mathematical and computational difficulty.  A key pair consists of a private key and a public key; the private key should never be shared with anyone.  Public-key algorithms support encryption and digital signatures.

Encryption provides confidentiality and works by using a recipients public key to encrypt a message.  Once the message is received, the recipient will use his or her own private key to decrypt the message.

Digital signatures provide integrity and non-repudiation.  Integrity refers to the assurance that the data is accurate and has not been changed.  Non-repudiation is the inability for someone to deny the origin or source of data.  An example use case would be for a service such as DocuSign, where authenticity of a signature is vital, and, once a contract is signed, it should not be possible for anyone to deny the authenticity of the signature.  In order to provide a digital signature, the sender of a message will encrypt the hash of the message using their private key.  The receiver of a message will use the senders public key to decrypt the hash, and compare the message hash computed by the sender with the hash computed locally.

### Math Background  

I will briefly introduce a few mathematical properties necessary for understanding Diffie-Hellman.  This is by no means comprehensive, and you are encouraged to do additional reading on your own.  You may be tempted to skip the math section; although it may not be the most exiting for some, it is necessary to understand Diffie-Hellman. `Don't Skip` 

#### Division and Remainder
- &forall; a, b &isin; Z (set of intergers) , &exist; q, r &isin; Z such that a = b * q + r :

    - a is the dividend
    - b is the divisor
    - q is the quotient
    - r is the remainder

- &lfloor; a/b &rfloor; = q
- a &equiv; r (mod b)


##### Greatest Common Divisor
- &forall; a, b &isin; Z, a \| b iff the remainder of dividing b by a is zero.
    - a \| b should be read "`a` divides `b`"
    - iff = if and only if

- The greatest common divisor of `a` and `b` is the largest number that is a divisor of both `a` and `b`.
    - gcd(a, b) = d
    - if gcd(a, b) = 1, `a` and `b` are said to be coprime or relatively prime.
    - gcd(a, 0) = a &forall; a &ne; 0.
    - &forall; a, b &isin; Z,  &exist; s, t &isin; Z such that gcd(a, b) = a * s + b * t

GCD(a, b) can easily be coded using the euclidian algorithm as show below.  `Note`: do not confuse the Euclidian algorithm with the Extended Euclidian algorithm; I will introduce that in my next post that will cover RSA.

```python
def gcd(a, b):
    if (b == 0 && a != 0):
        return a
    return gcd(b, a % b)
```
##### Prime Numbers
- p is a prime number iff its set of divisors is {1, p}
    - &forall; a &isin; Z, gcd(a, p) &isin; {1, p}
    - &forall; a &ne; 0 &isin; Z<sub>p</sub>, gcd(a, p) = 1

- How many prime numbers are &le; x ?
    - &pi;(x) = x / ln(x)

- Prime Factorization
    - All integers can be expressed as a product of prime numbers
        - E.g. 48 = 2<sup>4</sup>
        - Factorization is the process of finding that product
    - Prime factorization is computationally [Hard](https://en.wikipedia.org/wiki/Computational_hardness_assumption) for large numbers
        - Currently, there aren't any known algorithms that can compute this in polynomial time.

##### Multiplicative Group
- To keep this post brief, I will provide minimal explanation of Group Mathematics.  For further understanding, read more about [Group Mathematics](https://en.wikipedia.org/wiki/Group_(mathematics)).  If you have time, check out [Galois Groups](https://en.wikipedia.org/wiki/Galois_group) and [Galois Fields](https://en.wikipedia.org/wiki/Finite_field) as well.  Understanding Galois finite field mathematics isn't necessary for understanding Diffie-Hellman in particular, but it is important for understanding other types of cryptography, such as AES (Advanced Encryption Standard).  It is, however, crucial that you understand Group Mathematics, because it plays an important role in Diffie-Hellman.

- Groups are sets with operations that possess the following properties:
    - Closed
    - Associative
    - Identity Element
    - Inverse element

- For the purposes of this blog, I will focus on a multiplicative group G.  The multiplication operation provides includes the following group properties:
    - Closed: &forall; a, b &isin; G, a * b &isin; G
    - Associative: &forall; a, b, c &isin G, (a * b) * c = a * (b * c)
    - Identity Element: &exist; e<sub>0</sub> &isin; G, &forall; a &isin; G, e<sub>0</sub> * a = a * e<sub>0</sub> = a
    - Inverse Element: &forall; a &isin; G, &exist; b &isin; G, a * b = b * a = e<sub>0</sub>

- Group( Z<sub>n</sub><sup>*</sup> )

    - Z<sub>n</sub><sup>*</sup> = {a &isin; Z<sub>n</sub> \| gcd(a,b) = 1}
        - Removes 0 because gcd(a, 0) = a &forall; a &ne; 0.
        - If n is prime, only 0 is removed
        - This means Z<sub>n</sub><sup>*</sup> is the set of all integers &isin; Z<sub>n</sub> such that gcd(a, n) = 1.  That is, a and n are coprime.
    
    - Operation is multiplication (mod n)
        - closed, associative, and commutative
        - Identity element is (e<sub>0</sub> = 1)
    
    - Inverse Element
        - &forall; a &isin; Z<sub>n</sub><sup>*</sup>, gcd(a, n) = 1
        - `Note`: not &forall; a &isin; Z<sub>n</sub>, rather &forall; a &isin; Z<sub>n</sub><sup>*</sup>
            - gcd(a, n) = a * s + n * t
            - 1 = a * s + n * t
            - 1 = a * s + n * t (mod n)
            - 1 &equiv; a * s (mod n)
        - a<sup>-1</sup> &equiv; s (mod n)

- Subgroup
    - G<sub>a</sub> = {a<sup>1</sup>, a<sup>2</sup>, ..., a<sup>m-1</sup>, a<sup>m</sup>} &sube; G
    - G<sub>a</sub> is a subset of G and inherits all of the operations of G
    - &forall; a &isin; G, \|a\| = \|G<sub>a</sub>\|, \|G>sub>a</sub>\| divides \|G\|
    - In set theory \| G \| is the cardinality or number of elements in a set.  It is also referred to as "order of."
        - Z<sub>10</sub><sup>*</sup> = {1, 3, 7, 9}
        - \|Z<sub>10</sub><sup>*</sup>\| = 4
        - In Group Math, \|a\| is the smallest m &isin; N, such that a<sup>m</sup> (mod n) = e<sub>0</sub> identity element.
        - Select an element from the set Z<sub>10</sub><sup>*</sup> !  Let's choose 3.
            - You can see below that when m is 4, 3<sup>4</sup> = 1.  `Remember (mod n)`
            - {3<sup>1</sup>, 3<sup>2</sup>, 3<sup>3</sup>, 3<sup>4</sup>} = {3, 9, 7, 1}
        - Let's also try 9
            - {9<sup>1</sup>, 9<sup>2</sup>} = {9, 1}
    
    - Notice how \|3\| produces the set {3, 9, 7, 1}, which includes all of the numbers in Z<sub>10</sub><sup>*</sup>.  However, \|9\| only includes {9, 1}.  This is important because it means that 3 is a generator of Z<sub>10</sub><sup>*</sup>.
        - an element a &isin; Z<sub>n</sub><sup>*</sup> is a generator of Z<sub>n</sub><sup>*</sup> iff \|a\| = \|G\|

### Diffie-Hellman
Okay!  Now we're ready to explore the Diffie-Hellman algorithm.  Diffie-Hellman make heavy application of the math outlined above, so make sure you understand it before moving on.

Public-key cryptography is significantly slower than symmetric-key cryptography, and is typically used to bootstrap symmetric-key cryptography.  Diffie-Hellman facilitates key agreement or key exchange.

`The following outlines the Diffie-Hellman process for some user U and K`

- Step 1: The first step is for users to agree on a prime numbers `p` and `q`.
    - q is a prime number
    - p is a large prime number, ideally 4096 bits; 2048-bit is acceptable
        - `p must be a safe prime; that is, p = 2q + 1`
        - Find a prime number q, multiply it by 2, and add 1.
    - g &isin; Z<sub>p</sub><sup>*</sup>
        - g is a `generator` (see above for explanation of a generator) that generates an order `q` subgroup of Z<sub>p</sub><sup>*</sup>.
        - g<sup>q</sup> produces every element of Z<sub>p</sub><sup>*</sup>
    
    - Why is it necessary to use a generator g &isin; Z<sub>p</sub><sup>*</sup> and not any arbitrary member a &isin; Z<sub>p</sub><sup>*</sup> ?
        - Using a generator means all possible keys generated by g are reachable through the exponentiation of g, which is important for computing public keys.  When searching for the smallest m such that g<sup>m</sup> = \|G<sub>g</sub>\| = \| Z<sub>p</sub><sup>*</sup> \|, we stop at the identity element to ensure that the entire subgroup is covered.
            - There is likely a better explanation available online.
    
- Step 2: Select a random prime a that will be the private key
    - a &isin; Z<sub>p</sub><sup>*</sup>.
    - Should not be small
    - User K has the private key ak
    - User U has the private key au

- Step 3: Compute public keys
    - user K, the public key computed is K = g<sup>ak</sup> (mod p)
    - user U, the public key computed is U = g<sup>au</sup> (mod p)

- Step 4: Send Public Keys

- Step 5: Compute shared secret key
    - `Note:` a is the private key only known to the individual users U and K; au and ak, respectively.
    - User K, computes a shared secret key using the public key of user U.
        - `shared_key` = U<sup>ak</sup> (mod p)
    - User U, computes a shared secret key using the public key of user K.
        - `shared_key` = K<sup>au</sup> (mod p)

- Step 6: Both User K and U have a shared secret and turn it into a symmetric key
    - An example would be SHA256(shared_key) and using some agreed upon length `l`.

- Diffie-Hellman depends on the difficulty of the Discrete Logaraithm Problem (DLP), which is
    - Given A = g<sup>a</sup> (mod p), it is computationally hard to calculate `a` knowing only `A`, `g`, and `p`.

##### Additoinal Reading

- You may want to do some additional reading, here are a few topics to consider:
    - Miller-Rabin Primality Test
    - Eliptic Curves & Eliptic Curve Diffie-Hellman
    - Modular Exponentiation `Very Important`
        - Algorithm using repeated squared simplifies programming modular exponentiation. 


## Next Post: The RSA Algorithm

In my next post, I will introduce the RSA algorithm, including the foundation mathematical concepts.