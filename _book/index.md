--- 
title: "Learning Data Science with R"
author: "Peter Baumgartner"
date: "2021-12-29"
site: bookdown::bookdown_site
documentclass: book
bibliography: [book.bib, packages.bib]
# url: your book url like https://bookdown.org/yihui/bookdown
# cover-image: path to the social sharing image like images/cover.jpg
description: |
  This is a minimal example of using the bookdown package to write a book.
  The HTML output format for this example is bookdown::bs4_book,
  set in the _output.yml file.
biblio-style: apalike
csl: chicago-fullnote-bibliography.csl
---

# Preface {.unnumbered}

## Goal of this book {.unnumbered}

The bookdown project "Learning Data Science with R" will function as my personal notebook for learning data science with R. 

Judging my knowledge state personally, I believe I am currently (December 2021) on an intermediate level in R. This in-between status of my working knowledge (not a beginner and not an expert) is precisely the reason why I started this notebook. It is difficult to advance on this intermediate level as a self-determined learner. 

Reading newly published books on data science is not as effective as I would wish. Much of the material I already know, so I have to look for the hidden gems of knowledge that are new for me. But more importantly, some of the books do not incorporate the modern trend I am interested in: Using the `tidyverse` and `tidymodels` approach for data wrangling, data analysis, and data modeling.

Furthermore is my statistical knowledge poor. The historical reason is my distrust of the NHST (Null Hypothesis Significance Test) at a time I didn't even know of this notion and other related issues like p-hacking. Therefore I am also interested to learn more about the differences between frequentist and Bayesian approaches. 

Up to now, I have written personal notes and ran code examples by accompanying specific books. But in the last weeks, it turned out that this approach is not very practical. I am exposed to new material from many different sources (books, blogs, vignettes of packages, etc.), where I want to jot down my reflections and experiment with the code snippets. Therefore I thought that a general (meta) notebook would help me advance more effectively.

## Conventions Used In This Book {.unnumbered}

Colored paragraphs with icons give you a visual overview of things to watch out:

::: {.successbox}
This green-colored block summarizes crucial steps and is structured often as an ordered checklist.
:::

::: {.infobox}
The blue block offers you some essential information, tip, or hint. 
:::

::: {.warningbox}
The yellow-colored block tells you how to avoid troubles before it starts.
:::

::: {.stopbox}
The red-colored block explains error messages and how to recover from the problem.
::: 


:::{.todobox}
The pink-colored block is a reminder to do some specified action.
:::



The following conventions are taken from [Rmarkdown](https://bookdown.org/yihui/rmarkdown/software-info.html) [@Xie_Allaire_Grolemund_2018],  another book written with **{bookdown}**.

-   There are no prompts (`>` and `+`) to R source code.
-   The text output is commented out with two hashes `##` by default. This is for your convenience when you want to copy and run the code. The text output will be ignored since it is commented out.
-   Inline code and filenames are formatted in a typewriter font (e.g., `knitr::knit('foo.Rmd')`).
-   Package names are in bold face and surrounded by curly brakes (e.g., **{rmarkdown}**).
-   Function names are followed by parentheses (e.g., `bookdown::render_book()`). The double-colon operator `::` means accessing an object from a package.

::: {.graybox}
-   To distinguish my personal notes from (copied) text taken from the book, I highlight it with a gray background.
:::

