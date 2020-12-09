---
layout: post
author: Vladimir Fux
title: Budget @HOME. Introduction
topic: Operations Research
tags: [or@home, or, optimization, operations research, budget optimization, bank, account]
excerpt_separator: <!--more-->
---

<br/><br/>

I was once analyzing my bank account spendings. My bank UI was not very convenient: I cannot easily filter on interesting fields, some visualization were lacking, etc. But was there is an export as .csv button, which came quite handy. I decided to look at my expenses through my favorite tools. 

<img src="/images/budget_optimization/morgan-housel-cAQZuqdvba8-unsplash.jpg " alt="Budget" style="width:350px;"/>
<span>Photo by <a href="https://unsplash.com/@morganhousel?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Morgan Housel</a> on <a href="https://unsplash.com/s/photos/bag-of-money?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>

Nice! I can do whatever I want with my data! I started looking where I could have avoided spending to much money and then an idea came to my mind: this can be treated as optimization problem! So here it is: **Budget optimization @HOME**

<!--more-->
## What is the problem?

My bank account transactions contains a bunch of data. I would like to be able to understand easily on what I've spent the most during the past period of time. In addition, I would like to retroactively optimize my expenses, i.e. answer a question: if I wanted to **save X Euros** in the past year, what things should I have avoided to buy?

## What is the data?
I use the data which my bank account allows to export as csv. I think many banks do so. For obvious reasons I will show here only artifical data, created with data generator (code provided). This data has exactly the same shape as my bank account data, and most likely it bears strong simmilarities with other banks. That's how it looks like:

| 	Beneficiary / Originator     |  Payment Detils  | Debit | Credit | Booking date | Currency |
|----------|--------|--------|------------|------|
| Lidl  1245...   | NaN      |  -60.0    |   NaN |    09/12/2019      |  EUR  |
| Zalando Payments GmbH..  | NaN  |   -45.0    |  NaN |     09/12/2019      |  EUR  |
|NaN |AMAZON EU ... | -125.0    | NaN |     09/12/2019     |  EUR  | 



## Challenge
Our ambition is to build (step by step) something interactive, which can already be considered as a simple app. 
We will keep things simple, we will not deploy anything, but rather consider it to be a simple jupyter notebook based applicatoin. Sneak preview:

![Alt Text](/images/budget_optimization/interface.gif)

## Tech stack

We will use the following languages/libs:
* Python
* pandas, numpy for data preparation
* plotly for data visualization
* pulp/cbc for optimization part
* jupyter, iwidgets for interactive part


## The whole series of posts 
1. [Budget optimization intro](/2020/11/22/budget-optimization-intro)
1. [Data preparation](/2020/11/23/budget-data-preparation)
1. [Data visualizaion](/2020/11/26/budget-data-visualization)
1. [Optimization model](/2020/12/09/budget-model)
1. [Make it work](/2020/12/10/budget-go-live)
