---
layout: post
title:  "Word Vector Size vs Vocabulary Size in word2vec"
date:   2016-03-17 22:00:00
categories: wordvector
tags: wordvector word2vec
---
Inspiration
===========
This post was inspired by Stack Overflow question [Why does word2vec vocabulary length is different from the word vector length](
http://stackoverflow.com/questions/36013211/why-does-word2vec-vocabulary-length-is-different-from-the-word-vector-length/36021403)

I have read a number of word vector related papers and felt that this was something I should have been able to just answer.  The problem was I couldn’t.  So back to the research papers.

Before I was able to research, there was of course and answer.  The problem was, I didn’t understand it very well.  The clarifying edit helped, and it 
agreed with the second answer saying that the smaller vector size was largely to reduce computation and increase performance.

Increasing performance makes sense, but then there’s the follow up question, How does it pick what goes into the vector if it is not a relationship with all the other words in the vocabulary?

So, now back to the papers.

A Neural Probabilistic Language Model
=====================================
[Bengio et al](http://www.cs.columbia.edu/~blei/seminar/2016_discrete_data/readings/BengioDucharmeVincentJanvin2003.pdf) starts right off in the introduction talking about the curse of dimensionality.  That goes well with the performance points from the Stack Overflow answers.  This should be easy.

In the section about the curse of dimensionality is the following:

>The feature vector represents different aspects of the word: each word is associated with a point in a vector space. The number of features (e.g. m = 30, 60 or 100 in the experiments) is much smaller than the size of the vocabulary (e.g. 17,000).

In the calculations section, the feature vectors are represented by C a mapping from any element in the vocabulary to a real vector C(i).  Each C(i) is a 
Real Number vector of size m. The problem is there is no mention about how C is created or how m is selected.


From Frequency to Meaning: Vector Space Models of Semantics
===========================================================
[Turney and Pantel](http://www.jair.org/media/2934/live-2934-4846-jair.pdf) is a survey of the research of it’s time.  Section 4 describes a number of techniques for improving the results of the word frequency matrix culminating with:

> Deerwester et al. (1990) found an elegant way to improve similarity measurements with a mathematical operation on the term{document matrix, X, based on linear algebra. The operation is truncated Singular Value Decomposition (SVD), also called thin SVD.

Status
======
The initially calculated word vectors are the same size as the Vocabulary, but
they are too large to be efficient both for training the neural networks and 
for general use.

Onto the word2vec paper (at least it’s the first result when searching for word2vec paper)

Distributed Representations of Words and Phrases and their Compositionality
===========================================================================
[Mikolov et al]( http://www.cs.columbia.edu/~blei/seminar/2016_discrete_data/readings/MikolovSutskeverChenCorradoDean2013.pdf)

> Unlike most of the previously used neural network architectures for learning word vectors, training of the Skip-gram model (see Figure 1) does not involve dense matrix multiplications.

Rather than build up the word vectors, condense them down, and use them to build model, word2vec builds the word vectors in the hidden layer of it's neural network.  This is a much more efficient process and any loss in quality is made up for by training on orders of magnitdue more data.

This also explains why [deeplearning4j](http://deeplearning4j.org/word2vec) calls the parameter for the number of featuers in a word vector layerSize.

Alternatives
============

There is another way to pick the feature set.  Know ahead of time what you want to study.  [Gigasquid](http://gigasquidsoftware.com/blog/2016/02/10/fairy-tale-word-vectors/) chose the nouns in fairy tale books.  [A Probabilistic Model for Semantic Word Vectors](http://ai.stanford.edu/~ang/papers/nipsdlufl10-ProbabilisticModelSemanticWordVectors.pdf) pre-processes their
input data, 50K movie reviews, to select out the features they want to study.

> we build our dictionary by keeping the 20,000 most frequent unigram tokens without stop word removal.

Conclusions
===========
The initial thoughts in the Stack Overflow question, word vector size and 
vocabulary size should be the same, is true but not practical.  Initially, 
the vocabulary size vectors were built and then reduced, but word2vec 
is more efficient and builds a smaller size feature vector while it trains
its neural network.

I have a new personal record for time spent researching a Stack Overflow question.
