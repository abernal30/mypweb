# Big data and data cleaning with datapro{#clean}


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

For this chapter we will use the file credit_semioriginal.xlsx, which has historical information of lendingclub, https://www.lendingclub.com/ fintech marketplace bank at scale. The original data set has at least 2 million observations and 150 variables. You will find the credit_semioriginal.xlsx with the first 1,000 observations and the 150 variables. using the 2 million rows sample would make our processor very low, but I challenge you to try the original data set to see what big data is. 

dataset source:
https://www.kaggle.com/wordsforthewise/lending-club


```r
library(openxlsx)
data <- read.xlsx("data/credit_semioriginal.xlsx", sheet =1)
```





Review the data structure of the credit dataset and descriptive statistics, only of the first 10 columns. 

```r
str(data[,1:10])
#> 'data.frame':	1000 obs. of  10 variables:
#>  $ loan_amnt      : num  3600 24700 20000 35000 10400 ...
#>  $ funded_amnt    : num  3600 24700 20000 35000 10400 ...
#>  $ funded_amnt_inv: num  3600 24700 20000 35000 10400 ...
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



```r
#col <- "term"
#cat <- data[,col][!duplicated(data[,col])]
#cat

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
head(col_cat2)
#> [1] 1 1 2 3 1 2
tail(col_cat2)
#> [1] 1 2 2 4 1 1
```


Finally, if we are sure that we want to transform all the data set into numerical, the function "asnum" reviews detect the categorical columns and transform it into numeric, and as a result we would get a data frame. If we apply the function and review now winch are categorical columns, we do not get any.  

```r
data2 <- asnum(data1)
head(charname(data2))
#> NULL
```


## Missing values

To treat missing values, I suggest taking one of the following alternatives or a combination of those: i) eliminating columns with a significant amount of missing values; ii) eliminating the row where the missing(s) value(s) is(are) located; iii) replace missing values or Na´s by some statistic. 





For the firs alternative, lets first apply the function "summaryna" to detect columns with more than 50 percent of missing values: 


```r

na_perc <- datapro::summaryna(data2,.5)
head(na_perc)
#>                             Percentage of NAs Column number
#> mths_since_last_record              0.8064147            27
#> next_pymnt_d                        1.0000000            45
#> mths_since_last_major_derog         0.7090493            50
#> annual_inc_joint                    0.9919817            53
#> dti_joint                           0.9919817            54
#> mths_since_recent_bc_dlq            0.7502864            86
```



In this case there are  30 columns with more than 50 percent of missing values. if we would like to eliminate those columns we apply the following:



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
#data3_1 <- na.omit(data2)
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
#> hardship_loan_status      289.33333     0.4581901   FALSE TRUE
#> disbursement_method         0.00000     0.1145475    TRUE TRUE
#> debt_settlement_flag       38.68182     0.2290951   FALSE TRUE
#> debt_settlement_flag_date 170.40000     1.3745704   FALSE TRUE
#> settlement_status          78.00000     0.3436426   FALSE TRUE
#> settlement_date           284.00000     1.7182131   FALSE TRUE
```



To understand better what the "nearZeroVar" function is doing, lets estimate the metrics for the settlement_date  columns, first we apply the function "table", which gives the frequency per category:


```r
t<-table(data3[,col])
t
#> 
#>   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15 
#> 852   2   2   2   3   1   1   1   1   2   1   1   1   2   1
```





