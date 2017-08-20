---
title: ":art: Visualizing neural networks (Part 1)"
layout: post
date: 2017-08-20 17:34
image: /assets/images/neurons/output_iH6chi.gif
headerImage: true
tag:
- data-science
- deep-learning
- visualization
projects: true
hidden: true # don't count this post in blog pagination
description: "Sketches from my master's project"
category: project
author: aimeebarciauskas
externalLink: false
---

# The Project: Visualizing neural networks

## Part 1: Research and design

The [Algorritmo](http://www.bsc.es/viz/algorritmo/) project seeks to answer the question "Can computers create art?" For my final master's project, I collaborated with the [Barecelona Supercomputing Center (BSC)](https://www.bsc.es/) on an exhibit featuring Algorritmo at Sónar, a 3-day electronic and advanced music festival. Algorritmo is a word-play on rhythim and algorithm, and thus translates to Algorhythm - clever, right?

Representing complex ideas visually helps to communicate those complex ideas to others. My final project would be an interactive and explanatory visualization, using [D3.js](https://d3js.org/), of the recurrent neural network algorithm, being used by the team at BSC to create new music.

This post summarizes the first part of the project: research and design. I had to understand how neural networks work and iterate through different designs on how to visualize them.

## A simple example

The easiest way to start developing an understanding of how neural networks learn is to start with a toy example: Say we want to classify if a Pandora customer, let's call the customer Ben Folds, will like or dislike a song based on just 3 song features: Is it modern? Is it sad? Does it include piano?

The first sketch shows our existing "training" data on 4 songs for Ben Folds. Ben Folds likes 2 of the 4 songs and dislikes the other 2. Can this information help us predict if he will like other songs based on the same 3 features?

![data.png](/assets/images/neurons/data.png)

If we visualize this data in a plot, we can see that the training data we have so far will make it hard to predict what Ben Folds will like.

![data_plot.png](/assets/images/neurons/data_plot.png)

We want to learn some rules which will enable Pandora to queue new songs that Ben Folds will like. Unfortunately, no rules can be learned from this data set:

* All 4 songs have piano, so this feature is not helpful at all.
* Ben Folds likes one of the sad songs, but he dislikes the other sad song.
* Ben Folds likes one of the modern songs, but he dislikes the other modern song.

## Learning a decision boundary

With better training data, the neural network learns a "decision boundary". A decision boundary can be thought of as a set of rules which enable us to make predictions as to a certain outcome - for example if Ben Folds will like a new song. A decision boundary would have the majority of liked songs on one side and a majority of disliked songs on the other side.

![plane.JPG](/assets/images/neurons/plane.JPG)

How does a neural network learn where to put the decision boundary? Outputs of the training data are represented via 0's and 1's in our toy example - 1 means Ben Folds likes the song, 0 means Ben Folds dislikes the song. In addition to the outputs, we have the inputs as 0's and 1's indicating features present in each song, and a set of weights. The set of weights is initialized as a set of random numbers and one weight is associated with each feature of the data. In our toy example we have 3 weights because we have 3 features: modern, sad, piano.

The first iteration of the neural network algorithm uses the random weights to make song preference predictions for Ben Folds. Based on how wrong the predictions are (this is called the "loss"), the algorithm adjusts the weights up or down. For example, if the set of weights first weighted piano heavily and the first song featured piano but Ben Folds hates that song, the weight corresponding to the piano feature would be reduced.

Below is a representation of the steps of this process in a computational graph:

![output](/assets/images/output_xCrmbs.gif)

## Sketches

With a grasp on how neural networks work, I could start to come up with a design for the interactive visualization.

I sketched out multiple options for my colleagues to vet. I took inspirations from others exploring neural network visualizations and related concepts, like neurons, brain waves and circuits:

![simple neuron](/assets/images/output_laxktO.gif)
![another animation](/assets/images/neurons/output_iH6chi.gif)
![another animation](/assets/images/neurons/output_5XLW65.gif)
![full page view](/assets/images/neurons/full.JPG)
![waves1](/assets/images/neurons/waves1.png)
![waves2](/assets/images/neurons/waves2.png)
![circuit](/assets/images/neurons/circuit.png)
![waves3](/assets/images/neurons/waves3.png)

