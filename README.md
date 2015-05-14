##Question 1
Imagine one of Clever's districts is being criticized by a parent organization for class sizes that are too big. Spending less than an hour, use the Clever API & the demo data set to perform a quick analysis of class sizes.


```python
import requests
import json
import numpy as np

r = requests.get('https://api.clever.com/v1.1/sections?limit=10000',headers={'Authorization':'Bearer DEMO_TOKEN'})
records = json.loads(r.text)['data']
class_sizes = [len(record['data']['students']) for record in records]
```
The above code will yield a list of class sizes for each class available via the API call. Note that since the parent complaint was about class sizes in a district, one can do a quick check to ensure that we do not need to group these classes by district since they are all from a single sample district:
```python
 set(record['data']['district'] for record in records)
 =>  {u'4fd43cc56d11340000000005'}
```
Going back to the question of class sizes, we find:
* Sample Mean: 25.5
* Standard Deviation: 6.7
* Max class Size: 50
* Min class Size: 3
* Total Classes: 382

We could provide this analysis to the partner district and perhaps include information about other districts to contextualize these numbers. The district might also want to engage with the fact that some classes are very large (50 students), even if on average they are within the bounds of "normal".
##Question 2
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

##Exploring the Data
####How many apps are districts using?
Out of the 1680 districts included in this dataset, the vast majority (1336 ~= 79.5%) use only one app. 13.27% use 2 apps and only 7.2% use more than 2 apps.  
* Minimum Apps/District: 1
* Maximum Apps/District: 17

| Apps Per District  | Total Districts   | Percentage of Districts (%)  |
| ------------- |:-------------:| -----:|
| 1      | 1336 | 79.5% |
|2      | 223      |   13.27% |
| 3      |   57| 3.39%|
| 4      | 27| 1.60% |
| >4 | 37      |   2.2% |

![Plot of App Usage By District](https://raw.github.com/margo-K/clever-assn/master/app_usage_per_district.png) 

###How many districts is each app in?
![Plot of App Usage By District](https://raw.github.com/margo-K/clever-assn/master/district_per_app.png)
To get at the problem from another angle, we also want to look at how apps are used in each district:
* Min Districts an App is in: 1
* Max Districts an App is in: 285 (only one app)
* 25.53% of apps are in only 1 district (24 apps out of 94)
* 39.4% of apps are in more than 10 districts
* 8.5% of apps are in more than 100 districts (8 apps total)

###What do we know
So far, we've just scratched the surface of what this data set (or the larger data set of which its part) can tell us. We have a basic idea of how apps and district connections are made, but to really dig deeper, we need to think about what underlying questons we might want to answer. Clever has at least two entities that could be considered "customers" - app developers and school districts. As a company, Clever would presumably like to make both of these groups happy and help them meet their goals. We'd like to be able to use this data to understand how Clever is working for these two customer types. To do this, we have to first understand what their goals might be.

In a real-life situation, I would rely on colleagues who speak directly to these different user groups to get an understanding of what it means for each of these groups to be "happy customers". In lieu of those conversatios, I'll venture to guess that for app developers, getting more installs over time would make them happy. School districts, on the other hand, ultimately want to them to be able to find the apps that they find useful and use Clever to provide their students with access to these apps. Without actual usage data, we can't get a firm grasp on how well used the particular apps school districts use are. We can, however, assume that a school district that either (a) integrates more apps over time or (b) expands the proportion of their students having access to the apps is happy with the service that's being provided.

In this context, we notice a few things about the data above:

(1) The fact that most school districts have only integrated a single app means that there's probably huge room for improvement in terms of district usage. It could be that the districts are not happy with the service they've received so far or it could be that they just need a bit more help understanding how they could use Clever to better support their students.

(2) The total app catalog (in this data) is 94 apps, which means that even the most active districts use less than 20% of the catalog. It could be that they are using the max number of apps that are useful to them, but it could also be that there are issues in discovery.

(3) Roughly 1/4 of apps are only in one district. This suggests that there might be a bigger variety of apps being used than one might otherwise expect (think of this in opposition to the majority of districts using 1 or 2 of the biggest apps and then using almost none of the others). Also, in this data there are 94 different apps being used total. As a next stage, one could explore how bigger budget apps compare to apps made by individuals or smaller companies.

###Exploring The Bright Spots
To get a handle on what it might look like to use Clever well, we can take a look at the highest performing districts.

| District Id | Total Number of Apps |
| -------- |:----------:|
| 34 | 10 |
| 542 | 12 |
| 791 | 8 |
| 1089 | 17 |
| 1160 | 13 |
| 1734 | 11 |
| 1994 | 16 |
| 2442 | 9 | 
| 2641 | 15 |

When we look at the data for these districts individually, we see that there are many different types of usage patterns. Some districts install almost all of their apps in a short period of time (within a few months), some districts install apps all year long. Some try an app in the off season or a year before then come back and provision significantly more. There's also a fair amount of variety in the portion of their students they provide app installs to. All of these could be explored in more detail on a larger data set to determine whether any of these usage patterns lead to better overall adoption for districts.

###Assumptions
The following are assumptions on which this analysis is based. In practice, I would revise these assumptions using the domain knowledge of colleagues:
* Assume the "created" date is the date at which the apps were provisioned and that apps are only provisioned once (i.e. students can not be added to an app in a district piecemeal). This is supported in the data by the fact that each app-district combo only has a single creation date/row in the data.
* Rows with with district size = 0 and/or with accounts provisioned = 0 represent actual district-app connections but with missing data. For the purposes of this analysis, we will assume that the number of accounts provisioned for any app and any district with such characteristics is nonzero. This could be verified or falsified by speaking with someone with greater knowledge of how and when apps/districts are added to the database. An alternate assumption would be that, at least for the districts with non-negative district sizes, if installs = 0, the app has not yet been installed.

###Points for clarification
1. What does more than 100% provisioning for a given app imply? Does this mean accounts created for teachers or duplicate accounts for students or creation of accounts for students who no longer attend the school?
2. For districts that have extremely small district sizes (1 or 2 students), is this accurate or does this represent a different kind of intended usage (say for only special education students, for example) than the other apps?

##Next Steps
In addition to the questions raised above, the following are questions that could be explored in future work:

(1) Are there apps which are only deployed in "big districts" and apps that are only deployed in "small" ones?

(2) Do the following factors affect district engagement with apps:
* districts that try one app then try the rest later vs. all at once?
* districts that provision all apps to everyone vs. those that do just some to some?
* districts that provision apps all year long vs. in just one season?

(3) Is there a correlation between the district size and the portion of a district apps are deployed to (for example, do small districts provision to everyone)?  

