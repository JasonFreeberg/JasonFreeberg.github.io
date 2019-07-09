---
title: "Visualizing Wikipedia"
tags: 
    - technology
    - data science
    - data visualization
---

> This an introduction to a three-part guide on my favorite art project from college. ([Part 1](TODO), [Part 2](TODO), [Part 3](TODO))

## Background

Back in my third year of college I took a grad course, "Projects in Visualizing Information". I was aspiring to be a Data Scientist, so what could be better than a project-based course in visualizing data? I couldn't wait to make infographics and "tell a story with the data" just like new New York Times... boy was I wrong. In the first lecture our professor, [George Legrady](http://georgelegrady.com/), confidently told us, *"No bar charts, no line charts... and certainly no infographics! We will let the data express itself through art!"*. I was not only surprised, but scared shitless for my GPA because I never categorized myself as the creative type.

Our first two projects were restricted to the check-out and check-in records from the Seattle Public Library. I played within the proverbial box by ~~copying~~ drawing inspiration from projects done by past students. In our final project, we were allowed __and nudged__ to use data that interested us. The catch was that the visual had to be done in three dimensions with Processing.

![Professor Legrady's art installation at the Seattle Public Library. He was given permission to use the data in this course.](https://s.hdnux.com/photos/02/42/04/663822/3/628x471.jpg)

I was free to use **any** dataset I chose. I could have gone to Kaggle's [curated list of datasets](https://www.kaggle.com/datasets), Reddit's r/datasets, or UC Irvine's data library for Machine Learning. But I guess 21 year-old Jason wanted to make things as difficult as possible, because half the battle was collecting and preparing the data.

## The Idea

Did you know that Philosophy is at the center of Wikipedia? If you don't believe me pull up an article on Wikipedia, maybe your home town or alma-mater. Now click the first hyperlink in the description (skipping any parenthesized phonetic spelling)... now do it again, and again. If you click through enough, you will wind up at Philosophy for 97% of Wikipedia articles.

TODO: GIF of clicking through
(TODO: Make this a caption to the GIF) This phenomenon is actually [well-documented](https://en.wikipedia.org/wiki/Wikipedia:Getting_to_Philosophy) on Wikipedia itself.

I always thought this was a neat trick, so I set out to visualize Wikipedia and this phenomenon... in 3D... in three weeks... I was in way over my head.

## Data Collection

[See the detailed guide on collecting the data here](TODO)

First things first: data. We need information on the articles, and the professor said to *let the data speak for itself*. So what better data to use than the articles themselves? I already knew how to scrape information from web pages using Python, so I started planning the data format. 

In addition to the article text, we also needed to track how each article eventually resolves back to the Philosophy article. So the webscraping had to be done recursively: starting at Philosophy, it would grab the article text, write it to disk, then grab the URLs in the body and continue the process. When the scraper jumps to the next article, we will record the previous one as the "parent" so we know how we got to its "child".

TODO: Insert GIF of the terminal running the collection script.

At the end of the day, our data will be formatted something like the table below. `Level` tells us how many jumps we are away from Philosophy, `Parent` is the article that the webscraper was on previously, and `Text` is the raw HTML taken from within the `<body>` tags of the article. Note that Philosophy is at level 0 and has no parent. All articles at level 1 will have Philosophy as their parent.

| Title      | Level  | Parent            | Text                                                         |
|------------|--------|-------------------|--------------------------------------------------------------|
| Philosophy | 0      | None              | <p><strong>Philosophy</em> is the study of...                |
|            | 1      | Philosophy        |                 |
|            | 1      | Philosophy        |                 |
| Donald Trump | 2      |                   |                 |

It turned out that there was one more problem: Wikipedia doesn't particularly like people scraping its data and their servers will deny your scraper. To see how I got around this problem (and all the other little ones) see the full technical guide [here]().

## Data Preparation

Great, now we have semi-clean data in a neat, tabular format. But how the hell do we visualize something like raw HTML in 3D *and* show the relationship between the articles? Luckily I studied statistics in college, which is... well... the study of data. During my education I was eventually exposed to Natural Language Processing (NLP), a discipline that uses Statistics and Computer Science to process and analyze human language. This is a broad field that encompasses everything from document classification to personal assistants like Siri. To prepare the data for visualization we will tokenize and vectorize the data, then reduce the dimensionality. Don't know what the means? Good! I explain it in more detail in Part 2 (*coming soon!*).

### Information Retrieval

Before extracting more information from our raw HTML, we will clean up the data by removing all the markup tags, stop words, and numbers from the data in our `Text` column. Next, we will create a dictionary of all the unique words found across all the cleaned text. Since we are parsing each unique word from each article, this dictionary will be massive! (This process is called [tokenization](https://nlp.stanford.edu/IR-book/html/htmledition/tokenization-1.html).) Using this dictionary, we can count the occurrences of every dictionary entry in each article. This will create N new columns in our dataset, where N is the number of words--or tokens--in our dictionary. For the visual learners, this is what our data will look like after the tokenization and vectorization:

| Title      | Level  | Parent            | politics |         | small | hands  |...  |
|------------|--------|-------------------|----------|---------|-------|--------|-----|
| Philosophy | 0      | None              |          | 5       |       |        |...  |
|            | 1      | Philosophy        |          |         |       |        |...  |
|            | 1      | Philosophy        |          |         |       |        |...  |
| Donald Trump | 2    |                   |          | 7       | 8     | 8      |...  |

*Note:* I also applied Term-Frequency, Inverse Document Frquency (tf-idf) to the data at this point. But the explanation for that is outside the scope of this post. See Part 2 (*coming soon!*) of the guide for more details on the information retrieval process.

### Dimensionality Reduction

Our dataset is looking better after the vectorization: instead of one column with the raw text, we have several numerical columns for each unique word. Before we can start visualizing in Processing, we need *X*, *Y*, and *Z* coordinates for each article. After the information retrieval step our dataset has thousands of columns, so how do we decide which columns to use for those coordinates? This is where we introduce [dimensionality reduction](https://en.wikipedia.org/wiki/Dimensionality_reduction) to our process. Dimensionality reduction is a category of algorithms that take as input a "wide" dataset and output a *representative* dataset with fewer columns. It is about as close to magic as we can get. Some algorithms, such as [backward selection](https://www.jmp.com/en_us/statistics-knowledge-portal/what-is-multiple-regression/variable-selection.html) will simply drop variables that are determined to not be important in predicting an output variable. Other algorithms, such as [Principle Component Analysis](TODO) or [t-SNE](https://lvdmaaten.github.io/tsne/) will project the data onto an entirely new basis, so we can represent our input dataset with fewer columns.

In this project I used Principle Component Analysis (PCA) because it allows you to select the number of columns in the output dataset (up to the number of columns you had in the input set). This is great for our visualization because we not only get *X*, *Y*, and *Z* coordinates, but we can get two more columns for free and map them to the size and color of each sphere.

## Data Visualization

[Processing](https://processing.org/) is a "programming sketchbook" and uses a watered-down version of Java. This means it provides built-in methods to draw on the screen and there is less boilerplate code to write. Processing is often used as a teaching tool for introductory programming courses because it allows students to "see" their code working and causes fewer headaches. Our visualization can be broken down into four large categories: the `Article` class, the `Edge` class, the GUI (graphical user interface) methods, and the helper methods. The [`Article` class](https://github.com/JasonFreeberg/Wiki-Graph/blob/master/data-visualization/Article.pde) abstracts the data for each sphere and provides methods for drawing and interacting with them. To visualize how the webscraper made it from one article to the next, we will use the [`Edge` class](https://github.com/JasonFreeberg/Wiki-Graph/blob/master/data-visualization/Edge.pde). I grouped the methods for searching and filtering articles into the [GUI methods](https://github.com/JasonFreeberg/Wiki-Graph/blob/master/data-visualization/GUI_methods.pde) file. Everything else is thrown into the "helpers" file.

## Summary

This was meant to be a quick introduction to the project guide, but ended up being much longer than expected! I wanted to do this because back in college I started the Data Science club, and one of our big tenants was to write up and share your projects online for others to learn from. So partially out of guilt, this is the start of my guide to visualizing wikipedia.

If you are starting your data science journey, this project will introduce you to a variety of data science techniques and programming skills. Not only that, but there are plenty of places where you can improve on this project. At the end of each section I will call out things that you can try out on your own. Here's to learning by doing!