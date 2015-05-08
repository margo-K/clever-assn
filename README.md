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

##Usage Insights
To get a basic handle on the relationships between districts and apps, as provided in this data set, we first do a little exploratory work. ####How many apps are districts using?
Out of the 1680 districts included in this dataset, the vast majority (1336 ~= 79.5%) use only one app. 13.27% use 2 apps and only 7.2% use more than 2 apps.  [ADD WHAT HAPPENS IF WE ELIMINATE ROWS WHERE ACCOUNTS PROVISIONED == 0]
* Minimum Apps/District: 1
* Maximum Apps/District: 17
Given that the total app catalog (represented in this dataset) is 94 apps, this means that even the most active districts use less than 20% of the total app catalog. **** ADD SOMETHING HERE ****
Looking at the full distribution:
```python
apps_per_district = grouped_districts.size()
apps_per_district.value_counts()
```
![Plot of App Usage By District](https://raw.github.com/margo-K/clever-assn/master/app_usage_per_district.png) 
| Apps Per District  | Total Districts   | Percentage of Districts (%)  |
| ------------- |:-------------:| -----:|
| 1      | 1336 | 79.5% |
|2      | 223      |   13.27% |
| 3      |   57| 3.39%|
| 4      | 27| 1.60% |
| 5 | 11      |   <1% |
|7 | 10     |    <1%  |
| 6 | 7 | <1%  |
| 17  | 1      |   <1%  |
| 16| 1     |    <1% |
| 15 | 1      |   <1%  |
|13 | 1     |    <1%  |
| 12 | 1 | <1%  |
| 11  | 1      |   <1%  |
| 10| 1     |    <1%  |
|9 | 1     |    <1%  |
| 8 | 1| <1%  |

###How many districts is each app in?
![Plot of App Usage By District](https://raw.github.com/margo-K/clever-assn/master/district_per_app.png)
To get at the problem from another angle, we also want to look at how apps are used in each district:
* Min Districts an App is in: 1
* Max Districts an App is in: 285 (only one app)
* 25.53% of apps are in only 1 district (24 apps out of 94)
* 39.4% of apps are in more than 10 districts
* 8.5% of apps are in more than 100 districts (8 apps total)

###What does this tell us?
* There seems to be a fair amount of variety in 
Note: We are assuming that every connection (expressed as a row in the data provided) can be counted as a district "using" an app. For more see "Assumptions" and "What to do about missing data?"

##What to do about missing data?