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

So, we are concerned about our privacy with questions like: What is my data collected? How do they use my data? Are they going to share/sell my data to a 3rd party like what Facebook did to their users [1]? How my data is protected? etc.

From service providers, they certainly want to maximize the benefits from data they collected. For example, sharing/selling to their partners, doing analysis to draw insights 

From your side, you don't want to be a part of the data that service providers are utilizing. For example, if Facebook is selling their users' data, you hope that data does not contain yours. Whatever output drawn from the analysis won't reveal any things about you. That is the *core question* Differential Privacy is trying to resolve. 

## What is Differential Privacy ?

Differential Privacy (DP) is considered a promising approach to balance between protecting the individuals' privacy and leveraging data for insightful analysis. The formal concept $epsilon$-differential privacy was introduced by Dwork, McSherry, Nissim and Smith in 2006 [2] as followed:

> A randomized algorithm $\mathcal{M}$ with domain $\mathbb{N}^{\chi}$ is $(\epsilon, \delta)$-differentially private if for all $S \subset Range(\mathcal{M})$ and for all $D_1, D_2 \in \mathbb{N}^{\chi}$ such that $\| D_1 - D_2 \|_1 \le 1:$
> <div align="center"> $Pr[\mathcal{M}(D_1) \in \mathcal{S}] \le exp(\epsilon) Pr[\mathcal{M}(D_2) \in \mathcal{S}] + \delta$ </div>


### What are notions meaning?
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


In this example, 
- $\mathbb{N}^{\chi}$ can be seen as the entire dataset of all nine (9) employees
- $D_1$ and $D_2$ could be two datasets, which differ by only a single elemnet. For example $D_1$ is the set of first 8 employees and $D_2$ is the set of all 9 employees.



## References
[1] Wikipedia, [Facebook–Cambridge Analytica data scandal](https://en.wikipedia.org/wiki/Facebook%E2%80%93Cambridge_Analytica_data_scandal)
