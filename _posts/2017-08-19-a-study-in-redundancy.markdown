---
title: ":page_facing_up: Redundancy in Government Documents"
layout: post
date: 2017-08-19 19:10
tag:
- data-science
- nlp
- text-mining
projects: true
hidden: true # don't count this post in blog pagination
description: "It's not as dry as it sounds!"
category: project
author: aimeebarciauskas
externalLink: false
---

## The Project

Government documents are hard to read. When government documents are made public, they are hardly ever crafted to make them easy for the public to digest.

Text mining may be able to help citizens understand large corpii of government documents. Repetition in these texts conveys summary information about how a government entity operates and the actions of the government entity. Using Barcelona's Municipal Gazette and the minutes of the Federal Open Market Committee (FOMC), my data science master's practicum surfaced redundancies in these documents using the Needleman-Wunsch algorithm.

### Why those data sources?

* **Barcelona's Municipal Gazette:** I was working with a researcher at the Barcelona Supercomputing Center on this project, and they were interested in analyzing this corpus.
* **Minutes of the FOMC:** I don't speak very good Catalan (or any to be more accurate) and thus required an English data source to sanity check my work. Minutes of the FOMC are available in plain text format online making this an accessible data source to test my methodologies.

### Why Needleman-Wunsch?

The NW is a dynamic programming algorithm which finds the best global alignment in two sequences of text and returns a score indicating how similar or dissimilar they are. This algorithm was relatively simple to implement, understand and derive reasonable inferences from.

The scores returned however needed a reasonable threshold for redundancy. To determine this I used the human crowdsourcing platform Pybossa to poll experts and derive a threshold indicating when 2 sequences of text were redundant.


### What were the results?

I found this methodology to be successful for the minutes of the FOMC. Once all pairs of sentences had been scored, I found clusters of text sequences, in other words, groups sentences that were redundant with each other and identified centroids of those clusters as representatives of the cluster.

For the minutes of the FOMC, the centroids provided clear summary information about the operations and responsibilities of the FOMC. For example, three of the top 5 centroids were:

> The Manager also reported on developments in domestic financial markets and on System open market transactions in government securities and federal agency obligations during the period December 14, 2004 to February 1, 2005.

> There were no open market operations in foreign currencies for the Systems account in the period since the previous meeting.

> At its June meeting, the Federal Open Market Committee decided to increase the target level of the federal funds rate 25 basis points, to 3 percent.

## Final paper and presentation

The final paper and presentation can be found in [this google drive folder](https://drive.google.com/open?id=0B3JcGDJtrkntaEE1LVFoR2kwMHc).

## Some historical context

I wrote the original version of this post a year ago (almost, in six days!). In that post I wrote:

> **Inspired:** This project was hugely inspiring to helping define goals I want to achieve personally. Specifically, I wish to pursue work facilitating access to social goods and services.

I'm proud to say that's what I found with [Nava PBC](http://www.navapbc.com/), where we work on improving government services through user research and technology.

