---
layout: post
title: First post example
---

| info     |  type  | amount | importance | week |
|----------|--------|--------|------------|------|
| Lidl     |   1    |  60.0  |     1      |  3   |
| Zalando  |   2    |  45.0  |     2      |  28  |
| Rent Aug |   3    | 1450.0 |     10     |  17  | 

### Optimization goal, i.e. objective function
We want to minimize the total importance of all the transaction we decide to remove, in order to achieve certain savings goal.

If $$I_n$$ is importance of transaction $$n$$, then our goal is to

$$\begin{align*}
\min \sum_{n \in N} I_n z_n
\end{align*}$$


```python
def compare_expenses_by_type(solved_data, exp_type):
    before = solved_data[(solved_data['type']==exp_type) ].groupby('week')['amount'].sum()
    after = solved_data[(solved_data['solution']==1) & (solved_data['type']==exp_type) ].groupby('week')['amount'].sum()
    comparison = pd.DataFrame({"original":before, "optimized" :after})
    print(f"{transaction_type[exp_type]} savings by week")
    comparison.plot.bar()
```

![Budget optimized](/images/budget_optimization/Budget share.png)
[Github repo](https://github.com/nonvisual/budget_optimization) with project example with simple idea.
