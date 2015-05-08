##Question 1
Imagine one of Clever's districts is being criticized by a parent organization for class sizes that are too big. Spending less than an hour, use the Clever API & the demo data set to perform a quick analysis of class sizes.

```python
import requests
import json
import numpy as np

r = requests.get('https://api.clever.com/v1.1/sections?limit=10000',headers={'Authorization':'Bearer DEMO_TOKEN'})
records = json.loads(r.text)['data']
class_sizes = [len(records['data']['students']) for record in records]
```
The above code will yield a list of class sizes for each class available via the API call. Note that since the parent complaint was about class sizes in a district, one can do a quick check to ensure that we do not need to group these classes by district since they are all from a single sample district:
```python
 set(record['data']['district'] for record in records)
 =>  {u'4fd43cc56d11340000000005'}
```
Going back to the question of class sizes, we find:
* Sample Mean: 25.5
* Standard Deviation: 6.7
* Total Classes: 382
* Confidence Interval: [Insert Here]

##Question 2
Clever allows school districts to create user accounts in apps for their students and teachers. 
For each app, districts can provision a subset (accounts_provisioned) of their total userbase (district_size).
At Clever, we refer to each user account provisioned as an "install."

The linked dataset represents the installation pairings between applications and districts. 
Use this dataset to explore usage patterns of Clever. For example, what share of districts use 1, 2, or more apps? 
Do apps break down in patterns as to what portion of the district they are deployed to? 
What else can you discover about app installs in Clever?

###Assumptions
The following are assumptions on which this analysis is based. In practice, I would revise these assumptions using the domain knowledge of colleagues:
* Assume the "created" date is the date at which the apps were provisioned and that apps are only provisioned once (i.e. students can not be added to an app in a district piecemeal). This is supported in the data by the fact that each app-district combo only has a single creation date/row in the data
* Rows with with district size = 0 and/or with accounts provisioned = 0 represent actual district-app connections but with missing data. For the purposes of this analysis, we will assume that the number of accounts provisioned for any app and any district with such characteristics is nonzero. This could be verified or falsified by speaking with someone with greater knowledge of how and when apps/districts are added to the database. An alternate assumption would be that, at least for the districts with non-negative district sizes, if installs =0, the app has not been installed. A counterpoint to this would be that the number of districts and district-district-size combos is the same:
```python
df.groupby(['district_id','district_size']).ngroups == df.groupby('district_id').ngroups
==> True
```
###Basics
To explore the data, we will use the Python package pandas. As a reference, here are the basic entities to which I will be referring:
```python
import pandas as pd

df = pd.read_csv('accounts_provisioned_dataset.csv')
grouped_apps = df.groupby('app_id')
grouped_districts = df.groupby('district_id')
grouped_created_by = df.groupby('created')
```
* Total Apps: 94
* Total Districts: 1680
* Total Installation Pairings (including ones with district size=0 and/or installs=0): 2239
* Total Creation Dates: 460
