# Big data and data cleaning with datapro {#clean}


For this chapter, we will use a library I write for data processing, "datapro" To install it run the following code in a chunk:
To install it run:

library(devtools) 

Then:

remotes::install_github(“datanalyticss/datapro”)

or

devtools::install_github(“datanalyticss/datapro”)




```r
library(datapro)
```

For this chapter, we will use the file credit_semioriginal.xlsx, which has historical information on lendingclub, https://www.lendingclub.com/ fintech marketplace bank at scale. The original data set has at least 2 million observations and 150 variables. You will find the credit_semioriginal.xlsx with the first 1,000 observations and the 150 variables. Using the 2 million rows sample would make our processor very low, but I challenge you to try the original data set to see what big data is. 

dataset source:
https://www.kaggle.com/wordsforthewise/lending-club


```r
data<-read.csv("https://raw.githubusercontent.com/abernal30/BookAFP/main/data/credit_semioriginal.csv")
```


Review the data structure of the credit data set and descriptive statistics, only of the first 10 columns. 

```r
str(data[,1:10])
#> 'data.frame':	1000 obs. of  10 variables:
#>  $ loan_amnt      : int  3600 24700 20000 35000 10400 11950 20000 20000 10000 8000 ...
#>  $ funded_amnt    : int  3600 24700 20000 35000 10400 11950 20000 20000 10000 8000 ...
#>  $ funded_amnt_inv: int  3600 24700 20000 35000 10400 11950 20000 20000 10000 8000 ...
#>  $ term           : chr  "36 months" "36 months" "60 months" "60 months" ...
#>  $ int_rate       : num  14 12 10.8 14.8 22.4 ...
#>  $ installment    : num  123 820 433 830 290 ...
#>  $ grade          : chr  "C" "C" "B" "C" ...
#>  $ sub_grade      : chr  "C4" "C1" "B4" "C5" ...
#>  $ emp_title      : chr  "leadman" "Engineer" "truck driver" "Information Systems Officer" ...
#>  $ emp_length     : chr  "10+ years" "10+ years" "10+ years" "10+ years" ...
```


We could see that there are some numerical columns and some categorical. For categorical I mean that its elements are characters.

## Categorical into numerical: filtering and coditionals

There are several reasons to transform a numerical column or variable into categorical.  For a detailed explanation I suggest to review the chapter "Handling Text and Categorical Attributes" of the book "Machine learning introductory guide in R". For the moment that some functions we will use in this chapter will not work if the variables are categorical. 

If you see the "loan_status" variable is categorical. First we review how many categories does the column loan_status has:

data[,col][!duplicated(data[,"col"])]

where data is the name of the dataframe and col is the column name 

```r
col <- "loan_status"
data[,col][!duplicated(data[,col])]
#> [1] "Fully Paid"         "Current"            "Charged Off"       
#> [4] "In Grace Period"    "Late (31-120 days)"
```


Another possibility is applying the function categ of library datapro

```r
categ(data,col)
#> [1] "Fully Paid"         "Current"            "Charged Off"       
#> [4] "In Grace Period"    "Late (31-120 days)"
```



```
#> [1] 5
```

There are 5 categories, but we going to transform the column verification_status into numeric:

Create a filter, in such a way that the loan_status contains only Fully Paid and Charged Off.

data %>%
  filter(col== "categ1" |col== "categ2")

```r
library(dplyr)
#col <- "loan_status"
data1 <- data %>%
  filter(data[,"loan_status"] == "Fully Paid" | data[,"loan_status"] == "Charged Off")
```



```
#> [1] 873
```

As a result, now we only have 873 rows. 


Besides "loan_status" three are several categorical columns, for example term, winch has 2 categories:


```r
col <- "term"
cat <- categ(data,col)
cat
#> [1] "36 months" "60 months"
```

The method we use to transform is simple, in this example "36 months" will take the value of one and "60 months" the value of 2. If the column would have 3 categories, the 3rd categories would take value 3 and so on. 



```r
ncat <- c(1:length(cat))
ncat
#> [1] 1 2
```


```r
cat[1]
#> [1] "36 months"
```


```r
col_cat <- ifelse(data1[, col] == cat[1],ncat[1],data1[, col])
head(col_cat)
#> [1] "1"         "1"         "60 months" "60 months" "1"         "1"
```


```r
col_cat <- ifelse(data1[, col] == cat[1],ncat[1],ncat[2])
head(col_cat)
#> [1] 1 1 2 2 1 1
tail(col_cat)
#> [1] 1 1 1 1 1 1
```

The former example was easy because we only have 3 categories, however, there are other 


We use the charname function to see how many categorical variables there are. We print only the first rows using the head function.



```r
data1[1,"mths_since_recent_bc"]*2
#> [1] 8
```



