---
layout: post
author: Vladimir Fux
title: Budget @HOME. Data Visualization
topic: Operations Research
tags: [or@home, or , budget optimization, bank, data, visualization, plotly]
excerpt_separator: <!--more-->
---
In this part we visualize bank transactions data, aiming to get an idea 
* Types of expenses and there share
* Savings trends
* Expenses by monthes and types
Just to understand what do we spend money on, and where is potential for savings.

{% include /plotly/monthly_balance.html %}
<!--more-->

There are several choices for visualization libraries in python:
* plotnine (ggplot)
* matplotlib
* plotly
* seaborn

In all my previous projects I successfull used plotnine and was quite satisfied with it so far. For this post though, I decided to opt for plotly, which is new library for me. The main selling point for me was interactiveness of plots (just check the plot in the preview of the post).

As an outcome of data preparation post, we have a python function, which makes our data ready for further analysis. We again use synthetic data, and prepare it (in place):
```python
transactions = generate_transaction_data()
prepare_data(transactions)
```
To keep it simple (e.g. for monthly analysis), we focus on one year only:
```python
year = 2019
prepare_data(transactions)
selected_year = transactions[transactions['year']==year]
```


We can start with a simple pie chart, showing us expenses distribution by type. First observation is that pie chart does not like much negative values, so we first need to take absolute value of debit column:
```python
selected_year ['expense'] = abs(transactions['Debit'])
```
And let's start fancy: we can use donut instead of classical pie shape (wow, that's easy in plotly), and also show both percentage and label value on each donut piece:
```python
fig = px.pie(selected_year, values='expense', names='type', title=f'Ratio of expenses by type in year {year}',hole=.3)
fig.update_traces(textposition='inside', textinfo='percent+label')
fig.show()
```
And that's look nice:
{% include /plotly/budget_visualization/pie.html %}
Note it is interactive as promissed: one can switch on/off each transaction type, and double click to select only one label.

Actually, we can do something even nicer. In data prepartion step, for each transaction we not only definedt type, but also entity it is associated with. It is quite general as a term, because depending on context it can be a different thing. E.g. for shopping it can be a specific online shop and for travelling type of expense it can be, say country. 

With sunburst type of plot, we can have an additional layer of distribution, this time by entity. This way, on a single interactive plot we can see both type and entities distribution.
```python
fig = px.sunburst(selected_year, values='expense', path=['type','entity'], \
                  title=f'Ratio of expenses by type and entity in year {year}')
fig.update_traces(textinfo="label+percent entry")
fig.show()
```
Here we specify in path argument all the columns, on which we would like to see a distribution. That's the outcome:

{% include /plotly/budget_visualization/sunbirst.html %}
If we e.g. click on grocery type, then we see distribution by retailers. Nice!

This is all fancy, but that's not we would like to see. We would like to see how our balance evolves with time and also how are savings are trending.

Let's start with monthly balance. At first we aggregate by month, and sum in pandas. Further we introduce new columns:
* balance is to track difference between Debit and Credit transactions in month
* color - auxiliary column, which will allow us easily to color negative balances
* savings - cumulative sum of balances, which is basically accumulated savings



```python
balance = selected_year.groupby(['month']).sum().reset_index()
balance['balance'] =  balance['Credit'] + balance['Debit']
balance["color"] = np.where(balance["balance"]<0, 'Negative', 'Positive')
balance['savings'] = balance['balance'].cumsum()
```

Now, we can plot balance, with red color for negative entries:

```python
fig = px.bar(balance, x="month", y="balance",
             color='color',
             barmode='stack',
             labels={
                     "color": "Balance",
                     "balance": "Balance [EUR]",
                     "month": "Month"
             },
             title = f"Monthly balance in year {year}",
             height=400)

fig.show()
```
Resulting plot is visible on top of this post. If we are talking about monthly spendings, it would be nice to check total spendings and also splits by type:
```python
grouped = selected_year.groupby(['month','type']).sum().reset_index()
fig = px.bar(grouped, x="month", y="Debit",
             labels={
                     "Debit": "Total expenses [EUR]",
                     "month": "Month"
             },
             title = f"Total Expenses by month in year {year}",
             barmode='group',
             height=400)
fig.show()
fig = px.bar(grouped, x="month", y="Debit",
             color='type', barmode='group',
             labels={
                     "type": "Transaction type",
                     "month": "Month",
                     "Debit" : "Expenses [EUR]"
             },
             title = f"Total Expenses by month and type in year {year}",
             height=400)
fig.show()
```
{% include /plotly/budget_visualization/expense_month_type.html %}

The last important thing is to track our accumulative savings over time. We can do with simple area plot here:
```python
fig = px.area(balance, x="month", y="savings",labels={
                     "savings": "Savings [EUR]",
                     "month": "Month"
             },
             color_discrete_sequence=[ "green"],
             title=f'Savings trend for year {year}')
fig.show()
```
{% include /plotly/budget_visualization/savings.html %}

This is all visualization we need at this step. Last, very minor note: plotly is nicely embedible as html! You can do 
```python
fig.write_html("expense_month_type.html")
```
and include resulting html snippet on your webpage

## The whole series of posts 
1. [Budget optimization intro](/2020/11/22/budget-optimization-intro)
1. [Data preparation](/2020/11/23/budget-data-preparation)
1. [Data visualizaion](/2020/11/26/budget-data-visualization)
1. [Optimization model](/2020/12/09/budget-model)
1. [Make it work](/2020/12/10/budget-go-live)
