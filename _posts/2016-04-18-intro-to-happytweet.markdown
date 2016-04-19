---
layout: post
title:  "HappyTweet"
date:   2016-04-18 22:00:00
categories: happytweet
tags: sentiment
---
Intro
=====
HappyTweet is a project for doing sentiment analysis on Twitter data.  

Inspiration
===========
The name for the project is inspired by a Steve Martin bit:
[Happy Feet](https://www.youtube.com/watch?v=xnksquL557s&spfreload=10)
and I've imagined a logo based on the sliding footprint pictures used by
Go Feet Records on the back of [English Beat](http://englishbeat.net/) 
records.  That's probably getting a little ahead of the game though.

Research
========
Interesting articles about Word Vectors are what got this project
started in theory but also what prevented it from starting in practice.
I was spending a lot of time reading but not actually doing anything.  Then,
in a pretty short time span, I can across this blog post, 
[Imagining your future projects is holding you back]( http://jessicaabel.com/2016/01/27/idea-debt)  
and an episode of the Giant Robots Podcast 
titled [Always be Hustling](http://giantrobots.fm/171) where the guest
talked about saying 'do it anyway' when he didn't want to do things. Rather
than stressing and putting off a task he says 'F!!! it. Do it anyway' 
Accept that it may not be the best or be perfect, but doing it will
be better than not doing it.

So, prodded forward by these to items, I started down the path toward 
something even less than the agile minimum viable product.  This
is beginning with something I know is not good, but it will
provide a base to iterate and improve on.

Iteration 1
===========
The simplest form
of analysis that I could think of is to take each word in a Tweet and
look up a known sentiment value for that word.  Summing up all the words in
the Tweet gives an overall score.  

Two things are needed to build this version.  One is a set of scored 
Tweets and the other is a list of words with sentiment scores.  I found
a list tweets at [Sentiment140](http://help.sentiment140.com/for-students/)
There is a link to [test data](http://cs.stanford.edu/people/alecmgo/trainingandtestdata.zip)
that was used in the Sentiment140 research.  

The list of words was downloaded from [crr.ugent.be](http://crr.ugent.be/archives/1003 )
where they have collected

>affective norms of valence, arousal, and dominance for 13,915 English words 

The words are scored from 1 to 9 with 9 being the happiest.

For my algorithm, I like to think of happy as positive and unhappy as negative
so, I shifted the scale. Rather than having neutral being at 5
I subtract 5 to make neutral 0.  

As expected, it didn't perform very well.  Only 46% of the tweets were 
scored correctly.

What's Next
===========
First, I'd like to look at ways to increase the number of words that affect
the score.  I'm thinking about finding a library that calculates lemmas so that
simple variations of words can be converted into the standard form that has
been scored.  Another approach to that same problem is to use word vectors
to find similar words that are scored.