```r
head(charname(data1))
#> [1] "term"           "grade"          "sub_grade"      "emp_title"     
#> [5] "emp_length"     "home_ownership"
tail(charname(data1))
#> [1] "hardship_loan_status"      "disbursement_method"      
#> [3] "debt_settlement_flag"      "debt_settlement_flag_date"
#> [5] "settlement_status"         "settlement_date"
```



```
#> [1] 33
```

There are 33 categorical columns. The function "tonum" transform a categorical column into numeric, for example transforming column "grade", it has the following categories:

```r
col <- "grade"

cat <- categ(data,col)
#cat <- data[,col][!duplicated(data[,col])]
cat
#> [1] "C" "B" "F" "A" "E" "D" "G"
```

We need to specify the data source and the column name.

```r
col_cat2 <- tonum(data1,col)
head(col_cat2[,1:5])
#>   loan_amnt funded_amnt funded_amnt_inv      term int_rate
#> 1      3600        3600            3600 36 months    13.99
#> 2     24700       24700           24700 36 months    11.99
#> 3     20000       20000           20000 60 months    10.78
#> 4     10400       10400           10400 60 months    22.45
#> 5     11950       11950           11950 36 months    13.44
#> 6     20000       20000           20000 36 months     9.17
```



Finally, if we are sure that we want to transform all the data set into numerical, the function "asnum" reviews detect the categorical columns and transform it into numeric, and as a result we would get a data frame. If we apply the function and review now winch are categorical columns, we do not get any.  





```r
# Warning: this code may take a few minutes to finish, depending on the processor.
data2 <- datapro::asnum(data1)
head(charname(data2))
```


```
#> [1] "There are no character columns"
```

If, for some reason, you get an error, you could run the following code:

```r
data2<-read.csv("https://raw.githubusercontent.com/abernal30/BookAFP/main/data/credit_semioriginal_num.csv")
```


## Missing values

To treat missing values, I suggest taking one of the following alternatives or a combination of those: i) eliminating columns with a significant amount of missing values; ii) eliminating the row where the missing(s) value(s) is(are) located; iii) replace missing values or Na´s by some statistic. 





For the firs alternative, lets first apply the function "summaryna" to detect columns with more than 50 percent of missing values: 


```r

na_perc <- datapro::summaryna(data2,.5)
head(na_perc)
#>                                Percentage of NAs Column number
#> mths_since_last_record                 0.8064147            27
#> mths_since_last_major_derog            0.7090493            50
#> annual_inc_joint                       0.9919817            53
#> dti_joint                              0.9919817            54
#> mths_since_recent_bc_dlq               0.7502864            86
#> mths_since_recent_revol_delinq         0.6575029            88
```



In this case there are  29 columns with more than 50 percent of missing values. if we would like to eliminate those columns we apply the following:



```r
data3 <- data2[,-na_perc[,2]]
```
To confirm, we apply again the function summaryna

```r
summaryna(data3,.5)
#> [1] "There are no columns with missing values"
```


for the second alternative, which is eliminating the rows where the missing(s) value(s) is(are) located; we could applying the na.omit function. However, we have to be careful, because it could be the case that each row of the data frame has at least one missing value, in which cace it would delete all rows of the data frame, like this case: 

```r
data3_1 <- na.omit(data2)
```


The third alternative is replacing missing values by a metric. In this  we us the function "repnas", to the object data3 wich already has the drop the columns with more than 50 percent of missing values:

```r

data3 <- datapro::repnas(data3,"median")
```



## Zero- and Near Zero-Variance Predictors

We wil use the library [@R-caret] for this section. Zero- and Near Zero-Variance Predictors  are variables or columns that only have a single unique value, winch is refereed as a “zero-variance predictor”. Also, the variables might have only a a few unique values that occur with very low frequencies. In both cases it may cause troubles when estimating an econometric or machine learning model. 

The function nearZeroVar shows the columns number of the Zero- and Near Zero-Variance Predictors. 



```r
library(caret)
nzv <- nearZeroVar(data3,saveMetrics= TRUE)
head(nzv)
#>                 freqRatio percentUnique zeroVar   nzv
#> loan_amnt        1.173077    24.7422680   FALSE FALSE
#> funded_amnt      1.173077    24.7422680   FALSE FALSE
#> funded_amnt_inv  1.173077    24.7422680   FALSE FALSE
#> term             3.546875     0.2290951   FALSE FALSE
#> int_rate         1.129032     3.7800687   FALSE FALSE
#> installment      1.125000    70.4467354   FALSE FALSE
```

```r
tail(nzv)
#>                           freqRatio percentUnique zeroVar  nzv
#> hardship_loan_status      289.00000     0.5727377   FALSE TRUE
#> disbursement_method         0.00000     0.1145475    TRUE TRUE
#> debt_settlement_flag       38.68182     0.2290951   FALSE TRUE
#> debt_settlement_flag_date 170.20000     1.4891180   FALSE TRUE
#> settlement_status          77.36364     0.4581901   FALSE TRUE
#> settlement_date           283.66667     1.8327606   FALSE TRUE
```



