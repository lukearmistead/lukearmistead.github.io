
---
layout: post
title:  "Extracting health data from Fitbit (Work in Progress)"
date:   2019-10-20
categories: jekyll update
---

In this post, we'll cover getting data from Fitbit, doing a little cleaning, and storing it as a csv.

#### TODO:

- Write intro
- Write conclusion
- Use bolding to make steps a little more legible
- Make voice consistent

## Step 1: Register your personal app with Fitbit

Start by registering your personal app with Fitbit at their [fancy developer portal](https://dev.fitbit.com/apps/new). We're not going to get too nitty gritty here but [this blog post](https://towardsdatascience.com/collect-your-own-fitbit-data-with-python-ff145fa10873) has some very helpful details.

Navigate to your newly created app and grab the contents of the "OAuth 2.0 Client ID" and "Client Secret". In the spirit of not posting creds for all the internet to enjoy, I stored mine locally in a yaml file.


```python
import yaml
with open("../creds.yml", 'r') as stream:
    creds = yaml.safe_load(stream)
    
CLIENT_ID = creds['fitbit']['client_id']
CLIENT_SECRET = creds['fitbit']['client_secret']
```

## Step 2: Authorize and connect to Fitbit

We'll make heavy use of the `python-fitbit` [package](https://github.com/orcasgit/python-fitbit). You'll also need a specific `requests-oauthlib` and `oauthlib` to avoid errors when you try to authorize.


```python
!git clone https://github.com/orcasgit/python-fitbit
!pip3 install python-fitbit/requirements/base.txt
!pip3 install --upgrade requests-oauthlib==1.1.0 oauthlib==2.1.0
```

Mosey over to a directory where you can import the `gather_keys_oauth2` module from python-fitbit to get your access and refresh tokens and get that authorization you deserve!


```python
cd python-fitbit
```

    /Users/luke.armistead/workspace/projects/lukearmistead.github.io/notebooks/python-fitbit



```python
from gather_keys_oauth2 import *

server = OAuth2Server(CLIENT_ID, CLIENT_SECRET)
server.browser_authorize()
keys = server.fitbit.client.session.token
ACCESS_TOKEN, REFRESH_TOKEN = str(keys['access_token']), str(keys['refresh_token'])
```

If you're like me, you hit a 500 error here. [The Fitbit help forum suggests](https://community.fitbit.com/t5/Web-API-Development/Invalid-Client-Error/td-p/3290376) and my experience corroborates that you can get around this by downgrading to `requests-oauthlib==1.1.0` and `oauthlib==2.1.0`.

Finally, go ahead and create a client to ferry your fitbit data requests


```python
import fitbit

client = fitbit.Fitbit(
    client_id=CLIENT_ID,
    client_secret=CLIENT_SECRET,
    oauth2=True,
    access_token=ACCESS_TOKEN,
    refresh_token=REFRESH_TOKEN
)
```

## Step 3: Now for the good stuff. Let's get that data!

Hit [Fitbit's Web API](https://dev.fitbit.com/build/reference/web-api/) to pull in your data. Heads up, Fitbit [limits the number of daily requests you can make](https://dev.fitbit.com/build/reference/web-api/heart-rate/). Be careful not to over-ask or you'll have to wait a day to try again.

Start by establishing the range of time we want to pull. Fitbit gets testy if you request more than 100 days of data. We'll abide by this until we have a need for more historical data.


```python
from datetime import datetime, timedelta

today = datetime.now()
base_date = today - timedelta(days=100) # Fitbit doesn't like if we request more than 100 days. Meh. Fine.
```

Next, let's request our data! We'll simplify this by using the `time-series` method built into our `python-fitbit` package.


```python
sleeps = client.time_series(resource='sleep', base_date=base_date, end_date=today)

# Remove the minute by minute data of last night's sleep for nicer printing
example = sleeps['sleep'][0]
example.pop('minuteData', None) 
example
```




    {'awakeCount': 3,
     'awakeDuration': 4,
     'awakeningsCount': 13,
     'dateOfSleep': '2019-10-19',
     'duration': 28440000,
     'efficiency': 94,
     'endTime': '2019-10-19T08:02:30.000',
     'logId': 24320700816,
     'minutesAfterWakeup': 0,
     'minutesAsleep': 447,
     'minutesAwake': 27,
     'minutesToFallAsleep': 0,
     'restlessCount': 10,
     'restlessDuration': 23,
     'startTime': '2019-10-19T00:08:30.000',
     'timeInBed': 474}



Pull down the rest of your data. I grabbed the stuff that interests me, but there is a lot more! Check out the [Fitbit API documentation](https://dev.fitbit.com/build/reference/web-api/) to learn about what's possible.


```python
activities = client.time_series(resource='activities/distance', base_date=base_date, end_date=today)
weights = client.time_series(resource='body/weight', base_date=base_date, end_date=today)
fats = client.time_series(resource='body/fat', base_date=base_date, end_date=today)
bmis = client.time_series(resource='body/bmi', base_date=base_date, end_date=today)
```

Most of the payloads returned by requests are pretty simple. I wrote a function to handle those cases.


```python
def unload_simple_json(data, d=None):
    if not d:
        d = {'dateTime': [], 'value': []}
    for entry in data:
        for k in d.keys():
            d[k].append(entry[k])
    return d

d = {'efficiency': [],'minutesAsleep': [], 'startTime': [], 'endTime': [], 'awakeningsCount': [], 'dateOfSleep': []}
sleeps = unload_simple_json(sleeps['sleep'], d=d)
activities = unload_simple_json(activities['activities-distance'])
weights = unload_simple_json(weights['body-weight'])
fats = unload_simple_json(fats['body-fat'])
bmis = unload_simple_json(bmis['body-bmi'])
```

Some endpoints have different structures so we'll have to write some custom logic.


```python
hearts = client.time_series(resource='activities/heart', base_date=base_date, end_date=today)
hearts['activities-heart'][0]
```




    {'dateTime': '2019-07-12',
     'value': {'customHeartRateZones': [],
      'heartRateZones': [{'caloriesOut': 2273.24235,
        'max': 95,
        'min': 30,
        'minutes': 1167,
        'name': 'Out of Range'},
       {'caloriesOut': 1197.4905,
        'max': 133,
        'min': 95,
        'minutes': 200,
        'name': 'Fat Burn'},
       {'caloriesOut': 0, 'max': 161, 'min': 133, 'minutes': 0, 'name': 'Cardio'},
       {'caloriesOut': 0, 'max': 220, 'min': 161, 'minutes': 0, 'name': 'Peak'}],
      'restingHeartRate': 59}}




```python
d = {
    'Fat Burn': [],
    'Cardio': [],
    'Peak': [],
    'dateTime': [],
    'restingHeartRate': []
}

for entry in list(hearts['activities-heart']):
    d['dateTime'].append(entry.get('dateTime'))
    d['restingHeartRate'].append(entry['value'].get('restingHeartRate'))
    
    # Flattening embedded list
    for k in d.keys():
        for value in entry['value']['heartRateZones']:
            if value['name'] == k:
                d[k].append(value.get('minutes'))
hearts = d
```

Next we'll organize our data it into pandas dataframes. Let's start by setting up a foundational table with all the dates we'll need.


```python
import pandas as pd

df = pd.DataFrame(
    pd.date_range(start=base_date, end=today)
    ).rename(columns={0: 'dateTime'})
df['dateTime'] = pd.to_datetime(df['dateTime']).dt.date
df.head()
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
      <th>dateTime</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2019-07-12</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2019-07-13</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2019-07-14</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2019-07-15</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2019-07-16</td>
    </tr>
  </tbody>
</table>
</div>



I renamed these keys to avoid merging them as duplicate column names later.


```python
weights['weight'] = weights.pop('value')
fats['fat_pct'] = fats.pop('value')
bmis['bmi'] = bmis.pop('value')
sleeps['dateTime'] = sleeps.pop('dateOfSleep')
```

Now merge up your data!


```python
for d in [activities, weights, fats, bmis, hearts]:
    d = pd.DataFrame(d)
    d['dateTime'] = pd.to_datetime(d['dateTime']).dt.date
    df = df.merge(pd.DataFrame(d), how='left', on='dateTime')
```

The sleep data has multiple values for each date. A simple aggregation should be enough to de-duplicate with enough fidelity for the rough analyses we'll be running to start.


```python
sleeps = pd.DataFrame(sleeps)
sleeps[sleeps.duplicated(subset='dateTime', keep=False)]
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
      <th>efficiency</th>
      <th>minutesAsleep</th>
      <th>startTime</th>
      <th>endTime</th>
      <th>awakeningsCount</th>
      <th>dateTime</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>96</td>
      <td>71</td>
      <td>2019-10-17T22:02:00.000</td>
      <td>2019-10-17T23:16:30.000</td>
      <td>1</td>
      <td>2019-10-17</td>
    </tr>
    <tr>
      <th>3</th>
      <td>96</td>
      <td>358</td>
      <td>2019-10-17T00:21:00.000</td>
      <td>2019-10-17T06:34:00.000</td>
      <td>10</td>
      <td>2019-10-17</td>
    </tr>
    <tr>
      <th>44</th>
      <td>87</td>
      <td>109</td>
      <td>2019-09-06T10:37:30.000</td>
      <td>2019-09-06T12:43:30.000</td>
      <td>9</td>
      <td>2019-09-06</td>
    </tr>
    <tr>
      <th>45</th>
      <td>88</td>
      <td>278</td>
      <td>2019-09-06T01:41:00.000</td>
      <td>2019-09-06T06:57:00.000</td>
      <td>13</td>
      <td>2019-09-06</td>
    </tr>
    <tr>
      <th>85</th>
      <td>81</td>
      <td>60</td>
      <td>2019-07-14T08:21:00.000</td>
      <td>2019-07-14T09:35:30.000</td>
      <td>3</td>
      <td>2019-07-14</td>
    </tr>
    <tr>
      <th>86</th>
      <td>91</td>
      <td>329</td>
      <td>2019-07-14T01:02:30.000</td>
      <td>2019-07-14T07:12:00.000</td>
      <td>9</td>
      <td>2019-07-14</td>
    </tr>
    <tr>
      <th>87</th>
      <td>93</td>
      <td>56</td>
      <td>2019-07-13T19:25:30.000</td>
      <td>2019-07-13T20:36:30.000</td>
      <td>6</td>
      <td>2019-07-13</td>
    </tr>
    <tr>
      <th>88</th>
      <td>90</td>
      <td>572</td>
      <td>2019-07-12T22:28:00.000</td>
      <td>2019-07-13T09:04:30.000</td>
      <td>27</td>
      <td>2019-07-13</td>
    </tr>
  </tbody>
</table>
</div>



Looks like we got 'em.


```python
# sleeps['dateTime'] = sleeps.pop('dateOfSleep')
sleeps = sleeps \
    .groupby('dateTime', as_index=False) \
    .agg({'startTime': min, 'endTime': max, 'minutesAsleep': sum, 'efficiency': 'mean'})
sleeps[sleeps.duplicated(subset='dateTime', keep=False)]
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
      <th>dateTime</th>
      <th>startTime</th>
      <th>endTime</th>
      <th>minutesAsleep</th>
      <th>efficiency</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
sleeps['dateTime'] = pd.to_datetime(sleeps['dateTime']).dt.date
df = df.merge(pd.DataFrame(sleeps), how='left', on='dateTime')
```

Clean up column names. I must confess that I'm not crazy about camelcase.


```python
df = df.rename(columns={
    'dateTime': 'date',
    'efficiency': 'sleep_efficiency',
    'minutesAsleep': 'sleep_minutes', 
    'startTime': 'sleep_start_at',
    'endTime': 'sleep_end_at',
    'awakeningsCount': 'wake_count',
    'Fat Burn': 'fat_burn_minutes',
    'Cardio': 'cardio_minutes',
    'Peak': 'peak_minutes',
    'restingHeartRate': 'resting_heart_rate'
})
```

And finally export as a CSV!


```python
df.to_csv('../../data/{}_to_{}_fitbit_data.csv'.format(base_date.date(), today.date()), index=False)
```

- Facts => day / wk / mo tables
- Add in goals and metrics
- Daily pull
- Set up ETL to pull down historical data
- Store data remotely
