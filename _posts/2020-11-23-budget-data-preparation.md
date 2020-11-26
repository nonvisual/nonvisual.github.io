---
layout: post
author: Vladimir Fux
title: Budget @HOME. Data Preparation
topic: Operations Research
tags: [or@home, or, optimization, operations research, budget optimization, bank, data, preparation]
excerpt_separator: <!--more-->
---
<br/><br/>
In this part we explore our bank account data, prepare it and deal with the transaction classification.
{% include /plotly/sunbirst.html %}
This sunbirst chart from plotly library is a sneek preview from visualization part of this post series. Our task for now is to prepare data, in order to allow such type of visualizations.
<!--more-->


## Transactions Data Preparation
As it was mentioned before, I will use randomly generated data. There is already prepared data generator, drawing
transactions from simple distribution:
```python
transactions = generate_transaction_data()
```
But when I do it for myself, I use of course my real bank account data.
We can start with just getting a rough idea on what we are dealing with:
```python
transactions.head(5)
```

| Beneficiary / Originator   | Payment Details   |   Debit |   Credit | Booking date   | Currency   |
|:---------------------------|:------------------|--------:|---------:|:---------------|:-----------|
 Edeka     ..                 | Edeka             |  -55.46 |      nan | 09/12/2019     | EUR        |
 Rewe     ..                  | Rewe              |   -0.14 |      nan | 09/28/2019     | EUR        | 
 Edeka    ..                  | Edeka             |   -0.43 |      nan | 12/09/2019     | EUR        |
 Lidl      ..                 | Lidl              |   -5.3  |      nan | 12/22/2019     | EUR        |
 Edeka    ..                  | Edeka             |  -16.38 |      nan | 05/16/2019     | EUR        |

Let's now check the types. 
```python
transactions.dtypes
```
We see that "Booking date" is an object, not a date. We will need to change this.
```
Beneficiary / Originator     object
Payment Details              object
Debit                       float64
Credit                      float64
Booking date                 object
Currency                     object
dtype: object
```

Does Debit meet Credit? 
```python
transactions[['Debit','Credit']].sum()
```
Seems to be ok
```
Debit     33908.58
Credit    39000.00
dtype: float64
```

So, the very first step is to correctly treat date column. In addition we may want extract day, month, year and weekday from it in order to conveniently aggregated data later:
```python
transactions['Booking date'] = pd.to_datetime(transactions['Booking date'])
transactions['day'] = transactions['Booking date'].dt.day
transactions['month'] = transactions['Booking date'].dt.month
transactions['year'] = transactions['Booking date'].dt.year
transactions['weekday'] = transactions['Booking date'].dt.weekday
```

Now we can conveniently check expenses by month:
```python
transactions.groupby(['year','month'])[['Debit','Credit']].sum()
```


|   year |   month |    Debit |   Credit |
|-------:|--------:|---------:|---------:|
|   2019 |       1 | -2494.86 |     3000 |
|   2019 |       2 | -2375.05 |     3000 |
|   2019 |       3 | -2882.58 |     3000 |
|   2019 |       4 | -2641.56 |     3000 |
|   2019 |       5 | -3202.15 |     3000 |
|   2019 |       6 | -2260.02 |     3000 |
|   2019 |       7 | -2481.98 |     3000 |
|   2019 |       8 | -2561.8  |     3000 |
|   2019 |       9 | -2578.52 |     3000 |
|   2019 |      10 | -3339.24 |     3000 |
|   2019 |      11 | -2520.05 |     3000 |
|   2019 |      12 | -3270.77 |     3000 |
|   2020 |       1 | -1300    |     3000 |


## Inferring transaction type
For the further analysis we need to differentiate different transaction types, i.e. which expense
is related to grocery shopping, which is leisure and so on. In principle, one can train a classifier to solve this problem,
but in such case we will need labeled data. The easiest way forward is a rule-based approach for classification. E.g. we know 
that if Payment Details contain names such "Lidl", "Edeka" or "Rewe" (typical supermarkets in Germany), this is most likely a grocery shopping. If you can find "Booking.com" or "Lufthansa" - this is something to do with travelling. You can always adapt this rule-based approach to your specific data.

