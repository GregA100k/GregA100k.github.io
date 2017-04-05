---
layout: post
title:  "Quargle: the Kaggle Quora Competition"
date:   2017-04-04 12:00:00
categories: clojure
tags: clojure programming machinelearning kaggle quora
---
I've had some history with Kaggle competitions.  I worked the introductory Titanic competition using Naieve Bayes qualifiers to predict who would survive the Titanic disaster.  Last year, I began working on the [Bosch Production Line Performance](https://www.kaggle.com/c/bosch-production-line-performance)
trying to determine, based on line test data, which products were going to fail quality control measurements.  This was an interesting and
different sort of problem in that it is a classification problem where almost all of the data falls into one category. After going through the
work to process the training file and generate the rules for identifying failures, I lost interest and didn't write the program
that would apply those rules to the test data and generate a file to submit.

For the Quora competion, I'll eliminate the problem of losing interest in writing the infrastructure code to build a submission file by
writing that test submission program first.  This feels like a very agile approach.  Deploy early and often.  It also feels like a macro
level version of TDD.  The test in this case is not a standard unit test, but rather a program that generates a file to submit in the 
competition.

The similarity to TDD is this program calls a classification function that doesn't exist.  There is an Uncle Bob Martin blog post called 
the [Transformation Priority Premise](https://8thlight.com/blog/uncle-bob/2013/05/27/TheTransformationPriorityPremise.html)
which lists code enhancements that should be made to code in order to pass a failing test.  The first transformation goes from having no
code at all, to having code that returns nil.  Since the requirements for submission files do not accept nil, the first transformation
is not an option.  The second transformation goes from nil to a constant.  This I can do.  

The first submission file marked every question pair as having the same intent and it put me immediately at the bottom of the leader 
board with a score of 28.52056.  

Now that the boring infrastructure is taken care of, the more fun work of iterating on building a better model can begin.  The first 
improvement couldn't even wait until this writeup was done.  Changing the constant to saying that all question pairs are not related
landed in position 979 with a score of 6.01888 

The initial code is available on [GitHub](https://github.com/GregA100k/quargle)
