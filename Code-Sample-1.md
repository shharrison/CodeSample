Code Sample 1
================
Seth Harrison
4/30/2020

# Tidy Data

``` r
Data <- read_csv("SyriaData.csv")

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
```

#
