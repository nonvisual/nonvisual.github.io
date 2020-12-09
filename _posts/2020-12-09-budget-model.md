---
layout: post
author: Vladimir Fux
title: Budget @HOME. Optimization model
topic: Operations Research
tags: [or@home, or , budget optimization, bank, data, pulp, cbc, optimization]
excerpt_separator: <!--more-->
---


## Relative importance of transactions
For optimization to make sense, we as users, may want to specify relative importants of transactions. In the end we want to deduce a retroactive savings plan, and to do so we need to "cancel" some transactions. It is clear that one may save on grocery shopping, but saving on rent payments does not sound like a good idea.

<!--more-->

### Where the importance factors are coming from?
That can be user input. For each transaction type we can specify importance factor as integer value. Below are mine:

# Problem
You would like to understand which type of expenses you need to cut in future in order to achieve a certain savings per year.
Given a savings amount X, you need to return a list of transactions with minimal total importance, which you could have cut during current year, in order to save this amount.

Sanity rules:
* You know that there is a certain amount you need to spend on groceries (you need to eat)
* Every season you need to spend a certain amount on clothing 
* In general, you cannot avoid paying for household 


So what we would like to have is retroactive saving plan for our banc account: where I could have avoided additional expenses in order to achieve certain savings for the last year



# Model
### Decision
Take or not to take the corresponding transaction. Let transaction be indexed by $$n \in \mathcal{N}$$. Then the decision variable, corresponding to taking or not taking transaction $$n$$ is $$z_n \in \{0,1\}$$

### Optimization goal, i.e. objective function
We want to minimize the total importance of all the transaction we decide to keep, in order to achieve certain savings goal. If $$I_n$$ is importance of transaction $n$, then our goal is to
$$\begin{align*}
\max \sum_{n \in N} I_n z_n
\end{align*}$$


## Constraints

### Grocery
Well, we can never actually cut on grocery entirely. This retroactive savings plan would look ridiculous if some of the weeks will have 0 EUR spent on grocery. That's why we introduce a minimum bound for grocery expenses per week

$$\begin{align*}
\sum_{n \in \mathcal{N}^G_{w}} C_n z_n >= B^G, \quad \forall w \in \mathcal{W}
\end{align*}$$

where $$\mathcal{N}^G_{w}$$ is a set of transactions of type grocery, happening in a specific week $$w$$, $$B^G$$ is the minimum weekly allowance for grocery, which we find feasible (user input).

Now, there is a little trick here. If we keep constraint as it is, it may happen to be infeasible for some weeks. Imagine, one week you were paying all grocery expenses by cash, and thus they do not appear in your transactions. Yet, you ask solver to find solution, which exceeds your minimum allowance for groceries that week. That's simply impossible! That's why we modify our formulation, to account for such cases:

$$\begin{align*}
\sum_{n \in \mathcal{N}^G_{w}} C_n z_n >= \min(\sum_{n \in \mathcal{N}^G_{w}} C_n, B^G), \quad \forall w \in \mathcal{W}
\end{align*}$$

and here $$\sum_{n \in \mathcal{N}^G_{w}}$$ accounts for maximum transaction cost in grocery in week $$w$$ which you see in data. If this is above allowance limit, the latter will be used in the constraint. If it is smaller, we will use the observed cost from data as a lower bound limit.

### Household
We simply cannot avoid paying the rent. We would like our model to disallow saving plans which remove rent payments:
$$\begin{align*}
z_n = 1, \quad \forall n \in \mathcal{N}^H,
\end{align*}$$
where $$\mathcal{N}^H$$ is the set of transactions corresponding to rent payments.


### Savings
What we would like to know in the end is where do we need to cut, in order to have a certain savings number. We can put it is a target, or as a constraint. I've chosen the latter:
$$\begin{align*}
\sum_{n \in N} C_n (1-z_n) >= S,
\end{align*}$$

and here $$S$$ is our overall savings target. If target is given as a percentage of the total expenses $$P$$, then 
$$S = P * \sum_{n \in N} C_n$$


### Solution example

On the picture below, we see a small dataset, on which the problem is already solved. We can see that 2 transactions were removed, amounting to the total saving of 100 EUR. The objective value (total importance of kept transaction) equals 8.

<img src="/images/budget_optimization/budget_optimization_cropped.png">


