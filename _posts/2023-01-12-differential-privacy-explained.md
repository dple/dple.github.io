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

So, we are concerned about our privacy with questions like: What is my data collected? How do they use my data? Are they going to share/sell my data to a 3rd party like what Facebook did to their users? How my data is protected? etc.

From service providers, they certainly want to maximize the benefits from data they collected. For example, sharing/selling to their partners, doing analysis to draw insights 

From your side, you don't want to be a part of the data that service providers are utilizing. For example, if Facebook is selling their users' data, you hope that data does not contain yours. Whatever output drawn from the analysis won't reveal any things about you. That is the *core question* Differential Privacy is trying to resolve. 

## What is Differential Privacy ?