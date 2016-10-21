---
layout: post
title:  "Kaggle Bosch Data Mining Competition"
date:   2016-10-17 22:00:00
categories: data-mining kabosch
tags: kabosch clojure data-mining
---
The Kaggle Bosch Production Line Performance competition is trying to 
identify parts that will fail quality control.  I have tinkered with one 
other Kaggle competition, the one to predict who survived the sinking of the
Titanic.  The Bosch competition has much more data than Titanic.  The number
of parts that Bosch manufactures is much larger than the number of people
that can fit on a boat.  Because of the size of the data and the fact that
it is split into three files, it will be hard to just load all the data into
Weka like I did for Titanic.  At least that is the excuse I used to start
writing code to analyze the data.

There are three types of data files provided.  The numeric data file contains
the Response column that contains the 0 for a successfully manufactured part
and 1 for a part that failed quality control.  The second data file is
categorical data and the third is time data identifying when measurements
were taken.  I chose to start with the numeric data file for two reasons.
First, it has the Result column so it can be studied without joining to
another file, and second because I have less experience analyzing
numeric ranges and learning is the real goal of this activity.

Before starting to work up a model to predict success or failure, I needed
a better sense of the data.  The first run was to read in the all the rows
from the numeric training data and count up the ones and zeros for each 
Response value.  Results of the first run showed that only 6879 rows were 
products that failed quality control compared to 1176868 successful products.
The naive model of always saying a product was successfully manufactured 
would be correct over 99% of the time, but that is not likely to win the 
competition.

The second run is to take the counts of ones and zeros for each feature 
and look for ratios where the ones to zeros for that feature was greater than 
the overall ratio of ones to zeros.  That will be the topic of the next post.

