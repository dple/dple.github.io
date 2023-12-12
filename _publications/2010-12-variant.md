---
title: "A Variant of Miller Algorithm and its Formula"
collection: publications
permalink: /publication/2010-12-15-variant-miller
excerpt: ' In this paper, we revisit the double exponentiation countermeasure and propose faster methods to perform a double exponentiation. On the one hand, we present new heuristics for generating shorter double addition chains. On the other hand, we present an efficient double exponentiation algorithm based on a right-to-left sliding window approach.'
date: 2014-02-28
venue: 'CT-RSA'
paperurl: 'https://eprint.iacr.org/2015/657.pdf'
citation: 'Duc-Phong Le, Matthieu Rivain, Chik How Tan. (2014). &quot;On Double Exponentiation for Securing RSA against Fault Analysis.&quot; <i>CT-RSA 2014</i>.'
---
At CT-RSA 2009, a new principle to secure RSA (and modular/group exponentiation) against fault-analysis has been introduced by Rivain. The idea is to perform a so-called double exponentiation to compute a pair ($m^d, m^{\phi(N) − d}$) and then check that the output pair satisfies the consistency relation: $m^d, m^{\phi(N) − d} \equiv 1 \bmod N$. The author then proposed an efficient heuristic to derive an addition chain for the pair $(d, \phi(N) − d)$. In this paper, we revisit this idea and propose faster methods to perform a double exponentiation. On the one hand, we present new heuristics for generating shorter double addition chains. On the other hand, we present an efficient double exponentiation algorithm based on a right-to-left sliding window approach.

[Download paper here](https://dple.github.io/files/double-exponentiation.pdf)

**Recommended citation**: Duc-Phong Le, Matthieu Rivain, Chik How Tan. (2014). *On Double Exponentiation for Securing RSA against Fault Analysis*. <i>CT-RSA 2014</i>.