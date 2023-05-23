---
title: A Naive Approach to Monitoring Datasets [Draft]
categories: [Programming]
tags: [python,programming,data engineering]
date: 2023-05-23 16:00:00
---

For some time now, I have had this problem at my usual place of work to do with monitoring
of some data streams. The feeds are mostly set up as webhooks that are provided to other
teams within the organisation I work at and/or external organisations to send various data
through. The data ranges from notifications from mobile money partners to events from
various applications in use within the organisation. This data is dumped with some slight
modifications into our object storage solution (we made a poor decision
in the choice of our object storage solution -- believe it or not it is kind of
opinionated about the structure of the data dumped into it... Anyway, story for another
day). I was very much interested in the patterns of flow of the data. I wanted to be
alerted if we start getting an unusually low or high inflow of data. Unusual inflows of
data can sometimes indicate a problem that needs to be addressed ASAP before shit goes
too far south.

I scoured the Internet for possible solutions I could use for this. Turns out 99% of the
good ones require a good amount of data science knowledge. I know nothing about data
science so I really couldn't use any of them. The remaining 1% were extremely naive
solutions that really wouldn't work well for me. Examples of the extremely naive solutions
include simply checking for flatlines in the data (i.e. we haven't had any data in the past
24 hours). Checking for flatlines works for datasets that are expected to get data every
day but I have some datasets that can go for a couple of days without any data. Besides,
this was already something that was implemented on some of the datasets but behold there
were still issues we were missing. Flatlines alone are not enough. Like I have mentioned
elsewhere in this article, I was also interested in extremely high inflows of data.
Eventually, I settled on two solutions... They are quite naive as well but I felt they
were quite better than the rest of the other solutions and would complement the existing
monitoring solutions quite well.

# Difference of Means

The first solutions was the good old
[difference of means t-test](https://www.khanacademy.org/math/ap-statistics/xfb5d8e68:inference-quantitative-means/two-sample-t-test-means/v/two-sample-t-test-for-difference-of-means).
I did not run into anything that explictly mentioned this as a solution. I kind of inferred
it from the material I was reading... And to be honest I had already made the decision to
try this out even before I did any kind of research on this. I figured this could be used
to compare the current average inflow of data to the past average inflow of data on
a particular stream. There are some (a lot of) underlying assumptions I was making about
the data but what the hell, right??? Better to try something dumb than nothing at all.

Here is how I staged it all...

1. I gather data from the last 30 days on number of new rows added to a dataset daily
2. I split the data into two groups
  - Recent data (i.e. from the past 7 days)
  - Old data (i.e. everything before the last 7 days)
3. Compare the means from the two groups to see if there is a significant difference
   between them

Why did I pick 30 days... Well I figured that over long periods of time, the inflows of
data do change. In the beginning getting 100 news per day was the standard, 3 months
later the standard is now 10,000 rows per day. Trying to include all these data points
would result in very large standard deviations making almost everything acceptable. Over
a month the variations would be somewhat manageable.

Isn't 30 (ie... 22 data points vs 7 data points) too small for a sample? That was my
fear but again, no one's life depends on this. So, I winged it hoping for a good result
and I was handsomely rewarded for it in the end.

How does this look in code???

```python
from datetime import date, timedelta
from pandas import dataset

today = date.today()
start_date = today - datetime.timedelta(days=30)
dataset_name = "Sexy ass dataset"
inflows : Dataset = get_daily_inflows_for_dataset(dataset_name, from=start_date)

current = inflows[inflows["Date"] >= date]
historical = inflows[inflows["Date"] < date]

import stats

t_stat, p_value = stats.ttest_ind(historical["Count"], current["Count"]
# Using 5% significance level
if p_value <= 0.05:
    send_alert(dataset_name, {"t-stat": t_stat, "p-value": p_value})
```

**TBC**
