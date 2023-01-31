---
title: 'Differential Privacy: An illustrated primer'
date: 2023-01-12
permalink: /posts/2023/01/Differential-Privacy-an-illustrated-primer/
tags:
  - Data sharing
  - Data analysis
  - Data privacy 
  - Differential Privacy
  - Laplace noise
  - White noise
---

Nowadays, many things you can do online, some can be named as browsing news, shopping, socializing, registering services, attending events and courses, visiting healthcare clinic, etc. Your life seems getting more and more convenient. However, the trade-off is that more and more your personal information is collected and used/shared over online platforms. You is becoming more *visible* on Internet, and hence getting higher risk to be a victime of online fraudulent activities. 

So, we are concerned about our privacy with questions like: What is my data collected? How do they use my data? Are they going to share/sell my data to a 3rd party like what Facebook did to their users [1]? How my data is protected? etc. From service providers, they certainly want to maximize the benefits from data they collected. For example, sharing/selling to their partners, doing analysis to draw insights. From your side, you don't want to be a part of the data that service providers are utilizing. For example, if Facebook is selling their users' data, you hope that data does not contain yours. Whatever output drawn from the analysis won't reveal any things about you. 
That is the *core question* Differential Privacy is trying to resolve.

To keep both sides achieving their goals, privacy-enhancing techniques (PET) could be used. They could be either cryptographic solutions such as [zero-knowledge proofs](https://en.wikipedia.org/wiki/Zero-knowledge_proof), [secure multi-party computation (SMPC)](https://en.wikipedia.org/wiki/Secure_multi-party_computation), [fully homomorphic encryption (FHE)](https://en.wikipedia.org/wiki/Homomorphic_encryption), etc., or non-cryptographic solutions such as [data anonymization](https://en.wikipedia.org/wiki/Data_anonymization), [synthetic data](https://en.wikipedia.org/wiki/Synthetic_data), [differential privacy](https://en.wikipedia.org/wiki/Differential_privacy), etc. In which, Differential Privacy (DP) is a special tool that strikes a balance between your and service provider platforms' goals. 

In this post, I am going try to provide a (mostly) *non-mathematical* description of what differential privacy is and what makes it so promissing. 
 
## What is Differential Privacy ?

Differential Privacy (DP) is considered a promising approach to balance between protecting the individuals' privacy and leveraging data for insightful analysis. The formal concept $epsilon$-differential privacy was introduced in 2006 by Microsoft researchers Cynthia Dwork, Frank McSherry, Kobbi Nissim, and Adam Smith [2] as followed:

> A randomized algorithm $\mathcal{A}$ with domain $\mathbb{N}^{\chi}$ is $(\epsilon, \delta)$-differentially private if for all subsets $S$ of  $\mathrm{im} \; \mathcal{A})$ and for all $D_1, D_2 \in \mathbb{N}^{\chi}$ such that $\| D_1 - D_2 \|_1 \le 1:$
> <div align="center"> $\text{Pr}[\mathcal{A}(D_1) \in \mathcal{S}] \le exp(\epsilon) \cdot \text{Pr}[\mathcal{A}(D_2) \in \mathcal{S}], </div>

where $\epsilon$ is a positive real number and $\mathrm{im}$ is the image of $\mathcal{A}$. The probability is taken over the randomness used by the algorithm. 

### Interpretation
Let consider a dataset of information of employees and their salaries in a company as in the below table

|  Name                   |  Salary     |
|  ---------------------- |  ---------  |
|  Alice                  |  $90,000    |
|  Bob                    |  $125,000   |
|  Charlie                |  $80,000    |
|  Dragon                 |  $95,000    |
|  Eve                    |  $110,000   |
|  Floren                 |  $70,000    |
|  Geoffery               |  $65,000    |
|  Huff                   |  $100,000   |
|  Kevin                  |  $150,000   |
|  ...                    |  ...        |
|  Zayden                 |  $130,000   |


In this example, 
- $\mathbb{N}^{\chi}$ can be seen as the entire dataset of all nine (9) employees
- $D_1$ and $D_2$ could be two datasets, which differ by only a single elemnet. For example $D_1$ is the set of first 8 employees and $D_2$ is the set of all 9 employees.



## References
[1] Wikipedia, [Facebook–Cambridge Analytica data scandal](https://en.wikipedia.org/wiki/Facebook%E2%80%93Cambridge_Analytica_data_scandal)
[2] Cynthia Dwork, Frank McSherry, Kobbi Nissim, and Adam Smith, *Calibrating noise to sensitivity in private data analysis*. In Proceedings of the Third conference on Theory of Cryptography (TCC'06).
