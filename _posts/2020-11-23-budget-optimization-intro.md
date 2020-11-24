---
layout: post
author: Vladimir Fux
title: Budget optimization @HOME
topic: Operations Research
tags: [or@home, or, optimization, operations research, budget optimization, bank, account]
excerpt_separator: <!--more-->
---

I was once analyzing my bank account spendings. My bank UI was not very convenient: I cannot easily filter on interesting fields, some visualization were lacking, etc. But was there is an export as .csv button, which came quite handy. I decided to look at my expenses through my favorite tools. Nice! I can do whatever I want with my data! I started looking where I could have avoided spending to much money and then an idea came to my mind: this can be treated as optimization problem! So here it is: Budget optimization @HOME

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





## Next? 
1. [Data preparation](/2020/11/23/budget-data-preparation)
1. [Data visualizaion]()
