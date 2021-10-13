# Public-key Cryptography

## RSA Algorithm

### key generation

1. choose two distinct prime numbers $p$ and $q$
2. compute $n=pq$

### encryption

1. Bob sends a message $M$ to Alice
2. Bob first truns $M$ into an integer $m$, such that $0 \leq m < n$
3. compute $m^e \equiv c$ (mod $n$) where $e$ is alice's public key, $c$ is ciphertext

### decryption

1. Alice can recover $m$ from ciphertext $c$ by computing $c^d$ since $c^d = (m^e)^d \equiv m$ (mod n) where $d$ is her private key
2. Given $m$, alice can recover the original message $M$ by reversing the padding scheme