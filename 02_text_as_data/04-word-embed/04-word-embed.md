---
title: "Text as Data: Word embedding"
subtitle: "SICSS-Oxford, 2021"
author:
  name: Christopher Barrie
  affiliation: University of Edinburgh | [SICSS](https://github.com/cjbarrie/sicss_21)
output: 
  html_document:
    theme: flatly
    highlight: haddock
    # code_folding: show
    toc: yes
    toc_depth: 4
    toc_float: yes
    keep_md: true
    
bibliography: CTA.bib    
---


# Exercise 4: Word embedding

## Introduction

In this tutorial, you will learn how to:

* Load pre-trained word embeddings
* Train a local word embedding model
* Visualize and inspect results
* Determine over-time trends

Borrows from tutorial by Chris Bail [here](https://cbail.github.io/textasdata/word2vec/rmarkdown/word2vec.html) and Julia Silge [here](https://juliasilge.com/blog/tidy-word-vectors/).

## Setup 


```r
library(tidyverse) # loads dplyr, ggplot2, and others
library(stringr) # to handle text elements
library(tidytext) # includes set of functions useful for manipulating text
library(ggthemes) # to make your plots look nice
library(text2vec) # for word embedding implementation
library(widyr) #for reshaping the text data
library(irlba) #for svd
```


```r
twts_sample <- readRDS("data/twts_corpus_sample.rds")

#create tweet id
twts_sample$postID <- row.names(twts_sample)
```


```r
#create context window with length 6
tidy_skipgrams <- twts_sample %>%
    unnest_tokens(ngram, tweet, token = "ngrams", n = 6) %>%
    mutate(ngramID = row_number()) %>% 
    tidyr::unite(skipgramID, postID, ngramID) %>%
    unnest_tokens(word, ngram)
```




```r
#calculate unigram probabilities (used to normalize skipgram probabilities later)
unigram_probs <- twts_sample %>%
    unnest_tokens(word, tweet) %>%
    count(word, sort = TRUE) %>%
    mutate(p = n / sum(n))

#calculate probabilities
skipgram_probs <- tidy_skipgrams %>%
    pairwise_count(word, skipgramID, diag = TRUE, sort = TRUE) %>%
    mutate(p = n / sum(n))

#normalize probabilities
normalized_prob <- skipgram_probs %>%
    filter(n > 20) %>%
    rename(word1 = item1, word2 = item2) %>%
    left_join(unigram_probs %>%
                  select(word1 = word, p1 = p),
              by = "word1") %>%
    left_join(unigram_probs %>%
                  select(word2 = word, p2 = p),
              by = "word2") %>%
    mutate(p_together = p / p1 / p2)

normalized_prob %>% 
    filter(word1 == "brexit") %>%
    arrange(-p_together)
```

```
## # A tibble: 1,016 x 7
##    word1  word2                      n           p      p1         p2 p_together
##    <chr>  <chr>                  <dbl>       <dbl>   <dbl>      <dbl>      <dbl>
##  1 brexit brexit                 38517 0.000499    0.00279 0.00279          64.0
##  2 brexit softer                    50 0.000000648 0.00279 0.00000484       48.0
##  3 brexit dividend                 176 0.00000228  0.00279 0.0000201        40.7
##  4 brexit scotlandsplaceineurope    37 0.000000479 0.00279 0.00000446       38.5
##  5 brexit botched                  129 0.00000167  0.00279 0.0000208        28.7
##  6 brexit gridlock                  30 0.000000389 0.00279 0.00000521       26.7
##  7 brexit deadlock                 120 0.00000155  0.00279 0.0000242        23.0
##  8 brexit preparedness              22 0.000000285 0.00279 0.00000446       22.9
##  9 brexit soft                      89 0.00000115  0.00279 0.0000190        21.8
## 10 brexit weaken                    24 0.000000311 0.00279 0.00000521       21.4
## # … with 1,006 more rows
```



```r
pmi_matrix <- normalized_prob %>%
    mutate(pmi = log10(p_together)) %>%
    cast_sparse(word1, word2, pmi)

#remove missing data
pmi_matrix@x[is.na(pmi_matrix@x)] <- 0
#run SVD
pmi_svd <- irlba(pmi_matrix, 256, maxit = 500)
```




```r
#next we output the word vectors:
word_vectors <- pmi_svd$u
rownames(word_vectors) <- rownames(pmi_matrix)

dim(word_vectors)
```

```
## [1] 21172   256
```


```r
search_synonyms <- function(word_vectors, selected_vector){
  
  mult <- as.data.frame(word_vectors %*% selected_vector)
  
  mult %>%
  rownames_to_column() %>%
  rename(word = rowname,
         similarity = V1) %>%
  anti_join(get_stopwords(language = "en")) %>%
  arrange(-similarity)

}

boris_synonyms <- search_synonyms(word_vectors, word_vectors["boris",])
```

```
## Joining, by = "word"
```

```r
brexit_synonyms <- search_synonyms(word_vectors, word_vectors["brexit",])
```

```
## Joining, by = "word"
```

```r
head(boris_synonyms)
```

```
##      word similarity
## 1 johnson 0.10309556
## 2   boris 0.09940448
## 3  jeremy 0.04823204
## 4   trust 0.04800155
## 5  corbyn 0.04102031
## 6  farage 0.03973588
```

```r
head(brexit_synonyms)
```

```
##      word similarity
## 1  brexit 0.38737979
## 2    deal 0.15083433
## 3 botched 0.05003683
## 4    tory 0.04377030
## 5 unleash 0.04233445
## 6  impact 0.04139872
```


## Exercises

1. 
2. 

## References 
