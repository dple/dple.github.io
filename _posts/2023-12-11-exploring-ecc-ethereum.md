---
title: 'Exploring Elliptic Curve Cryptography in Ethereum Cryptography library'
date: 2023-12-11
permalink: /posts/2023/12/exploring-ecc-ethereum/
tags:
  - Ellitpic curve cryptography
  - Ethereum 
  - secp256k1
  - ed25519
  - ed448
  - Node js
---

Cryptography plays a crucial role in the world of blockchain and cryptocurrencies. It provides the foundational security elements that make decentralized and trustless systems possible. Basic cryptographic primitives such as [hash function](https://en.wikipedia.org/wiki/Hash_function), [Merkle tree](https://en.wikipedia.org/wiki/Merkle_tree), [digital signatures](https://en.wikipedia.org/wiki/Digital_signature), and consensus mechanisms (e.g. [Proof of work](https://en.wikipedia.org/wiki/Proof_of_work)) ensures secure transactions, preventing counterfeiting and [double-spending](https://en.wikipedia.org/wiki/Double-spending) in cryptocurrencies. They also ensure the integrity of the entire system due to collision-free property of hash functions. 

In this tutorial, we will be the [Js-Ethereum-Cryptography library](https://github.com/ethereum/js-ethereum-cryptography) in order to explore Elliptic Curve Cryptography (ECC) to:

- generate a private key, 
- derive a public key from a given private key
- convert the public key into its corresponding Ethereum addresss
- generate and verify a ECDSA signature 
- recover public key and Ethereum address from signature and digest's message

Many online resources cover ECC, for example,[cloudfare's ECC](https://blog.cloudflare.com/a-relatively-easy-to-understand-primer-on-elliptic-curve-cryptography/) from a mathematical perspective, or [geeks-for-geeks](https://www.geeksforgeeks.org/blockchain-elliptic-curve-cryptography/) for its applications in blockchain, but don't offer applicable code examples for our needs on Ethereum blockchain as a Web3 programmer. 

## What is Elliptic Curve Cryptography

I start with a brief introduction of Elliptic curve cryptography. If you want to have more details, please refer to the resources mentioned above. 

![Elliptic Curve Examples](https://upload.wikimedia.org/wikipedia/commons/d/d0/ECClines-3.svg)

Elliptic Curve Cryptography (ECC) is a branch of public key cryptography whose computational assumption is based on the difficulty of dicrete logarithm problem over a group of points defined on elliptic curve $E$ over a finite field $F_p$. The Weierstrass form of elliptic curves is as follows:

$$y^2 = x^3 + Ax + B$$

Given two points $P$ and $Q$ on $E$, the addition group law over elliptic curves is depicted in the below picture:

![Elliptic Curves' Group Law](https://upload.wikimedia.org/wikipedia/commons/c/c1/ECClines.svg)

Group operations could be performed in the [affine coordinate system](https://en.wikipedia.org/wiki/Affine_space) (a.k.a, two dimentions $x$ and $y$) or a [projective coordinate system](https://en.wikipedia.org/wiki/Projective_plane) (a.k.a, 3 dimentions with $X$, $Y$, and $Z$), which is more efficient as it don't require any finite field inversion operations. 

Given a private key $d$ (a big random number in $F_p$), its corresponding public key $PK$ (a point on the elliptic curve $E$), will be derived by raising the elliptic curve's generator $G$ by a scalar $s$, that is, $PK = d \times G$. This is a *one-way function*, that is, it is easy to calculate a public key from a private key, but it is *infeasible* to revert the private key, given its corresponding public key.  

As ECC provides a smaller key size compared to RSA for the same security level (e.g., the curve secp256k1 of 256-bits offers the equivalent security as the RSA-2048 bits), it is widely used in various applications, including digital signatures (i.e., ECDSA), secure key exchange protocols (i.e., ECDH), and more. Many blockchain networks, including Bitcoin and Ethereum, use elliptic curve cryptography for key management, digital signatures, and securing transactions.

### Curve secp256k1

The curve *secp256k1* is defined in Standards for Efficient Cryptography (SEC) [Certicom Research](https://www.secg.org/sec2-v2.pdf). Bitcoin and Ethereum use this curve for key generation and in the digital signature algorithm [ECDSA](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm). This is a Koblitz curve, that is a curve of form $y^2 = x^3 + B$, where $Bb$ is a small number. This special secp256k1 curve offers better performance in point additions and scalar multiplications. In particular, secp256k1 has the following parameters:

- $p$ = FFFFFFFF FFFFFFFF FFFFFFFF FFFFFFFF FFFFFFFF FFFFFFFF FFFFFFFE FFFFFC2F 
      = $2^{256} - 2^{32} - 2^9 - 2^8 - 2^7 - 2^6 - 2^4 - 1$
- $A = 0$
- $B = 7$
- Generator (compressed form): $G$ = 02 79BE667E F9DCBBAC 55A06295 CE870B07 029BFCDB 2DCE28D9 59F2815B 16F81798
- Order of $G$: $n$ = FFFFFFFF FFFFFFFF FFFFFFFF FFFFFFFE BAAEDCE6 AF48A03B BFD25E8C D0364141
- Co-factor: $h = 1$.

## What is Js-Ethereum-Cryptography?

Ethereum Cryptography is a library implemented in Javascript. It provides all Ethereum-related cryptographic primitives, including:

- **Hash functions**: SHA256, keccak-256, RIPEMD160, BLAKE2b
- **Key Derivation Functions** (KDFs): PBKDF2, Scrypt, CSPRNG (Cryptographically Secure Pseudorandom Number Generator)
- **Elliptic Curves**: such as secp256k1 elliptic curve
- **BIP32 HD Keygen**: Hierarchical deterministic (HD) wallets that conform to BIP32 standard
- **BIP39 Mnemonic phrases**: the implementation of a mnemonic code or mnemonic sentence -- a group of easy to remember words -- for the generation of deterministic wallets
- **AES Encryption**: symmetric encryption and decryption functions with different modes (i.e., counter *aes-128-ctr* or CBC *aes-128-cbc*, etc.) and and the pkcs7 padding scheme.

For more information, I recommend checking out the library's [github repo](https://github.com/ethereum/js-ethereum-cryptography). 


### Supported functions in secp256k1

The following functions are used to generate public key, generate and verify digital signatures:

```javascript
function getPublicKey(privateKey: Uint8Array, isCompressed = true): Uint8Array;
function sign(msgHash: Uint8Array, privateKey: Uint8Array): { r: bigint; s: bigint; recovery: number };
function verify(signature: Uint8Array, msgHash: Uint8Array, publicKey: Uint8Array): boolean
function getSharedSecret(privateKeyA: Uint8Array, publicKeyB: Uint8Array): Uint8Array;
function utils.randomPrivateKey(): Uint8Array;
function recoverPublicKey(msgHash: Uint8Array): Uint8Array;
```

In addition, to generate an Ethereum address, you must use Keccak hash function.


## Let's jump into the code

Get enough with the boring theory, let's see how ECC is used in Ethereum blockchain. 

Since the library comes in the form of a node module, we need to [install node.js](https://nodejs.org/en/learn/getting-started/how-to-install-nodejs). Then, install the library:

*npm install ethereum-cryptography*

Let create a project folder:

*mkdir ecc-in-ethereum* && *cd ecc-in-ethereum*

Then, create a new javascript source file:

*touch ecc.js*

In order to use this curve in the library, you must import the secp256k1 submodule from Ethereum-Cryptography library:

```javascript
const { secp256k1 } = require("ethereum-cryptography/secp256k1");
```

You also have to import the following functions in your code:

```javascript
const { utf8ToBytes, bytesToHex, hexToBytes } = require("ethereum-cryptography/utils");     // to convert message in different formats
const { sha256 } = require("ethereum-cryptography/sha256");     // to generate a cryptographic digest of a message
const { keccak256 } = require("ethereum-cryptography/keccak");     // to generate Ethereum address
const { getRandomBytes } = require("ethereum-cryptography/random");     // to generate a random private key using CSPRN method
```

Well, now you have all required tools to work with ECC in Ethereum. In this tutorial, I am using the CSPRN method to generate a private key. The other two methods that could be used to derive a private key supported in Ethereum-Cryptography are *pbkdf2* and *scrypt*. Samples code for these methods are on my [github](https://github.com/dple/crypto-in-ethereum/blob/main/kdf.js). 

On curve *secp256k1*, the size of a private key is $32$ bytes or $256$ bits. 

```javascript
const keylen = 32;      // Key length = 32 bytes
// Get a random private key using CSRPN
const privateKey = bytesToHex(await getRandomBytes(keylen)); 
```

After having a private key, you can generate its corresponding public key:

```javascript
// Generate public key from private key. Produce 33-byte compressed public key by default
const publicKey = secp256k1.getPublicKey(privateKey);   
```

By default, this returns you $33$ bytes compressed public key. If you want to get an uncompressed public key of $65$ bytes, you have to set argument *isCompressed* to *false*:

```javascript
// Produce 65-byte uncompressed public key
const publicKey = secp256k1.getPublicKey(privateKey, false);   
```

After having a public, you can generate its corresponding Ethereum address by taking the last 20 bytes of the hash digest of public by using Keccak hash function:

```javascript
// Generate Ethereum wallet address from public key => last 20 of public key's hash
const ethAddress = bytesToHex(await keccak256(publicKey));
const ethAddressWithPrefix = "0x" + ethAddress.slice(-20);
```



Ok, you had now all neccessary stuffs to generate a signature. Let sign this message "Web3 is Awesome":

```javascript
const messageHash = bytesToHex(await sha256(utf8ToBytes("Web3 is Awesome")));   // get message's digest

const sig = await secp256k1.sign(messageHash, privateKey);  // Signature generation
```

The message is a string in UFT8 format, you thus must convert it to bytes in order to feed it to the hash function *sha256*. Now, you have the signature for the message "Web3 is Awesome". Let verify if the signature was correctly generated:

```javascript
const verified = secp256k1.verify(sig, messageHash, publicKey);         // Signature verification 
console.log("Verified", verified);
```

You should see *true* in the output screen. If not, the signature generation is incorrect.

Once your transaction was successfully recorded on a blockchain platform, you can use a scanner, for example [Etherscan](https://etherscan.io/) for Ethereum transactions, to track it. Only transaction hash and signature are recorded on the blockchain. For a full transaction information, you must recover the signer's public key and signer's Ethereum address. 

```javascript
// Receover public key
publicKeyRecovered = sig.recoverPublicKey(messageHash).toHex(true);     
if (bytesToHex(publicKey) == publicKeyRecovered) 
    console.log("Public key recovered sucessfully!");

// Get signer's Ethereum address
const ethAddressRecovered = bytesToHex(await keccak256(hexToBytes(publicKeyRecovered)));
const ethAddressRecoveredWithPrefix = "0x" + ethAddressRecovered.slice(-20);

if (ethAddressRecoveredWithPrefix == ethAddressWithPrefix) 
    console.log("Signer's Ethereum address was sucessfully regenerated!");
```

The full code is on this [github repo](https://github.com/dple/crypto-in-ethereum/blob/main/challenge.js)

## Closing
This tutorial covered the basic functions of elliptic curve cryptography in the Js-Ethereum-Cryptography library. Hope that useful for you!