## Model creation with PuLP
PuLP is an LP modeler written in Python. It comes with open source CBC solver, which makes it an easy choice to start with.
We create a basic model object, already specifying optimization sense (maximization).

```python
model = pulp.LpProblem("Profit_maximizing_problem", constants.LpMaximize)
```

### Decision variables
The model has two essential parts: variables and constraints (which describe relation between variables). Typically, the usual first step with implemantation of optimization model is to define variables. In our case this is a set of binary variables (having values 0 or 1), which describe for each transaction whether it should be kept or removed from the retroactive savings plan, $$z_n \in \{0,1\}$$

```python
decision_vars = pulp.LpVariable.dicts(
    "Transaction", selected_year.index, 0, 1, LpInteger)
```

### Objective
We can already define the objective function, i.e. the function which we ask our solver to maximize. In our problem, this is the
total importance of transactions we decided to keep in the plan. So we simply multiply each decision variable $$z_n$$ by importance factor of the corresponding transaction, then sum them up.

```python
objective = pulp.lpSum(
    [decision_vars[t] * selected_year.loc[t, "importance"] for t in selected_year.index]
)
model += objective
```


### Constraints
Let's start with the easiest contstraint to formulate. We would like to always keep rent payments in our plan. 
This means that we want all decision variables, corresponding to rent payment transactions to be equal to 1 (i.e. we keep them)


```python
for t in selected_year.index:
    if selected_year.loc[t, "type"] == "rent":
        model += decision_vars[t] == 1
```
Now we have a savings constraint to add. What we want is to make sure that certain savings are achieved. We opted for relative
savings input. Note, we need to take absolute value of "Debit", because the original data contains negative values there. We could have dealt with it in data preparation step, but this will work as well


```python
total_expenditure = abs(selected_year.sum()["Debit"])
savings_constraint = (
    pulp.lpSum([decision_vars[t] * abs(selected_year.loc[t, "Debit"]) for t in selected_year.index])
    <= (1 - savings) * total_expenditure
)
model += savings_constraint
```

Now comes the tricky one: grocery expenses constraint. We would like to make sure that our savings plan does not deliver ridiculous outcomes and cuts food expenses to 0. Thus we would like to have a lower bound on grocery expenses. In addition, we would like to make sure that this lower bound is achievable, because otherwise model might become infeasible. E.g., if user specify lower bound of 100 EUR per week, but a certain week contains transactions amounting only to 50 EUR, lower bound for such week should be minimum between bound and transactions sum, which is 50 EUR in the described case.

We start with computing current grocery expenses per week:
```python
agg_per_week = selected_year[selected_year["type"] == "grocery"].groupby("week").sum()
```

Next, we add weekly constraints:
```python
for w in range(52):
        if w in agg_per_week.index:
            week_grocery_spending = abs(agg_per_week.loc[w, "Debit"])
            model += pulp.lpSum(
                [
                    decision_vars[t] * abs(selected_year.loc[t, "Debit"])
                    if (selected_year.loc[t, "week"] == w) and (selected_year.loc[t, "type"] == "grocery")
                    else 0.0
                    for t in selected_year.index
                ]
            ) >= min(week_grocery_spending, grocery_per_week)
```

### Solving the model
Now we are good to solve the model. An important thing is to check model status afterwards: if the solution is not optimal, most likely we would not like to use it 

```python
status = model.solve()

if LpStatus[status] == 'Optimal':
    print("Model soved optimally")
else:
    print(f"Model solving was unsuccessful with status {LpStatus[status]}")
```

Parsing model output: simply for each transaction assign value of the decision variable. In addition, we check value of the objective

```python
solution = pd.Series(
    [
        decision_vars[t].varValue if decision_vars[t].varValue is not None else 1
        for t in selected_year.index
    ],
    index=selected_year.index,
    name="solution",
    dtype=np.int64,
)
objective = pulp.value(model.objective)
```

And finally merging it with our inputs dataframe. Now for each transaction we have solution value, which tells 
us wether this transaction should be kept (1) or removed (0) in our retroactive savings plan.
```python
solved_data = selected_year.join(solution)
```

## The whole series of posts 
1. [Budget optimization intro](/2020/11/22/budget-optimization-intro)
1. [Data preparation](/2020/11/23/budget-data-preparation)
1. [Data visualizaion](/2020/11/26/budget-data-visualization)
1. [Optimization model](/2020/12/09/budget-model)
1. [Make it work](/2020/12/10/budget-go-live)
