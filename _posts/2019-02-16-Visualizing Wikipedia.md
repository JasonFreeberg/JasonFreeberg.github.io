---
title: "Visualizing Wikipedia"
tags: technology, data science, data visualization
---

> This an introduction to a three-part guide on my favorite art project from college. ([Part 1](TODO), [Part 2](TODO), [Part 3](TODO))

## Background

Back in my third year of college I took a grad course, "Projects in Visualizing Information". I was aspiring to be a Data Scientist, so what could be better than a project-based course in visualizing data? I couldn't wait to make infographics and "tell a story with the data" just like new New York Times... boy was I wrong. On the first day our professor, [George Legrady](http://georgelegrady.com/), confidently told us, *"No bar charts, no line charts... and certainly no infographics! We will let the data express itself through art!"*. I was not only suprised, but scared shitless for my GPA because I never categorized myself as the creative type.

Our first two projects were restricted to the checkout and checkin records from the Seattle Public Library. I played within the proverbial box by referrencing projects from past students. In our final project, we were allowed __and nudged__ to use data that interests us. The catch was that the visual had to be done in three dimentions with Processing.

![Professor Legrady's art installation at the Seattle Public Library. He was given permission to use the data in this course.](https://s.hdnux.com/photos/02/42/04/663822/3/628x471.jpg)

I was free to use **any** dataset I chose. I could have gone to Kaggle's [curated list of datasets](https://www.kaggle.com/datasets), Reddit's r/datasets, or UCI's datasets for Machine Learning. But I guess 21 year-old Jason wanted to make things as difficult as possible, because for my final project collecting the data was half the battle.

## The Idea

Did you know that Philosophy is at the center of Wikipedia? If you don't believe me, pull up an article on Wikipedia, maybe your home town or alma-mater. Now click the first hyperlink in the description (skipping any parenthesized language links)... now do it again, and again. If you click through enough, you will wind up at Philosophy for 97% of Wikipedia articles. 

TODO: GIF of clicking through

(TODO: Make this a caption to the GIF) This phenomenon is actually [well-documented](https://en.wikipedia.org/wiki/Wikipedia:Getting_to_Philosophy) on Wikipedia itself. I always thought this was a neat trick, so I set out to visualize Wikipedia and this phenomenon... in 3D... in three weeks... I was in way over my head.

## Data Collection

[See the detailed guide on collecting the data here](TODO)

First things first, the data. We need information on the articles, and the professor said to *let the data speak for itself*. So what better data to use than the articles themselves? I already knew how to scrape information from web pages using Python, so I had that much going for me. 

**But** we also needed to track how each article eventually resolves back to the Philosophy article. So I decided to do the webscraping recursively: starting at Philosophy, it would grab the article text, write it to disk, then grab the URLS in the body and continue the process. When the scraper jumps to the next article, we will record the previous one as the "parent" so we know how we got there. 

TODO: Insert GIF of the terminal running the collection script.

At the end of the day, our data format will look something like the table below. `Level` tells us how many jumps we are away from Philosophy, `Parent` is the article that the webscraper was on previously, and `Text` is the raw HTML taken from within the `<body>` tags of the article. Note that Philosophy is at level 0 and has no parent. All articles at level 1 will have Philosophy as their parent.

| Title      | Level  | Parent            | Text                                                         |
|------------|--------|-------------------|--------------------------------------------------------------|
| Philosophy | 0      | None              | <p><strong>Philosophy</em> is the study of...                |
|            | 1      |  Philosophy       |                 |
|            | 1      |  Philosophy       |                 |
|            | 2      |                   |                 |

It turned out that there was one more problem: Wikipedia doesn't particularly like people scraping its data and their servers will deny your scraper. To see how I got around this problem (and all the other little ones) see the full technical guide [here]().

## Data Preparation

Great, we have well-structured data. But now how the hell do we visualize something like raw HTML in 3D *and* show the relationship between the articles? Luckily I studied statistics in college, which is well, the study of data. Between my courses and self learning I was exposed to Natural Language Processing (NPL), a discipline that uses Statistics and Computer Science to analyze text and 

## Data Visualization



## Summary