To understand better what the "nearZeroVar" function is doing, lets estimate the metrics for the settlement_date  columns, first we apply the function "table", which gives the frequency per category:


```r
t<-table(data3[,col])
t
#> 
#>   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15  16 
#> 851   1   1   1   2   1   1   2   2   1   1   1   1   3   2   2
```





There are 851 rows  with label 1, there are 1 rows  with label 2 and so on. 


The "frequency ratio" is the frequency of the most prevalent value over the second most frequent value. It  would be near one for well-behaved predictors and very large for highly-unbalanced, for the "grade" column it would be:

To estimate the "frequency ratio" e apply the "which.max" function that gives the position of the frequency of the most prevalent value:

```r
w <- which.max(t)
w
#> 1 
#> 1
```
To get the most frequent value:

```r
t[w]
#>   1 
#> 851
```

The second most frequent value would be


```r
max(t[-w])
#> [1] 3
```
Then, the "frequency ratio" is:

```r
t[w]/max(t[-w])
#>        1 
#> 283.6667
```

By default, it has a threshold of 19 (or 95/5), which in terms of our object "nzv" would show only that column for which the "frequency ratio" are higher than 19.


Also, the nearZeroVar function shows the "percent of unique values," which is the number of unique values divided by the total number of rows of the data frame (times 100). It approaches zero as the granularity of the data increases.

The percent unique is the number of categories, which in the case of  the 851 column is estimated by applying first the function "length":


```r
length(table(data3[,col])) 
#> [1] 16
```
between the number of rows of the data frame, which we obtain applying the fucntio "dim":


```r
dim(data3)[1]
#> [1] 873
```


Then the "percent of unique values" is:

```r
(length(table(data3[,col]))/dim(data3)[1])*100
#> [1] 1.832761
```

The object "nzv" shows the "frequency ratio" and the "percent of unique values"; however, to apply the filter and get only those columns with a "frequency ratio" and "percent of unique values" higher than the respective threshold we apply the "nearZeroVar" again but this time without the argument "saveMetrics= TRUE":


```r
nzv_2 <- nearZeroVar(data3)
nzv_2 
#>  [1]  14  26  32  33  34  39  44  48  49  50  51  52  74  75  93  94  95 100 105
#> [20] 106 107 108 109 110 111 112 113 114 115 116 117
```
The object nzv_2  shows the position of the colums for which the tresholds are higher, then we create other object excluding that columns.


```r
data4<-data3[,-nzv_2]
```


## Collinearity 

Collinearity is when two or more variables are closely related to one another. The presence of collinearity can pose problems in model estimation, such as regression, because it could be difficult to separate the individual effects of collinear variables on the response  [@statistical_lerarning]. 


The function "cor" is to estimate the correlation matrix, and the function "findCorrelation" shows the correlated variables more than "n" (cutoff argument). For this case, we apply a cut-off of 80%. 
```{r}
descrCor <-  cor(data4)
hc <- findCorrelation(descrCor, cutoff = .8, names = T)
hc
#>  [1] "open_acc"                   "num_sats"                  
#>  [3] "total_rev_hi_lim"           "total_bc_limit"            
#>  [5] "total_rec_prncp"            "acc_open_past_24mths"      
#>  [7] "total_pymnt_inv"            "total_pymnt"               
#>  [9] "loan_amnt"                  "funded_amnt"               
#> [11] "funded_amnt_inv"            "num_tl_op_past_12m"        
#> [13] "sub_grade"                  "int_rate"                  
#> [15] "num_rev_accts"              "num_bc_sats"               
#> [17] "tot_hi_cred_lim"            "total_bal_ex_mort"         
#> [19] "num_actv_rev_tl"            "fico_range_low"            
#> [21] "last_fico_range_high"       "tot_cur_bal"               
#> [23] "revol_util"                 "total_il_high_credit_limit"
#> [25] "bc_util"                    "collection_recovery_fee"
```

The argument  "names = T" is only to get the column's names of correlated variables. Still, if we wish to cut off those variables from our data frame, we do not add that argument, getting only the column numbers:

```r
hc<-findCorrelation(descrCor, cutoff = .8)
hc
#>  [1] 25 78 54 85 32 58 31 30  1  2  3 79  8  5 76 72 83 84 71 22 39 42 28 86 61
#> [26] 35
```

We could cut those variables from data4 in the following way:


```r
data_corr <- data4[ , -hc]
head(data_corr[,1:5])
#>   term installment grade emp_title emp_length
#> 1    1      123.03     3       300          4
#> 2    1      820.28     3       210          4
#> 3    2      432.66     2       624          4
#> 4    2      289.91     6       127          6
#> 5    1      405.18     3       634          7
#> 6    1      637.58     2       637          4
```



