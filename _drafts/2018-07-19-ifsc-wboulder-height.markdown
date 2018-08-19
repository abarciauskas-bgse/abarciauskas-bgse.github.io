---
title: ":women_climbing: Does height matter in women's bouldering?"
layout: post
date: 2017-10-22 21:37
image: 
headerImage: false
tag:
- climbing
- data-science
- statistics
projects: false
description: Exploration of IFSC data
category: blog
author: aimeebarciauskas
externalLink: false
---

When not climbing or computing, I can often be found watching the [IFSC (International Federation of Sports Climbing)](http://www.ifsc-climbing.org/) World Cup Competitions.

IFSC world cup competitions, streamed on YouTube, are narrated by Charlie Bosco. Charlie often mentions the populare concern that taller climbers have an advantage in climbing. Given the objective is to get to the top of a wall, it seems reasonable that taller climbers have an advantage holds are far apart. However, there is an assumption route setters design problems to create equilibrium in the advantages or disadvantages given to climbers of different heights. 

## Does being tall help climbers win world cups?

I decided to look at the data to answer this question: Are taller climbers really at an advantage when it comes to IFSC world cup competitions?

It appears the answer is no, or at least the advantage is not statistically significant.

We can see that there is a difference in means for different groups of female boulderers, however in statistical tests these differences were not found to be statistically significant.

![barplot of heights](https://raw.githubusercontent.com/abarciauskas-bgse/ifsc/master/analysis/wboulder_heights.png)

## Methodology

### Collect data from the IFSC Website

The first step was to get the data. As always, getting the data took by far the most amount of time (and even more time than it should have because I spilled water on my laptop and lost the original code I wrote). To scrape the data I used python and selenium to iterate through web pages on the IFSC website. I stored data in a local PostgreSQL database in 3(ish) tables:

1. Athletes
2. Competitions
3. Competition Results

I have "3ish" tables because competition results are stored in multiple tables - 1 per competition category in the set bouldering, lead and speed. (Although I haven't stored speed results yet.) This was necessary to store result types - in bouldering we have, for each boulder, tops, top attempts, bonuses and bonuse attempts. In lead we just have a single value result for the highest position the climber reached.

### Run many t-tests to generate many p-values

I ran 10000 iterations of a t-test randomly sampling from the following comparision popultions:

1. All women competitors vs women who ranked first in finals or semifinals
1. All women competitors vs women who ranked first in finals or semifinals


