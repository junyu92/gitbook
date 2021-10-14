# Introduction

## Private-key encryption vs Public-key encryption

In private-key (symmetric) encryption, the key used to decrypt is the same as the key
used to encrypt. In public-key (asymmetric) encryption, the key used to decrypt is
different from the key used to encrypt.

## A basic form of symmetric encryption

encryption:
$$C = E(K, P)$$ where $$C$$ is a ciphertext, $$K$$ is a key and $$P$$ is a plaintext.

decryption:
$$P = D(K, C)$$

For some ciphers, the ciphertext is the same size as the plaintext; for some
others, the ciphertext is slightly longer. However, ciphertext can never be
shorter than plaintext.

## Classical Ciphers

Here are two simple ciphers working on alphabetic text.

### The Caesar Cipher

It encrypts a message by shifting each of letters down three positions in the
alphabet, wrapping back around to A if the shift reaches Z.

### The Vigenère cipher

The Vigenère cipher is similar to the Caesar Cipher, except the letters aren't
shifted by three place but rather by values defined by a $$key$$, a collection
of letters that represent numbers based on their position in the alphabet.

For example, if the key is DUH, letters in the plaintext are shifted using
3, 20, 7 because D is 3 letters after A, U is 20 letters after A and H is
7 letters after A.

#### The ways to break it

* Figure out the key's length
* Frenquency analysis, which exploits the uneven distribution of letters in
  language

## How Ciphers Work

A cipher contains two components:

* a permutation: a function that transforms an item such that each item
  has a unique inverse.
* a mode of operation: an algotithm that uses a permutation to process
  messages of arbitraty size.

### The Permutation

Not every permutation is secure, a cipher's permutation should satisfy three
criteria:

A permutation is **secure** if it satisfies three criteria:

* The permutation should be determinied by the key.
* Differnet keys should result in different permutations.
* The permutation should look random.

### The Mode of Operation

The mode of operation of a cipher mitigates the exposure of duplicate
letters in the plaintext by using different permutations for duplicate letters.

## The One-Time Pad: Perfect but not practical

### Encryption with the One-Time Pad

The one-time pad takes a plaintext, $$P$$, and a random key, $$k$$, that's the
same length as $$P$$ and produces a ciphertext $$C$$, defined as

$$C = P \oplus K$$

where $$C$$, $$P$$, and $$K$$ are bit strings of the same length and where
$$\oplus$$ is the bitwise exclusive OR operation (XOR), defined as
$$0 \oplus 0 = 0$$, $$0 \oplus 1 = 1$$, $$1 \oplus 0 = 1$$, $$1 \oplus 1 = 0$$.

The one-time pad's decryption is identical to encryption.

The important thing is that **a one-time pad can only be used one time**.

### Why Is the One-Time Pad Secure?

If $$K$$ is random, the resulting $$C$$ looks as random as $$K$$ to an attacker
because the XOR of a random string with any fixed string yields a random string.

### Reference

* [One-time pad](https://en.wikipedia.org/wiki/One-time_pad)

## Encryption Security

A cipher is secure if, even given a large number of plaintext-ciphertext
pairs, nothing can be learned about the cipher's behavior when applied
to other plaintexts or ciphertexts.

This opens up new questions:

* How does an attacker come by these pairs? How large is a "large number"?
This is all defined by **attack models**, assumptions about what the
attacker can and cannot do.
* What chould be "learned" and what "cipher's behavior" are we talking
about? This is defined by **security goals**, descriptions of what is
considered a successful attack.

## Asymmetric Encryption

In asymmetric encryption, there are two keys: one to encrypt and another
to decrypt. The encryption key is called a **public key** and is
generally considered publicly available to anyone who wants to send you
encrypted messages. The decryption key, however, must remain secrey and is
called a **private key**.

The public key can be computed from the private key, but obviously the
private key can't be computed from the public key. That's the key of
public-key cryptography, whose functions are easy to compute in one
direction but practically impossible to invert.