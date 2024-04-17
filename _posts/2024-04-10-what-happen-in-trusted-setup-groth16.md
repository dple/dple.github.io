---
title: 'What happens in the trusted setup phase of the Groth16 protocol'
date: 2024-02-11
permalink: /posts/2024/4/trusted-setup-groth16/
tags:
  - Groth16
  - Trusted setup
  - tau cerenomy  
  - Zero-Knowledge Proof
  - zk-SNARK
  - zk-STARK
  - Circom
---

## The Groth16 Protocol
Groth16, probably the most wide-used zk-SNARK (Zero-Knowledge Succinct Non-Interactive Arguments of Knowledge) on blockchains, was introduced by Jens Groth at Eurocrypt 2016 (*that's why it is named groth16*). The paper, entitled `On the Size of Pairing-Based Non-interactive Arguments`, could be on [eprint/2016/260](https://eprint.iacr.org/2016/260.pdf).

Groth16 is based on R1CS (*Rank One Constraint Systems*) arithmetization. While its proof consists of only 3 elliptic curve poinsts, defined in the base filed $G_1$, the verification is also fast with only 3 pairing computations. Think if you implement the protocol with the [BN curve](https://eprint.iacr.org/2005/133.pdf) for $128$-bit security level, the total size of 3 points in the proofs is about $200$ bytes ($3 \times 2 \times 254$ bits, each point represented by two 254-bit finite field elements). 
The below table compares it with other zk-SNARKs. 

|        | Setup | Size of proof | Proving time | Verifier time | Post-quantum |
| ------ | --------------------- | ------------ | ------------- | ------------ |
| Groth16 | ~ 200 bytes |       | ~ 1.5 ms | trusted per circuit (1) | No | 
| Plonk | ~ 400 bytes |       | ~ 3 ms | universal trusted setup (2) | No | 
| Bulletproofs | ~ 1.5 Kbytes |       | ~ 3 s | transparent (3) | No | 
| zk-STARK | ~ 100 Kbytes |       | ~ 10 ms | transparent |  | 

(1) Trusted per circuit requires the setup phase to run for every individual circuits, i.e., generating a Common Reference String (CRS) for one circuit; (2) Universal trusted setup runs only one time for all circuits; (3) Transparent requires nothing to setup.  

The zk-SNARK protocols often require a trusted setup in order to generate a CRS (`Common Reference String`), proving and verification keys for prover and verifier,respectively. Compared to other protocol, Groth16 protocol requires trusted setup per circuit, that means, if you implement a new circuit, you must execute another trusted setup. Otherwise it offers `smaller proof size` and `faster verification`, and these features probably makes this protocol more popular as it will consume `less gas cost` on Ethereum blockchain. Groth16 is suitable for applications generating many proofs for the same circuit and performance is crucial. 


This article will discuss about the two setup phases of the Groth16 protocol. 

## Gorth16 Setup Protocol

The trusted setup of the Groth16 protocol could be found in the page #17 of his paper. It is as the below picture:

<img src="http://dple.github.io/images/groth16setup.png" 
  alt="Trusted setup in the Groth16 protocol" 
  width="1000" 
  height="300" 
  style="display: block; margin: 0 auto" />


### Universal Setup

Assume that you are working with an elliptic curve $E$, and its generator $G$, i.e., $[p]G = \bf 1$.  This phase of setup will generate a random number $\tau$ and its powers $[\tau^0]G, [\tau^1]G, [\tau^2]G, ..., [\tau^d]G$. These values could be used for any circuits. 

```python
def generate_powers_of_tau(tau, degree, point):
  return [multiply(point, int(tau ** i)) for i in range(degree + 1)]

"""
  Trusted setup
    Phase 1: setup for all circuits
"""
# 1. Get a random value tau
tau = random.randint(1, p)  
# 2. Calculate tau*G1, tauˆ2*G1, ..., tauˆd*G1
powers_of_tau_for_G1 = generate_powers_of_tau(tau, d, G1)
# Calculate tau*G2, tauˆ2*G2, ..., tauˆd*G2
powers_of_tau_for_G2 = generate_powers_of_tau(tau, d, G2)
```

We need to generate powers of tau of both point $G_1$ and $G_2$. The former is a generator in the base field F_p and the later is the generator in the extension field $F_p^{k}$, where $k$ is the *embedding degree* of the elliptic curve $E$. You must be familiar with *pairing-friendly elliptic curves* to understand that concept and to know why we need powers of tau in both fields. A complete introduction of such curves could be found in the paper [A taxonomy of pairing-friendly elliptic curves](https://eprint.iacr.org/2006/372) by Freeman, Scott, and Teske. 

<ins>**Important**</ins>: $\tau$ must be discarded after this setup phase to prevent dishonest provers from iventing fake ZK proofs without using the knowledge about the witness. Given powers of tau $[\tau]G_i, [\tau^2]G_i, ..., [\tau^d]G_i$, for $i = 1, 2$, an adversary could not recover the value of $\tau$ unless he could solve the [discrete logarithm problem](https://en.wikipedia.org/wiki/Discrete_logarithm), believed a NP-Complete problem, and hence *infeasible* to solve with the classical computers. 


### Circuit-Specific Setup
At this phase, assume you are working on a computation problem and converted it into a R1CS system that can be encoded as:

$$Lw \cdot Rw = Ow,$$

where $L, R, O$ are matrices with $n$ rows (equal to the number of constraints) and $m$ columns (equal to the length of the witness), and $w$ is the witness vector. To understand *step-by-step* how to convert a computation problem to a R1CS system, the best online resource I found is [The Rareskills Book of Zero Knowledge](https://www.rareskills.io/zk-book). 


At the phase 2 setup per circuit, the trusted server generates random value $\alpha, \beta$ that will be used to prevent a potential cheating prover from making up three curve points of proof $A, B$, and $C$:


```python
alpha = random.randint(1, p)
beta = random.randint(1, p)
alpha_G1 = multiply(G1, int(alpha))     # alpha*G1
beta_G1 = multiply(G1, int(beta))       # beta*G1
beta_G2 = multiply(G2, int(beta))       # beta*G2
```

Trusted server also generates random $\gamma$ and $\delta$ to prevent dishonest prover from forgeries with public inputs to generate a false proof:

```python
gamma = random.randint(1, p)
gamma_G2 = multiply(G2, int(gamma))  # gamma*G2
gamma_inv = 1 / GFp(gamma)     # should be defined in a finite field GF(p)     
delta = random.randint(1, p)
delta_G1 = multiply(G1, int(delta))  # delta*G1
delta_G2 = multiply(G2, int(delta))  # delta*G2
delta_inv = 1 / GFp(delta)
```

Finally, trusted server compute powers of $\tau$ for public and private inputs

```python
"""
Given three matrices L, R and O, generate powers of tau for public and private input, 
i,e., C = beta*U + alpha*V + W
    @:param d: degree of the polynomials formed from columns in the matrices, d = n - 1, where n = #rows
    @:param m: #columns of the matrices = len(witness)

    @:return two list of powers of tau (elliptic curve points):
     - one related to public inputs (i.e., first l values in the witness vector) will be for verifier
     - one related to private inputs (i.e, last m - l values in the witness vector) will be for prover
"""
def generate_powers_of_tau_for_inputs(powers_of_tau, d, m, L_mat, R_mat, O_mat, alpha, beta, ell, gamma_inv, delta_inv):
  taus_for_public_inputs = []
  taus_for_private_inputs = []
  # require degree + 1 points to interpolate a polynomial of degree d
  xs = Fp(np.array([i + 1 for i in range(d + 1)]))
  # Each i-th col of matrices L, R, and W will be converted to a polynomial U_i(x), V_i(x) and W_i(x)
  for i in range(m):
    # U_i(x) = interpolate the i-th column of matrix L
    poly_Ui = galois.lagrange_poly(xs, L_mat[:, i])
    # Perform a random shift by multiplying the poly with a random factor beta
    beta_Ui = poly_Ui * beta        # multiply U with beta
    # V_i(x) = interpolate the i-th column of matrix R
    poly_Vi = galois.lagrange_poly(xs, R_mat[:, i])
    # Perform a random shift by multiplying the poly with a random factor alpha
    alpha_Vi = poly_Vi * alpha
    # W_i(x) = interpolate the i-th column of matrix W
    poly_Wi = galois.lagrange_poly(xs, O_mat[:, i])

    sum_poly = beta_Ui + alpha_Vi + poly_Wi

    if i < ell:
      taus_for_public_inputs.append(inner_product(powers_of_tau, (sum_poly.coeffs[::-1]) * gamma_inv))
    else:
      taus_for_private_inputs.append(inner_product(powers_of_tau, (sum_poly.coeffs[::-1]) * delta_inv))

  return taus_for_public_inputs, taus_for_private_inputs

taus_for_public_inputs, taus_for_private_inputs = \
        generate_powers_of_tau_for_inputs(powers_of_tau_for_G1, d, m, L, R, O, alpha, beta, ell, gamma_inv, delta_inv)

```


## Trusted Setup in practical ZKP systems

As mentioned above, the value of $\tau$ must be kept secret, it thus is risky if letting only a single entity to generate it. Practical ZKP systems ussually implement a MPC (Multi-Party Computation) protocol for [powers of tau cerenomy](https://medium.com/coinmonks/announcing-the-perpetual-powers-of-tau-ceremony-to-benefit-all-zk-snark-projects-c3da86af8377).  


### Groth16 setup by snarkjs

Let's see how `snarkjs` perform the setup phase:

You may need to install *snarkjs* first

```sh
npm install -g snarkjs@latest
```

```sh
snarkjs powersoftau new bn128 12 ../../ptau/pot12_0000.ptau -v
```

```sh
snarkjs groth16 setup circuit.r1cs final.ptau circuit.zkey
```

### 


## Trusted Setup in other ZKP system

### Zokrates



### 