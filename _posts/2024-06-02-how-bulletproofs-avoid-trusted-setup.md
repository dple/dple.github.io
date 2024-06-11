---
title: 'How does Bulletproof ZKP avoid the trusted setup phase?'
date: 2024-06-02
permalink: /posts/2024/6/bulletproofs-without-trusted-setup/
tags:
  - Bulletproof
  - Trusted setup phase  
  - Zero-Knowledge Proof
  - zk-SNARK
---

When you are just entering into the ZKP realm, digging to zk-SNARKs protocols such as Groth16, Plonk, bulletproofs, etc, you may pose yourself questions as follows: 


1) `What is the trusted setup protocol?`
2) `What is universal trusted setup vs circuit-specific setup? ` 
3) `Why other ZKP protocols such as zk-STARK or Bulletproofs don't require a trusted setup phase?`

In the post ["What happened in the trusted setup phase of the Groth16 protocol?"](https://dple.github.io/posts/2024/4/trusted-setup-groth16/), I provided insights for the first two questions. In this post, I will try to explain why Bulletproofs protocol can work without the trusted setup and its differences from Groth16 and Plonk-KZG zk-SNARKs. 


## A brief recall of zk-SNARKs

To have a complete knowledge of ZKP and zk-SNARKs, I would recommend you to watch the [ZKP MOOC Lectures](https://youtu.be/A0oZVEXav24?si=gQYhw2BvpSCL-eTk)). Personally, I see that lecture 2 [Overview of Modern SNARK construction](https://youtu.be/bGEXYpt3sj0?si=u-PifZaH5K0JXoeI) and lecture 5 [The Plonk SNARK](https://youtu.be/A0oZVEXav24?si=IZ5EGMPZ5dvqF2Wp) provided a great introduction of modern zk-SNARKs. I will summarize below in a few lines about what I got from those lectures.

Give a `correctness of computation` problem (for example, after calculations, you know a solution of a complex system of equations, and want to prove it the someone that is correct without reaveal the actual solution), you first transform it to an `arithmetic circuit` consisting of multiplication and addition gates as shown in the below picture (source: [ZKP MOOC Lecture 5](https://youtu.be/A0oZVEXav24?si=IZ5EGMPZ5dvqF2Wp)). Then, an `arithmetization` process will convert those circuits to a set of mathematical equations (i.e., `constraint satisfaction problem`). 

<img src="http://dple.github.io/images/arithmetic-circuit.png" 
  alt="Arithmetic Circuit" 
  width="300" 
  height="300" 
  style="display: block; margin: 0 auto" />


If you have been zk-SNARKs long enough, you might be familiar with terms like R1CS, Plonkish arithmetization. Briefly speaking, they are two ways you can express algebraic circuits. In the next step, a tool such as QAP (`Quadratic Arithmetic Program`) will help you to transform that set of mathematical equations (or algebraic matrices) into polynomials. The reason why we use polynomials is that a big matrix of many values can be equivalently represented by a polynomial of pre-defined degree $d$ (often the size of circuits). 

Eventually, instead of committing your computation to a proof system, you just need to commit those polynomials. Which `polynomial commitment scheme` you use, is probably the *most important factor* deciding the efficiency of your ZKP system. In this [post](https://dple.github.io/posts/2024/4/kzg-poly-commitment/), I introduced the KZG polynomial commitment scheme. 

Based on underlying cryptographic primitives, modern polynomial commitment schemes can be classified as follows:

- Pairing: KZG and Dory schemes
- Discrete-log: Bulletproofs, Hydrax 
- Hash-function: zk-STARK

Finally, as mentioned in [ZKP MOOC Lecture 2](https://youtu.be/bGEXYpt3sj0?si=u-PifZaH5K0JXoeI), any polynomial IOP combines with a polynomial commitment scheme can form a public coin interactive argument (see below picture, source: [ZKP MOOC Lecture 5](https://youtu.be/A0oZVEXav24?si=IZ5EGMPZ5dvqF2Wp)), which can be converted to a non-interactive ZKP or zk-SNARK via the well-known [Fiat-Shamir transformation](https://link.springer.com/chapter/10.1007/3-540-47721-7_12). Furthermore, a SNARK protocol requires the *proof* to be *succinct* and the verification should be much more quickly than the computation itself. For instance, The original Plonk ZKP is a combination of Plonk IOP and the KZG polynomial commitment. [Halo 2](https://zcash.github.io/halo2/) ZKP system combines Plonk IOP and Bulletproofs-based polynomial commitment. 

<img src="http://dple.github.io/images/IOP+Poly-commitment.png" 
  alt="Arithmetic Circuit" 
  width="450" 
  height="300" 
  style="display: block; margin: 0 auto" />


## What is Bulletproofs?
The Bulletproofs protocol was introduced by Benedikt Bünz, Jonathan Bootle, Dan Boneh, Andrew Poelstra, Pieter Wuille, and Greg Maxwell in their [paper](https://web.stanford.edu/~buenz/pubs/bulletproofs.pdf) published in 2017. Unlike some other zk-SNARKs, Bulletproofs do not require a trusted setup, i.e., `transparent setup`. This means there is no need for a preliminary phase where a trusted party generates certain parameters that, if compromised, could undermine the security of the entire system. The absence of a trusted setup increases the security and trustworthiness of Bulletproofs.

Bulletproofs was originally known as a range proof protocol (proving that a value $v$ lies within a certain range without revealing the value itself, i.e., $0 \le v \le 2^n$), it was then adapted to an efficient zero-knowledge argument for arbitrary `arithmetic circuits`, a.k.a, any computations. 

Bulletproofs protocol produces a proof of size $\mathcal{O}_{\lambda}(log |C|)$ and verification time asymtotically requires $\mathcal{O}_{\lambda}(|C|)$, where $C$ is the size of gates of the computation. These asymtotic complexities are actually pretty big compared to other zk-SNARKs protocols such as [Groth16](https://eprint.iacr.org/2016/260.pdf) or [Plonk-KZG](https://eprint.iacr.org/2019/953), in which those values are $\mathcal{O}_{\lambda}(1)$. 



## Why Bulletproofs don't require trusted setup?

We will discuss in the following how Bulletproofs are used to generate a general ZK circuit rather than a range proof. As mentioned earlier, if Bulletproofs could be used as a polynomial commitment, then it can be combined with a polynomial IOP such as Plonk to form a modern SNARK system. 

Let's commit a polynomial $f(X) = f_0 + f_1 x + \cdots + f_d x^d \in \mathbb{F}_p(X)$. The Bulletproofs-based commitment is $com_f = g_0^{f_0} g_1^{f_1}  \cdots g_d^{f_d}$, where $g_i \in \mathbf{G}$. It is form of [Pedersen vector commitment](https://zcash.github.io/halo2/background/groups.html). $com_f$ hence can be rewritten as $com_f = \mathbf{g}^{\mathbf{f}}$, where $\mathbf{g} = <g_0, g_1, ..., g_d>$, and $\mathbf{f} = <f_0, f_1, ..., f_d>$ are two vectors of group elements of $\mathbf{G}$ and elements in finite field $\mathbb{F}_p$, respectively. 

The below table compares how a polynomial will be committed using KZG and Bulletproofs protocols. 

| | KZG poly-commitment | Bulletproof poly-commitment|
| ------ | --------------------- | --------------------- | 
| | $\bf{G}_1,\bf{G}_2$ groups of EC points (same `pairing-friendly` ellitpic curve shape, but in defined in two different finite fields)|  $\bf{G}$ group of points of a (more generic) elliptic curve | 
| Setup | $g_1, g_2$, corresponding generators of $\bf{G}_1,\bf{G}_2$ | $g_0, g_1, \ldots, g_d \in \bf G$, arbitrary EC points |
|  | $e$, a cryptographic pairing, and $\tau \in \mathbb{F}_p$, a secret or `toxic waste`  | |
| CRS | $g_1, g_1^{\tau}, \ldots, g_1^{\tau^d}$, and $g_2^{\tau}$ | $g_i$ for $i \in [0, d]$ |
| Commitment | $com_f = g_1^{f(\tau)}$  | $\mathbf{g}^{\mathbf{f}}$  |
|Evaluation | | |
| Proof | $\pi = g_1^{q(\tau)}$ | |
| Verification | $e(com_f/g_1^{f(u)}, g_2) = e(\pi, g_2^{\tau}/g_2^{u})$ | $com_{f_{i+1}} = L_i^{r_i^2} \cdot com_{f_i} \cdot R_i^{r_i^{-2}} $ |



KZG poly-commitment proves that you know $f(x)$ and $q(x)$ such that $q(x) = \frac{f(x) - f(u)}{(x - u)}$. The remainder of $\frac{f(x) - f(u)}{(x - u)}$ should be $0$ as $u$ is a root of the polynomial $f(X) - f(u)$.  

As you can see in the KZG poly-commitment, the prover must compute the quotient poly $q(x)$. Thus, this technique is also called `quotient argument opening`.