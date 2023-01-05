---
title: "Algebraic Differential Fault Analysis on SIMON Block Cipher"
collection: publications
permalink: /publication/2019-11-01-simon
excerpt: 'This paper presents three attacks using three different algebraic techniques combined with a differential fault attack in the bit-flip fault model to break the SIMON block cipher.'
date: 2019-11-01
venue: 'IEEE Transactions on Computers'
paperurl: 'https://ieeexplore.ieee.org/abstract/document/8751983'
citation: 'Duc-Phong, Le; Sze-Ling Yeo; Khoongming, Khoo. (2019). &quot;Algebraic Differential Fault Analysis on SIMON Block Cipher.&quot; <i>IEEE Transactions on Computers</i>. 68(11).'
---
Algebraic differential fault attack (ADFA) is an attack in which an attacker combines a differential fault attack and an algebraic technique to break a targeted cipher. In this paper, we present three attacks using three different algebraic techniques combined with a differential fault attack in the bit-flip fault model to break the SIMON block cipher. First, we introduce a new analytic method which is based on a differential trail between the correct and faulty ciphertexts. This method is able to recover the entire master key of any member of the SIMON family by injecting faults into a single round of the cipher. In our second attack, we present a simplified Grobner basis algorithm to solve the faulty system. We show that this method could totally break SIMON ciphers with only 3 to 5 faults injected. Our third attack combines a fault attack with a modern SAT solver. By guessing some key bits and with only a single fault injected at the round T - 6, where T is the number of rounds of a SIMON cipher, this combined attack could manage to recover a master key of the cipher. For the last two attacks, we perform experiments to demonstrate the effectiveness of our attacks. These experiments are implemented on personal computers and run in very reasonable timing.

[Download paper here](https://dple.github.io/files/simon.pdf)

**Recommended citation**: Duc-Phong, Le; Sze-Ling Yeo; Khoongming, Khoo. (2019). "Algebraic Differential Fault Analysis on SIMON Block Cipher" <i>IEEE Transactions on Computers</i>. 68(11)