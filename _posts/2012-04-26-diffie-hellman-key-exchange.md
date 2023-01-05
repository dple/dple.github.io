---
title: 'Diffie-Hellman Key Agreement Protocol'
date: 2022-04-26
permalink: /posts/2012/08/diffie-hellman-key-exchange/
tags:
  - Diffie-Hellman
  - Key Exchange
  - Java implementation
---

This blog will discuss the Diffie-Hellman key exchange protocol, which is the most popular key exchange used on Internet nowadays. 

What is a Key agreement protocol?
======
Imagine that you (a.k.a. Alice) want to exchange messages with sensitive information with your friend, Bob, over an insecure network. You may think about a cipher such as AES to encrypt your messages so that Eve, a malicious internet user, won't learn anything even she could intercept them.

For just a piece of information, let's say a single message, a public key encryption algorithm such as RSA can be used, that is, you encrypt your message by using Bob's RSA public key (that is publicly shared by Bob on his website). Bob (and only Bob) will be able to decrypt that encrypted message by using his RSA private key.

However, if the information needs to be exchanged in bulk, RSA encryption is not recommended to use as its inefficiency. A symmetric encryption such as AES can be used. ``Symmetric'' means that both Alice and Bob must use the same secret key to encrypt and decrypt the data exchanged. How could they do that from distance?

> "Key agreement is a public key cryptographic protocol that allows two or more people to agree on a shared secret key that can be used later in a symmetric cryptographic algorithm".

Diffie-Hellman Key Agreement
======
[Whitfield Diffie](https://en.wikipedia.org/wiki/Whitfield_Diffie) and [Martin Hellman](https://en.wikipedia.org/wiki/Martin_Hellman), fathers of [public key cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography) introduced the first key agreement protocol in 1976. This is also one of the first public-key cryptographic protocols. The protocol works over a finite cyclic group, namely, G with a generator g of order n, that is g^n = 1, the identity element of the group G. In order to exchange a shared secret key, Alice and Bob execute the following steps:

1. Alice and Bob publicly agree to use the group G with the generator g;
2. Alice chooses a secret integer a, computes A = g<sup>a</sup>, and then sends A to Bob;
3. Bob chooses a secret integer b, computes B = g<sup>b</sup>, and then sends B to Alice;
4. Alice computes B<sup>a</sup>, Bob computes A<sup>b</sup>;
5. As B<sup>a</sup> = A<sup>b</sup> = g<sup>ab</sup>, Alice and Bob had the same value, so-called K, and can use that shared secret as the session key to encrypt/decrypt exchanged information between them.

![Diffie-Hellman Key Exchange](images/dh.webp) (Source: Wikipedia)



Implementation in Java
======
Java supported the Diffie-Hellman key agreement protocol in the package javax.crypto.KeyAgreement. Instances can be on multiplicative groups over integers or groups of points on elliptic curves.

For example, the following code gets an instance of the Diffie-Hellman key agreement protocol with elliptic curves as defined in RFC 7748, including X25519 and X448 (Edwards curves providing 128 and 224-bit security levels).

```java
// Diffie-Hellman over the curve X448
KeyAgreement ka = KeyAgreement.getInstance("XDH");
NamedParameterSpec params = new NamedParameterSpec("X448");
```

Now, you must have your own private key and Bob's public key to generate a shared key with Bob. Java class KeyAgreement supports the following functions:

```java
ka.init(priv);
ka.doPhase(pub, true);
byte [] secret = ka.generateSecret();
```

The full source code can be downloaded [here](https://github.com/dple/KeyAgreement).

Security Concerns
======
Mathematically, to break the Diffie-Hellman key exchange protocol, that is, recovering the shared key K = g<sup>ab</sup> from public information G, g, g<sup>a</sup>, g<sup>b</sup>, the hacker must be able to solve the Discrete Logarithm (DL) problem, an NP-Complete problem. That would be computationally infeasible with classical computers. However, a naïve implementation of this protocol will be suffered by the following attacks.

Man-In-The-Middle Attack
------
The first possible attack against the naïve protocol is the Man-In-The-Middle attack. Mallory, an attacker, standing in the middle of Alice and Bob, is able to intercept messages between them. The attack works as follows:

- Mallory captures A = g<sup>a</sup>, sent from Alice to Bob, produces C = g<sup>m</sup>, with m chosen by himself, and then sends it to Bob. At the same time, he also sends this value C to Alice;
- Once receiving C = g<sup>m</sup>, Bob chooses a secret integer b, computes B = g<sup>b</sup>, and then sends B to Alice (but captured by Mallory);
- Once receiving C = g<sup>m</sup>, Alice computes the secret key K<sub>A</sub> = C<sup>a</sup> = g<sup>am</sup>, and starts communicating with Bob using that key;
- Bob computes the secret key K<sub>B</sub> = g<sup>bm</sup>, and starts communicating with Alice using that key;
- Mallory in the middle computes two keys K<sub>A</sub> = A<sup>m</sup> = g<sup>am</sup> and K<sub>B</sub> = B<sup>m</sup> = g<sup>bm</sup>;
- Mallory is now able to decrypt messages from both Alice and Bob, perform any modifications, re-encrypt, and forward them to the next party.

[Man-in-the-Middle attack](images/mitm.webp) (Source: Wikipedia)

Authenticated Diffie-Hellman Protocol
------
The above vulnerability is present because the Diffie-Hellman key exchange does not authenticate the participants. Possible solutions are to authenticate participants' public keys due to the use of digital signatures or public-key certificates issued by a Certificate Authority (CA). Roughly speaking, the basic idea is as follows. Prior to the execution of the protocol, the two parties Alice and Bob each obtain a public/private key pair and a certificate for the public key. During the protocol, Alice computes a signature on certain messages, covering the public value g^a. Bob proceeds in a similar way. Even though Mallory is still able to intercept messages between Alice and Bob, he cannot forge signatures without Alice's private key and Bob's private key. Hence, the enhanced protocol defeats the man-in-the-middle attack.