# Case study: comparing Twitter archives {#twitter}

One type of text that gets plenty of attention is text shared online via Twitter. In fact, several of the sentiment lexicons used in this book (and commonly used in general) were designed for use with and validated on tweets. Both of the authors of this book are on Twitter and are fairly regular users of it, so in this case study, let's compare the entire Twitter archives of [Julia](https://twitter.com/juliasilge) and [David](https://twitter.com/drob).

## Getting the data and distribution of tweets

An individual can download their own Twitter archive by following [directions available on Twitter's website](https://support.twitter.com/articles/20170160). We each downloaded ours and will now open them up. Let's use the lubridate package to convert the string timestamps to date-time objects and initially take a look at our tweeting patterns overall (Figure \@ref(fig:tweets)).


```r
library(lubridate)
library(ggplot2)
library(dplyr)
library(readr)

tweets_julia <- read_csv("data/tweets_julia.csv")
tweets_dave <- read_csv("data/tweets_dave.csv")
tweets <- bind_rows(tweets_julia %>% 
                      mutate(person = "Julia"),
                    tweets_dave %>% 
                      mutate(person = "David")) %>%
  mutate(timestamp = ymd_hms(timestamp))

ggplot(tweets, aes(x = timestamp, fill = person)) +
  geom_histogram(position = "identity", bins = 20, show.legend = FALSE) +
  facet_wrap(~person, ncol = 1)
```

<div class="figure" style="text-align: center">
<img src="07-tweet-archives_files/figure-html/tweets-1.png" alt="All tweets from our accounts" width="90%" />
<p class="caption">(\#fig:tweets)All tweets from our accounts</p>
</div>

David and Julia tweet at about the same rate currently and joined Twitter about a year apart from each other, but there were about 5 years where David was not active on Twitter and Julia was. In total, Julia has about 4 times as many tweets as David.

## Word frequencies

Let's use `unnest_tokens()` to make a tidy data frame of all the words in our tweets, and remove the common English stop words. There are certain conventions in how people use text on Twitter, so we will use a specialized tokenizer and do a bit more work with our text here than, for example, we did with the narrative text from Project Gutenberg. 

First, we will remove tweets from this dataset that are retweets so that we only have tweets that we wrote ourselves. Next, the `mutate()` line cleans out some characters that we don't want like ampersands and such. 

<div class="rmdnote">
<p>In the call to <code>unnest_tokens()</code>, we unnest using the
specialized <code>"tweets"</code> tokenizer that is built in to the
tokenizers package <span class="citation">[@R-tokenizers]</span>. This
tool is very useful for dealing with Twitter text or other text from
online forums; it retains hashtags and mentions of usernames with the
<code>@</code> symbol.</p>
</div>

Because we have kept text such as hashtags and usernames in the dataset, we can't use a simple `anti_join()` to remove stop words. Instead, we can take the approach shown in the `filter()` line that uses `str_detect()` from the stringr package.


```r
library(tidytext)
library(stringr)

remove_reg <- "&amp;|&lt;|&gt;"
tidy_tweets <- tweets %>% 
  filter(!str_detect(text, "^RT")) %>%
  mutate(text = str_remove_all(text, remove_reg)) %>%
  unnest_tokens(word, text, token = "tweets") %>%
  filter(!word %in% stop_words$word,
         !word %in% str_remove_all(stop_words$word, "'"),
         str_detect(word, "[a-z]"))
```

Now we can calculate word frequencies for each person. First, we group by person and count how many times each person used each word. Then we use `left_join()` to add a column of the total number of words used by each person. (This is higher for Julia than David since she has more tweets than David.) Finally, we calculate a frequency for each person and word.


```r
frequency <- tidy_tweets %>% 
  count(person, word, sort = TRUE) %>% 
  left_join(tidy_tweets %>% 
              count(person, name = "total")) %>%
  mutate(freq = n/total)

frequency
#> # A tibble: 24,067 × 5
#>    person word               n total    freq
#>    <chr>  <chr>          <int> <int>   <dbl>
#>  1 Julia  @selkie1970      570 74152 0.00769
#>  2 Julia  time             557 74152 0.00751
#>  3 Julia  @skedman         531 74152 0.00716
#>  4 Julia  day              437 74152 0.00589
#>  5 Julia  baby             392 74152 0.00529
#>  6 David  @hadleywickham   308 20699 0.0149 
#>  7 Julia  love             302 74152 0.00407
#>  8 Julia  @haleynburke     298 74152 0.00402
#>  9 Julia  house            283 74152 0.00382
#> 10 Julia  morning          278 74152 0.00375
#> # … with 24,057 more rows
```

This is a nice and tidy data frame but we would actually like to plot those frequencies on the x- and y-axes of a plot, so we will need to use `pivot_wider()` from tidyr make a differently shaped data frame.


```r
library(tidyr)

frequency <- frequency %>% 
  select(person, word, freq) %>% 
  pivot_wider(names_from = person, values_from = freq) %>%
  arrange(Julia, David)

frequency
#> # A tibble: 21,071 × 3
#>    word               Julia     David
#>    <chr>              <dbl>     <dbl>
#>  1 @accidentalart 0.0000135 0.0000483
#>  2 @alicedata     0.0000135 0.0000483
#>  3 @alistaire     0.0000135 0.0000483
#>  4 @corynissen    0.0000135 0.0000483
#>  5 @jennybryans   0.0000135 0.0000483
#>  6 @jsvine        0.0000135 0.0000483
#>  7 @lewislab      0.0000135 0.0000483
#>  8 @lizasperling  0.0000135 0.0000483
#>  9 @ognyanova     0.0000135 0.0000483
#> 10 @rbloggers     0.0000135 0.0000483
#> # … with 21,061 more rows
```

Now this is ready for us to plot. Let's use `geom_jitter()` so that we don't see the discreteness at the low end of frequency as much, and `check_overlap = TRUE` so the text labels don't all print out on top of each other (only some will print).


```r
library(scales)

ggplot(frequency, aes(Julia, David)) +
  geom_jitter(alpha = 0.1, size = 2.5, width = 0.25, height = 0.25) +
  geom_text(aes(label = word), check_overlap = TRUE, vjust = 1.5) +
  scale_x_log10(labels = percent_format()) +
  scale_y_log10(labels = percent_format()) +
  geom_abline(color = "red")
```

<div class="figure" style="text-align: center">
<img src="07-tweet-archives_files/figure-html/wideplot-1.png" alt="Comparing the frequency of words used by Julia and David" width="90%" />
<p class="caption">(\#fig:wideplot)Comparing the frequency of words used by Julia and David</p>
</div>


Words near the line in Figure \@ref(fig:wideplot) are used with about equal frequencies by David and Julia, while words far away from the line are used much more by one person compared to the other. Words, hashtags, and usernames that appear in this plot are ones that we have both used at least once in tweets.

This may not even need to be pointed out, but David and Julia have used their Twitter accounts rather differently over the course of the past several years. David has used his Twitter account almost exclusively for professional purposes since he became more active, while Julia used it for entirely personal purposes until late 2015 and still uses it more personally than David. We see these differences immediately in this plot exploring word frequencies, and they will continue to be obvious in the rest of this chapter. 

## Comparing word usage 

We just made a plot comparing raw word frequencies over our whole Twitter histories; now let's find which words are more or less likely to come from each person's account using the log odds ratio. First, let's restrict the analysis moving forward to tweets from David and Julia sent during 2016. David was consistently active on Twitter for all of 2016 and this was about when Julia transitioned into data science as a career.


```r
tidy_tweets <- tidy_tweets %>%
  filter(timestamp >= as.Date("2016-01-01"),
         timestamp < as.Date("2017-01-01"))
```

Next, let's use `str_detect()` to remove Twitter usernames from the `word` column, because otherwise, the results here are dominated only by people who Julia or David know and the other does not. After removing these, we count how many times each person uses each word and keep only the words used more than 10 times. After a `pivot_wider()` operation, we can calculate the log odds ratio for each word, using


$$\text{log odds ratio} = \ln{\left(\frac{\left[\frac{n+1}{\text{total}+1}\right]_\text{David}}{\left[\frac{n+1}{\text{total}+1}\right]_\text{Julia}}\right)}$$

where $n$ is the number of times the word in question is used by each person and the total indicates the total words for each person.


```r
word_ratios <- tidy_tweets %>%
  filter(!str_detect(word, "^@")) %>%
  count(word, person) %>%
  group_by(word) %>%
  filter(sum(n) >= 10) %>%
  ungroup() %>%
  pivot_wider(names_from = person, values_from = n, values_fill = 0) %>%
  mutate_if(is.numeric, list(~(. + 1) / (sum(.) + 1))) %>%
  mutate(logratio = log(David / Julia)) %>%
  arrange(desc(logratio))
```

What are some words that have been about equally likely to come from David or Julia's account during 2016?


```r
word_ratios %>% 
  arrange(abs(logratio))
#> # A tibble: 351 × 4
#>    word      David   Julia logratio
#>    <chr>     <dbl>   <dbl>    <dbl>
#>  1 words   0.00377 0.00378 -0.00334
#>  2 science 0.00653 0.00648  0.00771
#>  3 idea    0.00577 0.00594 -0.0279 
#>  4 email   0.00251 0.00243  0.0330 
#>  5 file    0.00251 0.00243  0.0330 
#>  6 purrr   0.00251 0.00243  0.0330 
#>  7 test    0.00226 0.00216  0.0454 
#>  8 account 0.00201 0.00189  0.0612 
#>  9 api     0.00201 0.00189  0.0612 
#> 10 sad     0.00201 0.00189  0.0612 
#> # … with 341 more rows
```

We are about equally likely to tweet about words, science, ideas, and email.

Which words are most likely to be from Julia's account or from David's account? Let's just take the top 15 most distinctive words for each account and plot them in Figure \@ref(fig:plotratios).


```r
word_ratios %>%
  group_by(logratio < 0) %>%
  slice_max(abs(logratio), n = 15) %>% 
  ungroup() %>%
  mutate(word = reorder(word, logratio)) %>%
  ggplot(aes(word, logratio, fill = logratio < 0)) +
  geom_col(show.legend = FALSE) +
  coord_flip() +
  ylab("log odds ratio (David/Julia)") +
  scale_fill_discrete(name = "", labels = c("David", "Julia"))
```

<div class="figure" style="text-align: center">
<img src="07-tweet-archives_files/figure-html/plotratios-1.png" alt="Comparing the odds ratios of words from our accounts" width="90%" />
<p class="caption">(\#fig:plotratios)Comparing the odds ratios of words from our accounts</p>
</div>

So David has tweeted about specific conferences he has gone to and Stack Overflow, while Julia tweeted about Utah, Census data, and her family.

## Changes in word use

The section above looked at overall word use, but now let's ask a different question. Which words' frequencies have changed the fastest in our Twitter feeds? Or to state this another way, which words have we tweeted about at a higher or lower rate as time has passed? To do this, we will define a new time variable in the data frame that defines which unit of time each tweet was posted in. We can use `floor_date()` from lubridate to do this, with a unit of our choosing; using 1 month seems to work well for this year of tweets from both of us.

After we have the time bins defined, we count how many times each of us used each word in each time bin. After that, we add columns to the data frame for the total number of words used in each time bin by each person and the total number of times each word was used by each person. We can then `filter()` to only keep words used at least some minimum number of times (30, in this case).


```r
words_by_time <- tidy_tweets %>%
  filter(!str_detect(word, "^@")) %>%
  mutate(time_floor = floor_date(timestamp, unit = "1 month")) %>%
  count(time_floor, person, word) %>%
  group_by(person, time_floor) %>%
  mutate(time_total = sum(n)) %>%
  group_by(person, word) %>%
  mutate(word_total = sum(n)) %>%
  ungroup() %>%
  rename(count = n) %>%
  filter(word_total > 30)

words_by_time
#> # A tibble: 326 × 6
#>    time_floor          person word    count time_total word_total
#>    <dttm>              <chr>  <chr>   <int>      <int>      <int>
#>  1 2016-01-01 00:00:00 David  #rstats     2        315        205
#>  2 2016-01-01 00:00:00 David  broom       2        315         34
#>  3 2016-01-01 00:00:00 David  data        2        315        148
#>  4 2016-01-01 00:00:00 David  ggplot2     1        315         37
#>  5 2016-01-01 00:00:00 David  time        2        315         56
#>  6 2016-01-01 00:00:00 David  tweets      1        315         46
#>  7 2016-01-01 00:00:00 Julia  #rstats    10        437        116
#>  8 2016-01-01 00:00:00 Julia  blog        2        437         33
#>  9 2016-01-01 00:00:00 Julia  data        5        437        105
#> 10 2016-01-01 00:00:00 Julia  day         1        437         43
#> # … with 316 more rows
```

Each row in this data frame corresponds to one person using one word in a given time bin. The `count` column tells us how many times that person used that word in that time bin, the `time_total` column tells us how many words that person used during that time bin, and the `word_total` column tells us how many times that person used that word over the whole year. This is the data set we can use for modeling. 

We can use `nest()` from tidyr to make a data frame with a list column that contains little miniature data frames for each word. Let's do that now and take a look at the resulting structure.


```r
nested_data <- words_by_time %>%
  nest(data = c(-word, -person)) 

nested_data
#> # A tibble: 32 × 3
#>    person word    data             
#>    <chr>  <chr>   <list>           
#>  1 David  #rstats <tibble [12 × 4]>
#>  2 David  broom   <tibble [10 × 4]>
#>  3 David  data    <tibble [12 × 4]>
#>  4 David  ggplot2 <tibble [10 × 4]>
#>  5 David  time    <tibble [12 × 4]>
#>  6 David  tweets  <tibble [8 × 4]> 
#>  7 Julia  #rstats <tibble [12 × 4]>
#>  8 Julia  blog    <tibble [10 × 4]>
#>  9 Julia  data    <tibble [12 × 4]>
#> 10 Julia  day     <tibble [12 × 4]>
#> # … with 22 more rows
```

This data frame has one row for each person-word combination; the `data` column is a list column that contains data frames, one for each combination of person and word. Let's use `map()` from purrr [@R-purrr] to apply our modeling procedure to each of those little data frames inside our big data frame. This is count data so let’s use `glm()` with `family = "binomial"` for modeling. 

<div class="rmdtip">
<p>We can think about this modeling procedure answering a question like,
“Was a given word mentioned in a given time bin? Yes or no? How does the
count of word mentions depend on time?”</p>
</div>


```r
library(purrr)

nested_models <- nested_data %>%
  mutate(models = map(data, ~ glm(cbind(count, time_total) ~ time_floor, ., 
                                  family = "binomial")))

nested_models
#> # A tibble: 32 × 4
#>    person word    data              models
#>    <chr>  <chr>   <list>            <list>
#>  1 David  #rstats <tibble [12 × 4]> <glm> 
#>  2 David  broom   <tibble [10 × 4]> <glm> 
#>  3 David  data    <tibble [12 × 4]> <glm> 
#>  4 David  ggplot2 <tibble [10 × 4]> <glm> 
#>  5 David  time    <tibble [12 × 4]> <glm> 
#>  6 David  tweets  <tibble [8 × 4]>  <glm> 
#>  7 Julia  #rstats <tibble [12 × 4]> <glm> 
#>  8 Julia  blog    <tibble [10 × 4]> <glm> 
#>  9 Julia  data    <tibble [12 × 4]> <glm> 
#> 10 Julia  day     <tibble [12 × 4]> <glm> 
#> # … with 22 more rows
```

Now notice that we have a new column for the modeling results; it is another list column and contains `glm` objects. The next step is to use `map()` and `tidy()` from the broom package to pull out the slopes for each of these models and find the important ones. We are comparing many slopes here and some of them are not statistically significant, so let's apply an adjustment to the p-values for multiple comparisons.


```r
library(broom)

slopes <- nested_models %>%
  mutate(models = map(models, tidy)) %>%
  unnest(cols = c(models)) %>%
  filter(term == "time_floor") %>%
  mutate(adjusted.p.value = p.adjust(p.value))
```

Now let's find the most important slopes. Which words have changed in frequency at a moderately significant level in our tweets?


```r
top_slopes <- slopes %>% 
  filter(adjusted.p.value < 0.05)

top_slopes
#> # A tibble: 6 × 9
#>   person word      data              term   estimate std.error statistic p.value
#>   <chr>  <chr>     <list>            <chr>     <dbl>     <dbl>     <dbl>   <dbl>
#> 1 David  ggplot2   <tibble [10 × 4]> time_… -8.07e-8   2.00e-8     -4.04 5.23e-5
#> 2 Julia  #rstats   <tibble [12 × 4]> time_… -4.50e-8   1.11e-8     -4.04 5.41e-5
#> 3 Julia  post      <tibble [12 × 4]> time_… -5.14e-8   1.49e-8     -3.46 5.46e-4
#> 4 David  overflow  <tibble [10 × 4]> time_…  6.96e-8   2.23e-8      3.12 1.81e-3
#> 5 David  stack     <tibble [10 × 4]> time_…  7.38e-8   2.19e-8      3.37 7.51e-4
#> 6 David  #user2016 <tibble [3 × 4]>  time_… -8.17e-7   1.55e-7     -5.27 1.39e-7
#> # … with 1 more variable: adjusted.p.value <dbl>
```

To visualize our results, we can plot these words' use for both David and Julia over this year of tweets.


```r
words_by_time %>%
  inner_join(top_slopes, by = c("word", "person")) %>%
  filter(person == "David") %>%
  ggplot(aes(time_floor, count/time_total, color = word)) +
  geom_line(size = 1.3) +
  labs(x = NULL, y = "Word frequency")
```

<div class="figure" style="text-align: center">
<img src="07-tweet-archives_files/figure-html/topdave-1.png" alt="Trending words in David's tweets" width="90%" />
<p class="caption">(\#fig:topdave)Trending words in David's tweets</p>
</div>

We see in Figure \@ref(fig:topdave) that David tweeted a lot about the UseR conference while he was there and then quickly stopped. He has tweeted more about Stack Overflow toward the end of the year and less about ggplot2 as the year has progressed.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Me: I&#39;m so sick of data science wars. <a href="https://twitter.com/hashtag/rstats?src=hash">#rstats</a> vs Python, frequentist vs Bayesian...<br><br>Them: base vs ggplot2...<br><br>Me: WHY WHICH SIDE ARE YOU ON</p>&mdash; David Robinson (\@drob) <a href="https://twitter.com/drob/status/712639593703542785">March 23, 2016</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Now let's plot words that have changed frequency in Julia's tweets in Figure \@ref(fig:topjulia).


```r
words_by_time %>%
  inner_join(top_slopes, by = c("word", "person")) %>%
  filter(person == "Julia") %>%
  ggplot(aes(time_floor, count/time_total, color = word)) +
  geom_line(size = 1.3) +
  labs(x = NULL, y = "Word frequency")
```

<div class="figure" style="text-align: center">
<img src="07-tweet-archives_files/figure-html/topjulia-1.png" alt="Trending words in Julia's tweets" width="90%" />
<p class="caption">(\#fig:topjulia)Trending words in Julia's tweets</p>
</div>

Both the significant slopes for Julia are negative. This means she has not tweeted at a higher rate using any specific words, but instead using a variety of different words; her tweets earlier in the year contained the words shown in this plot at higher proportions. Words she uses when publicizing a new blog post like the #rstats hashtag and "post" have gone down in frequency.

## Favorites and retweets

Another important characteristic of tweets is how many times they are favorited or retweeted. Let's explore which words are more likely to be retweeted or favorited for Julia's and David's tweets. When a user downloads their own Twitter archive, favorites and retweets are not included, so we constructed another dataset of the authors' tweets that includes this information. We accessed our own tweets via the Twitter API and downloaded about 3200 tweets for each person. In both cases, that is about the last 18 months worth of Twitter activity. This corresponds to a period of increasing activity and increasing numbers of followers for both of us.


```r
tweets_julia <- read_csv("data/juliasilge_tweets.csv")
tweets_dave <- read_csv("data/drob_tweets.csv")
tweets <- bind_rows(tweets_julia %>% 
                      mutate(person = "Julia"),
                    tweets_dave %>% 
                      mutate(person = "David")) %>%
  mutate(created_at = ymd_hms(created_at))
```

Now that we have this second, smaller set of only recent tweets, let's again use `unnest_tokens()` to transform these tweets to a tidy data set. Let's remove all retweets and replies from this data set so we only look at regular tweets that David and Julia have posted directly.


```r
tidy_tweets <- tweets %>% 
  filter(!str_detect(text, "^(RT|@)")) %>%
  mutate(text = str_remove_all(text, remove_reg)) %>%
  unnest_tokens(word, text, token = "tweets", strip_url = TRUE) %>%
  filter(!word %in% stop_words$word,
         !word %in% str_remove_all(stop_words$word, "'"))

tidy_tweets
#> # A tibble: 11,014 × 7
#>         id created_at          source            retweets favorites person word 
#>      <dbl> <dttm>              <chr>                <dbl>     <dbl> <chr>  <chr>
#>  1 8.04e17 2016-12-01 16:44:03 Twitter Web Clie…        0         0 Julia  "sco…
#>  2 8.04e17 2016-12-01 16:44:03 Twitter Web Clie…        0         0 Julia  "50" 
#>  3 8.04e17 2016-12-01 16:42:03 Twitter Web Clie…        0         9 Julia  "sno…
#>  4 8.04e17 2016-12-01 16:42:03 Twitter Web Clie…        0         9 Julia  "\U0…
#>  5 8.04e17 2016-12-01 16:42:03 Twitter Web Clie…        0         9 Julia  "dri…
#>  6 8.04e17 2016-12-01 16:42:03 Twitter Web Clie…        0         9 Julia  "tea"
#>  7 8.04e17 2016-12-01 16:42:03 Twitter Web Clie…        0         9 Julia  "\U0…
#>  8 8.04e17 2016-12-01 16:42:03 Twitter Web Clie…        0         9 Julia  "#rs…
#>  9 8.04e17 2016-12-01 16:42:03 Twitter Web Clie…        0         9 Julia  "\U0…
#> 10 8.04e17 2016-12-01 02:56:10 Twitter Web Clie…        0        11 Julia  "jul…
#> # … with 11,004 more rows
```

Notice that the `word` column contains tokenized emoji.

To start with, let’s look at the number of times each of our tweets was retweeted. Let's find the total number of retweets for each person.


```r
totals <- tidy_tweets %>% 
  group_by(person, id) %>% 
  summarise(rts = first(retweets)) %>% 
  group_by(person) %>% 
  summarise(total_rts = sum(rts))

totals
#> # A tibble: 2 × 2
#>   person total_rts
#>   <chr>      <dbl>
#> 1 David      13014
#> 2 Julia       1750
```

Now let's find the median number of retweets for each word and person. We probably want to count each tweet/word combination only once, so we will use `group_by()` and `summarise()` twice, one right after the other. The first `summarise()` statement counts how many times each word was retweeted, for each tweet and person. In the second `summarise()` statement, we can find the median retweets for each person and word, also count the number of times each word was used ever by each person and keep that in `uses`. Next, we can join this to the data frame of retweet totals. Let's `filter()` to only keep words mentioned at least 5 times.


```r
word_by_rts <- tidy_tweets %>% 
  group_by(id, word, person) %>% 
  summarise(rts = first(retweets)) %>% 
  group_by(person, word) %>% 
  summarise(retweets = median(rts), uses = n()) %>%
  left_join(totals) %>%
  filter(retweets != 0) %>%
  ungroup()

word_by_rts %>% 
  filter(uses >= 5) %>%
  arrange(desc(retweets))
#> # A tibble: 170 × 5
#>    person word          retweets  uses total_rts
#>    <chr>  <chr>            <dbl> <int>     <dbl>
#>  1 David  animation           85     5     13014
#>  2 David  gganimate           75     6     13014
#>  3 David  error               56     7     13014
#>  4 David  start               56     6     13014
#>  5 David  download            52     5     13014
#>  6 Julia  tidytext            50     7      1750
#>  7 David  introducing         45     6     13014
#>  8 David  understanding       37     6     13014
#>  9 David  ab                  36     5     13014
#> 10 David  bayesian            34     7     13014
#> # … with 160 more rows
```

At the top of this sorted data frame, we see tweets from Julia and David about packages that they work on, like [gganimate](https://github.com/dgrtwo/gganimate) and [tidytext](https://cran.r-project.org/package=tidytext). Let's plot the words that have the highest median retweets for each of our accounts (Figure \@ref(fig:plotrts)).


```r
word_by_rts %>%
  filter(uses >= 5) %>%
  group_by(person) %>%
  slice_max(retweets, n = 10) %>% 
  arrange(retweets) %>%
  ungroup() %>%
  mutate(word = factor(word, unique(word))) %>%
  ungroup() %>%
  ggplot(aes(word, retweets, fill = person)) +
  geom_col(show.legend = FALSE) +
  facet_wrap(~ person, scales = "free", ncol = 2) +
  coord_flip() +
  labs(x = NULL, 
       y = "Median # of retweets for tweets containing each word")
```

<div class="figure" style="text-align: center">
<img src="07-tweet-archives_files/figure-html/plotrts-1.png" alt="Words with highest median retweets" width="90%" />
<p class="caption">(\#fig:plotrts)Words with highest median retweets</p>
</div>

We see lots of word about R packages, including tidytext, a package about which you are reading right now!

We can follow a similar procedure to see which words led to more favorites. Are they different than the words that lead to more retweets?


```r
totals <- tidy_tweets %>% 
  group_by(person, id) %>% 
  summarise(favs = first(favorites)) %>% 
  group_by(person) %>% 
  summarise(total_favs = sum(favs))

word_by_favs <- tidy_tweets %>% 
  group_by(id, word, person) %>% 
  summarise(favs = first(favorites)) %>% 
  group_by(person, word) %>% 
  summarise(favorites = median(favs), uses = n()) %>%
  left_join(totals) %>%
  filter(favorites != 0) %>%
  ungroup()
```

We have built the data frames we need. Now let's make our visualization in Figure \@ref(fig:plotfavs).


```r
word_by_favs %>%
  filter(uses >= 5) %>%
  group_by(person) %>%
  slice_max(favorites, n = 10) %>% 
  arrange(favorites) %>%
  ungroup() %>%
  mutate(word = factor(word, unique(word))) %>%
  ungroup() %>%
  ggplot(aes(word, favorites, fill = person)) +
  geom_col(show.legend = FALSE) +
  facet_wrap(~ person, scales = "free", ncol = 2) +
  coord_flip() +
  labs(x = NULL, 
       y = "Median # of favorites for tweets containing each word")
```

<div class="figure" style="text-align: center">
<img src="07-tweet-archives_files/figure-html/plotfavs-1.png" alt="Words with highest median favorites" width="90%" />
<p class="caption">(\#fig:plotfavs)Words with highest median favorites</p>
</div>

We see some minor differences between Figures \@ref(fig:plotrts) and \@ref(fig:plotfavs), especially near the bottom of the top 10 list, but these are largely the same words as for retweets. In general, the same words that lead to retweets lead to favorites. A prominent word for Julia in both plots is the hashtag for the NASA Datanauts program that she has participated in; read on to Chapter \@ref(nasa) to learn more about NASA data and what we can learn from text analysis of NASA datasets. Wondering about the "=" in David's list?

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Me: You can&#39;t just add two p-values together.<br><br>Dev: The hell I can&#39;t:<br><br>newPval = pval1 + pval2;<br><br>Me: But-<br><br>Dev: Is all statistics this easy</p>&mdash; David Robinson (@drob) <a href="https://twitter.com/drob/status/714879071725993986?ref_src=twsrc%5Etfw">March 29, 2016</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


## Summary

This chapter was our first case study, a beginning-to-end analysis that demonstrates how to bring together the concepts and code we have been exploring in a cohesive way to understand a text data set. Comparing word frequencies allows us to see which words we tweeted more and less frequently, and the log odds ratio shows us which words are more likely to be tweeted from each of our accounts. We can use `nest()` and `map()` with the `glm()` function to find which words we have tweeted at higher and lower rates as time has passed. Finally, we can find which words in our tweets led to higher numbers of retweets and favorites. All of these are examples of approaches to measure how we use words in similar and different ways and how the characteristics of our tweets are changing or compare with each other. These are flexible approaches to text mining that can be applied to other types of text as well.
