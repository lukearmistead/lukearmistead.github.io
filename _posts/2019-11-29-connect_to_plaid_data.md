---
layout: post
title:  "Extracting financial data using Plaid"
date:   2019-11-29
categories: jekyll update
---

Our quest to to overengineer the quantified self continues! Last time, we grabbed a plethora of [health data using Fitbit](http://luuuuke.com/jekyll/update/2019/10/20/connect_to_fitbit_data.html). Today, we're talking money! How much am I spending? How much am I making? How much do I have? We can answer all of these questions and more using the really cool, really robust API provided by [Plaid](http://plaid.com/). This post walks through setting up an account and connection with Plaid to access and store your data. Once stored, there's all sorts of cool analyses we could run to increase financial awareness and improve spending habits. Enough talk. Let's get to it!

## Step 1: Register for Plaid and connect a bank account

Start by getting your creds from [Plaid's website](https://dashboard.plaid.com/signup). Signing up gets you 1 free connected account but you can [request developer access](https://dashboard.plaid.com/overview/development) to get another 99 if you're like me and want to connect more.

Next, export the creds listed on your newly created account into your local environment. [Plaid's quickstart tool](https://github.com/plaid/quickstart/blob/master/python/server.py) specificaly expects the variable names printed below, so be sure to export them that way!

If you have trouble with these steps, the first half of [Zev's awesome tutorial](https://www.twilio.com/blog/2017/06/check-daily-spending-sms-python-plaid-twilio.html) provides a far more detailed walkthrough that could help you push through hangups.


```python
import os
import yaml

with open("../creds.yml", 'r') as stream:
    creds = yaml.safe_load(stream)['plaid']

print('Environment variable names for Plaid creds:')
for k, v in creds.items():
    if k in ['client_id', 'secret', 'public_key', 'env']:
        k = ('plaid_' + k).upper()
        print(k)
        os.environ[k] = v
    else:
        pass
```

    Environment variable names for Plaid creds:
    PLAID_CLIENT_ID
    PLAID_SECRET
    PLAID_PUBLIC_KEY
    PLAID_ENV


Next, clone Plaid quickstart guide to take advantage of the GUI they wrote to connect to your financial accounts.


```python
!git clone https://github.com/plaid/quickstart.git
```

Now run and navigate to Plaid's authorization server (`http://localhost:5000/` for me) and follow the instructions in the GUI to connect your account. If you run into trouble, check out [Plaid's quickstart guide](https://plaid.com/docs/quickstart/) or the [quickstart source code](https://github.com/plaid/quickstart/tree/master/python).


```python
!python quickstart/python/server.py
```

You should get back a payload that includes an `access_token` for your account. Hang on to that. You'll need it later.

## Step 2. Get transaction data

Now that we have the credentials to connect to a specific account using Plaid's API, we can get our data! We'll start by using the [plaid python package](https://github.com/plaid/plaid-python#install) to create a `Client` object.


```python
from plaid import Client

with open("../creds.yml", 'r') as stream:
    creds = yaml.safe_load(stream)['plaid']

client = Client(
    client_id=creds['client_id'],
    secret=creds['secret'],
    public_key=creds['public_key'],
    environment=creds['env']
)
```

Using this object, we can hit the API for our financial details. I'm interested in monitoring my spending, so I use the `Transactions` method to focus there. There is a [ton of additional functionality](https://plaid.github.io/plaid-python/index.html) like `AssetReport` or `Income`, so feel free to go wild!

I hit two gotchas. First, Plaid appears to limit the nuber of transactions per call to 100ish. If you're pulling years of data, you might have to chunk it to avoid this snag. Second, as of the publication of this post, Capital One and Plaid had [an ongoing disagreement](http://news.fintech.io/post/102ey7d/capital-one-restricts-third-party-data-access-to-plaid-upsets-customers) about the right way to secure user data. If you're like me and have a Capital One checking account, that data likely won't be accessible via Plaid until that dispute is resolved. 


```python
from datetime import datetime, timedelta
import pandas as pd

d = client.Transactions.get(
    access_token=creds['barclays']['access_token'],
    start_date='2019-06-30',
    end_date='2019-09-30'
)
```

The payload returned by the `Transactions` call fits very, very neatly into a dataframe. We can then use Pandas to manipulate and store our data in a csv.


```python
df = pd.DataFrame(d['transactions'])
df.loc[df['transaction_id'] == 'PND13M6VxJC3BekOR7R3CyNmDKQaEAFmVYgd8', ['category', 'date', 'amount', 'name', 'transaction_id']]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>category</th>
      <th>date</th>
      <th>amount</th>
      <th>name</th>
      <th>transaction_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>3</td>
      <td>[Shops]</td>
      <td>2019-09-26</td>
      <td>253.31</td>
      <td>THE PAWN SHOP</td>
      <td>PND13M6VxJC3BekOR7R3CyNmDKQaEAFmVYgd8</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.to_csv('../data/transactions.csv', index=False)
```

Now we have not one but TWO sources of data. Things are getting interesting. Could we identify trends across our different sources? Does going to the bar one night reduce the likelihood of hitting the gym the next day? Does sleeping more result in better frugality? So many hypotheses, so little time.