There are 852 rows  with label 1, there are 2 rows  with label 2 and so on. 


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
#> 852
```

The second most frequent value would be


```r
max(t[-w])
#> [1] 3
```
Then, the "frequency ratio" is:

```r
t[w]/max(t[-w])
#>   1 
#> 284
```

By default, it has a threshold of 19 (or 95/5), which in terms of our object "nzv" would show only those column for which the "frequency ratio" are higher than 19.


Also, the nearZeroVar  function shows the "percent of unique values", which is the number of unique values divided by the total number of rows of the data frame (times 100). It approaches zero as the granularity of the data increases.

The percent unique is the number of categories, which in the case of  the 852 column is estimated applying first the function "length":


```r
length(table(data3[,col])) 
#> [1] 15
```
between the number of rows of the data frame, which we obtain applying the fucntio "dim":


```r
dim(data3)[1]
#> [1] 873
```


Then the "percent of unique values" is:

```r
(length(table(data3[,col]))/dim(data3)[1])*100
#> [1] 1.718213
```

The object "nzv" shows the "frequency ratio" and the "percent of unique values", however, to apply the filter and get only those columns with a "frequency ratio" and "percent of unique values" higher than the respective threshold we apply again the "nearZeroVar" but this time whitout the argument "saveMetrics= TRUE":


```r
nzv_2 <- nearZeroVar(data3)
nzv_2 
#>  [1]  14  26  32  33  34  39  47  48  49  50  51  73  74  92  93  94  99 104 105
#> [20] 106 107 108 109 110 111 112 113 114 115 116
```
The object nzv_2  shows the position of the colums for which the tresholds are higher, then we create other object excluding that columns.


```r
data4<-data3[,-nzv_2]
```

## Collinearity 

Collinearity is the situation in which two or more variables are closely related to one another. The presence of collinearity can pose problems in a model esimatio, such as regression, becasue it could be difficult to separate out the individual effects of collinear variables on the response [@statistical_lerarning]. 



```
#>  [1] "open_acc"                   "num_sats"                  
#>  [3] "total_rev_hi_lim"           "total_rec_prncp"           
#>  [5] "total_pymnt_inv"            "total_pymnt"               
#>  [7] "total_bc_limit"             "acc_open_past_24mths"      
#>  [9] "loan_amnt"                  "funded_amnt"               
#> [11] "funded_amnt_inv"            "num_rev_accts"             
#> [13] "num_tl_op_past_12m"         "num_bc_sats"               
#> [15] "total_bal_ex_mort"          "num_actv_rev_tl"           
#> [17] "tot_hi_cred_lim"            "tot_cur_bal"               
#> [19] "fico_range_low"             "last_fico_range_high"      
#> [21] "total_il_high_credit_limit" "revol_util"                
#> [23] "bc_util"                    "collection_recovery_fee"   
#> [25] "purpose"
```



```r
cw<-c("open_acc", "total_rev_hi_lim","total_acc","num_sats","total_bc_limit","num_rev_accts")
descrCor <-  data.frame(cor(data4[,cw]))
descrCor
#>                   open_acc total_rev_hi_lim total_acc  num_sats total_bc_limit
#> open_acc         1.0000000        0.3719736 0.7000888 0.9992150      0.2718850
#> total_rev_hi_lim 0.3719736        1.0000000 0.2692880 0.3698295      0.8376838
#> total_acc        0.7000888        0.2692880 1.0000000 0.6983321      0.1808618
#> num_sats         0.9992150        0.3698295 0.6983321 1.0000000      0.2714660
#> total_bc_limit   0.2718850        0.8376838 0.1808618 0.2714660      1.0000000
#> num_rev_accts    0.6443270        0.3646198 0.7793596 0.6403936      0.2783977
#>                  num_rev_accts
#> open_acc             0.6443270
#> total_rev_hi_lim     0.3646198
#> total_acc            0.7793596
#> num_sats             0.6403936
#> total_bc_limit       0.2783977
#> num_rev_accts        1.0000000
```

```r
panel.cor <- function(x, y, digits = 2, prefix = "", cex.cor, ...)
{
    usr <- par("usr"); on.exit(par(usr))
    par(usr = c(0, 1, 0, 1))
    r <- abs(cor(x, y,use="pairwise.complete.obs"))
    txt <- format(c(r, 0.123456789), digits = digits)[1]
    txt <- paste0(prefix, txt)
    if(missing(cex.cor)) cex.cor <- 0.8/strwidth(txt)
    text(0.5, 0.5, txt, cex = cex.cor * r)
}
pairs(data4[,cw],upper.panel=panel.cor,na.action = na.omit)
```

<img src="02-data-cleaning_files/figure-html/unnamed-chunk-44-1.png" width="90%" style="display: block; margin: auto;" />

open_acc, revol_bal,total_rev_hi_lim

total_acc

```r
descrCor <-  cor(data4)
highlyCorDescr <- findCorrelation(descrCor, cutoff = .75,names=T)
highlyCorDescr
#>  [1] "open_acc"                   "num_sats"                  
#>  [3] "total_acc"                  "total_rev_hi_lim"          
#>  [5] "total_rec_prncp"            "total_pymnt_inv"           
#>  [7] "total_pymnt"                "total_bc_limit"            
#>  [9] "acc_open_past_24mths"       "loan_amnt"                 
#> [11] "funded_amnt"                "funded_amnt_inv"           
#> [13] "num_op_rev_tl"              "num_rev_accts"             
#> [15] "num_tl_op_past_12m"         "num_bc_sats"               
#> [17] "total_bal_ex_mort"          "num_actv_rev_tl"           
#> [19] "tot_hi_cred_lim"            "open_rv_24m"               
#> [21] "num_rev_tl_bal_gt_0"        "tot_cur_bal"               
#> [23] "fico_range_low"             "last_fico_range_high"      
#> [25] "total_il_high_credit_limit" "revol_util"                
#> [27] "bc_util"                    "collection_recovery_fee"   
#> [29] "purpose"
```



```r
highCorr <- sum(abs(descrCor[upper.tri(descrCor)]) > .999)
highCorr 
#> [1] 6
```



