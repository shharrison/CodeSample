Code Sample
================
Seth Harrison
05/03/2020

# Tidy Data

``` r
Data <- read_csv("SyriaData.csv") %>%
        select(c(1,5,8,9)) %>%
        na.omit()

stop_words <- read_lines("ARstopwords") %>%
              as_data_frame()

stop_words <- rename(stop_words, text = value)

tokens <- Data %>%
   unnest_tokens(output = text, input = text) %>%
   # remove stop words
   anti_join(stop_words) %>%
   # stem the words
   corpus::as_corpus_frame() %>%
   mutate(word = corpus::text_tokens(text, stemmer = "ar"))

tokens$word <- as.character(tokens$word)
```

# Create Document-Term Matrix

``` r
dtm <- tokens %>%
   # get count of each token in each document
   count(id, word) %>%
   # create a document-term matrix with all features and term frequency inverse document frequency 
   cast_dtm(document = id, term = word, value = n, weighting = tm::weightTfIdf)

  # remove sparse terms
dtm <- removeSparseTerms(dtm, sparse = .99)
```

# Estimate model

``` r
slice <- slice(Data, 1:614)

rf <- train(x = as.matrix(dtm),
                     y = factor(slice$classification),
                     method = "ranger",
                     num.trees = 200,
                     importance = "impurity",
                     trControl = trainControl(method = "oob"))

rf$finalModel
```

    ## Ranger result
    ## 
    ## Call:
    ##  ranger::ranger(dependent.variable.name = ".outcome", data = x,      mtry = min(param$mtry, ncol(x)), min.node.size = param$min.node.size,      splitrule = as.character(param$splitrule), write.forest = TRUE,      probability = classProbs, ...) 
    ## 
    ## Type:                             Classification 
    ## Number of trees:                  200 
    ## Sample size:                      614 
    ## Number of independent variables:  121 
    ## Mtry:                             2 
    ## Target node size:                 1 
    ## Variable importance mode:         impurity 
    ## Splitrule:                        gini 
    ## OOB prediction error:             51.14 %
