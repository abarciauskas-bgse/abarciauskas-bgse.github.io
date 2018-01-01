---
title: ":earth_americas: Topic Modeling in the Real World"
layout: post
date: 2018-01-01 16:21:16
image: 
headerImage: false
tag:
- nlp
- text-mining
- lda
projects: false
description: Exploring LDA
category: blog
author: aimeebarciauskas
externalLink: false
---

# Topic Modeling in the real world

At Nava, my team is working on an API for [Medicare's Quality Payments Program (QPP)](qpp.cms.gov). Some of my superhuman teammates support developers integrating with our API through a the [QPP APIs Google Group](https://groups.google.com/forum/#!forum/qpp-apis).

Searching for a tough nut to crack, I felt developing **an understanding of the QPP APIs google group discussions** would be a challenging and valuable project.

Plus - it would be an opportunity for topic modeling in real life!

## LDA: A topic modeling methodology for text

I applied topic modeling to QPP APIs Google Group forum text with the goal of understanding what was being discussed in the 239 (and growing daily) forum threads.

Latent Dirichlet Allocation (LDA) is perhaps the most well-known algorithm for [topic modeling](https://en.wikipedia.org/wiki/Topic_model). LDA is an unsupervised learning algorithm that discovers clusters of words which commonly appear together - in the same "document". Clusters of these words comprise the topics discovered by the algorithm.

What is a document? It depends on the corpus of text being analyzed. In the case of the QPP APIs Google Group, a document was the content of an single post, whether it be a topic or a response.

## How does the algorithm work?

A machine learning algorithm can often be decomposed into the following stages:

**1. Data retrieval and preparation**

**2. Variable setup or initialization** (This step often involves generating some random variables.)

**3. An update loop** (This step often updates the random variables to be less random.)

### LDA Step 1: Data retrieval and preparation

* Retieve a corpus (or collection) of documents. [Tokenize](https://nlp.stanford.edu/IR-book/html/htmledition/tokenization-1.html) and otherwise process the documents to achieve the most meaningful results. For example, this step may involve removing numeric characters.
* Generate a bag of words matrix. The matrix dimensions are `D x W` where `D` is the number of documents and `W` is the number of unique tokens. This matrix serves to generate a vocabulary index for each unique token in the corpus.

### LDA Step 2: Initialization

Define the following:
* Hyperparameters
  * `k` - A user-defined number of topics. A magic hyperparameter, in theory it represents the number of topics underlying the corpus's generative process.
  * `alpha` and `eta` - hyperparameters
* `document_vocabulary_indices`: A D-length array with variable-length inner arrays. Each inner array is a list of integers representing the vocabulary index for each token in the document.
* `topic_assignments`: A D-length array with variable-length inner arrays. Each inner array is a list of integers representing the document token's topic assignment.
* `document_topic_counts`: A D x K matrix having counts of tokens assigned to topic k for document d.
* `word_topic_counts`: A W x K matrix having counts of token `w` appear in topic `k`.

### Step 3: Loop!

Re-assign words to topics via [collapsed Gibbs sampling](https://en.wikipedia.org/wiki/Gibbs_sampling#Collapsed_Gibbs_sampler).

For `i` in iterations, `d` in `document_word_indices` and `w` in `d` do:
* `w_i` - vocabulary index for token in `d`
* `current_topic_assignment = topic_assignments[d][w_i]`
* Decrement count for `d,current_topic_assignment` in `document_topic_counts`
* Decrement count for `w_i,current_topic_assignment` in `word_topic_counts`
* Calculate `a = p(topic t | document d)`: proportion of words in document `d` that are assigned to topic `t`
* Calculate `b = p(word w| topic t): proportion of assignments to topic t, over all documents d, that come from word w`
* Calculate the K probabilities that w belongs to each k topic (`p_z = b * a`)
* `new_topic_assignment` is a sample from multinomial with K probabilities `p_z/sum(p_z)`
* Re-assign `topic_assignments[d,token_index] = new_topic_assignment`
* Increment counts for `new_topic_assignment` in `document_topic_counts` and `word_topic_counts`

### The code

I've written code for the LDA algorithm using collapsed Gibbs sampling, hosted [here](https://github.com/abarciauskas-bgse/abarciauskas-bgse.github.io/blob/master/jupyter/LDA%20Example.ipynb). However, in practice, I recommend using sklearn's [LatentDirichletAllocation](http://scikit-learn.org/stable/modules/generated/sklearn.decomposition.LatentDirichletAllocation.html) class.

## Is LDA useful?

The general consensus is that, like a lot of data science methods, LDA raises questions more often than it delivers answers. Raising questions is useful, but we are often in search of deeper understanding, so new questions as an outcome can be frustrating. Welcome to data science.

As detailed below, I did find LDA helpful in summarizing QPP APIs Google Groups forums' content - content I cannot myself read in entirety. However, it is not really a replacement for the content itself.

## So what are people discussing in the QPP APIs Google Groups?

I applied LDA to the QPP APIs Google Groups posts and found the following discernable topics:

<table border="1" class="table" cellpadding="10px" cellspacing="0" title=""><tr><td rowspan="1" colspan="1"><div><p style="font-weight:bold">Topic Summary</p></div></td><td rowspan="1" colspan="1"><div><p style="font-weight:bold">Tokens in Topic</p></div></td></tr><tr><td rowspan="1" colspan="1"><div><p>The sandbox topic</p></div></td><td rowspan="1" colspan="1"><div><p>submissions https qpp cms gov ap sandbox error navapbc endpoint</p></div></td></tr><tr><td rowspan="1" colspan="1"><div><p>The scoring topic</p></div></td><td rowspan="1" colspan="1"><div><p>value score eligiblepopulation decile null points performancenotmet true performancemet denominator</p></div></td></tr><tr><td rowspan="1" colspan="1"><div><p>The measures data topic</p></div></td><td rowspan="1" colspan="1"><div><p>measures data measure submit performance cmsgov github json master strata</p></div></td></tr><tr><td rowspan="1" colspan="1"><div><p>The data submission topic</p></div></td><td rowspan="1" colspan="1"><div><p>submission measurement sets set file request data using use submit</p></div></td></tr><tr><td rowspan="1" colspan="1"><div><p>The feedback and communication topic</p></div></td><td rowspan="1" colspan="1"><div><p>thanks update team like issue question reply measure feedback lot</p></div></td></tr><tr><td rowspan="1" colspan="1"><div><p>The forum / "navahq.com" topic</p></div></td><td rowspan="1" colspan="1"><div><p>com http navahq apis google groups forum qpp topic https</p></div></td></tr></table>


## Resources:

* [Latent Dirichlet Allocation - under the hood (Andrew Brooks)](https://brooksandrew.github.io/simpleblog/articles/latent-dirichlet-allocation-under-the-hood/)
* [Text Mining 101: Topic Modeling (kdnuggets.com)](https://www.kdnuggets.com/2016/07/text-mining-101-topic-modeling.html)
* [Latent Dirichlet Allocation (original paper by Blei, Ng and Jordan)](http://www.jmlr.org/papers/volume3/blei03a/blei03a.pdf)

