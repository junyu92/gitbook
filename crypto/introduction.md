# Introduction

## Private-key encryption vs Public-key encryption

In private-key (symmetric) encryption, the key used to decrypt is the same as the key
used to encrypt. In public-key (asymmetric) encryption, the key used to decrypt is
different from the key used to encrypt.

## A basic form of symmetric encryption

encryption:
$C = E(K, P)$ where $C$ is a ciphertext, $K$ is a key and $P$ is a plaintext.

decryption:
$P = D(K, C)$

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
shifted by three place but rather by values defined by a $key$, a collection
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

## The permutation

Not every permutation is secure, a cipher's permutation should satisfy three
criteria:

A permutation is **secure** if it satisfies three criteria:

* The permutation should be determinied by the key.
* Differnet keys should result in different permutations.
* The permutation should look random.