To keep it simple, let's focus on just 5 transaction types:
```python
types= ['grocery', 'fashion', 'shopping', 'travel', 'rent', "unknown"]
```
And let's say transaction is of type "grocery" whenever it contains one of the shops below:
```python
types_mapping = {}
types_mapping['grocery'] = [
    "lidl",
    "rewe",
    "edeka",
    "aldi",
]
```
Now we need to go row by row and see, which type we should assigin to the observed transaction. If we have no idea - we assign "unknown" value. 
I decide to do it with help of pandas apply and a simple function below:
```python
def assign_type(row,types_mapping: Dict[str,List[str]], info_columns:List[str]):
    matching = []

    for c in info_columns:
        if not pd.isna(row[c]):
            matching.extend([s for s in types_mapping 
                if any(xs in row[c] for xs in types_mapping[s])])
    matching.append('unknown')
    return matching[0]
```
Here, we've noticed in data that information about shop can be in either 'Payment Details' or in "Beneficiary / Originator" columns.
That's why we look in several columns contained in "info_columns" list. If value is not nan in each column, we check if any of the type-specific keywords (e.g. "Lidl" shop name) is contained in the column value. If yes - we add this class in the matching. Potentially, there can be several matchings for some reasons. That's why we append all of them and return the first entry only (simple conflict resolution). And we always add 'unknown' type for the case if no matching was found.

What's left is to prepare partial (because apply expects 1-argument function) and create a new column in the transactions dataframe:
```python
f = partial(assign_type, types_mapping=types_mapping, info_columns=["Beneficiary / Originator", "Payment Details"])
transactions['type'] = transactions.apply(f, axis=1)
```
Done, now our dataframe contains correct date data type, has separate columns for day, month and year, and transaction type, which should help us later:

| Beneficiary / Originator   | Payment Details   |   Debit |   Credit | Booking date        | Currency   |   day |   month |   year |   weekday | type    |
|:---------------------------|:------------------|--------:|---------:|:--------------------|:-----------|------:|--------:|-------:|----------:|:--------|
| Edeka ..                     | Edeka             |  -55.46 |      nan | 2019-09-12 00:00:00 | EUR        |    12 |       9 |   2019 |         3 | grocery |
| Rewe  ..                     | Rewe              |   -0.14 |      nan | 2019-09-28 00:00:00 | EUR        |    28 |       9 |   2019 |         5 | grocery |
| Edeka   ..                   | Edeka             |   -0.43 |      nan | 2019-12-09 00:00:00 | EUR        |     9 |      12 |   2019 |         0 | grocery |
| Lidl   ..                    | Lidl              |   -5.3  |      nan | 2019-12-22 00:00:00 | EUR        |    22 |      12 |   2019 |         6 | grocery |
| Edeka  ..                    | Edeka             |  -16.38 |      nan | 2019-05-16 00:00:00 | EUR        |    16 |       5 |   2019 |         3 | grocery |


## Adding new mappings
In terms of dealing with your data, one can always refine the rules. Let's check how many of transactions are rendered as unknown:

```python
transactions.groupby('type').size()
```
We see 40 unknown transactions:

| type     |   0 |
|:---------|--------:|
| fashion  |      22 |
| grocery  |     291 |
| rent     |      13 |
| shopping |      43 |
| travel   |       4 |
| unknown  |      40 |


If we look at those, we can see that indeed I forgot several often transaction classes:
```python
transactions[transactions['type']=='unknown']
```
One of the missing shops in grocery category is "ROSSMANN"

```python
types_mapping['grocery'] = types_mapping['grocery'].append('rossmann')
```

I also forgot about salary:
```python
types_mapping['salary'] = ["salary"]
```

If we rerun the method, we see that "unknown" class is gone:

| type     |   0 |
|:---------|----:|
| fashion  |  22 |
| grocery  | 331 |
| rent     |  13 |
| salary   |  13 |
| shopping |  43 |
| travel   |   4 |

I wouldn't expect this rule-based approach to be able to identify all transaction types, that's quite manual work, but for your own data you can achieve descent numbers with several iterations

This code can be found in repository as a [notebook](https://github.com/nonvisual/budget_optimization/blob/main/data/Data%20preparation.ipynb) and as python [module](https://github.com/nonvisual/budget_optimization/tree/main/preparation)

## Previous
1. [Budget optimization intro](/2020/11/23/budget-optimization-intro)

## Next 
1. [Data visualizaion]()
