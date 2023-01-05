---
title: "Improved algebraic attacks on lightweight block ciphers"
collection: publications
permalink: /publication/2020-11-07-lightweight
excerpt: 'This paper proposes improved algebraic attacks that are effective for lightweight block ciphers.'
date: 2020-11-07
venue: 'Journal of Cryptographic Engineering'
paperurl: 'https://link.springer.com/article/10.1007/s13389-020-00237-4'
citation: 'Sze Ling Yeo, Duc-Phong Le, Khoongming Khoo. (2010). &quot;Improved algebraic attacks on lightweight block ciphers.&quot; <i>Journal of Cryptographic Engineering</i>. 11(1).'
---
This paper proposes improved algebraic attacks that are effective for lightweight block ciphers. Concretely, we propose a new framework that leverages on algebraic preprocessing as well as modern SAT solvers to perform algebraic cryptanalysis on block ciphers. By combining with chosen plaintext attacks, we show that our framework can be applied to lightweight block ciphers that exhibit a nice differential trail. In particular, we demonstrate our techniques by performing algebraic cryptanalysis on both the Present cipher and the Simon cipher. For the Present cipher, we successfully solved up to 9 rounds with at most 32 key bits fixed and 8 chosen plaintexts. On the other hand, for the Simon cipher, we tested our method on Simon-32/64 and Simon-64/128. For these two versions, our attack can solve up to 13 rounds with only 8 chosen plaintexts by fixing 4 and 6 key bits for Simon-32/64 and Simon-64/128, respectively. Further, by considering a class of weak keys, we can extend our attacks to 16 rounds. As far as we are aware, these are the best algebraic attacks on these ciphers in the literature.

[Download paper here](https://dple.github.io/files/lightweight.pdf)

**Recommended citation**: Sze Ling Yeo, Duc-Phong Le, Khoongming Khoo. (2010). "Improved algebraic attacks on lightweight block ciphers." <i>Journal of Cryptographic Engineering</i>. 11(1).