# Relationships between words: n-grams and correlations {#ngrams}

So far we've considered words as individual units, and considered their relationships to sentiments or to documents. However, many interesting text analyses are based on the relationships between words, whether examining which words tend to follow others immediately, or that tend to co-occur within the same documents.

In this chapter, we'll explore some of the methods tidytext offers for calculating and visualizing relationships between words in your text dataset. This includes the `token = "ngrams"` argument, which tokenizes by pairs of adjacent words rather than by individual ones. We'll also introduce two new packages: [ggraph](https://github.com/thomasp85/ggraph), which extends ggplot2 to construct network plots, and [widyr](https://github.com/dgrtwo/widyr), which calculates pairwise correlations and distances within a tidy data frame. Together these expand our toolbox for exploring text within the tidy data framework.

## Tokenizing by n-gram

We've been using the `unnest_tokens` function to tokenize by word, or sometimes by sentence, which is useful for the kinds of sentiment and frequency analyses we've been doing so far. But we can also use the function to tokenize into consecutive sequences of words, called **n-grams**. By seeing how often word X is followed by word Y, we can then build a model of the relationships between them.

We do this by adding the `token = "ngrams"` option to `unnest_tokens()`, and setting `n` to the number of words we wish to capture in each n-gram. When we set `n` to 2, we are examining pairs of two consecutive words, often called "bigrams":


```r
library(dplyr)
library(tidytext)
library(janeaustenr)

austen_bigrams <- austen_books() %>%
  unnest_tokens(bigram, text, token = "ngrams", n = 2)

austen_bigrams
#> # A tibble: 675,025 × 2
#>    book                bigram         
#>    <fct>               <chr>          
#>  1 Sense & Sensibility sense and      
#>  2 Sense & Sensibility and sensibility
#>  3 Sense & Sensibility <NA>           
#>  4 Sense & Sensibility by jane        
#>  5 Sense & Sensibility jane austen    
#>  6 Sense & Sensibility <NA>           
#>  7 Sense & Sensibility <NA>           
#>  8 Sense & Sensibility <NA>           
#>  9 Sense & Sensibility <NA>           
#> 10 Sense & Sensibility <NA>           
#> # … with 675,015 more rows
```

This data structure is still a variation of the tidy text format. It is structured as one-token-per-row (with extra metadata, such as `book`, still preserved), but each token now represents a bigram.

<div class="rmdnote">
<p>Notice that these bigrams overlap: “sense and” is one token, while
“and sensibility” is another.</p>
</div>

### Counting and filtering n-grams

Our usual tidy tools apply equally well to n-gram analysis. We can examine the most common bigrams using dplyr's `count()`:


```r
austen_bigrams %>%
  count(bigram, sort = TRUE)
#> # A tibble: 193,210 × 2
#>    bigram      n
#>    <chr>   <int>
#>  1 <NA>    12242
#>  2 of the   2853
#>  3 to be    2670
#>  4 in the   2221
#>  5 it was   1691
#>  6 i am     1485
#>  7 she had  1405
#>  8 of her   1363
#>  9 to the   1315
#> 10 she was  1309
#> # … with 193,200 more rows
```

As one might expect, a lot of the most common bigrams are pairs of common (uninteresting) words, such as `of the` and `to be`: what we call "stop-words" (see Chapter \@ref(tidytext)). This is a useful time to use tidyr's `separate()`, which splits a column into multiple based on a delimiter. This lets us separate it into two columns, "word1" and "word2", at which point we can remove cases where either is a stop-word.


```r
library(tidyr)

bigrams_separated <- austen_bigrams %>%
  separate(bigram, c("word1", "word2"), sep = " ")

bigrams_filtered <- bigrams_separated %>%
  filter(!word1 %in% stop_words$word) %>%
  filter(!word2 %in% stop_words$word)

# new bigram counts:
bigram_counts <- bigrams_filtered %>% 
  count(word1, word2, sort = TRUE)

bigram_counts
#> # A tibble: 28,975 × 3
#>    word1   word2         n
#>    <chr>   <chr>     <int>
#>  1 <NA>    <NA>      12242
#>  2 sir     thomas      266
#>  3 miss    crawford    196
#>  4 captain wentworth   143
#>  5 miss    woodhouse   143
#>  6 frank   churchill   114
#>  7 lady    russell     110
#>  8 sir     walter      108
#>  9 lady    bertram     101
#> 10 miss    fairfax      98
#> # … with 28,965 more rows
```

We can see that names (whether first and last or with a salutation) are the most common pairs in Jane Austen books.

In other analyses, we may want to work with the recombined words. tidyr's `unite()` function is the inverse of `separate()`, and lets us recombine the columns into one. Thus, "separate/filter/count/unite" let us find the most common bigrams not containing stop-words.


```r
bigrams_united <- bigrams_filtered %>%
  unite(bigram, word1, word2, sep = " ")

bigrams_united
#> # A tibble: 51,155 × 2
#>    book                bigram     
#>    <fct>               <chr>      
#>  1 Sense & Sensibility NA NA      
#>  2 Sense & Sensibility jane austen
#>  3 Sense & Sensibility NA NA      
#>  4 Sense & Sensibility NA NA      
#>  5 Sense & Sensibility NA NA      
#>  6 Sense & Sensibility NA NA      
#>  7 Sense & Sensibility NA NA      
#>  8 Sense & Sensibility NA NA      
#>  9 Sense & Sensibility chapter 1  
#> 10 Sense & Sensibility NA NA      
#> # … with 51,145 more rows
```

In other analyses you may be interested in the most common trigrams, which are consecutive sequences of 3 words. We can find this by setting `n = 3`:


```r
austen_books() %>%
  unnest_tokens(trigram, text, token = "ngrams", n = 3) %>%
  separate(trigram, c("word1", "word2", "word3"), sep = " ") %>%
  filter(!word1 %in% stop_words$word,
         !word2 %in% stop_words$word,
         !word3 %in% stop_words$word) %>%
  count(word1, word2, word3, sort = TRUE)
#> # A tibble: 6,141 × 4
#>    word1     word2     word3         n
#>    <chr>     <chr>     <chr>     <int>
#>  1 <NA>      <NA>      <NA>      13260
#>  2 dear      miss      woodhouse    20
#>  3 miss      de        bourgh       17
#>  4 lady      catherine de           11
#>  5 poor      miss      taylor       11
#>  6 sir       walter    elliot       10
#>  7 catherine de        bourgh        9
#>  8 dear      sir       thomas        8
#>  9 replied   miss      crawford      7
#> 10 sir       william   lucas         7
#> # … with 6,131 more rows
```

### Analyzing bigrams

This one-bigram-per-row format is helpful for exploratory analyses of the text. As a simple example, we might be interested in the most common "streets" mentioned in each book:


```r
bigrams_filtered %>%
  filter(word2 == "street") %>%
  count(book, word1, sort = TRUE)
#> # A tibble: 33 × 3
#>    book                word1           n
#>    <fct>               <chr>       <int>
#>  1 Sense & Sensibility harley         16
#>  2 Sense & Sensibility berkeley       15
#>  3 Northanger Abbey    milsom         10
#>  4 Northanger Abbey    pulteney       10
#>  5 Mansfield Park      wimpole         9
#>  6 Pride & Prejudice   gracechurch     8
#>  7 Persuasion          milsom          5
#>  8 Sense & Sensibility bond            4
#>  9 Sense & Sensibility conduit         4
#> 10 Persuasion          rivers          4
#> # … with 23 more rows
```

A bigram can also be treated as a term in a document in the same way that we treated individual words. For example, we can look at the tf-idf (Chapter \@ref(tfidf)) of bigrams across Austen novels. These tf-idf values can be visualized within each book, just as we did for words (Figure \@ref(fig:bigramtfidf)).


```r
bigram_tf_idf <- bigrams_united %>%
  count(book, bigram) %>%
  bind_tf_idf(bigram, book, n) %>%
  arrange(desc(tf_idf))

bigram_tf_idf
#> # A tibble: 31,397 × 6
#>    book                bigram                n     tf   idf tf_idf
#>    <fct>               <chr>             <int>  <dbl> <dbl>  <dbl>
#>  1 Mansfield Park      sir thomas          266 0.0244  1.79 0.0438
#>  2 Persuasion          captain wentworth   143 0.0232  1.79 0.0416
#>  3 Mansfield Park      miss crawford       196 0.0180  1.79 0.0322
#>  4 Persuasion          lady russell        110 0.0179  1.79 0.0320
#>  5 Persuasion          sir walter          108 0.0175  1.79 0.0314
#>  6 Emma                miss woodhouse      143 0.0129  1.79 0.0231
#>  7 Northanger Abbey    miss tilney          74 0.0128  1.79 0.0229
#>  8 Sense & Sensibility colonel brandon      96 0.0115  1.79 0.0205
#>  9 Sense & Sensibility sir john             94 0.0112  1.79 0.0201
#> 10 Emma                frank churchill     114 0.0103  1.79 0.0184
#> # … with 31,387 more rows
```

<div class="figure" style="text-align: center">
<img src="04-word-combinations_files/figure-html/bigramtfidf-1.png" alt="Bigrams with the highest tf-idf from each Jane Austen novel" width="90%" />
<p class="caption">(\#fig:bigramtfidf)Bigrams with the highest tf-idf from each Jane Austen novel</p>
</div>

Much as we discovered in Chapter \@ref(tfidf), the units that distinguish each Austen book are almost exclusively names. We also notice some pairings of a common verb and a name, such as "replied elizabeth" in Pride & Prejudice, or "cried emma" in Emma.

There are advantages and disadvantages to examining the tf-idf of bigrams rather than individual words. Pairs of consecutive words might capture structure that isn't present when one is just counting single words, and may provide context that makes tokens more understandable (for example, "pulteney street", in Northanger Abbey, is more informative than "pulteney"). However, the per-bigram counts are also *sparser*: a typical two-word pair is rarer than either of its component words. Thus, bigrams can be especially useful when you have a very large text dataset.

### Using bigrams to provide context in sentiment analysis

Our sentiment analysis approach in Chapter \@ref(sentiment) simply counted the appearance of positive or negative words, according to a reference lexicon. One of the problems with this approach is that a word's context can matter nearly as much as its presence. For example, the words "happy" and "like" will be counted as positive, even in a sentence like "I'm not **happy** and I don't **like** it!"

Now that we have the data organized into bigrams, it's easy to tell how often words are preceded by a word like "not":


```r
bigrams_separated %>%
  filter(word1 == "not") %>%
  count(word1, word2, sort = TRUE)
#> # A tibble: 1,178 × 3
#>    word1 word2     n
#>    <chr> <chr> <int>
#>  1 not   be      580
#>  2 not   to      335
#>  3 not   have    307
#>  4 not   know    237
#>  5 not   a       184
#>  6 not   think   162
#>  7 not   been    151
#>  8 not   the     135
#>  9 not   at      126
#> 10 not   in      110
#> # … with 1,168 more rows
```

By performing sentiment analysis on the bigram data, we can examine how often sentiment-associated words are preceded by "not" or other negating words. We could use this to ignore or even reverse their contribution to the sentiment score.

Let's use the AFINN lexicon for sentiment analysis, which you may recall gives a numeric sentiment value for each word, with positive or negative numbers indicating the direction of the sentiment.


```r
AFINN <- get_sentiments("afinn")

AFINN
```


```
#> # A tibble: 2,477 × 2
#>    word       value
#>    <chr>      <dbl>
#>  1 abandon       -2
#>  2 abandoned     -2
#>  3 abandons      -2
#>  4 abducted      -2
#>  5 abduction     -2
#>  6 abductions    -2
#>  7 abhor         -3
#>  8 abhorred      -3
#>  9 abhorrent     -3
#> 10 abhors        -3
#> # … with 2,467 more rows
```


We can then examine the most frequent words that were preceded by "not" and were associated with a sentiment.


```r
not_words <- bigrams_separated %>%
  filter(word1 == "not") %>%
  inner_join(AFINN, by = c(word2 = "word")) %>%
  count(word2, value, sort = TRUE)

not_words
#> # A tibble: 229 × 3
#>    word2   value     n
#>    <chr>   <dbl> <int>
#>  1 like        2    95
#>  2 help        2    77
#>  3 want        1    41
#>  4 wish        1    39
#>  5 allow       1    30
#>  6 care        2    21
#>  7 sorry      -1    20
#>  8 leave      -1    17
#>  9 pretend    -1    17
#> 10 worth       2    17
#> # … with 219 more rows
```

For example, the most common sentiment-associated word to follow "not" was "like", which would normally have a (positive) score of 2.

It's worth asking which words contributed the most in the "wrong" direction. To compute that, we can multiply their value by the number of times they appear (so that a word with a value of +3 occurring 10 times has as much impact as a word with a sentiment value of +1 occurring 30 times). We visualize the result with a bar plot (Figure \@ref(fig:notwordsplot)).


```r

library(ggplot2)

not_words %>%
  mutate(contribution = n * value) %>%
  arrange(desc(abs(contribution))) %>%
  head(20) %>%
  mutate(word2 = reorder(word2, contribution)) %>%
  ggplot(aes(n * value, word2, fill = n * value > 0)) +
  geom_col(show.legend = FALSE) +
  labs(x = "Sentiment value * number of occurrences",
       y = "Words preceded by \"not\"")
```

<div class="figure" style="text-align: center">
<img src="04-word-combinations_files/figure-html/notwordsplot-1.png" alt="Words preceded by 'not' that had the greatest contribution to sentiment values, in either a positive or negative direction" width="90%" />
<p class="caption">(\#fig:notwordsplot)Words preceded by 'not' that had the greatest contribution to sentiment values, in either a positive or negative direction</p>
</div>

The bigrams "not like" and "not help" were overwhelmingly the largest causes of misidentification, making the text seem much more positive than it is. But we can see phrases like "not afraid" and "not fail" sometimes suggest text is more negative than it is.

"Not" isn't the only term that provides some context for the following word. We could pick four common words (or more) that negate the subsequent term, and use the same joining and counting approach to examine all of them at once.


```r
negation_words <- c("not", "no", "never", "without")

negated_words <- bigrams_separated %>%
  filter(word1 %in% negation_words) %>%
  inner_join(AFINN, by = c(word2 = "word")) %>%
  count(word1, word2, value, sort = TRUE)
```

We could then visualize what the most common words to follow each particular negation are (Figure \@ref(fig:negatedwords)). While "not like" and "not help" are still the two most common examples, we can also see pairings such as "no great" and "never loved." We could combine this with the approaches in Chapter \@ref(sentiment) to reverse the AFINN values of each word that follows a negation. These are just a few examples of how finding consecutive words can give context to text mining methods.

<div class="figure" style="text-align: center">
<img src="04-word-combinations_files/figure-html/negatedwords-1.png" alt="Most common positive or negative words to follow negations such as 'never', 'no', 'not', and 'without'" width="90%" />
<p class="caption">(\#fig:negatedwords)Most common positive or negative words to follow negations such as 'never', 'no', 'not', and 'without'</p>
</div>

### Visualizing a network of bigrams with ggraph

We may be interested in visualizing all of the relationships among words simultaneously, rather than just the top few at a time. As one common visualization, we can arrange the words into a network, or "graph." Here we'll be referring to a "graph" not in the sense of a visualization, but as a combination of connected nodes. A graph can be constructed from a tidy object since it has three variables:

* **from**: the node an edge is coming from
* **to**: the node an edge is going towards
* **weight**: A numeric value associated with each edge

The [igraph](http://igraph.org/) package has many powerful functions for manipulating and analyzing networks. One way to create an igraph object from tidy data is the `graph_from_data_frame()` function, which takes a data frame of edges with columns for "from", "to", and edge attributes (in this case `n`):


```r
library(igraph)

# original counts
bigram_counts
#> # A tibble: 28,975 × 3
#>    word1   word2         n
#>    <chr>   <chr>     <int>
#>  1 <NA>    <NA>      12242
#>  2 sir     thomas      266
#>  3 miss    crawford    196
#>  4 captain wentworth   143
#>  5 miss    woodhouse   143
#>  6 frank   churchill   114
#>  7 lady    russell     110
#>  8 sir     walter      108
#>  9 lady    bertram     101
#> 10 miss    fairfax      98
#> # … with 28,965 more rows

# filter for only relatively common combinations
bigram_graph <- bigram_counts %>%
  filter(n > 20) %>%
  graph_from_data_frame()

bigram_graph
#> IGRAPH 17b51c0 DN-- 86 71 -- 
#> + attr: name (v/c), n (e/n)
#> + edges from 17b51c0 (vertex names):
#>  [1] NA      ->NA         sir     ->thomas     miss    ->crawford  
#>  [4] captain ->wentworth  miss    ->woodhouse  frank   ->churchill 
#>  [7] lady    ->russell    sir     ->walter     lady    ->bertram   
#> [10] miss    ->fairfax    colonel ->brandon    sir     ->john      
#> [13] miss    ->bates      jane    ->fairfax    lady    ->catherine 
#> [16] lady    ->middleton  miss    ->tilney     miss    ->bingley   
#> [19] thousand->pounds     miss    ->dashwood   dear    ->miss      
#> [22] miss    ->bennet     miss    ->morland    captain ->benwick   
#> + ... omitted several edges
```

igraph has plotting functions built in, but they're not what the package is designed to do, so many other packages have developed visualization methods for graph objects. We recommend the ggraph package [@R-ggraph], because it implements these visualizations in terms of the grammar of graphics, which we are already familiar with from ggplot2.

We can convert an igraph object into a ggraph with the `ggraph` function, after which we add layers to it, much like layers are added in ggplot2. For example, for a basic graph we need to add three layers: nodes, edges, and text.


```r
library(ggraph)
set.seed(2017)

ggraph(bigram_graph, layout = "fr") +
  geom_edge_link() +
  geom_node_point() +
  geom_node_text(aes(label = name), vjust = 1, hjust = 1)
```

<div class="figure" style="text-align: center">
<img src="04-word-combinations_files/figure-html/bigramgraph-1.png" alt="Common bigrams in Jane Austen's novels, showing those that occurred more than 20 times and where neither word was a stop word" width="90%" />
<p class="caption">(\#fig:bigramgraph)Common bigrams in Jane Austen's novels, showing those that occurred more than 20 times and where neither word was a stop word</p>
</div>

In Figure \@ref(fig:bigramgraph), we can visualize some details of the text structure. For example, we see that salutations such as "miss", "lady", "sir", and "colonel" form common centers of nodes, which are often followed by names. We also see pairs or triplets along the outside that form common short phrases ("half hour", "thousand pounds", or "short time/pause").

We conclude with a few polishing operations to make a better looking graph (Figure \@ref(fig:bigramggraphausten2)):

* We add the `edge_alpha` aesthetic to the link layer to make links transparent based on how common or rare the bigram is
* We add directionality with an arrow, constructed using `grid::arrow()`, including an `end_cap` option that tells the arrow to end before touching the node
* We tinker with the options to the node layer to make the nodes more attractive (larger, blue points)
* We add a theme that's useful for plotting networks, `theme_void()`


```r
set.seed(2020)

a <- grid::arrow(type = "closed", length = unit(.15, "inches"))

ggraph(bigram_graph, layout = "fr") +
  geom_edge_link(aes(edge_alpha = n), show.legend = FALSE,
                 arrow = a, end_cap = circle(.07, 'inches')) +
  geom_node_point(color = "lightblue", size = 5) +
  geom_node_text(aes(label = name), vjust = 1, hjust = 1) +
  theme_void()
```

<div class="figure" style="text-align: center">
<img src="04-word-combinations_files/figure-html/bigramggraphausten2-1.png" alt="Common bigrams in Jane Austen's novels, with some polishing" width="90%" />
<p class="caption">(\#fig:bigramggraphausten2)Common bigrams in Jane Austen's novels, with some polishing</p>
</div>

It may take some experimentation with ggraph to get your networks into a presentable format like this, but the network structure is useful and flexible way to visualize relational tidy data.

<div class="rmdnote">
<p>Note that this is a visualization of a <strong>Markov chain</strong>,
a common model in text processing. In a Markov chain, each choice of
word depends only on the previous word. In this case, a random generator
following this model might spit out “dear”, then “sir”, then
“william/walter/thomas/thomas’s”, by following each word to the most
common words that follow it. To make the visualization interpretable, we
chose to show only the most common word to word connections, but one
could imagine an enormous graph representing all connections that occur
in the text.</p>
</div>

### Visualizing bigrams in other texts

We went to a good amount of work in cleaning and visualizing bigrams on a text dataset, so let's collect it into a function so that we easily perform it on other text datasets.

<div class="rmdnote">
<p>To make it easy to use the <code>count_bigrams()</code> and
<code>visualize_bigrams()</code> yourself, we’ve also reloaded the
packages necessary for them.</p>
</div>


```r
library(dplyr)
library(tidyr)
library(tidytext)
library(ggplot2)
library(igraph)
library(ggraph)

count_bigrams <- function(dataset) {
  dataset %>%
    unnest_tokens(bigram, text, token = "ngrams", n = 2) %>%
    separate(bigram, c("word1", "word2"), sep = " ") %>%
    filter(!word1 %in% stop_words$word,
           !word2 %in% stop_words$word) %>%
    count(word1, word2, sort = TRUE)
}

visualize_bigrams <- function(bigrams) {
  set.seed(2016)
  a <- grid::arrow(type = "closed", length = unit(.15, "inches"))
  
  bigrams %>%
    graph_from_data_frame() %>%
    ggraph(layout = "fr") +
    geom_edge_link(aes(edge_alpha = n), show.legend = FALSE, arrow = a) +
    geom_node_point(color = "lightblue", size = 5) +
    geom_node_text(aes(label = name), vjust = 1, hjust = 1) +
    theme_void()
}
```

At this point, we could visualize bigrams in other works, such as the King James Version of the Bible:


```r
# the King James version is book 10 on Project Gutenberg:
library(gutenbergr)
kjv <- gutenberg_download(10)
```




```r
library(stringr)

kjv_bigrams <- kjv %>%
  count_bigrams()

# filter out rare combinations, as well as digits
kjv_bigrams %>%
  filter(n > 40,
         !str_detect(word1, "\\d"),
         !str_detect(word2, "\\d")) %>%
  visualize_bigrams()
```

<div class="figure" style="text-align: center">
<img src="04-word-combinations_files/figure-html/kjvbigrams-1.png" alt="Directed graph of common bigrams in the King James Bible, showing those that occurred more than 40 times" width="90%" />
<p class="caption">(\#fig:kjvbigrams)Directed graph of common bigrams in the King James Bible, showing those that occurred more than 40 times</p>
</div>

Figure \@ref(fig:kjvbigrams) thus lays out a common "blueprint" of language within the Bible, particularly focused around "thy" and "thou" (which could probably be considered stopwords!) You can use the gutenbergr package and these `count_bigrams`/`visualize_bigrams` functions to visualize bigrams in other classic books you're interested in.

## Counting and correlating pairs of words with the widyr package

Tokenizing by n-gram is a useful way to explore pairs of adjacent words. However, we may also be interested in words that tend to co-occur within particular documents or particular chapters, even if they don't occur next to each other.

Tidy data is a useful structure for comparing between variables or grouping by rows, but it can be challenging to compare between rows: for example, to count the number of times that two words appear within the same document, or to see how correlated they are. Most operations for finding pairwise counts or correlations need to turn the data into a wide matrix first.

<div class="figure" style="text-align: center">
<img src="images/tmwr_0407.png" alt="The philosophy behind the widyr package, which can perform operations such as counting and correlating on pairs of values in a tidy dataset. The widyr package first 'casts' a tidy dataset into a wide matrix, performs an operation such as a correlation on it, then re-tidies the result." width="100%" />
<p class="caption">(\#fig:widyr)The philosophy behind the widyr package, which can perform operations such as counting and correlating on pairs of values in a tidy dataset. The widyr package first 'casts' a tidy dataset into a wide matrix, performs an operation such as a correlation on it, then re-tidies the result.</p>
</div>

We'll examine some of the ways tidy text can be turned into a wide matrix in Chapter \@ref(dtm), but in this case it isn't necessary. The [widyr](https://github.com/dgrtwo/widyr) package makes operations such as computing counts and correlations easy, by simplifying the pattern of "widen data, perform an operation, then re-tidy data" (Figure \@ref(fig:widyr)). We'll focus on a set of functions that make pairwise comparisons between groups of observations (for example, between documents, or sections of text).

### Counting and correlating among sections

Consider the book "Pride and Prejudice" divided into 10-line sections, as we did (with larger sections) for sentiment analysis in Chapter \@ref(sentiment). We may be interested in what words tend to appear within the same section.


```r
austen_section_words <- austen_books() %>%
  filter(book == "Pride & Prejudice") %>%
  mutate(section = row_number() %/% 10) %>%
  filter(section > 0) %>%
  unnest_tokens(word, text) %>%
  filter(!word %in% stop_words$word)

austen_section_words
#> # A tibble: 37,240 × 3
#>    book              section word        
#>    <fct>               <dbl> <chr>       
#>  1 Pride & Prejudice       1 truth       
#>  2 Pride & Prejudice       1 universally 
#>  3 Pride & Prejudice       1 acknowledged
#>  4 Pride & Prejudice       1 single      
#>  5 Pride & Prejudice       1 possession  
#>  6 Pride & Prejudice       1 fortune     
#>  7 Pride & Prejudice       1 wife        
#>  8 Pride & Prejudice       1 feelings    
#>  9 Pride & Prejudice       1 views       
#> 10 Pride & Prejudice       1 entering    
#> # … with 37,230 more rows
```

One useful function from widyr is the `pairwise_count()` function. The prefix `pairwise_` means it will result in one row for each pair of words in the `word` variable. This lets us count common pairs of words co-appearing within the same section:


```r
library(widyr)

# count words co-occuring within sections
word_pairs <- austen_section_words %>%
  pairwise_count(word, section, sort = TRUE)

word_pairs
#> # A tibble: 796,008 × 3
#>    item1     item2         n
#>    <chr>     <chr>     <dbl>
#>  1 darcy     elizabeth   144
#>  2 elizabeth darcy       144
#>  3 miss      elizabeth   110
#>  4 elizabeth miss        110
#>  5 elizabeth jane        106
#>  6 jane      elizabeth   106
#>  7 miss      darcy        92
#>  8 darcy     miss         92
#>  9 elizabeth bingley      91
#> 10 bingley   elizabeth    91
#> # … with 795,998 more rows
```

Notice that while the input had one row for each pair of a document (a 10-line section) and a word, the output has one row for each pair of words. This is also a tidy format, but of a very different structure that we can use to answer new questions.

For example, we can see that the most common pair of words in a section is "Elizabeth" and "Darcy" (the two main characters). We can easily find the words that most often occur with Darcy:


```r
word_pairs %>%
  filter(item1 == "darcy")
#> # A tibble: 2,930 × 3
#>    item1 item2         n
#>    <chr> <chr>     <dbl>
#>  1 darcy elizabeth   144
#>  2 darcy miss         92
#>  3 darcy bingley      86
#>  4 darcy jane         46
#>  5 darcy bennet       45
#>  6 darcy sister       45
#>  7 darcy time         41
#>  8 darcy lady         38
#>  9 darcy friend       37
#> 10 darcy wickham      37
#> # … with 2,920 more rows
```

### Pairwise correlation {#pairwise-correlation}

Pairs like "Elizabeth" and "Darcy" are the most common co-occurring words, but that's not particularly meaningful since *they're also the most common individual words.* We may instead want to examine **correlation** among words, which indicates how often they appear together relative to how often they appear separately.

In particular, here we'll focus on the [phi coefficient](https://en.wikipedia.org/wiki/Phi_coefficient), a common measure for binary correlation. The focus of the phi coefficient is how much more likely it is that either **both** word X and Y appear, or **neither** do, than that one appears without the other.

Consider the following table:

|  | Has word Y | No word Y | Total |  |
|------------|---------------|---------------|--------------|---|
| Has word X | $n_{11}$ | $n_{10}$ | $n_{1\cdot}$ |  |
| No word X | $n_{01}$ | $n_{00}$ | $n_{0\cdot}$ |  |
| Total | $n_{\cdot 1}$ | $n_{\cdot 0}$ | n |  |

For example, that $n_{11}$ represents the number of documents where both word X and word Y appear, $n_{00}$ the number where neither appears, and $n_{10}$ and $n_{01}$ the cases where one appears without the other. In terms of this table, the phi coefficient is:

$$\phi=\frac{n_{11}n_{00}-n_{10}n_{01}}{\sqrt{n_{1\cdot}n_{0\cdot}n_{\cdot0}n_{\cdot1}}}$$

<div class="rmdnote">
<p>The phi coefficient is equivalent to the Pearson correlation, which
you may have heard of elsewhere when it is applied to binary data.</p>
</div>

The `pairwise_cor()` function in widyr lets us find the phi coefficient between words based on how often they appear in the same section. Its syntax is similar to `pairwise_count()`.


```r
# we need to filter for at least relatively common words first
word_cors <- austen_section_words %>%
  group_by(word) %>%
  filter(n() >= 20) %>%
  pairwise_cor(word, section, sort = TRUE)

word_cors
#> # A tibble: 154,842 × 3
#>    item1     item2     correlation
#>    <chr>     <chr>           <dbl>
#>  1 bourgh    de              0.951
#>  2 de        bourgh          0.951
#>  3 pounds    thousand        0.701
#>  4 thousand  pounds          0.701
#>  5 william   sir             0.664
#>  6 sir       william         0.664
#>  7 catherine lady            0.663
#>  8 lady      catherine       0.663
#>  9 forster   colonel         0.622
#> 10 colonel   forster         0.622
#> # … with 154,832 more rows
```

This output format is helpful for exploration. For example, we could find the words most correlated with a word like "pounds" using a `filter` operation.


```r
word_cors %>%
  filter(item1 == "pounds")
#> # A tibble: 393 × 3
#>    item1  item2     correlation
#>    <chr>  <chr>           <dbl>
#>  1 pounds thousand       0.701 
#>  2 pounds ten            0.231 
#>  3 pounds fortune        0.164 
#>  4 pounds settled        0.149 
#>  5 pounds wickham's      0.142 
#>  6 pounds children       0.129 
#>  7 pounds mother's       0.119 
#>  8 pounds believed       0.0932
#>  9 pounds estate         0.0890
#> 10 pounds ready          0.0860
#> # … with 383 more rows
```

This lets us pick particular interesting words and find the other words most associated with them (Figure \@ref(fig:wordcors)).


```r
word_cors %>%
  filter(item1 %in% c("elizabeth", "pounds", "married", "pride")) %>%
  group_by(item1) %>%
  slice_max(correlation, n = 6) %>%
  ungroup() %>%
  mutate(item2 = reorder(item2, correlation)) %>%
  ggplot(aes(item2, correlation)) +
  geom_bar(stat = "identity") +
  facet_wrap(~ item1, scales = "free") +
  coord_flip()
```

<div class="figure" style="text-align: center">
<img src="04-word-combinations_files/figure-html/wordcors-1.png" alt="Words from Pride and Prejudice that were most correlated with 'elizabeth', 'pounds', 'married', and 'pride'" width="90%" />
<p class="caption">(\#fig:wordcors)Words from Pride and Prejudice that were most correlated with 'elizabeth', 'pounds', 'married', and 'pride'</p>
</div>

Just as we used ggraph to visualize bigrams, we can use it to visualize the correlations and clusters of words that were found by the widyr package (Figure \@ref(fig:wordcorsnetwork)).


```r
set.seed(2016)

word_cors %>%
  filter(correlation > .15) %>%
  graph_from_data_frame() %>%
  ggraph(layout = "fr") +
  geom_edge_link(aes(edge_alpha = correlation), show.legend = FALSE) +
  geom_node_point(color = "lightblue", size = 5) +
  geom_node_text(aes(label = name), repel = TRUE) +
  theme_void()
```

<div class="figure" style="text-align: center">
<img src="04-word-combinations_files/figure-html/wordcorsnetwork-1.png" alt="Pairs of words in Pride and Prejudice that show at least a .15 correlation of appearing within the same 10-line section" width="90%" />
<p class="caption">(\#fig:wordcorsnetwork)Pairs of words in Pride and Prejudice that show at least a .15 correlation of appearing within the same 10-line section</p>
</div>

Note that unlike the bigram analysis, the relationships here are symmetrical, rather than directional (there are no arrows). We can also see that while pairings of names and titles that dominated bigram pairings are common, such as "colonel/fitzwilliam", we can also see pairings of words that appear close to each other, such as "walk" and "park", or "dance" and "ball".

## Summary

This chapter showed how the tidy text approach is useful not only for analyzing individual words, but also for exploring the relationships and connections between words. Such relationships can involve n-grams, which enable us to see what words tend to appear after others, or co-occurences and correlations, for words that appear in proximity to each other. This chapter also demonstrated the ggraph package for visualizing both of these types of relationships as networks. These network visualizations are a flexible tool for exploring relationships, and will play an important role in the case studies in later chapters. 
