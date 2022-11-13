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
data<-read.csv("data/credit_semioriginal.csv")
```


Review the data structure of the credit data set and descriptive statistics, only of the first 10 columns. 

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
#>   loan_amnt funded_amnt funded_amnt_inv      term int_rate installment grade
#> 1      3600        3600            3600 36 months    13.99      123.03     3
#> 2     24700       24700           24700 36 months    11.99      820.28     3
#> 3     20000       20000           20000 60 months    10.78      432.66     2
#> 4     10400       10400           10400 60 months    22.45      289.91     6
#> 5     11950       11950           11950 36 months    13.44      405.18     3
#> 6     20000       20000           20000 36 months     9.17      637.58     2
#>   sub_grade                               emp_title emp_length home_ownership
#> 1        C4                                 leadman  10+ years       MORTGAGE
#> 2        C1                                Engineer  10+ years       MORTGAGE
#> 3        B4                            truck driver  10+ years       MORTGAGE
#> 4        F1                     Contract Specialist    3 years       MORTGAGE
#> 5        C3                    Veterinary Tecnician    4 years           RENT
#> 6        B2 Vice President of Recruiting Operations  10+ years       MORTGAGE
#>   annual_inc verification_status  issue_d loan_status            purpose
#> 1      55000        Not Verified Dec-2015  Fully Paid debt_consolidation
#> 2      65000        Not Verified Dec-2015  Fully Paid     small_business
#> 3      63000        Not Verified Dec-2015  Fully Paid   home_improvement
#> 4     104433     Source Verified Dec-2015  Fully Paid     major_purchase
#> 5      34000     Source Verified Dec-2015  Fully Paid debt_consolidation
#> 6     180000        Not Verified Dec-2015  Fully Paid debt_consolidation
#>                title zip_code addr_state   dti delinq_2yrs earliest_cr_line
#> 1 Debt consolidation    190xx         PA  5.91           0         Aug-2003
#> 2           Business    577xx         SD 16.06           1         Dec-1999
#> 3               <NA>    605xx         IL 10.78           0         Aug-2000
#> 4     Major purchase    174xx         PA 25.37           1         Jun-1998
#> 5 Debt consolidation    300xx         GA 10.20           0         Oct-1987
#> 6 Debt consolidation    550xx         MN 14.67           0         Jun-1990
#>   fico_range_low fico_range_high inq_last_6mths mths_since_last_delinq
#> 1            675             679              1                     30
#> 2            715             719              4                      6
#> 3            695             699              0                     NA
#> 4            695             699              3                     12
#> 5            690             694              0                     NA
#> 6            680             684              0                     49
#>   mths_since_last_record open_acc pub_rec revol_bal revol_util total_acc
#> 1                     NA        7       0      2765       29.7        13
#> 2                     NA       22       0     21470       19.2        38
#> 3                     NA        6       0      7869       56.2        18
#> 4                     NA       12       0     21929       64.5        35
#> 5                     NA        5       0      8822       68.4         6
#> 6                     NA       12       0     87329       84.5        27
#>   initial_list_status out_prncp out_prncp_inv total_pymnt total_pymnt_inv
#> 1                   w         0             0    4421.724         4421.72
#> 2                   w         0             0   25679.660        25679.66
#> 3                   w         0             0   22705.924        22705.92
#> 4                   w         0             0   11740.500        11740.50
#> 5                   w         0             0   13708.949        13708.95
#> 6                   f         0             0   21393.800        21393.80
#>   total_rec_prncp total_rec_int total_rec_late_fee recoveries
#> 1            3600        821.72                  0          0
#> 2           24700        979.66                  0          0
#> 3           20000       2705.92                  0          0
#> 4           10400       1340.50                  0          0
#> 5           11950       1758.95                  0          0
#> 6           20000       1393.80                  0          0
#>   collection_recovery_fee last_pymnt_d last_pymnt_amnt next_pymnt_d
#> 1                       0     Jan-2019          122.67         <NA>
#> 2                       0     Jun-2016          926.35         <NA>
#> 3                       0     Jun-2017        15813.30         <NA>
#> 4                       0     Jul-2016        10128.96         <NA>
#> 5                       0     May-2017         7653.56         <NA>
#> 6                       0     Nov-2016        15681.05         <NA>
#>   last_credit_pull_d last_fico_range_high last_fico_range_low
#> 1           Mar-2019                  564                 560
#> 2           Mar-2019                  699                 695
#> 3           Mar-2019                  704                 700
#> 4           Mar-2018                  704                 700
#> 5           May-2017                  759                 755
#> 6           Mar-2019                  654                 650
#>   collections_12_mths_ex_med mths_since_last_major_derog policy_code
#> 1                          0                          30           1
#> 2                          0                          NA           1
#> 3                          0                          NA           1
#> 4                          0                          NA           1
#> 5                          0                          NA           1
#> 6                          0                          NA           1
#>   application_type annual_inc_joint dti_joint verification_status_joint
#> 1       Individual               NA        NA                      <NA>
#> 2       Individual               NA        NA                      <NA>
#> 3        Joint App            71000     13.85              Not Verified
#> 4       Individual               NA        NA                      <NA>
#> 5       Individual               NA        NA                      <NA>
#> 6       Individual               NA        NA                      <NA>
#>   acc_now_delinq tot_coll_amt tot_cur_bal open_acc_6m open_act_il open_il_12m
#> 1              0          722      144904           2           2           0
#> 2              0            0      204396           1           1           0
#> 3              0            0      189699           0           1           0
#> 4              0            0      331730           1           3           0
#> 5              0            0       12798           0           1           0
#> 6              0            0      360358           0           2           0
#>   open_il_24m mths_since_rcnt_il total_bal_il il_util open_rv_12m open_rv_24m
#> 1           1                 21         4981      36           3           3
#> 2           1                 19        18005      73           2           3
#> 3           4                 19        10827      73           0           2
#> 4           3                 14        73839      84           4           7
#> 5           0                338         3976      99           0           0
#> 6           2                 18        29433      63           2           3
#>   max_bal_bc all_util total_rev_hi_lim inq_fi total_cu_tl inq_last_12m
#> 1        722       34             9300      3           1            4
#> 2       6472       29           111800      0           0            6
#> 3       2081       65            14000      2           5            1
#> 4       9702       78            34000      2           1            3
#> 5       4522       76            12900      0           0            0
#> 6      13048       74            94200      1           0            1
#>   acc_open_past_24mths avg_cur_bal bc_open_to_buy bc_util
#> 1                    4       20701           1506    37.2
#> 2                    4        9733          57830    27.1
#> 3                    6       31617           2737    55.9
#> 4                   10       27644           4567    77.5
#> 5                    0        2560            844    91.0
#> 6                    6       30030              0   102.9
#>   chargeoff_within_12_mths delinq_amnt mo_sin_old_il_acct mo_sin_old_rev_tl_op
#> 1                        0           0                148                  128
#> 2                        0           0                113                  192
#> 3                        0           0                125                  184
#> 4                        0           0                128                  210
#> 5                        0           0                338                   54
#> 6                        0           0                142                  306
#>   mo_sin_rcnt_rev_tl_op mo_sin_rcnt_tl mort_acc mths_since_recent_bc
#> 1                     3              3        1                    4
#> 2                     2              2        4                    2
#> 3                    14             14        5                  101
#> 4                     4              4        6                    4
#> 5                    32             32        0                   36
#> 6                    10             10        4                   12
#>   mths_since_recent_bc_dlq mths_since_recent_inq mths_since_recent_revol_delinq
#> 1                       69                     4                             69
#> 2                       NA                     0                              6
#> 3                       NA                    10                             NA
#> 4                       12                     1                             12
#> 5                       NA                    NA                             NA
#> 6                       NA                    10                             NA
#>   num_accts_ever_120_pd num_actv_bc_tl num_actv_rev_tl num_bc_sats num_bc_tl
#> 1                     2              2               4           2         5
#> 2                     0              5               5          13        17
#> 3                     0              2               3           2         4
#> 4                     0              4               6           5         9
#> 5                     0              2               3           2         2
#> 6                     0              4               6           4         5
#>   num_il_tl num_op_rev_tl num_rev_accts num_rev_tl_bal_gt_0 num_sats
#> 1         3             4             9                   4        7
#> 2         6            20            27                   5       22
#> 3         6             4             7                   3        6
#> 4        10             7            19                   6       12
#> 5         2             4             4                   3        5
#> 6         7             9            16                   6       12
#>   num_tl_120dpd_2m num_tl_30dpd num_tl_90g_dpd_24m num_tl_op_past_12m
#> 1                0            0                  0                  3
#> 2                0            0                  0                  2
#> 3                0            0                  0                  0
#> 4                0            0                  0                  4
#> 5                0            0                  0                  0
#> 6                0            0                  0                  2
#>   pct_tl_nvr_dlq percent_bc_gt_75 pub_rec_bankruptcies tax_liens
#> 1           76.9              0.0                    0         0
#> 2           97.4              7.7                    0         0
#> 3          100.0             50.0                    0         0
#> 4           96.6             60.0                    0         0
#> 5          100.0            100.0                    0         0
#> 6           96.3            100.0                    0         0
#>   tot_hi_cred_lim total_bal_ex_mort total_bc_limit total_il_high_credit_limit
#> 1          178050              7746           2400                      13734
#> 2          314017             39475          79300                      24667
#> 3          218418             18696           6200                      14877
#> 4          439570             95768          20300                      88097
#> 5           16900             12798           9400                       4000
#> 6          388852            116762          31500                      46452
#>   revol_bal_joint sec_app_fico_range_low sec_app_fico_range_high
#> 1              NA                     NA                      NA
#> 2              NA                     NA                      NA
#> 3              NA                     NA                      NA
#> 4              NA                     NA                      NA
#> 5              NA                     NA                      NA
#> 6              NA                     NA                      NA
#>   sec_app_earliest_cr_line sec_app_inq_last_6mths sec_app_mort_acc
#> 1                       NA                     NA               NA
#> 2                       NA                     NA               NA
#> 3                       NA                     NA               NA
#> 4                       NA                     NA               NA
#> 5                       NA                     NA               NA
#> 6                       NA                     NA               NA
#>   sec_app_open_acc sec_app_revol_util sec_app_open_act_il sec_app_num_rev_accts
#> 1               NA                 NA                  NA                    NA
#> 2               NA                 NA                  NA                    NA
#> 3               NA                 NA                  NA                    NA
#> 4               NA                 NA                  NA                    NA
#> 5               NA                 NA                  NA                    NA
#> 6               NA                 NA                  NA                    NA
#>   sec_app_chargeoff_within_12_mths sec_app_collections_12_mths_ex_med
#> 1                               NA                                 NA
#> 2                               NA                                 NA
#> 3                               NA                                 NA
#> 4                               NA                                 NA
#> 5                               NA                                 NA
#> 6                               NA                                 NA
#>   sec_app_mths_since_last_major_derog hardship_flag hardship_type
#> 1                                  NA             N          <NA>
#> 2                                  NA             N          <NA>
#> 3                                  NA             N          <NA>
#> 4                                  NA             N          <NA>
#> 5                                  NA             N          <NA>
#> 6                                  NA             N          <NA>
#>   hardship_reason hardship_status deferral_term hardship_amount
#> 1            <NA>            <NA>            NA              NA
#> 2            <NA>            <NA>            NA              NA
#> 3            <NA>            <NA>            NA              NA
#> 4            <NA>            <NA>            NA              NA
#> 5            <NA>            <NA>            NA              NA
#> 6            <NA>            <NA>            NA              NA
#>   hardship_start_date hardship_end_date payment_plan_start_date hardship_length
#> 1                <NA>              <NA>                    <NA>              NA
#> 2                <NA>              <NA>                    <NA>              NA
#> 3                <NA>              <NA>                    <NA>              NA
#> 4                <NA>              <NA>                    <NA>              NA
#> 5                <NA>              <NA>                    <NA>              NA
#> 6                <NA>              <NA>                    <NA>              NA
#>   hardship_dpd hardship_loan_status orig_projected_additional_accrued_interest
#> 1           NA                 <NA>                                         NA
#> 2           NA                 <NA>                                         NA
#> 3           NA                 <NA>                                         NA
#> 4           NA                 <NA>                                         NA
#> 5           NA                 <NA>                                         NA
#> 6           NA                 <NA>                                         NA
#>   hardship_payoff_balance_amount hardship_last_payment_amount
#> 1                             NA                           NA
#> 2                             NA                           NA
#> 3                             NA                           NA
#> 4                             NA                           NA
#> 5                             NA                           NA
#> 6                             NA                           NA
#>   disbursement_method debt_settlement_flag debt_settlement_flag_date
#> 1                Cash                    N                      <NA>
#> 2                Cash                    N                      <NA>
#> 3                Cash                    N                      <NA>
#> 4                Cash                    N                      <NA>
#> 5                Cash                    N                      <NA>
#> 6                Cash                    N                      <NA>
#>   settlement_status settlement_date settlement_amount settlement_percentage
#> 1              <NA>            <NA>                NA                    NA
#> 2              <NA>            <NA>                NA                    NA
#> 3              <NA>            <NA>                NA                    NA
#> 4              <NA>            <NA>                NA                    NA
#> 5              <NA>            <NA>                NA                    NA
#> 6              <NA>            <NA>                NA                    NA
#>   settlement_term
#> 1              NA
#> 2              NA
#> 3              NA
#> 4              NA
#> 5              NA
#> 6              NA
tail(col_cat2)
#>     loan_amnt funded_amnt funded_amnt_inv      term int_rate installment grade
#> 868      4175        4175            4175 36 months    12.88      140.44     3
#> 869      5000        5000            5000 36 months    11.48      164.84     2
#> 870     20000       20000           20000 36 months     9.17      637.58     2
#> 871     30000       30000           30000 36 months     6.99      926.18     1
#> 872      5500        5500            5500 36 months    11.99      182.66     3
#> 873     30000       30000           30000 36 months    12.88     1009.09     3
#>     sub_grade                emp_title emp_length home_ownership annual_inc
#> 868        C2    Child Support Officer    4 years       MORTGAGE      50720
#> 869        B5             County Clerk    8 years            OWN      43498
#> 870        B2                     <NA>       <NA>       MORTGAGE      60000
#> 871        A3 Director Strategic Sales  10+ years       MORTGAGE     202000
#> 872        C1      Service Coordinator    3 years           RENT      37000
#> 873        C2       Operations Manager    3 years           RENT     100000
#>     verification_status  issue_d loan_status            purpose
#> 868     Source Verified Dec-2015  Fully Paid debt_consolidation
#> 869            Verified Dec-2015 Charged Off debt_consolidation
#> 870     Source Verified Dec-2015  Fully Paid        credit_card
#> 871        Not Verified Dec-2015  Fully Paid        credit_card
#> 872     Source Verified Dec-2015  Fully Paid debt_consolidation
#> 873            Verified Dec-2015  Fully Paid debt_consolidation
#>                       title zip_code addr_state   dti delinq_2yrs
#> 868      Debt consolidation    563xx         MN  9.16           0
#> 869      Debt consolidation    768xx         TX 26.20           1
#> 870 Credit card refinancing    992xx         WA 10.45           0
#> 871 Credit card refinancing    038xx         NH 11.65           0
#> 872      Debt consolidation    955xx         CA 21.15           0
#> 873      Debt consolidation    944xx         CA 16.21           0
#>     earliest_cr_line fico_range_low fico_range_high inq_last_6mths
#> 868         Nov-2003            665             669              0
#> 869         Nov-1989            660             664              1
#> 870         Apr-2000            665             669              0
#> 871         Apr-1994            730             734              0
#> 872         Oct-2002            720             724              0
#> 873         Mar-2000            695             699              0
#>     mths_since_last_delinq mths_since_last_record open_acc pub_rec revol_bal
#> 868                     46                     NA        9       0      1779
#> 869                     21                     NA        5       0      3385
#> 870                     NA                     76       10       1     10874
#> 871                     NA                     NA        6       0     30349
#> 872                     NA                     NA        7       0      5914
#> 873                     NA                     NA       14       0     54554
#>     revol_util total_acc initial_list_status out_prncp out_prncp_inv
#> 868       85.0        20                   w         0             0
#> 869       75.2        24                   w         0             0
#> 870       52.3        36                   w         0             0
#> 871       82.2        27                   w         0             0
#> 872       81.0        21                   w         0             0
#> 873       56.0        24                   w         0             0
#>     total_pymnt total_pymnt_inv total_rec_prncp total_rec_int
#> 868    4646.682         4646.68         4175.00        471.68
#> 869    5104.710         5104.71         3602.61        925.10
#> 870   21664.666        21664.67        20000.00       1632.79
#> 871   31106.100        31106.10        30000.00       1106.10
#> 872    6065.314         6065.31         5500.00        565.31
#> 873   35509.700        35509.70        30000.00       5509.70
#>     total_rec_late_fee recoveries collection_recovery_fee last_pymnt_d
#> 868               0.00          0                    0.00     Jan-2017
#> 869              15.00        562                  101.16     Sep-2018
#> 870              31.88          0                    0.00     Jan-2017
#> 871               0.00          0                    0.00     Aug-2016
#> 872               0.00          0                    0.00     Jan-2017
#> 873               0.00          0                    0.00     Jan-2018
#>     last_pymnt_amnt next_pymnt_d last_credit_pull_d last_fico_range_high
#> 868         3109.31         <NA>           Sep-2017                  669
#> 869          100.00         <NA>           Dec-2018                  544
#> 870        14671.67         <NA>           Mar-2019                  674
#> 871        25578.14         <NA>           Mar-2018                  789
#> 872         3720.46         <NA>           Jan-2017                  694
#> 873        12354.30         <NA>           Sep-2017                  749
#>     last_fico_range_low collections_12_mths_ex_med mths_since_last_major_derog
#> 868                 665                          0                          40
#> 869                 540                          0                          73
#> 870                 670                          0                          NA
#> 871                 785                          0                          NA
#> 872                 690                          0                          NA
#> 873                 745                          0                          NA
#>     policy_code application_type annual_inc_joint dti_joint
#> 868           1       Individual               NA        NA
#> 869           1       Individual               NA        NA
#> 870           1       Individual               NA        NA
#> 871           1       Individual               NA        NA
#> 872           1       Individual               NA        NA
#> 873           1       Individual               NA        NA
#>     verification_status_joint acc_now_delinq tot_coll_amt tot_cur_bal
#> 868                      <NA>              0            0      105575
#> 869                      <NA>              0            0       22461
#> 870                      <NA>              0            0      189122
#> 871                      <NA>              0            0      457095
#> 872                      <NA>              0            0       33858
#> 873                      <NA>              0            0       61746
#>     open_acc_6m open_act_il open_il_12m open_il_24m mths_since_rcnt_il
#> 868           0           6           0           1                 17
#> 869           3           3           2           5                  2
#> 870           3           1           2           2                  5
#> 871           0           2           0           2                 21
#> 872           0           3           0           0                 28
#> 873           0           1           1           1                 11
#>     total_bal_il il_util open_rv_12m open_rv_24m max_bal_bc all_util
#> 868       103796     101           1           2        916       85
#> 869        19076      72           1           1       1613       73
#> 870         7050      94           4           4       2092       63
#> 871        14089      41           0           2      18978       63
#> 872        27944      77           0           0       2264       77
#> 873         7192      84           1           2      12445       59
#>     total_rev_hi_lim inq_fi total_cu_tl inq_last_12m acc_open_past_24mths
#> 868             2100      0           0            0                    3
#> 869             4500      0           0            1                    6
#> 870            20800      2          15            4                    6
#> 871            36900      1           5            2                    5
#> 872             7300      0           1            0                    0
#> 873           118500      0           0            0                    3
#>     avg_cur_bal bc_open_to_buy bc_util chargeoff_within_12_mths delinq_amnt
#> 868       11730            700    85.0                        0           0
#> 869        4492            687    70.1                        0           0
#> 870       23640           2510    54.4                        0           0
#> 871       76183           6551    82.2                        0           0
#> 872        5643             36    98.4                        0           0
#> 873        4750          63584    55.3                        0           0
#>     mo_sin_old_il_acct mo_sin_old_rev_tl_op mo_sin_rcnt_rev_tl_op
#> 868                145                  113                    11
#> 869                 41                  313                     2
#> 870                120                  188                     3
#> 871                117                  260                    17
#> 872                152                  158                    32
#> 873                117                  189                     7
#>     mo_sin_rcnt_tl mort_acc mths_since_recent_bc mths_since_recent_bc_dlq
#> 868             11        2                   11                       NA
#> 869              2        0                    2                       73
#> 870              3        5                    4                       NA
#> 871              8        6                   17                       NA
#> 872             28        0                   61                       NA
#> 873              7        0                    7                       NA
#>     mths_since_recent_inq mths_since_recent_revol_delinq num_accts_ever_120_pd
#> 868                    13                             NA                     1
#> 869                     5                             73                     3
#> 870                     7                             NA                     0
#> 871                     9                             NA                     0
#> 872                    NA                             NA                     0
#> 873                    NA                             NA                     0
#>     num_actv_bc_tl num_actv_rev_tl num_bc_sats num_bc_tl num_il_tl
#> 868              3               3           4         4        12
#> 869              1               2           1         9         8
#> 870              2               3           2         9         8
#> 871              3               3           3         5         7
#> 872              1               3           2         3        14
#> 873              6               7          11        15         2
#>     num_op_rev_tl num_rev_accts num_rev_tl_bal_gt_0 num_sats num_tl_120dpd_2m
#> 868             3             4                   3        9               NA
#> 869             2            16                   2        5                0
#> 870             8            23                   3       10                0
#> 871             3            14                   3        6                0
#> 872             4             7                   3        7                0
#> 873            13            22                   7       14                0
#>     num_tl_30dpd num_tl_90g_dpd_24m num_tl_op_past_12m pct_tl_nvr_dlq
#> 868            0                  0                  1             80
#> 869            0                  0                  3             75
#> 870            0                  0                  6            100
#> 871            0                  0                  1            100
#> 872            0                  0                  0            100
#> 873            0                  0                  2            100
#>     percent_bc_gt_75 pub_rec_bankruptcies tax_liens tot_hi_cred_lim
#> 868             66.7                    0         0          105046
#> 869              0.0                    0         0           30965
#> 870             50.0                    1         0          216650
#> 871             66.7                    0         0          488105
#> 872            100.0                    0         0           43788
#> 873             36.4                    0         0          127080
#>     total_bal_ex_mort total_bc_limit total_il_high_credit_limit revol_bal_joint
#> 868            105575           2100                     102946              NA
#> 869             22461           2300                      26465              NA
#> 870             17924           5500                       7500              NA
#> 871             44438          36900                      34205              NA
#> 872             33858           2300                      36488              NA
#> 873             61746         116600                       8580              NA
#>     sec_app_fico_range_low sec_app_fico_range_high sec_app_earliest_cr_line
#> 868                     NA                      NA                       NA
#> 869                     NA                      NA                       NA
#> 870                     NA                      NA                       NA
#> 871                     NA                      NA                       NA
#> 872                     NA                      NA                       NA
#> 873                     NA                      NA                       NA
#>     sec_app_inq_last_6mths sec_app_mort_acc sec_app_open_acc sec_app_revol_util
#> 868                     NA               NA               NA                 NA
#> 869                     NA               NA               NA                 NA
#> 870                     NA               NA               NA                 NA
#> 871                     NA               NA               NA                 NA
#> 872                     NA               NA               NA                 NA
#> 873                     NA               NA               NA                 NA
#>     sec_app_open_act_il sec_app_num_rev_accts sec_app_chargeoff_within_12_mths
#> 868                  NA                    NA                               NA
#> 869                  NA                    NA                               NA
#> 870                  NA                    NA                               NA
#> 871                  NA                    NA                               NA
#> 872                  NA                    NA                               NA
#> 873                  NA                    NA                               NA
#>     sec_app_collections_12_mths_ex_med sec_app_mths_since_last_major_derog
#> 868                                 NA                                  NA
#> 869                                 NA                                  NA
#> 870                                 NA                                  NA
#> 871                                 NA                                  NA
#> 872                                 NA                                  NA
#> 873                                 NA                                  NA
#>     hardship_flag hardship_type hardship_reason hardship_status deferral_term
#> 868             N          <NA>            <NA>            <NA>            NA
#> 869             N          <NA>            <NA>            <NA>            NA
#> 870             N          <NA>            <NA>            <NA>            NA
#> 871             N          <NA>            <NA>            <NA>            NA
#> 872             N          <NA>            <NA>            <NA>            NA
#> 873             N          <NA>            <NA>            <NA>            NA
#>     hardship_amount hardship_start_date hardship_end_date
#> 868              NA                <NA>              <NA>
#> 869              NA                <NA>              <NA>
#> 870              NA                <NA>              <NA>
#> 871              NA                <NA>              <NA>
#> 872              NA                <NA>              <NA>
#> 873              NA                <NA>              <NA>
#>     payment_plan_start_date hardship_length hardship_dpd hardship_loan_status
#> 868                    <NA>              NA           NA                 <NA>
#> 869                    <NA>              NA           NA                 <NA>
#> 870                    <NA>              NA           NA                 <NA>
#> 871                    <NA>              NA           NA                 <NA>
#> 872                    <NA>              NA           NA                 <NA>
#> 873                    <NA>              NA           NA                 <NA>
#>     orig_projected_additional_accrued_interest hardship_payoff_balance_amount
#> 868                                         NA                             NA
#> 869                                         NA                             NA
#> 870                                         NA                             NA
#> 871                                         NA                             NA
#> 872                                         NA                             NA
#> 873                                         NA                             NA
#>     hardship_last_payment_amount disbursement_method debt_settlement_flag
#> 868                           NA                Cash                    N
#> 869                           NA                Cash                    Y
#> 870                           NA                Cash                    N
#> 871                           NA                Cash                    N
#> 872                           NA                Cash                    N
#> 873                           NA                Cash                    N
#>     debt_settlement_flag_date settlement_status settlement_date
#> 868                      <NA>              <NA>            <NA>
#> 869                  Dec-2018          COMPLETE        Aug-2018
#> 870                      <NA>              <NA>            <NA>
#> 871                      <NA>              <NA>            <NA>
#> 872                      <NA>              <NA>            <NA>
#> 873                      <NA>              <NA>            <NA>
#>     settlement_amount settlement_percentage settlement_term
#> 868                NA                    NA              NA
#> 869               662                 45.02               4
#> 870                NA                    NA              NA
#> 871                NA                    NA              NA
#> 872                NA                    NA              NA
#> 873                NA                    NA              NA
```



Finally, if we are sure that we want to transform all the data set into numerical, the function "asnum" reviews detect the categorical columns and transform it into numeric, and as a result we would get a data frame. If we apply the function and review now winch are categorical columns, we do not get any.  





```r
# Warning: this code may take a few minutes to finish, depending on the processor.
data2 <- datapro::asnum(data1)
head(charname(data2))
```


If, for some reason, you get an error, you could run the following code:

```r
data2<-read.csv("data/credit_semioriginal_num.csv")
head(charname(data2))
#> [1] "There are no character columns"
```


## Missing values

To treat missing values, I suggest taking one of the following alternatives or a combination of those: i) eliminating columns with a significant amount of missing values; ii) eliminating the row where the missing(s) value(s) is(are) located; iii) replace missing values or Na´s by some statistic. 





For the firs alternative, lets first apply the function "summaryna" to detect columns with more than 50 percent of missing values: 


```r

na_perc <- datapro::summaryna(data2,.5)
head(na_perc)
#>                                Percentage of NAs Column number
#> mths_since_last_record                 0.8064147            28
#> mths_since_last_major_derog            0.7090493            51
#> annual_inc_joint                       0.9919817            54
#> dti_joint                              0.9919817            55
#> mths_since_recent_bc_dlq               0.7502864            87
#> mths_since_recent_revol_delinq         0.6575029            89
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
#> X                1.000000   100.0000000   FALSE FALSE
#> loan_amnt        1.173077    24.7422680   FALSE FALSE
#> funded_amnt      1.173077    24.7422680   FALSE FALSE
#> funded_amnt_inv  1.173077    24.7422680   FALSE FALSE
#> term             3.546875     0.2290951   FALSE FALSE
#> int_rate         1.129032     3.7800687   FALSE FALSE
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
#>  [1]  15  27  33  34  35  40  45  49  50  51  52  53  75  76  94  95  96 101 106
#> [20] 107 108 109 110 111 112 113 114 115 116 117 118
```
The object nzv_2  shows the position of the colums for which the tresholds are higher, then we create other object excluding that columns.


```r
data4<-data3[,-nzv_2]
```


## Collinearity 

Collinearity is when two or more variables are closely related to one another. The presence of collinearity can pose problems in model estimation, such as regression, because it could be difficult to separate the individual effects of collinear variables on the response  [@statistical_lerarning]. 


The function "cor" is to estimate the correlation matrix, and the function "findCorrelation" shows the correlated variables more than "n" (cutoff argument). For this case, we apply a cut-off of 80%. 

```
#>  [1] "open_acc"                   "num_sats"                  
#>  [3] "total_rev_hi_lim"           "total_bc_limit"            
#>  [5] "total_rec_prncp"            "acc_open_past_24mths"      
#>  [7] "total_pymnt_inv"            "total_pymnt"               
#>  [9] "loan_amnt"                  "funded_amnt"               
#> [11] "funded_amnt_inv"            "num_tl_op_past_12m"        
#> [13] "sub_grade"                  "int_rate"                  
#> [15] "num_rev_accts"              "num_bc_sats"               
#> [17] "total_bal_ex_mort"          "num_actv_rev_tl"           
#> [19] "tot_hi_cred_lim"            "fico_range_low"            
#> [21] "last_fico_range_high"       "tot_cur_bal"               
#> [23] "revol_util"                 "total_il_high_credit_limit"
#> [25] "bc_util"                    "collection_recovery_fee"
```

The argument  "names = T" is only to get the column's names of correlated variables. Still, if we wish to cut off those variables from our data frame, we do not add that argument, getting only the column numbers:

```r
hc<-findCorrelation(descrCor, cutoff = .8)
hc
#>  [1] 26 79 55 86 33 59 32 31  2  3  4 80  9  6 77 73 85 72 84 23 40 43 29 87 62
#> [26] 36
```

We could cut those variables from data4 in the following way:


```r
data_corr <- data4[ , -hc]
data_corr
#>       X term installment grade emp_title emp_length home_ownership annual_inc
#> 1     1    1      123.03     3       300          4              1   55000.00
#> 2     2    1      820.28     3       210          4              1   65000.00
#> 3     3    2      432.66     2       624          4              1   63000.00
#> 4     4    2      289.91     6       127          6              1  104433.00
#> 5     5    1      405.18     3       634          7              3   34000.00
#> 6     6    1      637.58     2       637          4              1  180000.00
#> 7     7    1      631.26     2       482          4              1   85000.00
#> 8     8    1      306.45     1       541          9              3   85000.00
#> 9     9    1      263.74     2       632          4              1   42000.00
#> 10   10    1       47.10     3       315          6              1   64000.00
#> 11   11    2      471.70     5       558         10              3  150000.00
#> 12   12    1      858.05     1       527          4              1   92000.00
#> 13   13    1      298.58     1       610         11              1   60000.00
#> 14   14    1      777.55     1       497          4              1  109000.00
#> 15   15    2      400.31     3       257          4              1  112000.00
#> 16   16    1      320.99     5       438         11              3   55000.00
#> 17   17    2      678.49     3       133          4              1   65000.00
#> 18   18    2      379.39     3        64          4              1   48000.00
#> 19   19    1      949.38     3       182          5              2  125000.00
#> 20   20    1      169.54     3       249          5              3   79000.00
#> 21   21    1      186.61     1        34          4              1  100000.00
#> 22   22    1      146.16     3       341          8              3   35000.00
#> 23   23    1      602.30     1       198         12              2   65000.00
#> 24   24    2      406.04     4       434          2              1  109000.00
#> 25   25    1      530.03     1       359          2              1   88000.00
#> 26   26    1      451.73     1       165          4              1   44000.00
#> 27   27    1      538.18     3       388          4              1   65000.00
#> 28   28    2      701.01     6       271          7              1   75000.00
#> 29   29    1      391.62     2       449          3              3   98000.00
#> 30   30    2      581.58     3       220         12              1   79000.00
#> 31   31    1      217.72     1        84          4              1   59000.00
#> 32   32    2      488.53     3       475          2              3   52000.00
#> 33   33    2      652.06     3       328          4              1  195000.00
#> 34   34    1      237.36     2       303          4              1   72500.00
#> 35   35    1      691.84     3       251          9              3  110000.00
#> 36   36    1      700.88     4       111          8              3   70000.00
#> 37   37    2      253.78     4       202          5              1   55000.00
#> 38   38    1      643.47     2       328          8              3   75000.00
#> 39   39    1      260.50     1        44          2              1  100000.00
#> 40   40    2      482.56     3       600          5              3   54649.00
#> 41   41    1      252.32     4        56          2              3   55000.00
#> 42   42    1      283.13     2       501          4              1   92000.00
#> 43   43    1       89.47     4       600          4              1   50000.00
#> 44   44    1      350.67     2       424          4              1   69000.00
#> 45   45    1      336.37     3       127          7              1   91392.00
#> 46   46    2      718.51     4       210          4              1   75000.00
#> 47   47    2      778.38     3       244          4              1  128000.00
#> 48   48    1      229.53     2       550          4              1   50000.00
#> 49   49    1      312.95     1       600          5              3   40100.00
#> 50   50    2      367.67     3       348          3              3   42000.00
#> 51   51    1      726.46     3       649          2              3   47590.00
#> 52   52    2      359.90     4        19          4              1   39000.00
#> 53   53    2      794.21     3       311         10              1  106000.00
#> 54   54    1      672.73     3       419          4              1  145000.00
#> 55   55    2      331.96     3        10          5              1   60000.00
#> 56   56    2      465.27     3       226          4              2   70000.00
#> 57   57    1      142.41     3       328          4              1  102500.00
#> 58   58    2      450.52     3        37         12              2   75000.00
#> 59   59    2      448.01     6       295          4              2   45000.00
#> 60   60    1     1051.31     4       635          5              1  175000.00
#> 61   61    1       45.97     1         1          1              1   45000.00
#> 62   62    1      752.87     1       172          3              1  150000.00
#> 63   63    2      518.35     4       328          7              3   60000.00
#> 64   64    2      633.73     5       422          4              3   70000.00
#> 65   65    2      324.50     2       517          4              1   65000.00
#> 66   66    1      326.35     2       316          6              1   41600.00
#> 67   67    1      473.45     2       513          9              1  110000.00
#> 68   68    2      618.46     5       465          5              2  110000.00
#> 69   69    1      977.00     4       255          4              1   65000.00
#> 70   70    2      549.57     2       445          4              1  135000.00
#> 71   71    1      489.52     2       116          3              3   40000.00
#> 72   72    2      246.33     4       208          2              3   30000.00
#> 73   73    2      419.80     3       588          4              1   70000.00
#> 74   74    1      156.48     1       446          9              3   32000.00
#> 75   75    1      286.92     2       626          3              2   50000.00
#> 76   76    1      337.09     1       339          4              1   85000.00
#> 77   77    1      612.89     1       608          9              1   55000.00
#> 78   78    1      451.73     1         1          1              3   80000.00
#> 79   79    1      361.38     1       358          7              1   53750.00
#> 80   80    1      261.08     2       137          2              1   45000.00
#> 81   81    1      391.62     2       503          6              3   40000.00
#> 82   82    1      559.83     1       597          8              1   75000.00
#> 83   83    1      118.68     2       594          2              1   85000.00
#> 84   84    1      257.67     4       160         12              2   20000.00
#> 85   85    2      344.69     3       137          7              3   77213.00
#> 86   86    1      160.87     2        17          9              2   72000.00
#> 87   87    2      284.51     4       248          8              3   38000.00
#> 88   88    1     1096.53     2        53         11              3  104000.00
#> 89   89    1      597.17     3       560          5              3   39750.00
#> 90   90    1      284.07     2       373         11              1   47000.00
#> 91   91    2      253.79     2       600          4              1   65000.00
#> 92   92    1      127.52     2       582          5              3   44500.00
#> 93   93    1      367.96     4       556          6              3   70100.00
#> 94   94    1      450.43     2       338          7              1  100000.00
#> 95   95    1      187.77     1       483          4              1  105000.00
#> 96   96    1      482.61     2       206          6              1   92000.00
#> 97   97    1      382.55     2       470          7              1   39400.00
#> 98   98    1      329.67     2       392          5              3   33000.00
#> 99   99    1      143.53     3       397          5              3   32000.00
#> 100 100    2      937.06     5       190         11              1  120000.00
#> 101 101    1      200.67     5        81          4              1  108000.00
#> 102 102    1      203.44     3       317          6              2   38700.00
#> 103 103    1      765.10     2        14          4              1   75000.00
#> 104 104    1      336.37     3       626          2              3   60000.00
#> 105 105    1      256.05     5       600          4              3   80000.00
#> 106 106    1      566.30     4       172          7              1   58800.00
#> 107 107    1      732.38     3       319         12              1   54000.00
#> 108 108    2      377.36     5       258          6              3   73320.00
#> 109 109    1      273.39     3       193          2              3   42000.00
#> 110 110    1      239.21     3       383          5              3   83000.00
#> 111 111    1      652.70     2       199          2              2   55000.00
#> 112 112    2      774.14     6       104          9              3   68000.00
#> 113 113    2      685.04     6       540          4              1   92000.00
#> 114 114    1      101.01     2       145          4              3   30407.00
#> 115 115    2      553.49     4        50          4              1  137000.00
#> 116 116    1     1186.72     3         6          2              1  125000.00
#> 117 117    1       85.89     4       361          5              1   52000.00
#> 118 118    1      777.55     1         1          1              1  132000.00
#> 119 119    1      296.00     3       169          5              3   55000.00
#> 120 120    1       82.42     2         1          1              3   16488.00
#> 121 121    1      240.92     1       499          3              3   50000.00
#> 122 122    1      664.20     3       435          4              1   78000.00
#> 123 123    2      794.21     3       136          4              1  110000.00
#> 124 124    1      466.53     1         1          1              1   45000.00
#> 125 125    1      560.70     4       419          3              1   70000.00
#> 126 126    2      251.25     3       346          6              3   40000.00
#> 127 127    1      271.04     4       389          4              2   15000.00
#> 128 128    1      522.16     2        57          8              3   63000.00
#> 129 129    1      271.04     1        15          4              1   49000.00
#> 130 130    1      332.92     4       323          4              3   28000.00
#> 131 131    1      312.95     1       345          5              2   25000.00
#> 132 132    1      187.77     1       600          4              1   49000.00
#> 133 133    2      358.48     3       374          4              1   52000.00
#> 134 134    1     1104.71     2       392          2              2   90000.00
#> 135 135    1      164.84     2       238         12              1   78000.00
#> 136 136    2      523.48     4       405          5              1   75000.00
#> 137 137    1      289.10     4         1          1              1   33130.00
#> 138 138    1      522.16     2       633          6              3   87500.00
#> 139 139    1      563.31     1       342          4              1   80000.00
#> 140 140    1     1115.77     2       172          3              3  105000.00
#> 141 141    1      396.01     1       393          3              3   40000.00
#> 142 142    1      178.53     2        72         11              3   48000.00
#> 143 143    1      739.99     2       230         12              1   90247.00
#> 144 144    2      298.77     3       642          3              1  175000.00
#> 145 145    1      389.94     4       600         11              1   55000.00
#> 146 146    1      132.84     3       507          6              3   41501.00
#> 147 147    1      659.33     2       249          8              3   60000.00
#> 148 148    1      318.79     2        33          4              1   59307.00
#> 149 149    1      714.83     5       553          2              1   55000.00
#> 150 150    2      467.03     3       474          4              1   82000.00
#> 151 151    1      331.42     2       583          7              1   60000.00
#> 152 152    1      163.73     4         2          8              3   56000.00
#> 153 153    1      164.84     2       171          3              3   17000.00
#> 154 154    1      246.99     1         1          1              1   45000.00
#> 155 155    1      166.05     3       158          3              3   30000.00
#> 156 156    1      273.30     1        29          4              1   40000.00
#> 157 157    1      388.78     1       327          4              3   52000.00
#> 158 158    2      474.25     4       500         11              1   80000.00
#> 159 159    1     1196.05     3       625          5              1   90000.00
#> 160 160    2      574.33     2       107          3              1   75000.00
#> 161 161    2      338.75     4        78          3              3   45500.00
#> 162 162    1      514.78     2         1          1              1   99906.00
#> 163 163    1      421.61     1       514          7              3  120000.00
#> 164 164    1      757.51     2       369         11              1   72000.00
#> 165 165    1     1196.05     3       485          9              2   75000.00
#> 166 166    1      263.74     2       630          2              3   21610.00
#> 167 167    1      122.07     3       253          3              3   25000.00
#> 168 168    1      358.82     3       590          4              1  110000.00
#> 169 169    1       98.90     2       615          4              1  105000.00
#> 170 170    1      553.48     3       362          5              3   43000.00
#> 171 171    1      248.82     1       600          4              1   57000.00
#> 172 172    1      606.49     1       428          2              3   85000.00
#> 173 173    1       63.13     2       156         11              1  140000.00
#> 174 174    1      481.84     1       336         11              1   96000.00
#> 175 175    1      114.23     2         9         11              1  122400.00
#> 176 176    1      791.20     2       516          6              2   95000.00
#> 177 177    1      180.69     1       184          5              2  120000.00
#> 178 178    1      751.08     1       469         12              1  100000.00
#> 179 179    2      381.71     4        85          6              3   37231.00
#> 180 180    1      332.10     3       385         10              3   45000.00
#> 181 181    1      463.09     1       453          5              1   93000.00
#> 182 182    1      301.15     1       504          4              3   68000.00
#> 183 183    2      628.93     5       246          4              3   97000.00
#> 184 184    2      597.00     4       220          8              1   70000.00
#> 185 185    1      358.99     2       404          4              3  297000.00
#> 186 186    1      765.10     2       363          5              1  210000.00
#> 187 187    1      505.01     2       210          4              1  140000.00
#> 188 188    2      303.51     1       268          8              3   90000.00
#> 189 189    1      166.05     3       477         11              1   60000.00
#> 190 190    1      489.52     2       497          6              2   85000.00
#> 191 191    1      482.61     2       323          4              3   60000.00
#> 192 192    1      119.61     3       203          4              3  105000.00
#> 193 193    1      637.58     2         1          5              3  130000.00
#> 194 194    2      898.13     4       286         12              1  132500.00
#> 195 195    1       66.42     3       235          4              1  180000.00
#> 196 196    1      386.09     2        92          5              3   39500.00
#> 197 197    1      246.99     1       469          3              1   80000.00
#> 198 198    1      321.74     2       456          4              1   92000.00
#> 199 199    1     1196.05     3       223          4              3  100000.00
#> 200 200    2      517.98     1       293          6              2  150000.00
#> 201 201    1      143.64     3         8          4              2   90000.00
#> 202 202    1      185.24     1       600          4              2   48000.00
#> 203 203    2      440.37     4        16         11              3   65000.00
#> 204 204    1      308.73     1       337          6              3   54000.00
#> 205 205    2      689.37     3       305          8              1   81000.00
#> 206 206    1      397.52     1       189          5              1  150000.00
#> 207 207    1      308.73     1       489          8              3  220000.00
#> 208 208    1       98.90     2       371          8              1   75000.00
#> 209 209    1      643.47     2       172          4              2   96000.00
#> 210 210    2      308.58     3       451          4              1   32500.00
#> 211 211    1      551.61     1       192          4              1   76000.00
#> 212 212    1      480.55     2       600          6              2   40000.00
#> 213 213    2      351.73     2       569          9              3   65000.00
#> 214 214    2      444.13     2       327         10              3  125000.00
#> 215 215    2      320.23     4       573          2              3   40000.00
#> 216 216    1      625.90     1       122          8              2  120000.00
#> 217 217    1      966.49     1       311          4              3   70000.00
#> 218 218    1      555.71     1       386          4              2  120000.00
#> 219 219    1      159.40     2         1          1              1   74967.00
#> 220 220    1      167.31     4       494          2              1   13000.00
#> 221 221    1      193.05     2        52          7              1   85140.00
#> 222 222    2      272.31     3       247          4              1   90000.00
#> 223 223    2      302.42     3       616          4              3  115000.00
#> 224 224    1       59.34     2        58          4              3   50000.00
#> 225 225    1      158.64     4       484          4              3  114000.00
#> 226 226    1      441.29     1       495         11              1   70000.00
#> 227 227    1      183.86     1       452          8              3   70000.00
#> 228 228    1      722.75     4       366         10              1  180000.00
#> 229 229    1      308.73     1       522          8              1  115000.00
#> 230 230    2      773.79     3       477         11              1   92892.00
#> 231 231    1      791.20     2       302          6              3   56457.00
#> 232 232    1      169.71     2       419         10              1  160000.00
#> 233 233    1      557.93     3       443          2              3   50000.00
#> 234 234    1      332.10     3       199         10              3   40000.00
#> 235 235    2      558.30     4       544          4              1   55000.00
#> 236 236    1      843.22     1       639          4              1  144000.00
#> 237 237    1      220.65     1       488          4              1   60000.00
#> 238 238    1      597.78     3       106         12              3   41000.00
#> 239 239    1      261.26     1       278          3              2   30000.00
#> 240 240    1      172.96     3       554          5              3   67000.00
#> 241 241    1      160.87     2       326          4              3   55000.00
#> 242 242    1       79.12     2        97          8              1   60000.00
#> 243 243    1      451.73     1       626         12              2   60000.00
#> 244 244    1      193.67     2       518         12              1   64400.00
#> 245 245    1      652.70     2        35          5              3   55000.00
#> 246 246    1      208.42     3       349          4              3  110000.00
#> 247 247    1      267.79     2        69          5              1   70000.00
#> 248 248    2      340.38     3       596          5              3   90000.00
#> 249 249    1      420.53     4       400          2              1  143000.00
#> 250 250    1      968.73     3       328         11              1   60000.00
#> 251 251    1      150.58     1       380          4              1   60000.00
#> 252 252    1      168.19     3        57         10              1   60000.00
#> 253 253    2      700.55     3       311          4              1  120000.00
#> 254 254    2      615.86     4       291          4              3  165000.00
#> 255 255    2      453.84     3       196          2              1  110000.00
#> 256 256    2      519.19     2       195          5              1  110000.00
#> 257 257    2      546.21     4       117          4              1   65000.00
#> 258 258    1      291.13     1       224          4              1  126500.00
#> 259 259    1      631.26     2       468          4              1   98000.00
#> 260 260    2      492.66     4       282          4              1  135000.00
#> 261 261    2      453.84     3       401          2              3   65000.00
#> 262 262    1      370.48     1       582          4              1   50000.00
#> 263 263    1      336.37     3       476          4              1   65000.00
#> 264 264    2      253.79     2        75          4              1   36500.00
#> 265 265    2      242.81     1       469         11              1   75000.00
#> 266 266    1      225.16     4       292          3              3   41000.00
#> 267 267    1      458.95     4        11          6              3   32760.00
#> 268 268    1     1196.05     3       600          4              2  121000.00
#> 269 269    1      512.60     3       598          5              2   48000.00
#> 270 270    2      506.21     4       241          4              3   60000.00
#> 271 271    1      771.82     1       436          4              1  127000.00
#> 272 272    2      366.88     5       245          4              3  115000.00
#> 273 273    2      462.46     5       220          4              3   47000.00
#> 274 274    1      159.36     3       403          6              3   22000.00
#> 275 275    1      157.82     2         1          1              2   25000.00
#> 276 276    1      415.11     3        77          5              1   92500.00
#> 277 277    1      386.09     2       270          4              3   55000.00
#> 278 278    1      463.09     1       260          3              1  117000.00
#> 279 279    1      166.15     3       445          5              1  123000.00
#> 280 280    1      559.83     1       469          2              1   70000.00
#> 281 281    2      426.81     3        24          8              3   48000.00
#> 282 282    1      894.69     4         8          4              1  120000.00
#> 283 283    1      504.55     3       312          4              1   50000.00
#> 284 284    1      118.68     2       413          3              1   97000.00
#> 285 285    1     1041.58     2       595          8              1   92250.00
#> 286 286    2      387.73     5       204          4              1   68000.00
#> 287 287    1      504.55     3        47          4              1  154000.00
#> 288 288    1      248.15     4       631          5              1   30100.00
#> 289 289    1     1177.27     3       570          7              3   98000.00
#> 290 290    1      252.51     2         1          1              1   58000.00
#> 291 291    1      664.20     3       199         11              1   40000.00
#> 292 292    2      379.98     5       571          2              1   80000.00
#> 293 293    1      669.46     2       391          2              1   85000.00
#> 294 294    1      124.46     3       645          4              2   65000.00
#> 295 295    1      267.79     2       613          4              3   85000.00
#> 296 296    1      237.36     2       617          3              3   22000.00
#> 297 297    1      530.26     2       173         11              1   85400.00
#> 298 298    1      271.04     1       233         10              1  150000.00
#> 299 299    1      205.04     3         3          9              1   43700.00
#> 300 300    1     1109.58     2       110         10              3   76497.00
#> 301 301    1      538.18     3       572          4              1   91000.00
#> 302 302    1      353.51     2       215          4              1   70000.00
#> 303 303    1      187.77     1       354          4              1   50000.00
#> 304 304    1      278.87     1       301          7              3   84000.00
#> 305 305    1      308.73     1       243          4              1   60000.00
#> 306 306    2      924.87     7       543          3              1  115000.00
#> 307 307    1      280.63     3       546          4              3   37380.00
#> 308 308    1      337.17     6       542          4              3   75000.00
#> 309 309    1      180.69     1       212          8              1   72000.00
#> 310 310    1      301.15     1       307          4              1   75000.00
#> 311 311    1      350.44     4       249          7              3  128000.00
#> 312 312    1      257.67     4       394          6              3   22000.00
#> 313 313    2      474.23     3        14          8              2   75000.00
#> 314 314    1      637.58     2       538          5              3   95750.00
#> 315 315    1      150.58     1       517         11              1  130000.00
#> 316 316    2      317.24     2       219          2              1   35179.00
#> 317 317    1      865.49     4       498          6              3  160000.00
#> 318 318    1      489.52     2        96          4              1  160000.00
#> 319 319    1       71.02     2       448          2              1   60000.00
#> 320 320    1      474.69     3       327          3              1   40000.00
#> 321 321    1      470.91     3       492          5              3   80000.00
#> 322 322    1      559.83     1       574          6              1   83000.00
#> 323 323    1      361.38     1       225          6              1   72500.00
#> 324 324    1      182.76     2       159          4              2   46000.00
#> 325 325    1     1240.72     4        94          3              3   83000.00
#> 326 326    1      269.09     3        21          6              3   43500.00
#> 327 327    1       81.38     3       314          6              3   26000.00
#> 328 328    1     1252.56     4       279          2              1   98000.00
#> 329 329    1      235.46     3        61          4              2   55000.00
#> 330 330    1      265.68     3       333          5              2   23200.00
#> 331 331    1      219.20     4       263          7              3   70000.00
#> 332 332    1      352.63     3       521          4              1  150000.00
#> 333 333    1      938.85     1       193          8              1  134000.00
#> 334 334    1      138.37     3       114          8              1   30000.00
#> 335 335    1      326.35     2       402          4              1  125000.00
#> 336 336    1      500.72     1       296          2              1   50000.00
#> 337 337    2      599.96     4       563          4              1   67000.00
#> 338 338    2      586.93     7       381          2              1   56600.00
#> 339 339    2      295.00     3       576          8              1  130000.00
#> 340 340    1       61.29     1       250          6              2   35037.00
#> 341 341    1      578.21     1       625          2              1   50000.00
#> 342 342    2      229.79     3       249          4              3  140000.00
#> 343 343    1     1095.33     1        30          7              1  165000.00
#> 344 344    1      504.55     3        38          4              1   60000.00
#> 345 345    1     1226.53     4       469          2              3   84468.00
#> 346 346    1      568.14     2       249          2              3   55000.00
#> 347 347    1      257.39     2       198          9              3   40000.00
#> 348 348    2      340.38     3       143          4              2   50000.00
#> 349 349    2      356.91     2       386          4              1  175000.00
#> 350 350    1     1162.34     3        43          4              1  156000.00
#> 351 351    1      382.55     2       513          6              3  100000.00
#> 352 352    1      765.10     2       606          4              3   66000.00
#> 353 353    1      250.36     1       168          8              2   66500.00
#> 354 354    2      443.41     5       344          3              1   74000.00
#> 355 355    1      252.51     2       626          6              1   48000.00
#> 356 356    1      685.33     2       425          2              1  100000.00
#> 357 357    1      336.37     3       240          2              1  113000.00
#> 358 358    1      187.77     1        59          2              1   81120.00
#> 359 359    1      218.40     4         1          1              1   54744.00
#> 360 360    1      276.92     2       147          2              3   80000.00
#> 361 361    2      226.92     3       603          4              3   33000.00
#> 362 362    1      191.28     2       130          5              2   72000.00
#> 363 363    1      631.26     2       222          4              3   70000.00
#> 364 364    1      152.77     3         1          1              3   14688.00
#> 365 365    1      715.75     4       205          4              1   50000.00
#> 366 366    1      478.19     2       151          5              3   48112.00
#> 367 367    1      403.64     3       537          7              1   40000.00
#> 368 368    1      163.18     2       472          4              2   75000.00
#> 369 369    1      195.81     2       427          2              1   57000.00
#> 370 370    1      180.69     1       587          4              3   43000.00
#> 371 371    2      749.65     5       211          4              3   84000.00
#> 372 372    1      451.73     1       604          6              2   81000.00
#> 373 373    1      331.42     2       445          4              1  168400.00
#> 374 374    1      339.07     3       350          6              1   34000.00
#> 375 375    1      111.97     1        18          2              1   62000.00
#> 376 376    1      329.67     2        12          4              1   72000.00
#> 377 377    1      946.89     2       408          4              3  115000.00
#> 378 378    1      265.68     3       149          5              3   35000.00
#> 379 379    2      369.49     4       114          3              1   55000.00
#> 380 380    1      681.76     2       239          5              1   60000.00
#> 381 381    1      382.55     2       382          4              1  105000.00
#> 382 382    1      876.26     1       367         11              1  102000.00
#> 383 383    1      210.99     2       493          4              3   33000.00
#> 384 384    1      478.19     2       276          4              3   48000.00
#> 385 385    1      406.84     1       124          5              3   70000.00
#> 386 386    1      339.07     3        42          6              3   80000.00
#> 387 387    1      776.12     1         7          4              1   95000.00
#> 388 388    2      259.60     2       322          4              2   56000.00
#> 389 389    1      498.15     3       299          3              1   65000.00
#> 390 390    1      722.76     1       402          6              1  120000.00
#> 391 391    1       67.82     3       210          4              2   75600.00
#> 392 392    1      791.99     4       600          6              1   53867.00
#> 393 393    1      523.50     6       205          4              3   50000.00
#> 394 394    1      195.81     2       396          4              1   66000.00
#> 395 395    1      153.23     1       421          4              2   55000.00
#> 396 396    1      622.04     1       405          4              1  100000.00
#> 397 397    1      386.09     2       536          4              1  103000.00
#> 398 398    1     1054.93     2       435          4              1  107000.00
#> 399 399    2      353.60     3       210          4              2   69998.00
#> 400 400    2      278.00     3       386          2              1   60000.00
#> 401 401    1      685.33     2       285          7              1   81000.00
#> 402 402    1      505.01     2       148          6              3   35000.00
#> 403 403    1      812.63     2       585          4              1  100000.00
#> 404 404    1      344.25     1       526          6              1   77000.00
#> 405 405    2      531.14     3       308          4              3   70000.00
#> 406 406    2      388.12     4       198          4              1   61000.00
#> 407 407    1      250.36     1       600          4              1   63800.00
#> 408 408    1      255.04     2       648          5              3   52000.00
#> 409 409    1      380.52     3       231          5              3   42240.00
#> 410 410    2      739.15     5       488          2              1   70000.00
#> 411 411    1      766.12     1       328          8              2  120000.00
#> 412 412    1      223.16     2       365          8              3   70000.00
#> 413 413    1      386.09     2       177          2              1  100000.00
#> 414 414    1      205.89     4       334          4              3   56000.00
#> 415 415    2      461.90     4       491          4              3  145000.00
#> 416 416    1      876.26     1       256          3              1  160000.00
#> 417 417    2      740.21     2       405          4              3  205000.00
#> 418 418    2      783.32     4       232          2              1   65000.00
#> 419 419    2      634.43     4       199          4              1  105000.00
#> 420 420    1      572.60     4       370          9              3   54000.00
#> 421 421    2      862.15     4       469          8              1  139000.00
#> 422 422    1      563.31     1       488          3              2   60000.00
#> 423 423    1      311.02     1       589          2              1   55500.00
#> 424 424    1      276.74     3       216          5              3   53040.00
#> 425 425    1      451.73     1         1          1              1   75000.00
#> 426 426    2      444.79     3        91          7              1  155000.00
#> 427 427    2      370.06     5       439          4              1   80000.00
#> 428 428    1      647.04     2         1          1              3   77700.00
#> 429 429    1      160.62     3       488          7              1   88000.00
#> 430 430    2      583.55     2        51          7              1   76008.00
#> 431 431    1      375.54     1       265          4              1   71000.00
#> 432 432    1      225.33     1       494          2              3   42500.00
#> 433 433    1      342.12     1         1          1              2   33000.00
#> 434 434    2      255.90     3       502          8              1   52000.00
#> 435 435    1      112.61     2       376          3              1   45000.00
#> 436 436    1      808.67     3       591          6              2  111000.00
#> 437 437    1      126.16     4       432          6              3   70000.00
#> 438 438    1      547.96     3       272          4              1   76000.00
#> 439 439    1      270.06     2         1          1              1   55000.00
#> 440 440    1     1252.56     4       176          2              1   80000.00
#> 441 441    1      602.30     1       460          2              1  106000.00
#> 442 442    1      686.62     1       386          9              1   76000.00
#> 443 443    1      315.63     2        13          7              2   35000.00
#> 444 444    1      620.06     2       186          4              1  155000.00
#> 445 445    1      212.70     4       294          5              3   40000.00
#> 446 446    1     1240.72     4       379          4              1  105000.00
#> 447 447    1      205.04     3       567          4              1  210000.00
#> 448 448    1      546.77     3       399          2              3   52000.00
#> 449 449    1      177.25     4         1          1              3   19000.00
#> 450 450    1      109.20     4        87          3              3   54000.00
#> 451 451    1      691.84     3       551          4              2  120000.00
#> 452 452    2      678.37     6       132          9              1   43435.00
#> 453 453    1      622.04     1       283          4              1  106000.00
#> 454 454    1      255.04     2       454          2              3   65000.00
#> 455 455    1      168.19     3       249          2              3   47000.00
#> 456 456    1      602.30     1       620          4              1  135000.00
#> 457 457    1      169.54     3       459          9              3  110000.00
#> 458 458    1      391.62     2       487          8              3   29500.00
#> 459 459    2      537.61     5       556          2              1   86000.00
#> 460 460    1      772.17     2        98          6              2  125000.00
#> 461 461    1      391.62     2       228          5              1   85000.00
#> 462 462    1      361.38     1       509          7              2   45000.00
#> 463 463    1      876.26     1       619          5              1  112000.00
#> 464 464    1      456.89     2       527         10              3  105000.00
#> 465 465    1      386.81     6       580         12              2   31200.00
#> 466 466    1      301.15     1       429         11              3   72000.00
#> 467 467    1      123.03     3        82          8              1   34000.00
#> 468 468    1     1096.15     6       519          4              1   55702.00
#> 469 469    1      450.43     2       364          7              1  175000.00
#> 470 470    1      375.69     1        93          5              2   26000.00
#> 471 471    2      551.50     3       622          3              1   96000.00
#> 472 472    2      531.14     3       533          6              3   96000.00
#> 473 473    1      796.76     1       618          9              1  124400.00
#> 474 474    2      574.48     3         1          1              2   50000.00
#> 475 475    1      450.43     2       524          4              1  130000.00
#> 476 476    1      306.45     1       584         11              2   52600.00
#> 477 477    1      423.32     5         1          1              1   28948.92
#> 478 478    1      432.22     1       163          9              1  100200.00
#> 479 479    1      281.66     1       192          4              1   90000.00
#> 480 480    1      261.08     2        25          6              3   38000.00
#> 481 481    1      344.25     1       600          5              1   60000.00
#> 482 482    1      337.29     1       458          5              1   82500.00
#> 483 483    2      482.56     3       478          5              1   75000.00
#> 484 484    1     1162.34     3       327          4              3  130000.00
#> 485 485    1      678.13     3       647          6              1   64000.00
#> 486 486    1      735.47     1       410          4              1  105000.00
#> 487 487    1      657.55     3       328          6              3   45000.00
#> 488 488    1     1009.09     3       318         12              3   84000.00
#> 489 489    1      843.22     1       523          6              1  205200.00
#> 490 490    2      692.80     4       172          7              1   65000.00
#> 491 491    2      544.61     3       611          4              3   73000.00
#> 492 492    1       46.23     2       607          5              1   35000.00
#> 493 493    1      128.70     2       213          5              1   47500.00
#> 494 494    1       32.97     2       431          4              3   86000.00
#> 495 495    1      814.24     2       328          8              1   65000.00
#> 496 496    1      752.87     1       463          4              1  300000.00
#> 497 497    1      187.77     1       565          4              1  250000.00
#> 498 498    1      669.46     2       284          2              1  110000.00
#> 499 499    1       97.97     3         1          1              1   22000.00
#> 500 500    2      956.84     6       469          4              1   72000.00
#> 501 501    1      214.73     4       593          4              1  120000.00
#> 502 502    2      266.46     4       170          3              3   28832.00
#> 503 503    1      398.52     3       486          8              2   35000.00
#> 504 504    1      617.46     1       332          4              1  120000.00
#> 505 505    1      369.00     5       198          3              1  150000.00
#> 506 506    1      325.50     3        11          9              1   55000.00
#> 507 507    1      602.30     1       643          5              1  187000.00
#> 508 508    1      476.37     2        66         10              1  116300.00
#> 509 509    1      666.85     1       172          6              1  120000.00
#> 510 510    1      389.00     1       220          3              3   73000.00
#> 511 511    1      518.65     4       437         10              3   48000.00
#> 512 512    1      678.13     3       100          3              3   53000.00
#> 513 513    1      316.48     2        41          4              3  110000.00
#> 514 514    1      430.04     4       490          6              1   34000.00
#> 515 515    2      539.45     5        70          2              3   60000.00
#> 516 516    1      979.04     2        71          4              1   70000.00
#> 517 517    1      386.09     2       508          6              2   31200.00
#> 518 518    1      164.84     2       447          5              1   85000.00
#> 519 519    2      380.66     4        63          5              3   45500.00
#> 520 520    1      635.73     3       600          8              3   45000.00
#> 521 521    1      156.48     1       309          4              1   67000.00
#> 522 522    1      647.80     2        76         10              3   85005.00
#> 523 523    1      273.39     3       213          5              1   80000.00
#> 524 524    2      492.66     4       298          7              1  100000.00
#> 525 525    2      465.27     3        40          4              2   60000.00
#> 526 526    2      780.52     6       164         11              3   93000.00
#> 527 527    1      410.08     3       142          6              3   50000.00
#> 528 528    1      482.61     2       306          7              3   52500.00
#> 529 529    1      933.05     1       331          6              1  130000.00
#> 530 530    1      511.96     1        88          8              1   98000.00
#> 531 531    1      558.33     4       600         11              3   50000.00
#> 532 532    1      163.18     2       600          4              2   55000.00
#> 533 533    2      558.32     3       221          4              1  115000.00
#> 534 534    1      478.19     2       450          4              3   45000.00
#> 535 535    2      689.37     3       599          4              1   70000.00
#> 536 536    1      130.63     1       534          4              1   90000.00
#> 537 537    1      200.09     3        54         10              3   25000.00
#> 538 538    2      381.07     4        62          4              1   45000.00
#> 539 539    1      306.45     1       414          4              1  106000.00
#> 540 540    1      329.67     2       413          5              1  105000.00
#> 541 541    1     1177.27     3       636          3              1  240000.00
#> 542 542    1      504.55     3       335          2              3   89000.00
#> 543 543    1      166.05     3       488         11              3   60000.00
#> 544 544    1      375.91     3        37          6              3   65000.00
#> 545 545    1      192.74     4       109          5              1  101000.00
#> 546 546    1       83.03     3        25          2              1   50400.00
#> 547 547    1      741.18     4       386          8              1   52000.00
#> 548 548    1      659.33     2        74          3              3  185000.00
#> 549 549    1      193.68     2       352          6              3   25200.00
#> 550 550    1      746.44     1        46          4              3  107000.00
#> 551 551    1      386.09     2       621          3              3   46000.00
#> 552 552    2      497.96     5       530          4              2  113000.00
#> 553 553    1      273.39     3         1          1              1   50000.00
#> 554 554    1      981.23     4       357         11              1  105000.00
#> 555 555    1      440.57     2       249          4              3   77000.00
#> 556 556    1      347.20     2        65          4              1  125000.00
#> 557 557    1      489.52     2       600          6              1   46000.00
#> 558 558    1      643.47     2       283          6              3   95000.00
#> 559 559    1     1126.07     2       181          4              2  173000.00
#> 560 560    2      219.83     2       328          4              1   55000.00
#> 561 561    1      286.92     2       422          5              1  105000.00
#> 562 562    1      297.78     4       390          6              3   51000.00
#> 563 563    1      672.73     3       140          4              3   65000.00
#> 564 564    2      455.26     1        10          4              1   93000.00
#> 565 565    1      727.98     4       188         11              3  137000.00
#> 566 566    1      697.41     3       385          8              1   63000.00
#> 567 567    2      435.02     3        73          2              1   55000.00
#> 568 568    2      778.38     3        32          5              1  175000.00
#> 569 569    1      648.33     1       497          4              1   85000.00
#> 570 570    1     1017.19     3       442          8              1  106000.00
#> 571 571    1      496.29     4         1          1              1  124596.00
#> 572 572    1      722.76     1       640          5              3  450000.00
#> 573 573    2      757.16     2       112          4              3   89000.00
#> 574 574    1      120.68     5        13          8              1   35000.00
#> 575 575    1      858.05     1       564          6              1  108000.00
#> 576 576    1      229.53     2       485          4              1   60000.00
#> 577 577    1      386.09     2       217          8              3   43000.00
#> 578 578    2      917.19     5        99          4              1  123000.00
#> 579 579    1      361.38     1       274          2              1  118000.00
#> 580 580    1      156.48     1       395         10              2   38000.00
#> 581 581    1      854.32     3       178          9              3   85000.00
#> 582 582    1      320.70     5       373          3              3   29524.32
#> 583 583    1      243.54     5       131          7              1  240000.00
#> 584 584    1      318.79     2       575          5              2  110000.00
#> 585 585    1      407.94     2       488          4              1   34000.00
#> 586 586    2      610.66     3       135          4              1   65000.00
#> 587 587    2      544.61     3       413          4              1  186000.00
#> 588 588    1      494.50     2       480         12              3   90000.00
#> 589 589    1      487.43     4       627          4              1   27500.00
#> 590 590    2     1011.77     6       157          2              1  117000.00
#> 591 591    1      235.46     3       304          4              3   65000.00
#> 592 592    1      180.18     2       360          4              1   91500.00
#> 593 593    1      669.46     2       510         11              1  150000.00
#> 594 594    1      451.73     1       435          7              1   85000.00
#> 595 595    1      204.03     2       512          9              3   53000.00
#> 596 596    1      270.26     2       600          4              3   72701.00
#> 597 597    2      483.93     4       487          9              1  150000.00
#> 598 598    1      518.88     3         1          1              1   37534.44
#> 599 599    1      448.35     2       134          4              1   61983.00
#> 600 600    1      615.11     3       126          6              2   63000.00
#> 601 601    2      898.13     4       162          4              2  160000.00
#> 602 602    1      631.26     2       412          4              2   58000.00
#> 603 603    1      602.30     1        89          4              3  139000.00
#> 604 604    1      357.05     2        55          4              1   53000.00
#> 605 605    1      766.12     1       167          5              1   62000.00
#> 606 606    1       78.24     1       150          9              3   29000.00
#> 607 607    1      505.01     2       108          3              3   75000.00
#> 608 608    1      189.88     3       361          4              3   50000.00
#> 609 609    1      311.02     1       145          4              3   70000.00
#> 610 610    1      102.96     2         1          1              1   24000.00
#> 611 611    1      843.22     1        60          7              1  135000.00
#> 612 612    1      489.52     2       467          4              1   75000.00
#> 613 613    1      666.85     1       384          3              1  120000.00
#> 614 614    1      301.15     1       174         12              1   75000.00
#> 615 615    1      160.87     2       194          4              2   92000.00
#> 616 616    1      367.74     1        90          6              1   73000.00
#> 617 617    1      543.08     4       321          5              1   58000.00
#> 618 618    1      669.46     2        36          2              3   80000.00
#> 619 619    1      210.81     1         1          1              3   16968.00
#> 620 620    1      373.22     1       578          6              1  108000.00
#> 621 621    1      104.44     2       600          4              1   49000.00
#> 622 622    1      249.07     3       166          4              3   52000.00
#> 623 623    1      444.00     3       587          4              1   45000.00
#> 624 624    1      672.73     3         1          1              2   65000.00
#> 625 625    2      369.22     2       146          5              1  120000.00
#> 626 626    1      638.09     4       310          9              1   50000.00
#> 627 627    1      262.83     4       561          6              3   75000.00
#> 628 628    1      504.55     3        49          4              1  118000.00
#> 629 629    1       74.65     1        83          4              3   50000.00
#> 630 630    1       67.82     3        34          4              1  119945.00
#> 631 631    1      336.37     3       638          4              3  100000.00
#> 632 632    2      769.83     4       407          2              1  103000.00
#> 633 633    1      782.38     1       183          4              1  105000.00
#> 634 634    1      376.39     2       351          4              1   50000.00
#> 635 635    1       91.94     1       289          4              1   75000.00
#> 636 636    1      420.81     2        67          3              1   98000.00
#> 637 637    1      168.19     3       202          4              3   48000.00
#> 638 638    1      289.57     2       600          4              1   81000.00
#> 639 639    1      318.79     2       433          6              1   43000.00
#> 640 640    1       33.21     3       153          4              1   36522.00
#> 641 641    2      279.16     3        68          4              2   35000.00
#> 642 642    1      722.76     1        80          4              1  200000.00
#> 643 643    1      270.98     2       120          3              2   35500.00
#> 644 644    1      107.26     1       568         11              1   56000.00
#> 645 645    1      576.18     4       141          4              1   50000.00
#> 646 646    2      262.06     5       489          2              1  117000.00
#> 647 647    1      189.38     2       154          7              2   42000.00
#> 648 648    1      154.37     1        22          4              1   61000.00
#> 649 649    1      551.61     1        31          4              1  100000.00
#> 650 650    1       34.18     3       602          4              3   30000.00
#> 651 651    1      130.54     3       277          2              1   45000.00
#> 652 652    1       66.42     3       105          7              3   35000.00
#> 653 653    2      268.96     5       121          6              1   63900.00
#> 654 654    1      308.73     1       614          2              1   70000.00
#> 655 655    2      422.98     2       406          8              1   52000.00
#> 656 656    1     1142.22     2       495          4              1  110000.00
#> 657 657    1      672.73     3        28          4              3   93534.00
#> 658 658    1      612.89     1       328          5              1   95000.00
#> 659 659    1      423.76     3       461          7              3   26679.38
#> 660 660    1      451.73     1       488         11              1   90000.00
#> 661 661    1      301.15     1       227          4              3   80000.00
#> 662 662    1      117.49     2       233         10              1   39000.00
#> 663 663    1      166.05     3       139          4              1   68500.00
#> 664 664    2      778.38     3       594          4              1   80000.00
#> 665 665    1      437.28     3        26          4              1   38000.00
#> 666 666    1      141.80     4       273          8              2   35000.00
#> 667 667    2      582.25     5       535          4              1   74000.00
#> 668 668    1      159.41     3       254          3              3   45000.00
#> 669 669    1      735.95     3       200          7              1   56035.00
#> 670 670    1      451.73     1       481          4              3  100000.00
#> 671 671    1      403.64     3       283          7              1  110000.00
#> 672 672    1      324.59     3       267          2              3   55500.00
#> 673 673    1      772.17     2       328          4              1  100000.00
#> 674 674    1     1142.22     2       586          4              1  134000.00
#> 675 675    1      752.87     1       218          4              1  125000.00
#> 676 676    1      284.79     1       324          4              1   40000.00
#> 677 677    2      592.79     3       629          5              1  118450.00
#> 678 678    1      559.83     1       462          9              2   60000.00
#> 679 679    1      356.04     2       297          4              2  123000.00
#> 680 680    1      306.04     2       172          5              1  150000.00
#> 681 681    1      118.68     2       214          6              1   55000.00
#> 682 682    2      272.31     3       413          4              2  100000.00
#> 683 683    1      367.74     1       398          4              1   65000.00
#> 684 684    1      555.71     1       329         11              1   89000.00
#> 685 685    2      564.71     4       129          2              3   51000.00
#> 686 686    1      245.16     1       261          6              3  120000.00
#> 687 687    1      220.95     2       197          6              1  100000.00
#> 688 688    1      144.79     2       623          8              1   39600.00
#> 689 689    1      424.35     5         1          1              3   33888.00
#> 690 690    1      159.40     2       600          7              1   52000.00
#> 691 691    1      157.82     2       330          5              1  141934.00
#> 692 692    1      100.21     4       280         10              3  114400.00
#> 693 693    1      386.09     2       124         11              3   75000.00
#> 694 694    1      540.71     3       624          6              2   66666.00
#> 695 695    1      669.46     2       242          7              1   85000.00
#> 696 696    1      602.30     1       479          4              1   71665.00
#> 697 697    2      574.33     2        48          8              1   95000.00
#> 698 698    1      482.61     2       531          5              3   89000.00
#> 699 699    1      536.81     4        86         10              3  101000.00
#> 700 700    1      820.15     3       566          4              1  120000.00
#> 701 701    1     1226.53     4       355          7              1  120000.00
#> 702 702    2      533.10     7       392          7              3   45000.00
#> 703 703    2      697.90     3       591         12              1   70000.00
#> 704 704    1      261.08     2       600          5              1   43500.00
#> 705 705    1      159.40     2       103          6              1  113000.00
#> 706 706    1      290.10     2       549          8              1   70000.00
#> 707 707    1       55.99     1       471          4              1   50000.00
#> 708 708    1      150.85     1       125          2              1   56000.00
#> 709 709    1      740.95     1       328          4              1  127000.00
#> 710 710    1      240.92     1       191          4              2   31500.00
#> 711 711    1      318.79     2       488          6              1   35000.00
#> 712 712    1      294.19     1       466          7              3   64567.05
#> 713 713    2      518.71     5       373          3              2   52000.00
#> 714 714    1      301.15     1       327          4              2   32500.00
#> 715 715    1      180.69     1         4          4              1   80000.00
#> 716 716    1      662.53     1         1          1              1  110000.00
#> 717 717    2      622.71     3       416          3              1   82000.00
#> 718 718    1      156.48     1       119          7              3   93000.00
#> 719 719    1      542.07     1       644          4              2   76000.00
#> 720 720    1      301.15     1       555          4              1   80000.00
#> 721 721    1      371.95     3        39          2              3   37000.00
#> 722 722    1       76.95     4       368          6              2   18300.00
#> 723 723    1      302.73     3       592          4              3  110000.00
#> 724 724    1      388.78     1       209          4              1   99500.00
#> 725 725    1      180.69     1       268          2              1   52000.00
#> 726 726    1      691.84     3       328          9              3   60000.00
#> 727 727    2      517.29     4       124          4              3   90000.00
#> 728 728    2      778.38     3       527          2              1  148000.00
#> 729 729    2      738.98     4       199          4              1   94000.00
#> 730 730    1      189.38     2       403          6              3   36000.00
#> 731 731    1      876.26     1       356          4              1  111000.00
#> 732 732    1      199.26     3       262          4              3   51000.00
#> 733 733    2      333.60     3       528          4              1  103000.00
#> 734 734    2      781.62     4       175          4              1  111000.00
#> 735 735    1      391.62     2       375          4              3   86308.00
#> 736 736    1       67.28     3       201          9              3   86564.00
#> 737 737    1      153.00     4       115          3              1   38500.00
#> 738 738    1      180.69     1       457          5              1   42000.00
#> 739 739    1      563.31     1       113          3              3   45000.00
#> 740 740    1      830.24     3       155          4              1   70000.00
#> 741 741    1      193.05     2       290          6              3   41500.00
#> 742 742    1      315.61     2       229          4              3   57000.00
#> 743 743    1     1088.56     1       325          8              1  125000.00
#> 744 744    2      482.40     5        51          3              2   36500.00
#> 745 745    1      361.38     1         1          1              1  110000.00
#> 746 746    1     1009.09     3       577          4              3  139000.00
#> 747 747    1      669.46     2       515          7              3   80000.00
#> 748 748    1       33.64     3       600          4              3   45000.00
#> 749 749    1      119.56     3       409          4              3   55000.00
#> 750 750    1      395.60     2       508          6              1   47140.00
#> 751 751    1      617.46     1       264          3              1  100000.00
#> 752 752    1      451.73     1        23          2              3   55000.00
#> 753 753    1      230.77     2       123          7              2   41000.00
#> 754 754    1     1037.76     3       287          4              1  112000.00
#> 755 755    1       67.82     3        10          4              1   43500.00
#> 756 756    1      424.26     2         5          4              1   45000.00
#> 757 757    1      469.43     1       180          4              2  170000.00
#> 758 758    1      571.82     3       628         12              3   89000.00
#> 759 759    1      432.22     1       494          8              2   65000.00
#> 760 760    2      269.44     4       192         11              1   47000.00
#> 761 761    2      344.69     3       441         10              2   43500.00
#> 762 762    1      394.32     1       473         10              3   73000.00
#> 763 763    1      361.38     1       207         12              1   98000.00
#> 764 764    2      677.49     4       415          3              3  118000.00
#> 765 765    2      369.33     7       505          3              2   28000.00
#> 766 766    2      507.58     2       419          5              1  105000.00
#> 767 767    1      259.75     1         1          1              3   52000.00
#> 768 768    1      466.53     1       600          4              1   67879.00
#> 769 769    1      261.08     2       525          8              1   53000.00
#> 770 770    2      622.45     5       496          7              3  230000.00
#> 771 771    2      829.90     3       320          4              2   90000.00
#> 772 772    1      395.60     2       236         10              1   85000.00
#> 773 773    2      391.19     6       600          4              1   43500.00
#> 774 774    1      252.97     1       552          4              1   95000.00
#> 775 775    1      306.04     2       378         11              1   90000.00
#> 776 776    1      318.79     2       420          7              3   35000.00
#> 777 777    1       57.39     2       237          3              1   72000.00
#> 778 778    1      504.55     3       353          8              1   94000.00
#> 779 779    1      191.28     2       418          3              3   20000.00
#> 780 780    1      306.45     1       102          2              1   35000.00
#> 781 781    1      752.87     1       152          4              3   99000.00
#> 782 782    1      321.91     5         1          1              1   36000.00
#> 783 783    1      484.29     3       548          5              3   44500.00
#> 784 784    2      363.07     3       646          4              1   64000.00
#> 785 785    2      604.90     6       220          4              2   60300.00
#> 786 786    1      101.19     4       234          5              3   58000.00
#> 787 787    1      111.15     1       185          4              1  150000.00
#> 788 788    1      617.46     1       249          4              3  105000.00
#> 789 789    2      786.17     5       422          4              1   88000.00
#> 790 790    1      622.04     1       641          7              3  105000.00
#> 791 791    2      803.19     5       417          2              1   98000.00
#> 792 792    1      321.74     2        51          3              3   75000.00
#> 793 793    2      697.90     3       532          2              2   70000.00
#> 794 794    1      104.84     1         1          1              1   42251.00
#> 795 795    2      311.23     5       626          2              3   50000.00
#> 796 796    1      186.61     1       582          4              1   67000.00
#> 797 797    2      415.36     2        71          2              3   65000.00
#> 798 798    2      263.80     2       422          8              1  110000.00
#> 799 799    1      612.89     1       288          4              2   78000.00
#> 800 800    1      456.62     3       444          3              2   61000.00
#> 801 801    1     1095.33     1       423          7              2  145000.00
#> 802 802    1      454.51     2       438          2              3   47000.00
#> 803 803    1       98.90     2       144          8              1   45000.00
#> 804 804    1      739.60     2       529          2              3  110000.00
#> 805 805    1      601.64     2         1          1              1   39696.00
#> 806 806    2      370.06     5       430          6              1   75000.00
#> 807 807    1      410.08     3       605          4              1   90000.00
#> 808 808    1      101.72     3       626          5              3   50740.00
#> 809 809    1      124.41     1       411          8              3   83500.00
#> 810 810    1      166.05     3       386          5              3   65000.00
#> 811 811    2      322.25     5         1          1              3   24897.72
#> 812 812    1     1097.75     4        45          3              3  157000.00
#> 813 813    1      858.05     1       372          4              1   82000.00
#> 814 814    1       86.48     3       612          4              3   32000.00
#> 815 815    1      157.82     2       436          2              1   29000.00
#> 816 816    1       49.63     4       266          4              1   54000.00
#> 817 817    1      290.69     4       387          6              1   79000.00
#> 818 818    1      162.62     1       347          5              3   41000.00
#> 819 819    1      824.17     2       464          4              3  185000.00
#> 820 820    1      672.73     3       259          4              2   50000.00
#> 821 821    1      846.85     3       559         10              3   72000.00
#> 822 822    1      421.70     1       343          4              2  120000.00
#> 823 823    1      227.26     2       579          7              3   42000.00
#> 824 824    1     1037.76     3       179          9              1  124000.00
#> 825 825    1      473.45     2       440          2              3   50500.00
#> 826 826    2      566.04     5       210          8              1  121000.00
#> 827 827    2      333.60     3       313          6              1  100000.00
#> 828 828    2      285.29     5       547          4              3   55000.00
#> 829 829    1      217.01     3        27          4              3   54000.00
#> 830 830    1      315.63     2       600          7              3   63000.00
#> 831 831    1      176.75     1       601          6              3   25000.00
#> 832 832    1      492.39     2        95          8              2   38000.00
#> 833 833    2      499.96     4       269         12              3  124000.00
#> 834 834    1      148.49     2       445          8              1   78000.00
#> 835 835    1      362.64     2       520          7              1   75000.00
#> 836 836    2      695.71     5        70          4              1   66548.00
#> 837 837    1      286.20     3       393          4              2   32000.00
#> 838 838    1      789.08     2       129          6              1  135000.00
#> 839 839    2      396.50     5       581         12              3   50000.00
#> 840 840    1      578.21     1       275         11              3   60000.00
#> 841 841    1      553.48     3       385          8              1   70000.00
#> 842 842    1      746.44     1         1          1              1   48000.00
#> 843 843    1      361.38     1       506          5              2   50000.00
#> 844 844    1      459.06     2       209          4              3   82000.00
#> 845 845    1      139.15     2       161          5              1   40000.00
#> 846 846    1      265.87     4       340          6              2   50000.00
#> 847 847    2      540.83     2       562          9              3   50000.00
#> 848 848    1      456.89     2         1          1              1  120000.00
#> 849 849    1      116.21     2       545          3              3   20000.00
#> 850 850    1      173.74     2       557          5              3   80000.00
#> 851 851    2      678.37     6       436          9              1  104000.00
#> 852 852    1      520.96     1       118          5              1   52000.00
#> 853 853    1      339.07     3       252          4              1   90000.00
#> 854 854    2      888.20     4       281          4              1  150000.00
#> 855 855    2      246.15     2       426          2              1   68000.00
#> 856 856    2      512.17     3       455          7              1   86000.00
#> 857 857    1      140.18     4       137          3              3   60000.00
#> 858 858    1      542.07     1        79         10              2  200000.00
#> 859 859    1      760.74     4       511          5              2   55000.00
#> 860 860    2      512.98     4       357          4              1   81500.00
#> 861 861    1      327.37     2       477          6              1   65000.00
#> 862 862    2      295.60     4       581          4              1  101000.00
#> 863 863    2      483.93     4       609          9              3   57000.00
#> 864 864    1      128.70     2        20          4              1   47000.00
#> 865 865    1      489.52     2       283          4              1  200000.00
#> 866 866    2      410.58     4       487          4              3   65000.00
#> 867 867    2      338.75     4       128          4              3  132700.00
#> 868 868    1      140.44     3       101          7              1   50720.00
#> 869 869    1      164.84     2       138         11              2   43498.00
#> 870 870    1      637.58     2         1          1              1   60000.00
#> 871 871    1      926.18     1       187          4              1  202000.00
#> 872 872    1      182.66     3       539          6              3   37000.00
#> 873 873    1     1009.09     3       377          6              3  100000.00
#>     verification_status loan_status purpose title zip_code addr_state   dti
#> 1                     1           2       3     5       80         34  5.91
#> 2                     1           2      10     2      248         37 16.06
#> 3                     1           2       4     1      253         12 10.78
#> 4                     2           2       6     8       76         34 25.37
#> 5                     2           2       3     5      137         10 10.20
#> 6                     1           2       3     5      241         20 14.67
#> 7                     1           2       6     8      131         36 17.61
#> 8                     1           2       2     4       71         34 13.07
#> 9                     1           2       2     4       16         35 34.80
#> 10                    1           2       9     1      121         24 34.95
#> 11                    1           1       3     5      342          4  9.39
#> 12                    1           2       3     5      121         24 21.60
#> 13                    1           2       2     4      136         36 22.44
#> 14                    1           2       3     5      106         41 26.02
#> 15                    1           2       3     5      325          3  8.68
#> 16                    3           2       3     5      217         13 25.49
#> 17                    3           2       3     5      100         18 21.77
#> 18                    1           2       2     4      135         36 33.18
#> 19                    2           2       3     5       41         30  7.49
#> 20                    3           2       2     4       95         18 12.70
#> 21                    1           2       3     5      290         39 13.28
#> 22                    2           1       3     5      275         14 15.22
#> 23                    1           2       3     5       53         30 18.83
#> 24                    2           2       3     5      328         28 23.35
#> 25                    2           2       2     4      275         14 26.59
#> 26                    2           2       4     7      111         41 15.34
#> 27                    1           1      10     2      170          1 18.96
#> 28                    1           1       3     1       47         30 20.84
#> 29                    1           2       3     5      386         43 24.04
#> 30                    1           1       3     5       94         18 34.53
#> 31                    1           2       2     4       96         18 13.06
#> 32                    2           2       2     4      208         31 14.47
#> 33                    1           2       9    11      282         16  6.79
#> 34                    3           2       4     7      137         10  8.47
#> 35                    1           2       2     4      254         12 12.45
#> 36                    1           2       5     6      157          9 22.21
#> 37                    1           1       2     4      314          5 35.70
#> 38                    1           2       3     5      381         43 23.45
#> 39                    1           2       2     4      217         13  7.28
#> 40                    2           2       3     5      296         39 12.14
#> 41                    1           2       3     1       62         30 17.35
#> 42                    2           2       2     4      232         19  5.18
#> 43                    3           2       9    11      156          9 17.60
#> 44                    1           2       2     4      208         31 21.34
#> 45                    1           2       5     6      268         21  9.44
#> 46                    3           2       3     5      220         13 31.88
#> 47                    2           2       4     7       86          7  6.46
#> 48                    1           2       3     5        5         17 19.25
#> 49                    1           2       2     4      213         31 10.81
#> 50                    1           1       2     4        9         17  9.60
#> 51                    2           2       2     4      166          9 17.76
#> 52                    3           2       3     5      240         44 18.00
#> 53                    2           1       3     5      169          1 17.36
#> 54                    1           2       3     5       16         35 12.28
#> 55                    1           1       3     1      323          3 24.29
#> 56                    2           1       3     5       92         18 16.90
#> 57                    1           2       3     5       56         30 26.21
#> 58                    1           2       2     4       45         30 13.09
#> 59                    1           2       3     5      207         31 34.85
#> 60                    1           2       4     7      201         31 18.50
#> 61                    1           2      11    12      371         11 16.11
#> 62                    1           2       2     4       22         42  9.54
#> 63                    2           1       3     5      368          4 34.84
#> 64                    1           1       2     4      101         41 33.20
#> 65                    1           2       3     5      168          1 29.28
#> 66                    1           2       3     5      221         13 15.78
#> 67                    1           2       2     4       38         27 13.24
#> 68                    1           1       1     3       84          8 20.43
#> 69                    1           1       4     7       49         30 25.63
#> 70                    1           2       2     4      303         39  4.65
#> 71                    3           2       2     4      310         39 10.17
#> 72                    2           2       3     5      209         31 30.84
#> 73                    2           2       3     5      292         39 13.20
#> 74                    2           2       2     4      204         31 13.73
#> 75                    1           2       3     5      274         14 11.98
#> 76                    1           2       2     4        6         17 15.00
#> 77                    1           2       3     5      186         38 29.15
#> 78                    1           2       3     5      110         41 20.18
#> 79                    2           2       3     5      322          3 15.30
#> 80                    1           2       3     5      226         19 21.23
#> 81                    1           2       2     4       28         27 31.95
#> 82                    1           2       3     5       17         26 25.65
#> 83                    2           2       3     5      101         41 17.44
#> 84                    3           1       3     5       12         17 13.21
#> 85                    2           2       2     4      166          9 18.17
#> 86                    1           2       9    11      125         24  6.52
#> 87                    1           1       2     4      205         31 21.07
#> 88                    2           2       3     5      208         31 14.01
#> 89                    1           1       3     5      252         12 23.58
#> 90                    1           2       3     5      165          9  8.43
#> 91                    1           2       3     5      271         14 23.84
#> 92                    3           2       3     5      276         25 15.83
#> 93                    2           2       2     4      272         14  8.51
#> 94                    1           2       7     9      206         31 15.97
#> 95                    1           2       1     3      202         31 14.45
#> 96                    1           2       3     5      375         33 13.89
#> 97                    1           2       3     5      368          4 26.32
#> 98                    1           2       3     5      223         13 21.75
#> 99                    2           2       9    11      183         38 31.02
#> 100                   2           2       3     5      138         10 35.69
#> 101                   3           1       3     5      167          9 33.47
#> 102                   2           2       3     5      127         24 23.26
#> 103                   2           2       3     5       16         35 10.11
#> 104                   3           2       3     5      383         43  3.30
#> 105                   2           2       3     5       45         30  9.57
#> 106                   1           2       3     5      338          4 20.45
#> 107                   3           2       3     5      239         44 17.64
#> 108                   2           1       3     5       57         30 17.02
#> 109                   1           2       9     1      375         33 23.26
#> 110                   2           2       3     5       84          8 21.96
#> 111                   1           2       3     5      171          1 36.94
#> 112                   3           1       3     5       89         18 31.68
#> 113                   3           2       3     5      210         31 24.16
#> 114                   1           2       7     9      241         20 20.61
#> 115                   1           2       3     5      340          4 32.30
#> 116                   2           2       3     5      242         20 24.19
#> 117                   3           2       4     7      217         13 28.38
#> 118                   1           2       3     5      235         19 13.83
#> 119                   1           2       2     4        1         17 35.52
#> 120                   1           2       3     5       25          6 38.06
#> 121                   1           2       2     4      225         19 14.86
#> 122                   1           1       3     5      172          1 27.10
#> 123                   2           1       6     8       93         18 13.39
#> 124                   3           2       3     5      340          4 10.67
#> 125                   1           1       3     5       39         27  8.78
#> 126                   2           2       3     5      159          9 26.13
#> 127                   2           2       3     5      153          9 37.20
#> 128                   1           1       3     5       44         30 35.19
#> 129                   1           2       2     4      350          4 21.63
#> 130                   1           2       9     1      224         19 27.95
#> 131                   1           1       3     5      340          4 29.91
#> 132                   1           2       3     5      315          5 17.92
#> 133                   1           2       6     8       58         30 15.37
#> 134                   2           2       3     5      352          4 13.36
#> 135                   1           2       3     5      301         39 21.21
#> 136                   2           2       2     4      287          2 38.65
#> 137                   1           1       3     5      283         16 11.52
#> 138                   1           2       3     5      252         12 13.74
#> 139                   1           1       3     5       75         34 15.56
#> 140                   2           2       3     5      148          9 25.07
#> 141                   1           2       6     8      301         39 20.25
#> 142                   2           2       2     4      291         39 19.85
#> 143                   1           2       2     1      107         41 17.09
#> 144                   1           2       2     5       88         18 11.75
#> 145                   2           2       3     5      254         12 16.85
#> 146                   2           2       3     5       46         30 12.90
#> 147                   1           2       3     5      109         41 22.62
#> 148                   1           2       2     4      207         31 32.70
#> 149                   1           1       3     5      332         29 15.56
#> 150                   1           2       3     5       64         30 15.07
#> 151                   1           2       3     5      117         45 15.39
#> 152                   1           1       5     6       36         27 10.03
#> 153                   2           2       9    11       18         26 27.90
#> 154                   1           2       4     7       15         35 15.15
#> 155                   2           1       3     5       90         18  9.16
#> 156                   1           2       2     4      249         23  8.39
#> 157                   1           2       2     4      361          4  6.83
#> 158                   2           1       3     5       20         26 18.99
#> 159                   2           2       3     5      166          9 10.97
#> 160                   1           2       3     5      138         10 24.83
#> 161                   1           1       3     5      301         39 19.24
#> 162                   1           2       2     4       49         30  6.97
#> 163                   1           2       3     5      358          4  6.73
#> 164                   1           2       3     5      301         39 22.80
#> 165                   2           2       3     5      165          9 18.19
#> 166                   3           2       2     4      179         38 31.06
#> 167                   2           2       3     5       81         34 26.45
#> 168                   1           2       3     5      347          4 30.73
#> 169                   3           2       4     7      335          4 12.09
#> 170                   2           2       2     4       43         30 17.11
#> 171                   1           2       3     5       70         34 25.01
#> 172                   1           2       3     5      333          4 13.74
#> 173                   2           2       9    11      132         36  6.78
#> 174                   1           2       3     5      242         20 14.45
#> 175                   2           2      11    12      367          4 17.97
#> 176                   3           2       3     5      254         12 26.89
#> 177                   1           2       2     4       46         30  3.98
#> 178                   1           2       3     5      231         19 26.83
#> 179                   3           1       3     5      179         38 36.30
#> 180                   1           2       9     1       92         18 37.55
#> 181                   1           2       2     4       10         17 14.86
#> 182                   1           2       2     4      219         13 14.61
#> 183                   3           2       3     5      312          5  7.23
#> 184                   1           2       3     5       37         27 13.33
#> 185                   3           2       9    11       89         18 12.69
#> 186                   2           2       3     5      246         20 20.16
#> 187                   1           2       2     4      309         39 14.68
#> 188                   3           1       3     5       49         30 17.07
#> 189                   1           1       3     5      138         10 19.56
#> 190                   2           2       2     4      218         13 10.14
#> 191                   1           2       2     4      205         31 25.56
#> 192                   3           2       9    11      335          4 17.65
#> 193                   2           2       3     5       43         30 12.77
#> 194                   2           2       3     5      290         39 14.48
#> 195                   1           2       9    11      255         12 12.66
#> 196                   3           2       2     4      301         39 25.43
#> 197                   1           2       2     4       72         34 18.92
#> 198                   2           1       3     5      377         43 10.55
#> 199                   2           2       3     5      366          4 19.43
#> 200                   1           2       3     5      301         39 10.88
#> 201                   1           2       6     8      108         41 16.96
#> 202                   1           2       2     1      150          9 33.78
#> 203                   1           1       2     4      137         10 29.28
#> 204                   1           2       2     4      244         20 19.38
#> 205                   2           2       3     5       89         18 12.03
#> 206                   1           2       4     7      302         39 15.45
#> 207                   1           2       3     5      250         12  4.76
#> 208                   2           2       3     5      291         39 22.82
#> 209                   1           2       3     5      163          9 19.04
#> 210                   2           2       2     4      280         16 23.53
#> 211                   3           2       3     5      290         39 14.40
#> 212                   1           1       3     5      178          1 21.21
#> 213                   1           1       3     5      153          9 21.16
#> 214                   1           1       9    11      357          4  6.20
#> 215                   3           1       3     5      371         11 16.74
#> 216                   1           2       6     8      343          4  4.99
#> 217                   2           2       2     4      138         10 17.18
#> 218                   1           2       2     4      318          5 14.88
#> 219                   3           2       9    11      320         46 33.39
#> 220                   1           2       2     4      234         19 34.07
#> 221                   1           2       3     5      351          4 14.42
#> 222                   1           2       3     5      351          4 12.62
#> 223                   1           2       4     7      323          3  9.90
#> 224                   1           2       9    11       60         30 20.38
#> 225                   1           2       2     4      185         38  8.36
#> 226                   1           2       3     5      120         24 16.32
#> 227                   1           2       3     5      350          4  9.53
#> 228                   1           2       3     5      304         39 25.10
#> 229                   1           2       3     5       15         35 23.26
#> 230                   3           2       3     5      139         10 29.14
#> 231                   3           2       2     4       11         17 12.37
#> 232                   1           2       3     5      290         39  8.00
#> 233                   2           2       3     5      315          5  5.76
#> 234                   1           2       3     5       36         27 10.95
#> 235                   1           2       3     5      230         19 22.34
#> 236                   3           2       3     5      260         12 29.88
#> 237                   1           2       2     4      182         38 20.68
#> 238                   1           2       3     5      343          4 38.35
#> 239                   1           2       3     5      289         32 29.20
#> 240                   2           2       2     4       81         34 31.08
#> 241                   1           1       3     5      356          4 10.93
#> 242                   3           1       3     5      125         24 24.14
#> 243                   1           2       3     5      298         39 21.48
#> 244                   3           2       3     1       39         27 27.37
#> 245                   3           2       3     5      250         12 22.58
#> 246                   3           2       3     5       14         17 21.32
#> 247                   2           2       3     5      140         10 12.96
#> 248                   1           2       3     5      366          4 21.17
#> 249                   2           2       3     5      361          4  3.61
#> 250                   1           2       4     7       87         41  6.68
#> 251                   1           2       3     5      343          4 19.22
#> 252                   1           2       4     7      103         41 30.24
#> 253                   2           2       3     5      361          4 10.55
#> 254                   2           2       3     5      357          4 13.05
#> 255                   3           1       3     5      303         39 23.89
#> 256                   3           2       3     5      227         19 20.82
#> 257                   1           2       3     5       58         30 12.74
#> 258                   1           2       4     7      172          1  8.67
#> 259                   1           2       3     5      242         20 16.70
#> 260                   2           1       4     7      291         39 26.92
#> 261                   1           2       2     1       11         17 16.93
#> 262                   1           2       2     4      113         41 20.12
#> 263                   2           2       3     5      177          1 14.73
#> 264                   1           2       3     5      134         36 18.91
#> 265                   1           2       3     5      145         10  6.40
#> 266                   1           2       3     5       39         27  2.87
#> 267                   1           2       3     5       80         34 34.32
#> 268                   2           2       2     4       48         30  9.23
#> 269                   2           1       2     4      330         29 31.29
#> 270                   1           2       3     5      331         29 26.38
#> 271                   2           2       4     7      295         39 25.48
#> 272                   2           2       3     5      357          4 13.36
#> 273                   3           1       6     8      243         20 22.09
#> 274                   3           2       3     5      157          9 17.64
#> 275                   1           2       3     5      199         15 13.63
#> 276                   2           2       3     5      139         10 17.66
#> 277                   2           2       3     5       92         18 12.61
#> 278                   2           2       3     5      383         43  4.66
#> 279                   1           2       3     5      166          9 25.60
#> 280                   2           2       2     4      270         21 16.82
#> 281                   2           1       3     5      316          5 19.76
#> 282                   1           1       3     5      168          1 21.33
#> 283                   1           2       3     5      231         19 13.92
#> 284                   2           2       1     3      257         12 18.93
#> 285                   3           2       9    11       87         41 20.68
#> 286                   3           2       3     5      238         44 31.36
#> 287                   1           2       3     5       34         27 35.77
#> 288                   3           2       3     5      213         31 25.68
#> 289                   2           2       3     5      267         21 18.93
#> 290                   1           2       2     4      348          4  5.90
#> 291                   1           1       2     4       49         30 22.84
#> 292                   2           2       3     5      295         39 37.32
#> 293                   1           2       3     5        9         17 10.51
#> 294                   2           2       3     5      115         41 20.81
#> 295                   1           1       2     4       17         26 26.77
#> 296                   1           2       3     5      309         39 23.57
#> 297                   1           2       3     5       78         34 17.27
#> 298                   1           2       6     8      372         11  0.63
#> 299                   3           2       4     7      155          9 31.87
#> 300                   3           2       3     5      331         29 13.32
#> 301                   2           1       3     5       28         27 11.78
#> 302                   1           2       3     5       14         17 15.88
#> 303                   1           2       3     5      197         15 19.42
#> 304                   1           2       2     4      214         31 15.57
#> 305                   1           2       9    11      183         38 29.15
#> 306                   3           1       9    11      315          5 17.73
#> 307                   1           2       3     5      129         24 15.09
#> 308                   2           2       2     4      142         10 20.37
#> 309                   2           2       3     5      303         39 17.08
#> 310                   2           2       3     5      319         46  5.17
#> 311                   3           1       3     5      364          4 36.23
#> 312                   3           2       3     5       92         18 38.63
#> 313                   1           2       3     5      341          4 18.76
#> 314                   2           1       3     5      155          9 19.14
#> 315                   1           2       3     5      350          4 10.44
#> 316                   1           1       3     5      317          5 23.47
#> 317                   3           1       2     4       82         34 18.41
#> 318                   1           2       3     5      151          9 24.79
#> 319                   2           2       2     4      218         13 16.88
#> 320                   1           2       3     5      281         16 37.39
#> 321                   2           1       3     5      334          4  5.04
#> 322                   2           2       3     5      110         41  7.93
#> 323                   1           2       9    11       91         18  2.71
#> 324                   2           2       2     4       98         18 13.18
#> 325                   3           2       3     5      347          4 25.52
#> 326                   2           1       9    11      144         10 22.48
#> 327                   2           2       3     5      378         43 16.39
#> 328                   2           2       3     5      164          9 19.74
#> 329                   2           2       9    11      322          3 37.22
#> 330                   3           2       3     5      376         33 33.78
#> 331                   2           2       3     5       62         30 17.36
#> 332                   3           1       3     5      263         21 18.51
#> 333                   1           2       3     5      173          1 17.10
#> 334                   1           2       3     5      251         12  8.18
#> 335                   1           2       3     5      166          9 19.92
#> 336                   1           2       3     5      194         15 24.41
#> 337                   1           1       4     7       54         30 29.07
#> 338                   3           1       3     1      212         31 18.21
#> 339                   1           1       4     7      288         32  5.51
#> 340                   1           2       2     4       94         18  8.51
#> 341                   2           2       3     5      131         36 19.24
#> 342                   2           2       3     5      349          4  9.99
#> 343                   2           2       3     5       45         30 18.89
#> 344                   1           2       3     5      169          1 36.40
#> 345                   2           2       3     5      130         24 16.85
#> 346                   2           2       2     4      213         31 27.17
#> 347                   1           1       3     5      361          4 25.15
#> 348                   1           2       3     5      323          3 30.08
#> 349                   2           2       3     5      375         33  7.04
#> 350                   3           2       2     4      181         38 14.98
#> 351                   2           2       3     5      251         12 12.13
#> 352                   3           2       3     5      349          4 23.98
#> 353                   3           2       4     7      302         39 29.22
#> 354                   3           2       3     5      137         10 29.91
#> 355                   1           2       2     4      322          3 15.15
#> 356                   1           2       3     5      313          5 29.75
#> 357                   2           2       2     4      376         33 21.59
#> 358                   1           2       3     5      123         24 18.45
#> 359                   3           2       3     5       28         27 23.96
#> 360                   2           2       2     4       64         30 13.92
#> 361                   1           2       2     4      372         11 23.23
#> 362                   1           2       2     4      315          5  2.80
#> 363                   2           2       2     4       41         30 12.51
#> 364                   3           2       2     4      158          9 14.79
#> 365                   1           2       9     1      128         24 33.75
#> 366                   1           2       2     4      363          4 31.62
#> 367                   1           2       3     5      323          3 23.98
#> 368                   2           2       3     5      125         24 16.70
#> 369                   2           2       3     5       93         18 18.34
#> 370                   2           2       3     5       55         30 20.37
#> 371                   2           1       3     5      347          4 37.26
#> 372                   1           2       2     4      292         39 16.95
#> 373                   3           2       9    11      294         39 27.58
#> 374                   2           2       3     5        8         17 11.19
#> 375                   1           2       3     5      225         19 22.42
#> 376                   1           2       9    11       87         41 26.90
#> 377                   2           2       2     4      330         29 13.54
#> 378                   2           2       3     5       49         30 10.12
#> 379                   1           1       3     5       26          6 10.17
#> 380                   3           2       3     5      314          5 17.42
#> 381                   2           2       2     1      180         38 16.86
#> 382                   2           2       2     4      215         13 21.05
#> 383                   3           2       2     4      157          9 15.60
#> 384                   2           2       1     3       46         30 12.20
#> 385                   1           2       2     4      310         39 26.32
#> 386                   1           2       2     4       24          6 15.56
#> 387                   2           2       3     1      312          5 14.76
#> 388                   1           1       9    11      285          2 25.98
#> 389                   2           2       3     1      344          4 17.56
#> 390                   1           2       2     4       28         27 15.82
#> 391                   3           2       3     5      200         15 10.17
#> 392                   1           2       3     5      349          4 25.31
#> 393                   2           1       3     5      154          9 25.30
#> 394                   1           1       1     1      243         20 19.96
#> 395                   1           2       3     5       19         26 29.15
#> 396                   1           2       2     4        5         17 12.18
#> 397                   2           2       3     5      102         41 18.54
#> 398                   3           2       2     4       21         42 19.13
#> 399                   3           1       3     5      265         21 14.52
#> 400                   1           2       3     5      129         24 16.98
#> 401                   1           2       3     5      217         13 17.19
#> 402                   1           2       2     4      276         25 24.49
#> 403                   3           2       3     5      370          4 22.64
#> 404                   1           2       3     5      211         31 28.73
#> 405                   3           2       2     4       15         35 18.69
#> 406                   3           1       3     5      162          9 25.38
#> 407                   1           2       3     5      290         39 23.76
#> 408                   2           2       6     8      337          4 11.86
#> 409                   2           2       4     1       11         17 19.18
#> 410                   2           2       3     5      378         43 29.37
#> 411                   1           2       2     4      301         39 21.30
#> 412                   1           2       8    10      333          4 16.10
#> 413                   2           2       3     5      251         12 12.07
#> 414                   3           2       3     5      252         12 25.65
#> 415                   2           2       3     5      361          4 15.54
#> 416                   1           2       4     7      377         43 23.69
#> 417                   2           2       8    10      358          4  5.71
#> 418                   2           2       3     5      286          2 24.15
#> 419                   1           2       3     5       87         41 22.57
#> 420                   3           2       3     5       43         30 20.73
#> 421                   2           2       2     4      361          4 14.95
#> 422                   1           1       2     4      166          9 11.52
#> 423                   2           2       3     5       66         34 15.05
#> 424                   3           2       2     4      371         11 20.54
#> 425                   1           2       3     5       33         27  8.75
#> 426                   2           2       5     6      122         24 22.47
#> 427                   2           2      10     2      183         38 14.39
#> 428                   3           2       3     5      366          4 18.56
#> 429                   2           2       3     5      108         41 14.21
#> 430                   2           2       3     5      361          4 14.47
#> 431                   1           2       3     5      182         38 24.86
#> 432                   2           2       2     4      157          9 11.07
#> 433                   1           2       3     5       85          8 37.56
#> 434                   1           1       3     5      349          4  5.70
#> 435                   2           2       1     3      236         44 22.29
#> 436                   2           2       6     8      355          4 13.58
#> 437                   2           2       9    11       23         42 14.33
#> 438                   2           2       2     4      375         33  2.56
#> 439                   3           2       2     4       37         27 38.05
#> 440                   2           2       4     7      316          5 15.77
#> 441                   1           2       3     5       11         17 20.04
#> 442                   1           2       3     5       62         30 21.96
#> 443                   1           2       2     4      188         22 33.54
#> 444                   1           2       3     5       28         27 18.42
#> 445                   1           1       3     5      374         33 10.80
#> 446                   3           2       2     4       69         34 27.22
#> 447                   2           2       3     5      302         39 12.55
#> 448                   2           1       3     5       28         27 33.12
#> 449                   3           2       3     5       66         34 14.28
#> 450                   1           1       5     6       46         30 32.18
#> 451                   1           1       9    11       41         30 33.29
#> 452                   3           2       3     1      302         39 35.70
#> 453                   1           2       3     5      291         39 27.20
#> 454                   3           2       6     8      202         31 22.05
#> 455                   2           2       3     5      295         39 38.61
#> 456                   1           2       3     5       49         30 15.90
#> 457                   3           2       3     5      349          4 24.66
#> 458                   2           2       3     5      354          4 16.27
#> 459                   2           2       3     5       90         18  8.09
#> 460                   1           1       4     7      253         12  2.96
#> 461                   2           2       3     5       81         34 24.06
#> 462                   1           2       3     5      369          4 29.87
#> 463                   2           2       2     4      314          5 11.95
#> 464                   2           2       2     4       41         30 26.66
#> 465                   3           2       9    11      345          4 21.14
#> 466                   2           2       3     5      374         33 16.42
#> 467                   3           2       2     4      125         24 29.79
#> 468                   3           2       3     5      230         19 26.18
#> 469                   2           2       2     4      278         16 25.43
#> 470                   1           2       2     4      346          4 28.49
#> 471                   3           2       2     1      269         21 22.70
#> 472                   3           2       3     5      295         39 20.45
#> 473                   1           2       2     4      175          1 26.03
#> 474                   1           2       4     7      173          1  6.39
#> 475                   2           2       2     4      253         12 11.99
#> 476                   1           2       3     5       48         30 13.16
#> 477                   3           2       2     4      386         43 18.41
#> 478                   2           2       2     4      114         41 19.34
#> 479                   1           2       2     4       38         27  9.28
#> 480                   3           2       3     5      271         14 25.36
#> 481                   1           2       9    11      256         12 28.36
#> 482                   1           2       2     4      124         24 14.79
#> 483                   1           1       3     5      190         22 21.56
#> 484                   2           2       3     5       45         30 19.75
#> 485                   3           2       2     4       61         30 23.85
#> 486                   2           2       3     5      334          4 17.15
#> 487                   1           2       3     5      148          9 23.47
#> 488                   1           1       3     5      307         39 24.39
#> 489                   2           2       2     4      138         10 21.60
#> 490                   2           2       3     5      326         28 17.02
#> 491                   1           1       2     4      252         12 19.76
#> 492                   2           2       4     7      116         41 18.59
#> 493                   2           2       3     5       52         30  5.79
#> 494                   1           2       6     8      327         28 20.64
#> 495                   1           2       6     8      158          9 15.95
#> 496                   1           2       4     7      158          9  9.14
#> 497                   2           2       9    11      379         43 16.17
#> 498                   1           1       3     5      137         10 22.50
#> 499                   3           2       3     5      228         19 16.58
#> 500                   2           1       3     5      174          1 23.77
#> 501                   3           2       3     5      313          5 23.03
#> 502                   3           1       3     5      132         36 23.62
#> 503                   1           2       3     5       77         34 20.82
#> 504                   1           2       2     4      254         12 16.80
#> 505                   2           2       9    11      329         28 14.89
#> 506                   2           1       2     4      359          4 30.96
#> 507                   1           2       3     5      350          4 14.68
#> 508                   3           2       3     5      133         36  2.86
#> 509                   1           2       3     5      301         39 20.73
#> 510                   1           2       2     4       45         30  5.56
#> 511                   3           1       3     5      382         43 20.55
#> 512                   3           1       3     5       40         27 15.29
#> 513                   1           2       3     5      340          4 13.83
#> 514                   3           2       4     7       65         34 32.83
#> 515                   2           2       2     4      104         41 29.60
#> 516                   2           1       2     4      161          9 22.99
#> 517                   3           2       2     4      162          9 30.04
#> 518                   2           2       4     7      122         24  8.28
#> 519                   3           1       3     5       29         27 30.95
#> 520                   2           2       3     5      105         41 30.45
#> 521                   2           2       3     5      106         41 36.53
#> 522                   2           2       2     4       46         30 17.20
#> 523                   1           2       3     5      101         41 19.74
#> 524                   2           2       3     5      224         19 20.00
#> 525                   1           1       3     5      276         25 23.24
#> 526                   2           2       3     1      355          4 17.73
#> 527                   3           2       3     5      336          4 20.31
#> 528                   1           2       3     5      292         39  4.96
#> 529                   3           2       2     4      361          4  9.82
#> 530                   2           2       2     4       65         34  9.21
#> 531                   3           2       3     5      273         14 21.96
#> 532                   1           2       6     8      301         39 30.98
#> 533                   3           2       3     5      218         13 17.71
#> 534                   1           2       3     5      114         41 16.93
#> 535                   2           2       4     7       60         30 12.08
#> 536                   2           2       2     4      322          3 24.85
#> 537                   2           2       3     1        7         17  6.82
#> 538                   3           1       3     5      305         39 13.57
#> 539                   1           2       2     4      252         12 16.45
#> 540                   3           1      10     2      138         10 36.81
#> 541                   2           2       2     4      331         29  4.30
#> 542                   3           1       3     5       83         34 34.58
#> 543                   3           2       8    10       32         27 19.27
#> 544                   3           2       3     5       81          4 21.88
#> 545                   3           2      11    12      361          4 25.55
#> 546                   2           2       4     7      316          5 12.48
#> 547                   2           2       3     5       17         26 26.17
#> 548                   3           2       3     5      266         21 25.70
#> 549                   2           2       3     5      333          4 10.48
#> 550                   2           2       1     3      250         12 15.15
#> 551                   3           1       3     5      266         21 27.42
#> 552                   2           1       3     5       82         34 23.81
#> 553                   1           2       3     1       89         18 19.85
#> 554                   1           2       3     5      259         12 28.35
#> 555                   2           1       2     4      349          4 14.45
#> 556                   3           2       3     5       97         18 27.87
#> 557                   1           2       3     5      195         15 30.65
#> 558                   3           2       3     5       45         30 21.31
#> 559                   3           2       2     4       36         27 22.42
#> 560                   1           2       2     4       74         34 22.74
#> 561                   2           2       3     5       45         30  6.31
#> 562                   2           1       9    11       85          8 13.27
#> 563                   2           2       3     5      162          9 16.49
#> 564                   1           2       3     5       92         18 14.34
#> 565                   2           2       3     5       64         30 18.48
#> 566                   1           2       3     5      233         19 28.69
#> 567                   1           2       3     5      291         39 30.81
#> 568                   2           2       3     5      357          4 33.80
#> 569                   1           2       3     1        2         17 24.00
#> 570                   2           2       2     4      109         41 14.19
#> 571                   3           2       9    11      166          9 23.73
#> 572                   2           2       6     8       27          6  0.70
#> 573                   2           2       2     4       44         30 19.26
#> 574                   1           1       3     5      251         12 24.01
#> 575                   2           2       2     4      295         39 14.63
#> 576                   1           2       3     5      293         39 24.60
#> 577                   2           2       3     5      251         12 18.28
#> 578                   2           2       3     5      142         10 21.53
#> 579                   1           2       3     5      295         39 14.06
#> 580                   2           2       2     4       67         34 11.78
#> 581                   2           2       3     5      314          5 11.11
#> 582                   3           1       3     5      254         12 46.71
#> 583                   2           2       3     5      212         31  7.04
#> 584                   1           2       2     4      121         24 28.37
#> 585                   3           2       3     5      165          9 27.11
#> 586                   3           2       3     5       51         30 26.90
#> 587                   2           2       3     5      363          4  8.29
#> 588                   2           2       3     5      137         10 14.89
#> 589                   2           2       3     5      129         24 36.36
#> 590                   2           2       3     5      251         12 16.82
#> 591                   2           2      11    12      349          4  6.68
#> 592                   2           2       3     5      127         24 30.79
#> 593                   2           2       3     5       49         30 13.09
#> 594                   1           2       3     5      247         37 13.56
#> 595                   1           2       3     5       50         30 14.40
#> 596                   3           2       2     4      139         10 20.42
#> 597                   1           2       9    11       92         18 11.66
#> 598                   3           2       3     5      350          4 31.44
#> 599                   2           2       3     5       99         18  2.87
#> 600                   1           2       3     5       89         18 17.49
#> 601                   2           2       3     5       92         18 14.65
#> 602                   3           2       3     5      291         39 31.24
#> 603                   1           2       2     4       46         30  9.24
#> 604                   1           2       3     5      324          3 11.96
#> 605                   2           2       2     1      144         10 27.24
#> 606                   2           2       3     5      290         39 21.19
#> 607                   1           2       3     5      301         39 19.44
#> 608                   2           1       3     5       87         41 28.73
#> 609                   1           2       4     7       35         27 13.30
#> 610                   1           1       2     4      155          9 18.05
#> 611                   2           2       2     4      252         12  8.56
#> 612                   2           2       3     5      176          1 26.24
#> 613                   2           2       3     5      137         10 15.84
#> 614                   1           2       2     4       25          6  3.86
#> 615                   3           1       2     4      207         31 16.01
#> 616                   1           2       2     4      181         38 14.75
#> 617                   2           1       4     7      254         12  7.57
#> 618                   2           2       3     1      154          9 14.69
#> 619                   3           2       2     4      214         31 12.87
#> 620                   2           2       3     5      376         33  6.92
#> 621                   1           2       3     5      182         38 25.91
#> 622                   1           1       2     4      128         24 39.21
#> 623                   2           2       3     5      243         20 32.56
#> 624                   1           1       3     5      386         43 17.38
#> 625                   1           1       2     4      311         39  6.42
#> 626                   1           2       3     5      378         43 14.69
#> 627                   2           2       3     5      312          5  9.74
#> 628                   3           2       2     4       94         18 23.70
#> 629                   2           2       2     4       36         27 11.67
#> 630                   3           2       3     5       89         18 15.61
#> 631                   3           2       2     4      148          9 16.18
#> 632                   3           1       3     5      301         39 22.36
#> 633                   2           2       9    11      141         10 16.87
#> 634                   1           2       3     5      280         16 28.44
#> 635                   2           2       4     7       51         30 13.38
#> 636                   2           1       2     4      384         43 10.03
#> 637                   2           2       3     5      149          9 17.48
#> 638                   1           2       3     5      339          4 22.64
#> 639                   2           2       3     5      241         20 12.39
#> 640                   3           2       9    11       59         30 35.72
#> 641                   2           1       3     5      206         31 32.68
#> 642                   2           2       3     5      156          9 22.55
#> 643                   1           2       2     4      254         12  5.88
#> 644                   2           2       3     5      243         20 19.03
#> 645                   1           2       3     5       49         30 15.96
#> 646                   2           2      10     2      368          4 12.73
#> 647                   1           2       3     5      233         19 26.06
#> 648                   2           2       3     5      227         19 30.53
#> 649                   1           2       3     5      282         16 10.49
#> 650                   2           1       9    11      186         38 31.08
#> 651                   3           2       3     5      277         25 22.59
#> 652                   2           2       3     5      225         19  4.01
#> 653                   2           2       2     4       68         34 20.28
#> 654                   1           2       4     7      323          3 21.79
#> 655                   1           2       3     5      109         41 17.75
#> 656                   2           2       2     4      137         10 25.38
#> 657                   1           2       3     5      362          4 13.95
#> 658                   1           2       3     5      180         38 12.63
#> 659                   3           1       3     5      345          4 34.10
#> 660                   1           2       2     4      265         21 16.91
#> 661                   2           2       2     4      333          4  5.27
#> 662                   1           2       2     4      254         12  9.48
#> 663                   2           2       4     7       28         27 16.36
#> 664                   3           2       2     4      182         38 26.84
#> 665                   2           2       4     7      189         22 25.89
#> 666                   3           1       3     5      308         39 11.45
#> 667                   2           1       3     5      382         43 33.18
#> 668                   2           2       2     4      292         39 10.29
#> 669                   3           2       3     5      245         20 32.79
#> 670                   1           2       2     4      346          4 15.76
#> 671                   2           2       2     4      309         39  9.44
#> 672                   2           2       3     5       13         17 15.18
#> 673                   2           2       3     5      167          9 15.46
#> 674                   3           2       3     1      297         39 23.09
#> 675                   1           2       2     4      245         20  6.66
#> 676                   3           2       3     5      230         19 12.57
#> 677                   1           2       3     5        7         17 25.97
#> 678                   1           2       2     4      157          9 35.00
#> 679                   2           2       2     4      323          3 23.62
#> 680                   3           2       3     5      155          9  5.54
#> 681                   2           2       3     5      208         31 14.10
#> 682                   1           1       2     4       43         30 12.31
#> 683                   1           2       4     7       80         34 19.24
#> 684                   2           2       2     4      241         20 13.88
#> 685                   1           2       3     5      352          4 31.04
#> 686                   2           2       2     4      361          4  4.91
#> 687                   1           2       4     7      225         19 29.75
#> 688                   3           2       3     5       89         18 22.82
#> 689                   3           2       9    11      109         41 39.16
#> 690                   1           2       9    11      303         39 24.79
#> 691                   1           1       2     4      306         39 25.58
#> 692                   3           2       3     5      160          9  6.34
#> 693                   2           2      10     2      237         44 14.11
#> 694                   3           2       3     5      214         31 13.29
#> 695                   2           2       2     4       77         34 14.65
#> 696                   3           2       3     5      261         12 12.31
#> 697                   2           1       2     4      295         39 19.60
#> 698                   1           2       3     5      362          4 19.50
#> 699                   3           2       3     5      355          4 39.87
#> 700                   2           2       3     5      279         16  8.91
#> 701                   2           2       2     4      108         41 13.34
#> 702                   2           2       3     5       15         35 17.44
#> 703                   2           1       3     5      264         21 19.90
#> 704                   2           1       3     5      299         39 11.98
#> 705                   2           2       2     4      202         31 26.45
#> 706                   1           2       2     4       12         17 25.60
#> 707                   2           2       2     4      157          9  8.28
#> 708                   3           2       3     5      242         20  6.15
#> 709                   1           2       4     7      137         10 12.07
#> 710                   1           2       3     5      296         39 15.62
#> 711                   2           2       2     4      385         43 11.69
#> 712                   1           2       2     4      161          9 10.48
#> 713                   2           2       2     4      191         22 26.93
#> 714                   1           2       2     4       85          8 12.15
#> 715                   1           2       2     4      118         45 13.50
#> 716                   1           2       2     4      303         39  7.21
#> 717                   1           2       4     7      358          4 10.57
#> 718                   2           2       2     4      157          9 14.30
#> 719                   1           2       4     7      146         10 18.44
#> 720                   1           2       2     4      131         36 25.73
#> 721                   3           1       3     5      222         13  7.78
#> 722                   3           1       7     9      202         31 11.74
#> 723                   3           2       3     5      267         21 17.29
#> 724                   1           2       3     5       15         35 12.18
#> 725                   1           2       2     4      327         28 15.82
#> 726                   3           1       3     5      134         36 26.02
#> 727                   1           1       3     5      361          4 12.09
#> 728                   2           2       2     4      121         24 13.33
#> 729                   2           2       3     5      182         38 33.15
#> 730                   1           2       3     5       30         27 23.84
#> 731                   2           2       3     5      137         10 17.76
#> 732                   3           2       3     5      359         33 25.04
#> 733                   3           1       4     7      301         39 32.33
#> 734                   3           2       2     4      115         41 33.98
#> 735                   2           1       3     5       92         18 19.13
#> 736                   3           2       2     4      102         41 27.73
#> 737                   3           2       3     5       58         30 28.55
#> 738                   1           2       4     7       92         18 16.11
#> 739                   2           2       2     4      244         20 10.48
#> 740                   1           2       3     5      296         39 23.21
#> 741                   3           2       2     4       39         27 19.81
#> 742                   1           2       2     4        3         17 21.71
#> 743                   2           2       2     4      380         43 21.72
#> 744                   2           1       3     5      184         38 27.52
#> 745                   1           2       2     4       73         34 16.19
#> 746                   3           2       2     4      377         43 21.38
#> 747                   1           2       3     5      140         10 23.42
#> 748                   1           2       9    11      105         41 39.11
#> 749                   1           1       3     5      157          9 27.93
#> 750                   2           2       3     1       38         27 33.10
#> 751                   1           2       3     5       41         30  7.10
#> 752                   2           2       2     4       40         27 25.25
#> 753                   3           2       4     7      232         19 19.79
#> 754                   2           2       3     5      152          9 19.07
#> 755                   2           2       3     5      287          2 27.61
#> 756                   3           2       3     5      198         15 26.64
#> 757                   3           2       2     4       79         34  7.24
#> 758                   3           2       3     5      345          4 21.02
#> 759                   1           2       3     5       29         27 15.22
#> 760                   1           1       3     5      125         24 29.55
#> 761                   3           2       3     5      309         39 24.61
#> 762                   1           2       2     4      215         13 21.58
#> 763                   1           2       6     8      353          4 17.73
#> 764                   3           2       3     5      215         13 35.91
#> 765                   3           1       3     5      188         22 22.16
#> 766                   1           2       3     5      303         39  8.32
#> 767                   1           2       3     5      313          5  9.46
#> 768                   1           2       2     4       54         30 29.60
#> 769                   3           2       3     5      154          9 20.13
#> 770                   2           1       3     5      373         33 14.44
#> 771                   3           2       3     5      213         31 19.13
#> 772                   2           2       3     5       80         34 22.87
#> 773                   3           2       3     5      121         24 22.79
#> 774                   1           1       3     5       36         27 21.79
#> 775                   1           2       3     5      193         15 15.15
#> 776                   1           2       3     5      157          9 10.55
#> 777                   1           2       3     5      137         10 13.42
#> 778                   2           2       3     5      201         31 13.21
#> 779                   2           2       3     5      378         43 16.99
#> 780                   2           2       4     7      229         19 19.44
#> 781                   1           2       2     4      255         12 18.47
#> 782                   3           2       3     5       94         18 33.93
#> 783                   3           1       3     5      281         16 20.05
#> 784                   1           2       3     5      160          9  5.76
#> 785                   3           2       3     5       98         18 24.74
#> 786                   1           2      10     2      354          4 18.60
#> 787                   2           2       3     5        5         17 28.08
#> 788                   3           2       2     4      203         31 19.02
#> 789                   2           2       3     5      292         39 18.47
#> 790                   1           2       3     5      130         24  9.71
#> 791                   2           2       3     5       12         17  7.96
#> 792                   2           2       3     5      358          4 14.16
#> 793                   3           2       4     7      125         24 20.72
#> 794                   1           2       2     4      192         22 23.49
#> 795                   3           1       2     4      333          4 21.87
#> 796                   1           2       2     4      321         40 29.05
#> 797                   1           2       3     5      137         10 15.50
#> 798                   2           2       3     5      143         10  4.90
#> 799                   1           2       2     4      147          9 21.60
#> 800                   3           2       3     5       42         30 15.42
#> 801                   2           2       2     4      126         24 13.77
#> 802                   1           2       3     5      310         39 27.96
#> 803                   2           2       3     5      141         10 15.73
#> 804                   3           2       2     4       13         17 18.84
#> 805                   1           2       2     4      140         10 24.55
#> 806                   2           2       3     5      206         31 16.03
#> 807                   2           1       3     5      210         31 29.15
#> 808                   2           2       9    11      323          3 17.24
#> 809                   3           2       1     3      224         19  2.17
#> 810                   3           1       6     8      290         39 18.52
#> 811                   1           2       3     5       52         30  6.62
#> 812                   2           2       2     4       41         30 30.13
#> 813                   3           2       2     4       25          6 12.67
#> 814                   2           2       6     8      343          4 16.73
#> 815                   1           2       3     5      172          1  6.46
#> 816                   2           2       4     7      171          1 20.24
#> 817                   1           2       3     5      233         19  0.76
#> 818                   1           2       2     4       11         17 26.23
#> 819                   1           2       3     5      323          3 14.03
#> 820                   2           2       3     5      201         31  4.94
#> 821                   1           2       2     4       28         27 16.37
#> 822                   3           2       9    11       31         27 19.79
#> 823                   2           2       6     8      140         10 13.54
#> 824                   1           2       3     5       63         30 15.61
#> 825                   1           2       3     5      243         20 24.76
#> 826                   3           1       3     5      271         14 19.93
#> 827                   2           2       3     5       79         34  7.58
#> 828                   3           2       8    10       49         30  0.94
#> 829                   2           1       3     5       11         17 27.44
#> 830                   1           1       3     5      301         39 24.67
#> 831                   1           2       3     5      363          4 12.67
#> 832                   1           1       3     5      282         16 17.59
#> 833                   1           1       2     4      344          4 31.23
#> 834                   1           2      10     2      222         13 26.08
#> 835                   3           2       3     5      243         20 10.78
#> 836                   3           2       3     5      241         20 31.61
#> 837                   2           1       3     5      284          2  7.50
#> 838                   1           2       3     5       47         30 22.37
#> 839                   3           1       3     5      137         10 21.87
#> 840                   1           2       2     4       45         30 17.84
#> 841                   2           2      10     2      258         12 26.16
#> 842                   1           2       2     4      324          3 16.58
#> 843                   2           2       2     4      196         15  7.54
#> 844                   1           2       6     8      216         13 12.47
#> 845                   1           2       1     3      270         21 14.79
#> 846                   2           2       4     7      119         45 26.76
#> 847                   3           2       2     4      301         39 39.30
#> 848                   2           2       2     4      187         38 12.52
#> 849                   1           2       2     4      158          9 25.63
#> 850                   2           1       3     5      310         39 18.17
#> 851                   3           2       3     5       89         18 24.75
#> 852                   1           2       3     5      167          9 12.29
#> 853                   3           2       2     4      154          9 16.28
#> 854                   1           2       3     5       80         34 25.11
#> 855                   1           2       3     5      123         24 12.02
#> 856                   2           2       9    11        4         17 13.58
#> 857                   3           2       2     4      112         41 38.02
#> 858                   1           2       3     5      292         39 13.57
#> 859                   1           2       1     3       11         17 18.96
#> 860                   3           2       3     5      137         10 24.59
#> 861                   3           2       3     5      156          9 28.03
#> 862                   1           2       3     5      340          4 22.96
#> 863                   3           1       3     5      128         24 21.09
#> 864                   1           2       4     7      252         12 19.59
#> 865                   2           1       2     4      262         12 13.57
#> 866                   3           2       2     1      137         10 15.73
#> 867                   2           2       3     5       75         34 25.38
#> 868                   2           2       3     5      246         20  9.16
#> 869                   3           1       3     5      300         39 26.20
#> 870                   2           2       2     4      385         43 10.45
#> 871                   1           2       2     4       20         26 11.65
#> 872                   2           2       3     5      365          4 21.15
#> 873                   3           2       3     5      360          4 16.21
#>     delinq_2yrs earliest_cr_line fico_range_high inq_last_6mths pub_rec
#> 1             0               49             679              1       0
#> 2             1               71             719              4       0
#> 3             0               46             699              0       0
#> 4             1              174             699              3       0
#> 5             0              272             694              0       0
#> 6             0              166             684              0       0
#> 7             1               97             709              0       0
#> 8             0               21             689              1       1
#> 9             0              251             704              0       0
#> 10            0              172             704              0       0
#> 11            0              181             669              1       1
#> 12            0              212             724              0       0
#> 13            0              172             699              0       0
#> 14            0               73             749              1       0
#> 15            0              250             804              0       0
#> 16            0              202             679              4       1
#> 17            0              232             719              0       0
#> 18            0              167             689              0       2
#> 19            0              228             699              0       0
#> 20            0              291             669              0       0
#> 21            1              222             699              1       0
#> 22            2              158             729              0       0
#> 23            0              219             839              1       0
#> 24            0               21             684              1       0
#> 25            0              229             784              0       0
#> 26            0              178             689              1       0
#> 27            0               61             679              0       1
#> 28            0               26             664              0       0
#> 29            0              100             694              0       1
#> 30            0              177             734              1       0
#> 31            1              278             734              1       0
#> 32            0              128             684              1       0
#> 33            0              150             704              0       0
#> 34            0               50             729              1       0
#> 35            0              183             694              0       0
#> 36            0              152             684              0       0
#> 37            0               20             689              0       0
#> 38            0              220             729              0       0
#> 39            0              150             754              0       0
#> 40            0              281             674              0       1
#> 41            0              181             684              0       1
#> 42            0              258             669              1       0
#> 43            3              124             674              4       0
#> 44            0              311             684              1       0
#> 45            0              312             714              0       0
#> 46            0              284             689              0       0
#> 47            0               90             689              0       0
#> 48            0               36             679              0       0
#> 49            0              280             684              1       0
#> 50            0               21             674              0       0
#> 51            0               20             704              0       0
#> 52            0               47             679              1       0
#> 53            0               21             734              0       0
#> 54            0              102             699              0       0
#> 55            0               55             694              0       0
#> 56            0              177             684              0       0
#> 57            0              123             704              0       0
#> 58            0              176             669              2       0
#> 59            0              130             759              1       0
#> 60            0               43             729              1       0
#> 61            0               63             749              0       0
#> 62            0               94             799              0       0
#> 63            0              180             724              0       1
#> 64            0              171             689              1       0
#> 65            0               47             724              0       0
#> 66            0              286             699              0       0
#> 67            0               74             674              0       1
#> 68            0               35             664              0       0
#> 69            0              259             664              0       0
#> 70            0              286             709              0       0
#> 71            0              288             674              0       0
#> 72            0              307             719              2       1
#> 73            0              178             679              1       0
#> 74            0              108             689              0       1
#> 75            0              177             804              2       0
#> 76            0               24             759              0       0
#> 77            0              302             754              0       0
#> 78            0               13             779              1       0
#> 79            1              313             704              0       0
#> 80            0              122             674              0       0
#> 81            3              235             669              0       0
#> 82            0              198             719              1       0
#> 83            0               14             679              2       1
#> 84            1               22             709              1       0
#> 85            0               73             674              2       1
#> 86            0              125             714              0       1
#> 87            0              132             704              0       0
#> 88            0              257             694              2       0
#> 89            0              230             699              0       0
#> 90            0              257             679              0       0
#> 91            0              260             789              0       0
#> 92            0              310             674              0       1
#> 93            0               54             664              0       0
#> 94            0               69             689              0       0
#> 95            0              234             689              1       3
#> 96            0              258             684              1       2
#> 97            0              312             709              1       0
#> 98            0               55             714              0       1
#> 99            0              265             709              2       0
#> 100           1              276             704              0       0
#> 101           4               71             674              2       0
#> 102           0              251             694              1       0
#> 103           0              213             664              3       1
#> 104           0              309             719              0       0
#> 105           0              181             669              3       0
#> 106           0              175             669              1       0
#> 107           0              177             694              1       0
#> 108           0              263             664              0       0
#> 109           0              234             709              1       0
#> 110           0              259             684              0       0
#> 111           0              260             744              1       1
#> 112           0              125             679              1       0
#> 113           0              253             729              0       0
#> 114           0               98             714              0       0
#> 115           0              109             704              0       0
#> 116           0              182             714              0       0
#> 117           1              150             674              1       1
#> 118           0               30             749              1       0
#> 119           0              107             719              2       0
#> 120           0              262             774              1       0
#> 121           1              259             719              0       0
#> 122           0               71             709              0       0
#> 123           0               60             699              0       0
#> 124           3              118             714              0       0
#> 125           0              201             709              1       0
#> 126           0              281             669              0       2
#> 127           0               44             689              0       1
#> 128           0               78             674              1       0
#> 129           0              248             759              0       0
#> 130           0               25             699              0       0
#> 131           0              290             744              0       0
#> 132           0              261             799              0       0
#> 133           0               93             724              1       0
#> 134           0              274             754              0       0
#> 135           0              171             704              1       0
#> 136           2              293             729              0       0
#> 137           1              222             669              1       0
#> 138           0              131             699              0       0
#> 139           0              259             744              0       0
#> 140           0              171             694              0       0
#> 141           0              229             769              0       0
#> 142           0              145             664              0       0
#> 143           0              228             684              0       0
#> 144           0               38             694              0       1
#> 145           0              282             739              0       0
#> 146           0              262             689              0       0
#> 147           0               72             689              2       0
#> 148           1              278             694              1       0
#> 149           0              173             704              2       0
#> 150           0              297             734              0       0
#> 151           1              226             689              0       1
#> 152           1              229             669              1       0
#> 153           0               57             694              1       0
#> 154           0              309             794              0       0
#> 155           0               24             679              0       2
#> 156           0              294             819              0       0
#> 157           0               23             709              0       0
#> 158           0              285             714              1       0
#> 159           0              238             744              0       0
#> 160           0              127             689              0       0
#> 161           0              161             694              0       0
#> 162           0              143             664              1       1
#> 163           0              256             779              0       0
#> 164           0               43             719              0       0
#> 165           0              309             689              0       0
#> 166           0               80             709              0       0
#> 167           0              183             714              0       0
#> 168           1              147             664              1       0
#> 169           0              144             689              1       0
#> 170           0              316             679              0       0
#> 171           0              197             744              0       0
#> 172           0               47             734              0       0
#> 173           0              242             850              1       0
#> 174           0              298             729              1       0
#> 175           0              195             664              2       3
#> 176           0              290             679              1       0
#> 177           0              307             784              0       0
#> 178           1              221             679              0       0
#> 179           1              176             669              3       1
#> 180           0              164             689              0       1
#> 181           0              275             759              1       0
#> 182           0              272             734              0       0
#> 183           0               43             689              0       0
#> 184           0              226             679              0       1
#> 185           1               12             674              1       0
#> 186           0               48             694              1       0
#> 187           2               98             674              0       0
#> 188           0               47             689              1       0
#> 189           0              206             679              0       0
#> 190           0              315             689              0       0
#> 191           0               58             724              2       0
#> 192           0              164             669              0       0
#> 193           0              291             719              0       0
#> 194           0              151             674              1       0
#> 195           0              212             669              0       0
#> 196           0               49             729              0       0
#> 197           0               23             744              1       0
#> 198           1               42             684              0       0
#> 199           6              112             669              0       0
#> 200           0              103             744              0       0
#> 201           1               43             734              1       1
#> 202           0                5             764              0       0
#> 203           0              102             664              1       1
#> 204           0              307             704              0       0
#> 205           3              217             729              0       0
#> 206           0              146             744              0       0
#> 207           0              260             714              1       1
#> 208           1               23             669              0       0
#> 209           0              194             769              0       0
#> 210           0              198             714              0       0
#> 211           1              251             759              0       0
#> 212           0              227             689              3       1
#> 213           0               99             709              0       1
#> 214           0              231             754              2       0
#> 215           0              307             664              1       0
#> 216           3              295             709              1       0
#> 217           0               11             729              1       0
#> 218           1              240             714              0       0
#> 219           0              280             689              0       1
#> 220           1              253             679              1       0
#> 221           0              208             714              0       0
#> 222           0               21             664              0       1
#> 223           0               47             704              0       0
#> 224           0              125             664              0       0
#> 225           4              149             674              2       1
#> 226           0              151             699              1       0
#> 227           0              259             674              0       1
#> 228           0                9             664              0       0
#> 229           0              178             719              0       0
#> 230           0              251             739              0       0
#> 231           0               76             684              0       0
#> 232           1              244             664              1       0
#> 233           0               77             679              1       1
#> 234           0              105             669              3       0
#> 235           1              149             669              1       0
#> 236           0              269             719              0       0
#> 237           0              309             729              0       0
#> 238           0              236             694              0       0
#> 239           0               53             734              0       0
#> 240           0              281             674              2       0
#> 241           0              151             669              1       0
#> 242           0               79             674              0       0
#> 243           0                8             794              0       0
#> 244           0               71             734              2       0
#> 245           0              155             674              1       0
#> 246           1              281             669              2       1
#> 247           0              121             694              0       0
#> 248           0               45             714              0       0
#> 249           1              231             699              1       0
#> 250           0              146             704              0       0
#> 251           0              258             734              0       0
#> 252           2              131             689              2       0
#> 253           0              173             759              0       0
#> 254           1              197             664              1       1
#> 255           0               88             674              2       0
#> 256           0              162             739              0       0
#> 257           2               33             664              0       0
#> 258           1              224             684              0       1
#> 259           1              303             679              0       0
#> 260           0              120             689              4       0
#> 261           0              277             739              0       0
#> 262           0              124             794              0       0
#> 263           0               75             674              0       1
#> 264           0                3             809              0       0
#> 265           0               49             754              0       0
#> 266           1              199             674              2       1
#> 267           0              311             679              2       0
#> 268           0              136             684              4       1
#> 269           0               28             674              0       0
#> 270           0              230             719              0       0
#> 271           1              241             729              0       0
#> 272           0              130             664              0       0
#> 273           0               96             684              1       1
#> 274           0              183             709              0       0
#> 275           0              138             679              0       1
#> 276           1              200             674              0       3
#> 277           1              312             664              0       0
#> 278           2              168             704              2       0
#> 279           0              113             689              0       0
#> 280           1              124             694              0       0
#> 281           5              175             664              1       1
#> 282           2              261             664              0       1
#> 283           0               50             669              4       0
#> 284           1               77             664              1       1
#> 285           0              289             739              1       0
#> 286           1              153             689              1       1
#> 287           1               93             734              3       0
#> 288           3               41             684              0       0
#> 289           0              147             694              0       0
#> 290           0               58             734              0       0
#> 291           0              281             714              1       0
#> 292           0              285             699              0       0
#> 293           0               75             709              0       0
#> 294           1               95             704              0       0
#> 295           1               49             664              1       2
#> 296           0               45             754              3       0
#> 297           0              251             709              0       0
#> 298           0               88             799              0       0
#> 299           1              103             664              0       0
#> 300           0              303             709              0       0
#> 301           3              260             669              0       0
#> 302           1               68             689              0       0
#> 303           0              189             789              1       0
#> 304           0              312             734              0       0
#> 305           0              171             714              1       0
#> 306           0              283             679              2       0
#> 307           0              126             674              0       1
#> 308           0              312             664              4       1
#> 309           1                6             694              1       0
#> 310           0              285             774              1       0
#> 311           1               74             689              1       0
#> 312           0              229             704              0       0
#> 313           0               12             709              0       0
#> 314           1               69             689              0       0
#> 315           0              255             724              1       0
#> 316           0              149             709              0       0
#> 317           0               94             679              1       1
#> 318           0               44             674              0       0
#> 319           0               54             694              1       1
#> 320           0              299             674              1       0
#> 321           0               76             664              2       0
#> 322           0              245             704              1       0
#> 323           0              146             749              0       0
#> 324           0               76             664              0       0
#> 325           0              211             704              1       0
#> 326           0              200             704              1       1
#> 327           0              266             674              0       0
#> 328           0              241             689              3       0
#> 329           0              216             739              0       0
#> 330           0              263             679              1       0
#> 331           0              301             694              0       1
#> 332           1              192             674              1       0
#> 333           0              165             729              1       0
#> 334           1              144             669              2       1
#> 335           1              174             689              0       0
#> 336           0               50             719              0       0
#> 337           4              116             694              1       1
#> 338           0               10             714              1       1
#> 339           0              126             709              2       0
#> 340           0              153             764              0       0
#> 341           0               73             694              0       1
#> 342           0              202             664              1       1
#> 343           0              195             714              1       1
#> 344           0              228             679              0       1
#> 345           1              251             669              0       0
#> 346           0               51             674              0       0
#> 347           0               56             689              0       1
#> 348           0              241             809              0       0
#> 349           0              256             699              0       0
#> 350           0               74             714              0       0
#> 351           0               56             749              0       0
#> 352           1              222             669              0       1
#> 353           0              196             729              0       0
#> 354           0               70             679              5       1
#> 355           0              177             674              4       0
#> 356           0              222             734              0       0
#> 357           0               34             694              0       0
#> 358           0               46             689              1       1
#> 359           1              264             669              1       0
#> 360           0              289             714              0       0
#> 361           1              140             669              2       0
#> 362           0              296             664              0       1
#> 363           0              175             679              0       0
#> 364           0              200             674              1       1
#> 365           0              200             709              0       0
#> 366           1              283             689              0       0
#> 367           0              313             694              1       0
#> 368           0               68             709              0       0
#> 369           2              179             694              0       0
#> 370           0              204             684              0       0
#> 371           0              281             704              1       0
#> 372           0              271             804              0       0
#> 373           0              256             719              0       0
#> 374           0               82             724              2       0
#> 375           0              201             724              1       0
#> 376           0               18             684              0       0
#> 377           1              283             694              0       0
#> 378           0              256             724              1       0
#> 379           0              309             664              1       0
#> 380           0              264             689              0       0
#> 381           1               10             674              0       0
#> 382           1               83             704              2       0
#> 383           0              179             689              0       1
#> 384           0               90             684              0       1
#> 385           0              157             714              0       0
#> 386           1               49             674              1       1
#> 387           0              142             769              0       0
#> 388           0              227             709              0       1
#> 389           0               50             669              0       0
#> 390           0              311             729              1       0
#> 391           1              303             689              0       0
#> 392           0              311             679              2       0
#> 393           0              199             664              3       1
#> 394           1              244             699              1       0
#> 395           0              313             704              0       1
#> 396           0              148             739              0       0
#> 397           1              119             684              0       0
#> 398           0               96             714              1       0
#> 399           0              262             664              0       1
#> 400           0               25             689              0       0
#> 401           0              151             704              0       0
#> 402           0               25             709              0       1
#> 403           0                8             729              0       0
#> 404           2              201             689              0       0
#> 405           0              126             699              1       0
#> 406           0               29             684              0       0
#> 407           1              167             734              1       0
#> 408           1              102             669              0       0
#> 409           1              132             689              0       0
#> 410           0              308             694              0       0
#> 411           0               62             749              0       0
#> 412           0              264             664              2       0
#> 413           0              307             669              0       0
#> 414           0              287             674              2       0
#> 415           0              197             674              0       0
#> 416           0              299             714              1       0
#> 417           0              187             714              1       0
#> 418           0              285             814              0       0
#> 419           0               89             684              0       1
#> 420           0              178             719              1       0
#> 421           0              314             674              2       0
#> 422           0              184             684              0       0
#> 423           1              101             704              0       0
#> 424           0              150             664              3       1
#> 425           0              133             784              2       0
#> 426           1              251             714              0       0
#> 427           0               49             704              0       0
#> 428           0               18             689              0       1
#> 429           0              150             664              0       2
#> 430           0              303             694              1       0
#> 431           1              297             719              0       0
#> 432           5              198             674              1       0
#> 433           0               60             804              0       0
#> 434           0              237             664              0       0
#> 435           1              250             729              1       0
#> 436           0              309             664              2       1
#> 437           1               50             689              0       0
#> 438           0              129             729              1       0
#> 439           0              219             719              0       0
#> 440           2              190             694              0       0
#> 441           0              123             739              2       0
#> 442           0              281             739              1       0
#> 443           0              251             709              1       0
#> 444           0              310             669              0       0
#> 445           0              106             664              3       0
#> 446           0               66             704              1       0
#> 447           0              236             669              1       0
#> 448           0              303             709              0       1
#> 449           0              204             669              0       0
#> 450           1              175             684              2       0
#> 451           0              296             689              1       0
#> 452           0              147             684              0       1
#> 453           0              124             709              0       0
#> 454           2              194             689              0       0
#> 455           0               22             684              0       0
#> 456           0              111             709              1       0
#> 457           0               14             674              1       0
#> 458           0              130             684              0       1
#> 459           1              156             664              0       0
#> 460           0              231             704              2       0
#> 461           0               19             684              2       0
#> 462           0              138             784              0       0
#> 463           0              191             719              0       0
#> 464           0              284             694              0       0
#> 465           0              210             689              0       0
#> 466           0              186             694              1       0
#> 467           0              223             664              0       1
#> 468           1              148             679              2       1
#> 469           0               71             694              1       2
#> 470           0               54             744              0       0
#> 471           0              177             679              1       1
#> 472           1              100             679              0       0
#> 473           0              229             739              0       0
#> 474           0              145             779              0       0
#> 475           0              123             679              1       0
#> 476           0              285             724              0       0
#> 477           0              258             689              0       0
#> 478           0              193             744              0       0
#> 479           2               17             719              0       0
#> 480           0              258             664              0       1
#> 481           0              178             714              1       0
#> 482           0               86             829              0       0
#> 483           0              261             679              0       1
#> 484           0              225             714              0       0
#> 485           0               55             699              0       0
#> 486           0              257             694              0       0
#> 487           0               23             699              0       0
#> 488           1              230             704              1       0
#> 489           0              209             684              0       0
#> 490           1              150             679              0       0
#> 491           2              101             664              0       1
#> 492           0               49             679              1       0
#> 493           0              205             689              0       0
#> 494           0               46             664              0       0
#> 495           0                6             794              0       0
#> 496           0              143             754              0       1
#> 497           1              249             684              1       0
#> 498           1              254             704              0       5
#> 499           0               73             669              2       1
#> 500           0              117             689              0       0
#> 501           0              203             664              0       0
#> 502           1              167             694              1       0
#> 503           0              130             754              2       0
#> 504           0               36             689              1       0
#> 505           1              194             709              0       0
#> 506           0              229             694              1       0
#> 507           0               72             769              0       0
#> 508           0               74             729              0       0
#> 509           0              298             744              1       0
#> 510           0              142             684              1       0
#> 511           0              228             724              2       1
#> 512           0               24             679              0       0
#> 513           0              113             669              0       1
#> 514           0              221             679              1       1
#> 515           0               52             679              0       0
#> 516           0              174             674              0       0
#> 517           0              257             689              0       0
#> 518          15              247             674              1       0
#> 519           0              147             704              2       0
#> 520           0              283             664              0       0
#> 521           0              188             704              0       0
#> 522           0              234             694              2       0
#> 523           0              125             684              1       1
#> 524           0              287             684              2       0
#> 525           0              224             739              1       1
#> 526           0              264             719              1       0
#> 527           0              276             664              2       1
#> 528           0              100             699              0       1
#> 529           0                7             724              2       0
#> 530           0              198             759              0       0
#> 531           0              258             679              1       0
#> 532           1              280             679              0       0
#> 533           0              251             694              2       1
#> 534           1               92             664              1       1
#> 535           0              170             704              0       0
#> 536           0               47             729              1       0
#> 537           0              267             699              0       0
#> 538           0              227             674              3       0
#> 539           0              163             814              0       0
#> 540           7               37             664              2       2
#> 541           0              172             664              0       0
#> 542           0               43             664              1       1
#> 543           0               23             709              2       0
#> 544           0               74             674              0       0
#> 545           0              289             679              2       0
#> 546           0              286             669              0       1
#> 547           0               17             689              1       1
#> 548           0               19             689              1       0
#> 549           0              291             684              1       0
#> 550           0              136             769              0       0
#> 551           0              122             704              2       0
#> 552           0              177             669              1       0
#> 553           0              214             684              0       0
#> 554           0              311             674              3       0
#> 555           1              127             664              0       2
#> 556           1                9             674              0       0
#> 557           1              194             689              0       0
#> 558           0              229             714              0       0
#> 559           0               85             704              1       0
#> 560           0               22             764              1       0
#> 561           3               71             674              1       0
#> 562           0               25             694              2       0
#> 563           0              197             674              1       1
#> 564           0              302             814              0       0
#> 565           0              308             669              2       0
#> 566           0              283             689              0       0
#> 567           0              160             739              0       0
#> 568           0               91             729              0       0
#> 569           3              115             714              0       0
#> 570           0              313             679              2       0
#> 571           0              304             689              1       0
#> 572           1               45             724              0       0
#> 573           0              119             759              1       0
#> 574           3              256             664              2       1
#> 575           0               44             709              0       0
#> 576           0              195             739              1       0
#> 577           0              125             699              2       0
#> 578           0              143             704              1       0
#> 579           0               65             749              0       0
#> 580           0              301             699              1       0
#> 581           0              284             674              0       0
#> 582           0               44             684              0       1
#> 583           2               48             664              3       5
#> 584           2               59             719              0       1
#> 585           0              280             689              0       0
#> 586           0               26             704              1       1
#> 587           0              144             679              1       0
#> 588           1              173             684              0       0
#> 589           0               97             684              1       0
#> 590           0              199             679              1       0
#> 591           1              261             674              0       1
#> 592           3               73             664              1       0
#> 593           0               47             669              0       1
#> 594           1              251             749              0       0
#> 595           0              295             699              0       0
#> 596           1               93             674              0       0
#> 597           1               74             689              0       0
#> 598           0              247             684              1       0
#> 599           0               51             694              0       1
#> 600           0               42             679              0       0
#> 601           1               84             699              0       0
#> 602           0              212             684              1       0
#> 603           0              258             709              0       1
#> 604           0              159             724              0       0
#> 605           0              104             729              0       0
#> 606           0              137             759              0       0
#> 607           1              285             689              0       0
#> 608           0               44             744              0       3
#> 609           0              178             779              0       0
#> 610           1               13             674              0       0
#> 611           0               20             704              0       0
#> 612           0               17             709              1       0
#> 613           0               49             719              0       0
#> 614           0              177             714              0       0
#> 615           1              117             674              1       0
#> 616           0               19             719              0       0
#> 617           0              207             749              1       0
#> 618           0              102             709              0       0
#> 619           0               68             744              0       0
#> 620           1              218             719              1       0
#> 621           0               19             669              0       3
#> 622           1              308             674              1       0
#> 623           0              284             679              1       0
#> 624           2               11             734              0       0
#> 625           0              234             704              0       0
#> 626           0              141             709              2       0
#> 627           0              223             669              0       1
#> 628           0              224             709              0       1
#> 629           0               14             739              0       0
#> 630           0              225             664              0       2
#> 631           0              243             679              0       0
#> 632           0              151             679              4       0
#> 633           0               45             709              0       0
#> 634           0              302             774              0       0
#> 635           0              194             689              1       1
#> 636           0              225             669              0       0
#> 637           0              310             694              0       1
#> 638           0              292             664              0       1
#> 639           0              102             714              0       0
#> 640           0              246             664              0       0
#> 641           0              103             704              0       0
#> 642           0              281             714              0       0
#> 643           0              156             704              0       0
#> 644           3              123             684              0       0
#> 645           0              200             689              0       0
#> 646           0              231             739              0       0
#> 647           0              281             734              0       0
#> 648           4              229             714              0       0
#> 649           0              284             724              0       0
#> 650           0               91             684              0       0
#> 651           1              152             694              0       0
#> 652           0              203             664              2       1
#> 653           1              149             689              1       0
#> 654           0              215             734              0       1
#> 655           0              261             749              0       0
#> 656           0              273             709              1       0
#> 657           0               41             754              1       0
#> 658           0              152             744              1       0
#> 659           0               81             714              0       0
#> 660           0              242             769              1       0
#> 661           0              308             689              0       0
#> 662           0              165             769              0       0
#> 663           3              257             679              0       0
#> 664           0               87             729              1       0
#> 665           0               17             679              1       1
#> 666           0              309             669              1       2
#> 667           1              196             679              0       0
#> 668           0               50             674              0       0
#> 669           0               98             689              1       0
#> 670           0              252             749              0       0
#> 671           0              253             684              2       0
#> 672           1               46             664              0       1
#> 673           0              253             669              0       0
#> 674           0              270             729              1       0
#> 675           1               11             719              0       0
#> 676           0               46             744              1       0
#> 677           0              248             699              0       0
#> 678           0              285             744              0       0
#> 679           0              251             684              4       0
#> 680           0               22             664              0       1
#> 681           0              233             724              0       1
#> 682           3              230             664              0       0
#> 683           0              230             769              1       0
#> 684           1              113             684              0       0
#> 685           0              251             724              0       0
#> 686           0              136             779              0       0
#> 687           2              302             679              1       0
#> 688           0              154             709              0       0
#> 689           0              227             664              0       1
#> 690           0              311             759              0       0
#> 691           1               93             714              0       1
#> 692           0              312             699              5       0
#> 693           0              308             689              0       0
#> 694           0              286             669              3       0
#> 695           1              148             689              0       0
#> 696           0                2             719              0       0
#> 697           0               75             714              0       0
#> 698           0              278             764              1       0
#> 699           0              114             664              3       1
#> 700           0              236             704              0       0
#> 701           0               49             694              1       0
#> 702           0               27             664              3       1
#> 703           0              311             694              0       0
#> 704           0              129             669              2       0
#> 705           1               45             674              0       0
#> 706           0               16             729              0       0
#> 707           0               73             694              0       0
#> 708           0              173             774              0       0
#> 709           1               36             714              0       0
#> 710           0                1             754              0       0
#> 711           0               88             679              0       2
#> 712           0               72             684              0       0
#> 713           0               48             669              1       0
#> 714           0               15             799              1       0
#> 715           0               62             749              1       0
#> 716           0              134             774              0       0
#> 717           0               78             824              0       0
#> 718           2              135             684              0       0
#> 719           1               16             754              1       0
#> 720           0              172             734              0       0
#> 721           0              276             679              0       0
#> 722           0              315             699              1       0
#> 723           0               74             679              4       1
#> 724           0               91             709              0       0
#> 725           0              101             769              1       0
#> 726           0               44             719              4       0
#> 727           1               96             714              1       0
#> 728           0              242             739              0       0
#> 729           1              305             689              0       0
#> 730           2              102             714              0       0
#> 731           0               64             719              1       0
#> 732           0               99             709              0       1
#> 733           0              251             684              2       0
#> 734           0              112             704              0       0
#> 735           0              126             694              1       0
#> 736           0              310             664              2       0
#> 737           1              145             664              2       1
#> 738           0              278             749              0       0
#> 739           1              181             694              0       0
#> 740           0              239             674              1       0
#> 741           1              261             679              2       0
#> 742           0              199             694              0       0
#> 743           0               40             724              0       0
#> 744           0              310             674              1       0
#> 745           0               41             749              2       0
#> 746           1               90             664              1       0
#> 747           0               78             709              1       0
#> 748           0               51             699              0       0
#> 749           0              104             664              0       0
#> 750           2              232             699              0       0
#> 751           0               22             764              0       0
#> 752           0               49             694              0       0
#> 753           0              232             674              2       0
#> 754           2              124             674              0       0
#> 755           6              144             689              0       0
#> 756           0              198             699              0       0
#> 757           0               36             749              0       0
#> 758           0              203             684              0       0
#> 759           1              309             739              0       0
#> 760           0              103             689              0       0
#> 761           0               67             669              0       0
#> 762           0              201             689              1       0
#> 763           0              297             779              0       0
#> 764           0              256             714              0       0
#> 765           0              124             694              1       0
#> 766           0              120             709              0       0
#> 767           0              227             674              1       1
#> 768           0              199             734              0       0
#> 769           0               72             674              1       1
#> 770           0               49             679              2       2
#> 771           0              247             729              0       0
#> 772           1              200             724              3       0
#> 773           0              169             719              0       0
#> 774           0               92             729              0       0
#> 775           1              128             709              1       1
#> 776           0              224             709              0       0
#> 777           0              289             759              1       0
#> 778           0               69             674              5       2
#> 779           0               48             719              0       0
#> 780           0               41             759              0       0
#> 781           0              279             749              0       0
#> 782           0               32             679              1       1
#> 783           0              291             674              0       0
#> 784           0              247             689              1       0
#> 785           1              226             674              0       0
#> 786           0              192             699              1       0
#> 787           0               64             729              0       0
#> 788           0              306             714              1       0
#> 789           0              282             669              0       0
#> 790           1              100             704              0       0
#> 791           2              120             664              0       0
#> 792           0               21             704              3       0
#> 793           0               22             769              1       0
#> 794           0               45             684              0       0
#> 795           0               25             669              5       1
#> 796           0               47             719              0       0
#> 797           0               73             694              0       0
#> 798           5              172             679              0       0
#> 799           0              198             734              0       0
#> 800           0               28             684              1       0
#> 801           0               10             714              0       0
#> 802           0              310             719              0       0
#> 803           0              258             769              0       0
#> 804           0              309             669              0       0
#> 805           0              110             669              0       0
#> 806           0              102             664              1       0
#> 807           2              125             674              2       0
#> 808           0               48             674              1       1
#> 809           0              223             759              0       0
#> 810           0              251             694              0       0
#> 811           0              276             724              0       0
#> 812           0              209             694              0       0
#> 813           0              256             704              0       0
#> 814           0              216             674              1       0
#> 815           0              277             694              0       0
#> 816           0              139             669              2       0
#> 817           1               53             674              1       0
#> 818           0              243             774              0       0
#> 819           0              309             709              2       0
#> 820           0              127             664              0       1
#> 821           0               19             719              0       0
#> 822           0              246             749              0       0
#> 823           0              202             694              0       0
#> 824           0               34             679              0       0
#> 825           0              180             784              0       0
#> 826           2              115             669              1       0
#> 827           0               31             664              1       1
#> 828           0               51             734              3       0
#> 829           2              177             684              1       0
#> 830           0              113             694              1       0
#> 831           0              268             704              0       0
#> 832           0              270             714              0       0
#> 833           0              303             674              1       0
#> 834           0              254             674              0       1
#> 835           0              300             669              0       1
#> 836           0               16             689              1       1
#> 837           0              150             664              0       1
#> 838           2              280             719              0       1
#> 839           0              185             694              0       0
#> 840           0               17             724              0       0
#> 841           0              261             739              2       0
#> 842           0              273             739              0       0
#> 843           0              275             774              0       0
#> 844           1              250             674              1       3
#> 845           0              195             664              0       1
#> 846           1              152             669              1       0
#> 847           0              286             729              0       0
#> 848           1              126             709              0       0
#> 849           0              309             679              1       0
#> 850           3              287             669              1       0
#> 851           0               74             669              1       0
#> 852           1               46             684              2       0
#> 853           0               39             689              2       1
#> 854           1                4             674              1       0
#> 855           0               74             744              0       0
#> 856           0              128             689              0       0
#> 857           0               49             699              1       0
#> 858           0              164             779              2       0
#> 859           0              286             669              2       0
#> 860           2              252             679              0       1
#> 861           0              282             694              1       0
#> 862           0              180             689              3       0
#> 863           0              177             684              0       0
#> 864           0               26             759              0       0
#> 865           0              149             714              2       0
#> 866           0              232             684              0       1
#> 867           2              309             664              5       0
#> 868           0              260             669              0       0
#> 869           1              246             664              1       0
#> 870           0               19             669              0       1
#> 871           0               14             734              0       0
#> 872           0              284             724              0       0
#> 873           0              197             699              0       0
#>     revol_bal total_acc total_rec_int recoveries last_pymnt_d last_pymnt_amnt
#> 1        2765        13        821.72       0.00           16          122.67
#> 2       21470        38        979.66       0.00           20          926.35
#> 3        7869        18       2705.92       0.00           21        15813.30
#> 4       21929        35       1340.50       0.00           17        10128.96
#> 5        8822         6       1758.95       0.00           28         7653.56
#> 6       87329        27       1393.80       0.00           30        15681.05
#> 7         826        15       1538.51       0.00           14        14618.23
#> 8       10464        23        998.97       0.00            6         1814.48
#> 9        7034        18        939.58       0.00            2         4996.24
#> 10      37828        24        175.16       0.00           24          965.36
#> 11      14052        27       4351.98    1618.90           28          471.70
#> 12      51507        24       1939.02       0.00           28        17093.51
#> 13       7722         9       1036.10       0.00           12         3480.17
#> 14      20862        19       1224.23       0.00           36        20807.39
#> 15      10711        27        387.22       0.00           23        18004.90
#> 16       9568        19        540.49       0.00           27         8251.42
#> 17      31682        31       4133.21       0.00           24        25234.24
#> 18      19108        19       3498.09       0.00           37        12322.68
#> 19      22780        22       5320.39       0.00            8        12486.30
#> 20       9441         5       1117.45       0.00           16          169.17
#> 21       8563        16        711.71       0.00           16          186.60
#> 22       1058         6        653.60     368.37            2          146.16
#> 23       2269        17       1631.72       0.00            6         3577.50
#> 24      42469        33       1471.57       0.00           17        15481.30
#> 25      12203        42        705.08       0.00           30           17.88
#> 26      17003        21       1257.97       0.00           16          455.59
#> 27       5157        20       3402.05       0.00           32          565.09
#> 28      24799        21       2791.73       0.00           17          701.01
#> 29      20462        39       1349.67       0.00            2         7884.96
#> 30      22519        72       6126.37       0.00           12          581.58
#> 31      10467        34        806.10       0.00           35          425.19
#> 32      20374        15       6285.81       0.00            6        12548.30
#> 33      34974        19       2253.17       0.00            4        25897.53
#> 34       5449         7       1053.86       0.00           37         3755.50
#> 35      21374        12       4889.01       0.00            9         1407.70
#> 36      19077        63       1780.59       0.00            7         2815.60
#> 37      38623        28       2871.05       0.00           31          253.78
#> 38       7558        26       1918.82       0.00           24        13580.93
#> 39       7158        24        613.85       0.00            8            2.08
#> 40       1581        44       3473.23       0.00           28        17274.03
#> 41       5938        43       1694.32       0.00           25         2602.09
#> 42       9195        24       1052.88       0.00            5         4768.52
#> 43        732        22        713.30       0.00           32          266.79
#> 44       8137        18       1610.03       0.00           16          350.59
#> 45       6959        33       1316.89       0.00           24         6961.97
#> 46      43413        46      11607.45       0.00           35        16687.04
#> 47      14277        46       9358.91       0.00            9        17952.27
#> 48       9051        25        398.05       0.00            4         6206.89
#> 49       6664         7       1255.15       0.00           16          312.89
#> 50      11834        48       1975.88       0.00           14          367.67
#> 51      24882        14       3976.27       0.00           25         7662.31
#> 52      15646        21       2316.39       0.00            7        12764.01
#> 53      39055        27       7086.13    5558.60           31          794.21
#> 54      22551        21       4206.60       0.00           26          764.11
#> 55      12718        24       2770.73    1495.90           18          331.96
#> 56      31200        35       2592.85    9915.00           14          465.27
#> 57      11947        37        918.76       0.00           16          142.25
#> 58      17771        34       2165.04       0.00           11        10797.99
#> 59       8799        24       5631.26       0.00           37        13019.99
#> 60      21831        23       3903.77       0.00            7        23456.38
#> 61       3717        37         98.98       0.00            2          956.75
#> 62      19339        18       2033.11       0.00            9          752.86
#> 63       8284        16       5019.82       0.00           18          518.35
#> 64      44807        19       2450.73       0.00           17          633.73
#> 65      24723        32       3520.28       0.00           13         2536.24
#> 66      12958        33       1289.95       0.00           18         5756.97
#> 67      19029        33        657.93       0.00           17        13308.37
#> 68      20594        24       1868.46    2278.59           20          618.46
#> 69      31035        38       3157.54       0.00           33          977.00
#> 70       7340        33       1482.58       0.00           17        23774.59
#> 71      12272         9       2653.90       0.00           32         1464.61
#> 72       6118        36        757.52       0.00           20         9795.24
#> 73      15883        19       3412.00       0.00            5        14388.69
#> 74       5552        10        627.56       0.00           16          156.25
#> 75       7518        11       1232.91       0.00           16          167.11
#> 76       9488        31        513.41       0.00           33         8826.61
#> 77       4851        23       2045.97       0.00           16          612.85
#> 78      41593        47       1241.28       0.00           38         2244.30
#> 79       3172        19        463.54       0.00           30         9219.99
#> 80      11141        69       1257.10       0.00           25          252.27
#> 81      11008        22       1096.73       0.00           14         8659.70
#> 82       7758        47       1771.10       0.00           34         8073.57
#> 83       6952        23        449.63       0.00           28         2275.17
#> 84       7302        17       1041.44     723.81            7          257.67
#> 85      59237        44       2307.02       0.00           24        12793.18
#> 86       4838        20        736.46       0.00            3         1560.65
#> 87       7179        12       3048.38    1165.01           31          284.51
#> 88      28475        31        709.15       0.00           23        33262.93
#> 89      10987        13       3104.20    4382.00           37          597.17
#> 90       9747        22       1213.63       0.00           32          849.93
#> 91       9786        37       1842.63       0.00           31         8529.37
#> 92        502        24        399.02       0.00           28         2491.31
#> 93       8280        10       2548.52       0.00           29         3036.60
#> 94      34250        14       2205.37       0.00           16          450.79
#> 95       8353        11        753.10       0.00           16          187.74
#> 96      12703        31        415.46       0.00            1        14450.25
#> 97      10495        25       1277.57       0.00           21         7172.05
#> 98       8052        20       1833.28       0.00            9          329.46
#> 99       1294        16        836.37       0.00           15         1743.34
#> 100     13993        54       4312.22       0.00            4        33789.51
#> 101      3045        27       1393.48     404.32           31           47.01
#> 102      1960        10       1282.37       0.00           19         1393.81
#> 103     11763        25       3505.01       0.00           16          732.08
#> 104      6727        42        665.53       0.00           17         9001.57
#> 105      4091        16       2090.02       0.00           25         2607.83
#> 106      9196        23        790.51       0.00            1        15662.52
#> 107     26438        30       4725.16       0.00           16          732.18
#> 108     12105        19        881.94    1054.97           27          377.36
#> 109      4414        10       1835.59       0.00           16          273.24
#> 110     17866        28       1542.44       0.00           22         1858.16
#> 111     24365        32       3390.24       0.00            9          503.38
#> 112     20769        33       2146.66     912.41           20           25.00
#> 113     25225        27      10776.17       0.00            3        17616.76
#> 114      1061        13        401.09       0.00           25          983.41
#> 115     44174        62       7879.31       0.00           35        13092.73
#> 116     47300        26       5803.98       0.00           18        20564.41
#> 117      7763        18        590.16       0.00            8         1106.34
#> 118     33231        32       2965.41       0.00           16          777.17
#> 119     11331        29       1826.44       0.00           35         1170.18
#> 120      2469        18        462.97       0.00           16           82.26
#> 121      6452        64        198.14       0.00           17         6999.45
#> 122     40405        37       3417.20    1650.00           22           50.00
#> 123     23315        32       5092.10   13403.00            2          794.21
#> 124     16173        20       1435.97       0.00           37         7587.50
#> 125     15986        25       4140.43       0.00           32          560.70
#> 126      8192        14       2151.65       0.00            5         8450.13
#> 127      7933        14        719.46       0.00            4         6611.94
#> 128     20672        22        786.83    1278.31           17          522.16
#> 129      8775        29        381.76       0.00            7         6678.01
#> 130      1844        14        850.38       0.00            4         8373.67
#> 131      9053        30        873.74    2729.00           21          350.00
#> 132      2212        16        463.98       0.00           24         4025.61
#> 133     12329        13       2919.63       0.00            5        12096.11
#> 134     15448        26       3047.28       0.00           24        22713.41
#> 135     34764        21        840.28       0.00           12         1892.09
#> 136     24345        49       2766.25       0.00           33        18930.43
#> 137      7610        26       1241.66     611.63           14          289.10
#> 138     21151        17       2706.17       0.00            6         3065.33
#> 139     20724         6       1978.36       0.00           15          563.31
#> 140     92939        26       2242.44       0.00           33        28360.86
#> 141     11221        17       1083.38       0.00           38         1966.79
#> 142     12503        68        467.39       0.00           11         3930.74
#> 143     21039        25       3429.04       0.00            3         7176.79
#> 144     16659        25        431.39       0.00           23        12758.61
#> 145      5739        14       2579.46       0.00            8         4826.13
#> 146      8485        12        743.00       0.00           22         1030.14
#> 147     29929        20       3657.70       0.00           38         3250.36
#> 148     24399        30       1502.50       0.00            9          638.18
#> 149      6069        26       1909.67       0.00            4           35.74
#> 150     27092        34       1353.07       0.00           17        20052.89
#> 151     10760        19       1237.50       0.00            8         4458.64
#> 152      2380        17        614.48       0.00           30          163.73
#> 153      6378        11        852.44       0.00           25         1739.41
#> 154      5890        26        890.66       0.00           16          246.61
#> 155      5426        15        945.52     181.05            6          166.05
#> 156     12854        26        648.38       0.00           31         3990.79
#> 157     10009         7       1122.94       0.00           18         7026.68
#> 158     11038        26       6073.27    1737.71           22          497.96
#> 159      2193         7       8771.29       0.00            9         7315.86
#> 160     18002        23       4339.77       0.00            3        17440.21
#> 161      6297        12       3892.58     450.00           12          338.75
#> 162     12632        26       2510.05       0.00           16          514.53
#> 163     15686        20        937.24       0.00           19          320.89
#> 164     20014        52       3064.65       0.00           29         6640.18
#> 165     22755        32       1128.45       0.00            1        33804.36
#> 166      8898        10       1155.96       0.00            5         4421.40
#> 167      4419        39        680.19       0.00            8         1601.37
#> 168     28822        38        899.80       0.00           36         8908.46
#> 169      5560        22        555.12       0.00            9          197.30
#> 170     10130        16       3924.30       0.00           16          567.76
#> 171     15916        31        973.89       0.00           16          248.55
#> 172     18662        20       2316.26       0.00           16          609.40
#> 173      4110        16         76.18       0.00           20         1960.21
#> 174     21922        38        700.54       0.00            7        11893.96
#> 175     27242        30        398.86       0.00            2         2304.88
#> 176     18255        12       4466.97       0.00           16         1604.44
#> 177      7763        21        269.20       0.00            7         4465.28
#> 178     35994        39       3012.38       0.00           16          750.95
#> 179      4665        49       1936.88    1415.92           33          381.71
#> 180      8339        40        972.86       0.00            7         7668.51
#> 181     14914        17       1162.62       0.00           28         8796.87
#> 182     15206        32        740.21       0.00           15         3821.15
#> 183     16424        38       5546.32       0.00            2        20806.23
#> 184     28738        22       6105.97       0.00            5        18943.68
#> 185     34746        33       1891.22       0.00           35         1420.01
#> 186     16417        35       3512.78       0.00           16          764.85
#> 187     28430        25       2216.56       0.00           16          504.87
#> 188     20125        25       2271.53       0.00           29         4486.00
#> 189     15996        19        358.20       0.00           36          166.05
#> 190      8768        21       2005.47       0.00            5         8216.57
#> 191     15448        53        133.97       0.00           10        15154.39
#> 192     22712        19        780.27       0.00           16            0.59
#> 193     12187        13       2833.15       0.00           19         5038.26
#> 194     27946        25      12716.46       0.00           29        23556.83
#> 195      4016        32        378.02       0.00           19          455.17
#> 196      9332        10       1882.51       0.00           16          385.69
#> 197      5387        25        213.49       0.00           20         7233.30
#> 198      9560        27       1192.88     681.01            5          321.74
#> 199     31610        22       7873.77       0.00           16          680.03
#> 200     15837        39       2185.35       0.00           25         4848.59
#> 201     15676        23        838.46       0.00           16          143.26
#> 202      7204        30        662.63       0.00           16          185.05
#> 203      7411        38       1617.50    1749.94            4          440.37
#> 204     13447        18       1104.38       0.00           16          308.54
#> 205     18471        18       9247.17       0.00           16        15175.22
#> 206     42605        30        406.39       0.00            4        11231.02
#> 207      6666        14        205.53       0.00            1         9597.78
#> 208     11907        25        555.91       0.00           13            0.29
#> 209      8516        14       1436.70       0.00           11         3135.21
#> 210     10580        15       2140.88       0.00           21        11101.71
#> 211     19182        41       1746.74       0.00           29         4869.49
#> 212      5278        30       1910.70     573.30            3          480.55
#> 213     12669        18       2112.17    1746.80            2          351.73
#> 214      4994        21        312.19       0.00           23          444.13
#> 215     18114        17       1018.72     727.70           17          320.23
#> 216      1801        31       1670.83       0.00            2        12930.20
#> 217     31640        28       1090.83       0.00           17        27333.39
#> 218     41266        13       1994.47       0.00           16          555.52
#> 219      8770        35         89.72       0.00           23         4936.69
#> 220      3394        23       1341.22       0.00           16          167.10
#> 221     11210        12        314.02       0.00           17         5178.59
#> 222      8275        30       2494.33       0.00           31         8793.00
#> 223     16440        21       2733.79       0.00           34         8646.13
#> 224     10140        15        329.56       0.00           38          292.89
#> 225      5153        40       1225.47       0.00           16          158.38
#> 226     14446        26       1433.06       0.00            6          957.34
#> 227      5814        18        544.01       0.00            5            4.12
#> 228     59126        30       3123.77       0.00            7        15283.34
#> 229     25824        39        623.23       0.00           14         7236.91
#> 230     21449        39      10165.42       0.00           13        16469.98
#> 231     22849         6       4447.15       0.00            9          755.16
#> 232      8211        23        900.35       0.00            9          338.00
#> 233      6920        30       2177.61       0.00           28        10636.64
#> 234      1252         7       1939.29       0.00           16          332.44
#> 235     39840        25       8846.88       0.00           32          153.79
#> 236     82471        29       2244.56       0.00           22         6655.09
#> 237      4727        22        623.33       0.00           31         3196.17
#> 238     18982        20       3358.39       0.00           19         1752.75
#> 239      4008        13       1022.58       0.00           16          261.01
#> 240     16036        38       1246.12       0.00           13            3.93
#> 241      6846        11        593.37     343.29            5          160.87
#> 242     13450        22        355.05     163.29           37           79.12
#> 243     12370        16       1177.49       0.00            3         4443.59
#> 244      5633        40        399.56       0.00           33         4761.92
#> 245     23324        22       3466.95       0.00           16          652.39
#> 246      7047        37        105.88       0.00           10         6143.31
#> 247      4595        36       1100.52       0.00           12         3084.26
#> 248     18385        29       3521.69       0.00           25        10034.99
#> 249      6198        12        656.29       0.00           20         1000.45
#> 250     25166        19       6022.41       0.00           16          968.38
#> 251      5832        19         79.51       0.00            1         4782.04
#> 252     11687        31       1041.83       0.00            9            1.31
#> 253      9374        20       8644.93       0.00           26        10596.15
#> 254     15347        38       1778.36       0.00           27        23809.41
#> 255     28645        33       1612.56    1862.71           36          930.37
#> 256     26534        35       5368.84       0.00            6        13829.07
#> 257      8964        32       5542.45       0.00            5        17455.86
#> 258      5951        47        976.04       0.00           13            3.92
#> 259     16876        30       1315.67       0.00           30        15597.75
#> 260     49831        30       3067.13    2192.67           14          492.66
#> 261     17336        41       5917.64       0.00           16        10069.02
#> 262     12242        21        984.87       0.00           18         6698.36
#> 263      8358        22       1952.73       0.00            3         3225.00
#> 264      5940        24        964.52       0.00           30        10696.74
#> 265      6873        10       1274.69       0.00           18         9160.10
#> 266       476        13       1666.46       0.00           16          224.93
#> 267     11472        30       3790.23       0.00           16          458.68
#> 268     21876        87       4285.10       0.00           14        26196.56
#> 269      3044        11       1194.57     722.98           36          512.60
#> 270     34612        20        545.58       0.00           23        20636.72
#> 271     18828        26       2799.05       0.00           32          767.54
#> 272      4916        11       3138.81       0.00           28          503.50
#> 273      6022        27       8466.02       0.00            9          462.46
#> 274      4192        19        847.13       0.00           31           14.34
#> 275      7259        24        682.38       0.00           35          627.42
#> 276       291        19       2944.09       0.00           16          439.99
#> 277      5116        13       1612.06       0.00            8         5127.50
#> 278      2976        13       1243.31       0.00           18         7963.75
#> 279      4792        16       1007.31       0.00           29         1430.41
#> 280     19100        31       2135.13       0.00           16          559.80
#> 281      3862        40       2652.44    3392.11           11          426.81
#> 282     35492        26       6337.15    1843.00           12          894.69
#> 283     14435        34       2842.86       0.00           12         1241.36
#> 284      7696        30        591.80       0.00           15         1467.90
#> 285     12502        18       4231.94       0.00           29         9148.19
#> 286      5549        40       1141.62       0.00           20        14580.13
#> 287     19565        58        699.73       0.00           27        14132.41
#> 288     13663        30       1727.25       0.00           12         2571.89
#> 289     40023        24       7318.95       0.00           16         1177.11
#> 290     42741        42        328.44       0.00           17         7075.32
#> 291     13519        65       2816.15    1490.66           18          664.20
#> 292     27510        39        488.97       0.00           23        14523.70
#> 293     22910        30       2413.66       0.00           37        10720.67
#> 294      3971        34        789.56       0.00           16          124.20
#> 295     14877        34       1293.73       0.00            6          282.79
#> 296      1335        10       1304.90       0.00            6         1395.58
#> 297     10223        21        886.41       0.00           36        13994.40
#> 298      4842        30         79.21       0.00           23         6814.82
#> 299      2773        33       1329.23       0.00           19         1394.73
#> 300     34886         7       5686.53       0.00           19         6541.67
#> 301      9290        36       1884.72       0.00           11          538.18
#> 302     21478        28       1512.93       0.00           16          353.29
#> 303      5049        20        237.20       0.00           17         5304.94
#> 304      8610        33        898.31       0.00           22         2198.15
#> 305      7056        45        818.68       0.00           18         5579.98
#> 306     50634        20       7124.27    2956.20            7          924.87
#> 307      3855        14       1490.88       0.00           25         2939.20
#> 308      7613        44        594.55       0.00            1         8717.10
#> 309     23785        22        499.91       0.00            9          360.88
#> 310     13435        30        256.41       0.00           17         8758.05
#> 311     23125        29        606.02     903.80           20          350.44
#> 312     20086        13       1802.84       0.00           15         3093.70
#> 313     59351        36       5615.42       0.00            3        13326.69
#> 314      5248        22       1290.69    1542.53           33          637.58
#> 315     16709        27        210.04       0.00           14         2557.35
#> 316     12706        40        576.05       0.00           20          317.24
#> 317     13883        46          0.00    3595.36           23          100.00
#> 318     38681        30       2360.70       0.00           12         5634.68
#> 319      1725        23        183.17       0.00           11         1575.63
#> 320     16328        20       3062.61       0.00           16          474.59
#> 321      7002        18       2286.96     939.80           37          470.91
#> 322     13186        28       2108.50       0.00           16          497.74
#> 323      7804        19        480.55       0.00           30         9237.00
#> 324      8773        14        694.33       0.00           21         3362.68
#> 325     35575        33       9674.07       0.00           16         1248.88
#> 326      2015        16        508.88     710.94           17          307.00
#> 327      1869         8        501.46       0.00           35           61.78
#> 328     13676        43      10024.70       0.00           16         1252.26
#> 329     34441         6       1463.75       0.00           16          235.17
#> 330     13545        22       1551.00       0.00           16          265.52
#> 331      2275        11       1573.87       0.00           25           52.76
#> 332     29336        44       2332.77     298.58           35          564.21
#> 333     11694        49       2926.35       0.00            5        16060.01
#> 334      5339        22        764.00       0.00            5         1757.39
#> 335     13001        38       1465.47       0.00           31         4627.09
#> 336     12638        22       1445.36       0.00           28         9899.41
#> 337      5727        32       9213.70       0.00           35          599.96
#> 338      7055        22       9167.61       0.00           15          586.93
#> 339      6739        32       3594.18       0.00           35          295.00
#> 340      2113        14        204.60       0.00           16           61.25
#> 341     10848        11       1601.13       0.00           16          577.97
#> 342      9190        24       2786.95       0.00            6         5911.92
#> 343     53593        34       4391.60       0.00            9         2188.83
#> 344      5576        42       3136.65       0.00           16          504.23
#> 345     23127        35       6549.44       0.00           21        21917.85
#> 346     22088        24       2400.56       0.00            6          563.63
#> 347     11247        25       1097.14     395.59           15          257.39
#> 348     75205        11       1268.93       0.00           36        13913.10
#> 349      3048        18       2348.54       0.00           31        12273.95
#> 350     39669        22       5127.93       0.00            5        19264.09
#> 351      1435        17       1756.39       0.00           16          382.42
#> 352     11718        41       2787.93       0.00           37        12281.60
#> 353     15224        13        910.47       0.00           12         2910.62
#> 354      9911        29       3244.12       0.00           14        15187.95
#> 355      7764        24       1039.50       0.00           19         1488.73
#> 356     45269        48       1734.71       0.00           30        16598.18
#> 357     20061        27        686.86       0.00            4         8686.53
#> 358      6151        22        730.58       0.00           19         1291.84
#> 359      1373        26       1259.71       0.00           18           27.49
#> 360      6597        61        111.45       0.00           10         8524.84
#> 361     13034        18       2602.26       0.00           19         6039.47
#> 362      4752        24        677.35       0.00            5         3240.43
#> 363     32224         4       2492.94       0.00           25         6735.02
#> 364      3810        26        898.25       0.00            9          304.07
#> 365      4354        26       5718.79       0.00           16          715.51
#> 366     21070        23       2195.48       0.00           16          477.93
#> 367      8700        22       2370.90       0.00           29         3494.09
#> 368     22869        53        388.04       0.00           33         4090.09
#> 369     10706        23       1038.63       0.00            9          390.07
#> 370      4929        11        296.53       0.00           11         4132.68
#> 371     71192        52       9623.24    2930.67            8          749.65
#> 372     37068        26       1250.88       0.00           16          451.41
#> 373     43536        36       1418.35       0.00           16          331.03
#> 374      8425        21       2187.53       0.00           16          338.75
#> 375      3886        43         94.23       0.00           27         3362.06
#> 376     16758        26       1858.19       0.00           16          331.30
#> 377     22022        44       2823.76       0.00           28        18655.79
#> 378      6352        23       1404.24       0.00           12         3041.24
#> 379      9419        22       5410.49    1275.31           38          387.96
#> 380     11037        22       2429.93       0.00           34        10420.20
#> 381     11143        29       1711.61       0.00           19         2632.94
#> 382     55427        48       3442.34       0.00            6         5185.30
#> 383      5707        18       1077.37       0.00           12         2411.52
#> 384      8588        11       2195.48       0.00           16          477.93
#> 385     13025        27       1123.16       0.00           28         8034.84
#> 386      8297        24       2189.39       0.00           16          340.61
#> 387     29719        24       1107.62       0.00            4        21278.15
#> 388      3114        15       1620.57       0.00           21          259.60
#> 389     13028        16        183.46       0.00           10        15208.44
#> 390     42613        31       1138.94       0.00           14        17199.22
#> 391     15568        19        437.46       0.00           16           67.49
#> 392     23959        46       3051.11       0.00           14           44.10
#> 393     10292        21       4343.57     907.31           31          523.50
#> 394      3731        28        863.92    1203.00           31          195.81
#> 395     11449        15        511.46       0.00           16          152.92
#> 396     44737        18       2376.95       0.00           13            4.32
#> 397     20218        50       1884.07       0.00           13            1.16
#> 398     95821        27       4053.67       0.00           28        20260.34
#> 399     19434        15       1489.25     562.25           33          353.60
#> 400      8619         8       2665.38       0.00           25            2.71
#> 401     14360        39        261.59       0.00           10        21293.03
#> 402     12989        13       2161.35       0.00           16          504.87
#> 403     31110        24       3389.00       0.00           18        14225.21
#> 404     10919        25       1382.90       0.00           32         1026.03
#> 405     15292        20       4179.61       0.00           28        18520.11
#> 406     19934        18       3680.39    1689.53           21          388.12
#> 407      5025        35        893.19       0.00           15         3143.70
#> 408      4715        16       1158.70       0.00           38         1262.65
#> 409      4903        16       2675.63       0.00           16          380.12
#> 410     21710        29      10095.78       0.00           25        20192.20
#> 411     29581        33       2557.44       0.00           16          765.77
#> 412      2978        20        771.96       0.00           37          142.84
#> 413     24801        44       1196.79       0.00           28         2821.77
#> 414       928        11       1390.06       0.00           25         2130.68
#> 415     36333        49       7721.38       0.00           32        10386.23
#> 416    158563        31       3531.24       0.00           13           60.45
#> 417     13356        44        917.70       0.00            1        34484.92
#> 418     17044        23       4464.31       0.00           30        29287.70
#> 419      5787        41      10118.45       0.00           32        13961.76
#> 420      4280        16       4598.51       0.00           13           23.29
#> 421     24127        28       7004.45       0.00           28        29152.85
#> 422     16649        18       2082.21     724.80           25          563.31
#> 423     13908        16       1053.24       0.00           15         3910.18
#> 424      8195        35       2113.20       0.00           38         2465.78
#> 425      6138        34        931.01       0.00           18         8262.68
#> 426     12740        25       4696.29       0.00           22        12275.48
#> 427     28820        28        522.34       0.00           23        14190.96
#> 428     11495        51       1496.40       0.00           14        14903.13
#> 429      2328        13       1072.84       0.00           16          160.27
#> 430     38235        38       5258.68       0.00            6        15787.84
#> 431     42125        65       1136.02       0.00           21         7101.01
#> 432      8187        18        462.02       0.00            7         5416.63
#> 433      9867        44       1304.79       0.00           16          342.03
#> 434      8800         8       2335.82       0.00           34          255.90
#> 435      3236        66        551.57       0.00           16          114.74
#> 436     20298        38        696.50       0.00           23        23773.45
#> 437      2442        30        935.03       0.00           13            0.73
#> 438      7256        15       1794.76       0.00           14        11821.39
#> 439     46623        23       1389.38       0.00           19         1845.03
#> 440     13859        19       9777.88       0.00            9         1252.27
#> 441     16210        28       1599.05       0.00           32          737.93
#> 442     15801        27       1901.35       0.00           16          686.50
#> 443     23294        30       1350.85       0.00           16          315.59
#> 444     14815        27       2936.80       0.00           15         7512.72
#> 445      3510        10       1183.50     469.97           18          212.70
#> 446     40781        24       9107.09       0.00           22         8412.73
#> 447     43074        19        822.22       0.00           24         4168.36
#> 448      7971        19       1492.33       0.00           33          546.77
#> 449      4040         8       1334.69       0.00            9          177.03
#> 450     24977        89        702.65       0.00           37          120.00
#> 451     50651        19       3905.76    1322.09           34          691.84
#> 452     12508        22       3488.25       0.00           36          417.35
#> 453     27910        32       2374.00       0.00           13            1.37
#> 454     13171        35       1177.67       0.00           13            2.35
#> 455     10099        35        951.51       0.00           25         1755.70
#> 456     14467        32       1666.70       0.00            9         1203.28
#> 457     26360        59       1093.98       0.00           16          169.41
#> 458     12710        10       1383.19       0.00            2         7918.48
#> 459     10734        10       6454.15       0.00           37        15973.26
#> 460      7280        27       3130.84       0.00           31          772.17
#> 461     13616        40        656.28       0.00            4          786.81
#> 462      8994        23        413.65       0.00           14         4351.38
#> 463     32983        15       2687.32       0.00            5        14945.40
#> 464     36206        30       1279.82       0.00            7        10731.88
#> 465      9075        12       3980.75       0.00           16          386.61
#> 466     31789        14        824.09       0.00           16          191.23
#> 467     10572        19        823.09       0.00           16          119.04
#> 468      7657        27       3634.71       0.00           17        27193.87
#> 469     41039        38       2196.30       0.00           16          450.31
#> 470      6643         7        753.65       0.00           21         7226.83
#> 471     19403        24       6785.08       0.00            6        14284.88
#> 472     10802        19       6890.87       0.00           13          524.89
#> 473     20052        36       2427.17       0.00           25         6626.93
#> 474      8291        34       1266.66       0.00           27        24589.89
#> 475     78106        28       1397.28       0.00            2         9110.32
#> 476      4788        20        844.34       0.00           34         4724.35
#> 477     14858        13       3602.03       0.00           38         2059.57
#> 478     26355        23        939.54       0.00            5         2143.17
#> 479      3218        40        702.15       0.00           24         6050.46
#> 480      7065        27       1411.99       0.00           16          260.95
#> 481     16701        24       1355.71       0.00           38         1596.04
#> 482      2721        25        766.48       0.00            3          205.22
#> 483     14236        52       2593.35    1846.42           14          482.56
#> 484     41206        20       3854.57       0.00           11        24964.77
#> 485     25020        22       3406.31       0.00           15         1225.38
#> 486     30763        13       2388.44       0.00           19         5081.44
#> 487      6477        18       3263.93       0.00           31         9288.35
#> 488      2204        26       4902.54    2110.95           37         1059.54
#> 489     24236        40       2228.37       0.00           35          867.94
#> 490     29015        32       6066.09       0.00            2        23222.79
#> 491      9016        20       4987.84    1900.00           31          544.61
#> 492      6916        16        212.20       0.00           16           46.00
#> 493      5854        19        568.60       0.00           12         1485.24
#> 494     26498        22        130.36       0.00           21          604.43
#> 495      9520        23       4271.31       0.00           38         4017.23
#> 496      7543        27       2049.00       0.00           22         5934.55
#> 497     67109        33        114.01       0.00            1         5745.06
#> 498     24093        20       2625.42    1104.30            8          669.46
#> 499      4285        22        574.88       0.00           16           97.90
#> 500     23791        23      18502.04       0.00           26           50.00
#> 501     20408        21       1689.68       0.00            9          214.43
#> 502      7739        23       2574.61    1250.00           18          308.41
#> 503     19850        16       2326.50       0.00           16          398.28
#> 504     35896        40       2239.83       0.00           16          617.09
#> 505      3318        28       3042.15       0.00            3         3475.21
#> 506     16992        32        975.87     736.85           30          325.50
#> 507     18711        37       1256.68       0.00           21        11599.20
#> 508       883        23        648.01       0.00           27        13691.94
#> 509    143395        36       2385.35       0.00            9         1333.42
#> 510      8699         7       1390.10       0.00            9          776.33
#> 511      2659        22       2160.18    1232.46           11          518.65
#> 512     16180        25       3167.97    1503.30           18          678.13
#> 513      7544        24       1313.21       0.00           18         5548.36
#> 514      8890        41       3539.60       0.00           16          429.79
#> 515     14899        27       3061.30       0.00           33        19490.84
#> 516     33542        26       5623.65       0.00           13          203.15
#> 517      5652        11       1882.51       0.00           16          385.69
#> 518       327        38        701.26       0.00            5         2742.11
#> 519     13021        50       5095.07       0.00            3          380.66
#> 520     13106        31       2951.40       0.00           18        11057.52
#> 521     44464        56        627.56       0.00           16          156.25
#> 522     15506        20       1848.28       0.00           14        14602.20
#> 523      6779        21       1428.58       0.00           37         4145.05
#> 524     18478        25       2114.98       0.00           36        11212.44
#> 525      4985        27       3357.69    2178.72           28          465.27
#> 526     17698        19       5983.94       0.00           14        22366.93
#> 527      7167        37       2761.45       0.00           32         1153.49
#> 528     10459        16       2291.90       0.00            9          482.22
#> 529     39025        23       3558.54       0.00           16          933.00
#> 530     18067        24       1313.18       0.00           12         6003.53
#> 531      8770        49       1892.44       0.00           33        13212.09
#> 532     17669        12        865.91       0.00           32          488.46
#> 533     20359        32       6594.04       0.00           29        15566.03
#> 534      9481        22       1875.78       0.00           34         7282.27
#> 535     19212        33       5193.85       0.00           21        23530.56
#> 536     15478        38        387.84       0.00            5         2240.87
#> 537      4877         6        756.73       0.00           28          290.41
#> 538     16081        32       3320.06       0.00           28          381.07
#> 539     45381        32       1019.57       0.00           32          915.73
#> 540      4268        37       1773.25       0.00           19          346.15
#> 541     37389        17       7318.95       0.00           16         1177.11
#> 542     32901        43        445.07    1315.29            1          504.55
#> 543     18425        30        969.37       0.00           16          165.95
#> 544      9862        28       2399.13       0.00           38         1434.29
#> 545     27478        21        964.43       0.00           28         3585.38
#> 546        35        12        165.05       0.00           17         2179.55
#> 547     16582        32       5485.79       0.00           16          740.81
#> 548     31201        28       3703.99       0.00           16          659.33
#> 549      4446        11       1048.88       0.00           19         1138.43
#> 550     23594        22       2765.19       0.00           19         5143.40
#> 551     18383        27       1607.62       0.00            8          386.09
#> 552     25878        49       2291.29    1770.80           36          497.96
#> 553     29588        18        375.90       0.00           27         7571.27
#> 554     18749        40       6036.70       0.00           31        13492.20
#> 555      6875        10       1829.78     852.50           37          440.57
#> 556     58332        23       1485.90       0.00           16          346.87
#> 557     17771        36       2619.50       0.00           38           14.28
#> 558     37071        22       3068.51       0.00            9          643.36
#> 559     63825        31       5608.85       0.00           32         3353.27
#> 560     31359        31       1064.72       0.00           14         8662.53
#> 561      7937        32       1000.96       0.00            5         4847.86
#> 562      4072        12       2010.13     486.38            8          297.78
#> 563      6711        18       4182.22       0.00           16          672.45
#> 564    109973        26       2739.29       0.00           34        16158.81
#> 565     27103        32        267.02       0.00           23           20.47
#> 566     20519        24       4076.65       0.00           13            4.86
#> 567     25268        46       1916.29       0.00           33        17172.47
#> 568    290575        28       8636.43       0.00            6        20343.31
#> 569     18025        30       2158.35       0.00            3         6322.16
#> 570     19778        31       5611.39       0.00            8        13282.21
#> 571     52760        36       3834.00       0.00           16          496.11
#> 572     18792        17       1862.38       0.00            3         7088.35
#> 573     29704        50       3457.11       0.00           14        30317.00
#> 574      3072        12        254.89       0.00           20          120.68
#> 575     34722        28       2565.12       0.00           12         9997.16
#> 576      8307        45        575.57       0.00           11            2.87
#> 577     13131        31       1463.79       0.00            5         6164.01
#> 578     42372        32      13259.60       0.00            3        24223.27
#> 579     24537        41        999.86       0.00            9          721.81
#> 580      5348        14        627.97       0.00           16          156.66
#> 581     21139        26       5706.91       0.00           16          854.29
#> 582      2412        12       2153.90     924.07           37          320.70
#> 583      1604        17        543.81       0.00           20         6180.52
#> 584     21745        42       1440.10       0.00            6         1882.64
#> 585     28554        30       1405.90       0.00            2         8213.46
#> 586     11401        19        486.93       0.00           10        26787.94
#> 587     19850        44       5962.98       0.00           29        15301.44
#> 588     13377        14       1184.68       0.00           33         7952.60
#> 589      6468        20       1370.74       0.00           36        11740.41
#> 590     20137        35       7610.06       0.00           30        33268.46
#> 591      5123        16       1334.52       0.00           19          218.70
#> 592     24056        39        327.00       0.00            4         4853.54
#> 593     10920        22       3025.99       0.00           38         3299.48
#> 594     18084        21       1188.27       0.00            9          340.53
#> 595      4980        14        366.38       0.00           36         5346.32
#> 596      6998        48       1317.78       0.00           16          270.11
#> 597     21235        33       1822.27       0.00            4        18962.50
#> 598     39981        28       3710.56       0.00           16          518.83
#> 599      5455         5       1426.64       0.00           11         9668.12
#> 600     14100        10       4129.96       0.00           16          615.11
#> 601     90009        31      11342.57       0.00           15        25775.46
#> 602     19093        16       2511.23       0.00           25         6753.31
#> 603     19300        27       1672.84       0.00           16          602.13
#> 604     14005        19       1494.75       0.00           25         3782.76
#> 605     11141        15       2311.68       0.00           12         8221.78
#> 606      1409        16        314.03       0.00           13            0.14
#> 607     15196        26       1452.87       0.00           28         9896.59
#> 608      5967        22        689.35     487.57           11          189.88
#> 609      1709        18       1021.35       0.00            8         4189.31
#> 610      5090        24        477.83       0.00           32          101.00
#> 611     38029        25       2206.30       0.00           29         7460.05
#> 612      8499        13       2336.06       0.00           12         5601.06
#> 613     24339        26        678.99       0.00           20        19632.56
#> 614      9902        36        461.18       0.00           37          246.72
#> 615     22216        38        432.03     411.43           14          160.87
#> 616     12494         8        979.25       0.00           37         5503.01
#> 617       166         7       1576.08       0.00           36          543.08
#> 618     21065        28       3014.85       0.00            9          669.35
#> 619      6092         8        519.64       0.00           12         2269.68
#> 620     19903        38       1377.79       0.00           29         3276.82
#> 621      5958        21        337.86       0.00           24         2184.93
#> 622     10881        45       1363.87     492.23           37          249.07
#> 623     14895        19       1183.61       0.00           33        10855.22
#> 624     20914        21       1377.04    1776.11            4          706.37
#> 625      1641        14       1867.60       0.00           28          369.22
#> 626     43858        27       3535.86       0.00           18          107.81
#> 627      4980        21       1895.06       0.00            6         1526.59
#> 628     18893        19       2941.77       0.00           25         5276.20
#> 629      4014         7        284.65       0.00           16           74.40
#> 630     20627        37        445.49       0.00           16           67.50
#> 631     14403        40        429.04       0.00           27         9437.82
#> 632     25530        33       7475.10       0.00            5          808.15
#> 633      4122        25       2060.10       0.00            2        16087.60
#> 634     14212        52        508.42       0.00           17        10551.47
#> 635      7682        18        306.45       0.00            9          183.19
#> 636     14301        21       1702.95     712.68            8          420.81
#> 637      1688        10       1045.50       0.00           16          167.79
#> 638     21819        38        723.91       0.00            7         6825.76
#> 639      5806        13       1351.87       0.00           25         3410.80
#> 640     12138        28        126.72       0.00            2          663.45
#> 641     12627        16       1423.09       0.00            7          279.16
#> 642     99614        26       1995.58       0.00           32         2162.23
#> 643      6022         6       1244.07       0.00           16          270.60
#> 644      3770        19        358.09       0.00           16          107.14
#> 645     24765        17       4603.61       0.00           16          575.93
#> 646      1931        12       2816.56       0.00            5         8126.54
#> 647     17033        35        757.95       0.00            3         1841.15
#> 648     11513        44        290.76       0.00            2         2034.43
#> 649     16114        29       1801.95       0.00           19         3808.50
#> 650      8676        33        119.94       0.00           14           34.18
#> 651     11901        28        820.84       0.00           29         1131.89
#> 652      1024        16        387.31       0.00           32            1.17
#> 653     16428        52       3951.95       0.00            3         7156.60
#> 654      5605        23        543.79       0.00           30         7761.77
#> 655     20576        28       3772.97       0.00           29        12298.07
#> 656     48591        20       4197.03       0.00           28        22116.13
#> 657      5341        20       4146.74       0.00           35         2655.16
#> 658     11020        18        536.26       0.00           20        18102.73
#> 659     10914        17       1333.99       0.00           30          423.76
#> 660     22631        37       1058.15       0.00           31         6582.90
#> 661     19012         9        833.92       0.00           16          301.06
#> 662     18838         8        624.02       0.00           16          117.26
#> 663     22065        35        969.37       0.00           16          165.95
#> 664     33364        21       6625.73       0.00           31        25338.03
#> 665     10410        35       1834.31       0.00            2         8698.45
#> 666      3618        12        619.70     336.33           11          141.80
#> 667     37064        21       7094.13       0.00            8          582.25
#> 668      6174        31        906.62       0.00            6          852.31
#> 669     17826        17       2731.84       0.00           14        15955.27
#> 670     13605        47        892.85       0.00           21         8676.25
#> 671      6112        15       1808.80       0.00           21         7372.03
#> 672      9419        29       2027.10       0.00           16          333.71
#> 673     12057        35       3096.72       0.00           37        12369.06
#> 674     37482        15       6047.57       0.00           32         3406.71
#> 675     34415        17       2084.83       0.00           16          752.85
#> 676      8133        15       1142.17       0.00           16          284.52
#> 677     50627        23       1486.63       0.00           27        24605.14
#> 678     18792        43       2124.50       0.00            9          380.28
#> 679     70397        23        987.47       0.00            7         8244.29
#> 680      7493        26       1076.37       0.00            5         5179.88
#> 681      3279        27        666.71       0.00           16          118.65
#> 682     11078        20       2844.93    1095.82           25          287.31
#> 683      7626        20       1001.18       0.00           34         5657.20
#> 684     20555        21       1917.28       0.00           22         4374.88
#> 685     21147        23       7791.81       0.00           19        14393.04
#> 686      9233        20        814.08       0.00            9          244.98
#> 687     63792        28        945.56       0.00           16          220.56
#> 688      9374        11         75.83       0.00           23         4437.16
#> 689     10585        23       3745.02       0.00           16          423.88
#> 690     10502        13        728.67       0.00           32          474.84
#> 691     32677        49        692.65       0.00           13          364.00
#> 692         0        22        805.97       0.00            9          198.84
#> 693     12020        17       1872.85       0.00           35         1534.30
#> 694      8881        19       2103.34       0.00           24        10723.43
#> 695     28624        28       2258.89       0.00           18        11904.82
#> 696     19997        16       1667.85       0.00           16          602.13
#> 697     30169        62       2806.37    4736.26           28          574.33
#> 698      6273        24       1072.39       0.00           30        11749.32
#> 699     23901        36       3498.61       0.00           34         7798.39
#> 700     16349        10       5049.78       0.00           19          312.06
#> 701     31184        26       8400.12       0.00            3        11587.00
#> 702     14276        42       8155.54       0.00            8        13792.01
#> 703     15676        38       1989.34    2888.32           17          708.86
#> 704     16344        24       1248.65     410.27           12          276.08
#> 705     45576        21        427.99       0.00           11         3521.56
#> 706     37505        23       1331.93       0.00           16          290.02
#> 707     12327        12         25.83       0.00           17          247.75
#> 708      4865        28        254.69       0.00           33         3902.94
#> 709     21112        56       2096.69       0.00           37        12041.94
#> 710      4795        22        667.00       0.00            9          481.63
#> 711     10705        32       1044.60       0.00           21         5956.70
#> 712      8073        15        663.81       0.00           28         5859.61
#> 713     14786        43       5722.46       0.00           37        15919.72
#> 714     11685        14        772.44       0.00           29         1236.70
#> 715     48408        39        487.81       0.00           35           10.16
#> 716     24075        23       1837.63       0.00           16          662.35
#> 717      2112        29       1501.20       0.00           20        27056.99
#> 718     11220        30        365.94       0.00           11         3498.07
#> 719      2139        29       1486.12       0.00           38         2695.25
#> 720     37657        39        322.13       0.00           36         8221.47
#> 721     16425        13        413.28    1019.47           27          371.95
#> 722       512        19        155.21     190.94           20           76.95
#> 723     11701        42       1916.52       0.00           16          302.54
#> 724      4739        12       1049.82       0.00           21         7342.34
#> 725     14849        20        496.77       0.00           32          177.05
#> 726      3961        18       4049.17    1242.64           31          691.84
#> 727      1392        29        616.12    2996.27           20           47.00
#> 728     43397        26       1974.87       0.00           17        33141.25
#> 729     37424        42      11841.02       0.00           13        15306.86
#> 730      6582        28        707.69       0.00           19          557.75
#> 731     24758        21       3514.45       0.00           16          876.11
#> 732      7346        39        719.17       0.00           24         4128.79
#> 733     45796        21       1723.17       0.00           14          333.60
#> 734     61770        18       2745.92       0.00           17          163.69
#> 735      8208        15       1978.41       0.00           29          391.62
#> 736     19821        35        418.87       0.00           16           67.65
#> 737     11287        16       1058.11       0.00            8         1977.36
#> 738     15437         9        279.93       0.00           14         4296.77
#> 739     14691        22       2035.95       0.00           32          566.50
#> 740     19951        46       3713.65       0.00            5        13050.39
#> 741      5513        27        943.27       0.00           32          504.90
#> 742     17526        15       1448.83       0.00            9          630.70
#> 743     48446        37       4157.81       0.00           16         1088.45
#> 744     12330        36       3371.41    2047.92           14          482.40
#> 745     16265        65        952.05       0.00            6          814.01
#> 746     27281        33       6273.37       0.00           16         1008.89
#> 747     24345        19       1975.30       0.00            2        13629.61
#> 748     10177        19        202.35       0.00           19          228.58
#> 749     11570        18        205.80       0.00           17          239.12
#> 750     35362        20       2222.38       0.00           16          395.51
#> 751      2227        16        418.24       0.00            1        19202.74
#> 752     21754        22       1072.00       0.00           31         6596.75
#> 753      1974        52        138.23       0.00           23         6918.62
#> 754     26552        43       2515.71       0.00            4        26351.03
#> 755      2118        40        157.71       0.00            4         1744.63
#> 756      7198        23       2011.47       0.00           12         4848.69
#> 757     28964        17       1882.71       0.00           16          469.14
#> 758     13978        18       2516.98       0.00           21        10398.27
#> 759      2363        32        682.27       0.00           36        11629.55
#> 760      7634        18       1513.12     205.56           30          269.44
#> 761      7989        18       3179.50       0.00           34        11296.90
#> 762     11234        15       1491.32       0.00           29         3096.61
#> 763     27483        50        867.52       0.00            8         4926.03
#> 764     10568        27       6844.64       0.00           37        21491.10
#> 765      9758        20       2640.24    1151.95           30          369.33
#> 766      6826        26       4174.63       0.00           12        15992.72
#> 767      7585        25        210.16       0.00           27         7736.39
#> 768     25249        51       1794.86       0.00           16          466.31
#> 769      6515        18       1386.78       0.00           16          260.96
#> 770     14966        65      10098.17       0.00           35          622.45
#> 771     17669        24       9012.29       0.00           15        27330.01
#> 772      8288        63       2113.62       0.00           22         1867.95
#> 773     12799        21       6740.22       0.00           35          302.37
#> 774      8766        20        660.71       0.00           29          252.97
#> 775      7421        23       1405.11       0.00           16          305.94
#> 776      7697        12       1464.55       0.00           13            0.85
#> 777      4726        29         94.14       0.00            4         1500.74
#> 778     11431        28       1908.46       0.00           24        10376.14
#> 779      4997        15        881.92       0.00           13            2.14
#> 780      4024        14       1017.22       0.00           35         1219.83
#> 781     45023        24       2084.83       0.00           16          752.85
#> 782     12997        27       1046.20       0.00            4         7743.29
#> 783     12184        13       3154.23     577.47            3          484.29
#> 784     28495        14       2852.08       0.00           18        12708.51
#> 785     32236        17       1225.30       0.00           27          578.26
#> 786     13627        32        819.41       0.00           19          682.11
#> 787     28415        42        127.42       0.00           17         3175.17
#> 788     50541        28       2167.92       0.00           38           12.37
#> 789     15564        29       6716.40       0.00            2        25391.19
#> 790     17047        10       2372.33       0.00           16          621.74
#> 791     20312        25       9090.11       0.00            5        24718.11
#> 792     27849        25       1570.76       0.00           16          339.56
#> 793      6644        52       5826.90       0.00           18        24020.89
#> 794      4055        29        369.19       0.00           15         1310.82
#> 795      5272        22       1078.33    1188.95           17          311.23
#> 796     17685        21        705.02       0.00           35          739.74
#> 797     19014        26       3666.87       0.00           12        12926.98
#> 798      5492        54       3240.79       0.00           13         5499.32
#> 799     21041        41       1849.25       0.00           15         7742.12
#> 800     12932        16       2887.87       0.00           12         5156.21
#> 801     26666        19       4393.03       0.00           16         1094.93
#> 802     16702        24       2014.01       0.00           13           24.02
#> 803      1888        24        246.27       0.00           33         2451.05
#> 804     29095        20       3307.03       0.00            9          739.23
#> 805     37981        10       3379.88       0.00           16          601.58
#> 806      7029        15       6842.92       0.00           13         7189.38
#> 807      5795        27       2360.96     786.73           31          430.58
#> 808      7847        13        643.48       0.00            9          101.67
#> 809        94        15        472.56       0.00           32          371.19
#> 810     20381        14        418.72     395.16           33          186.22
#> 811     11239        31       1603.78       0.00           36        11792.70
#> 812     32416        30       6301.31       0.00            5        17935.42
#> 813     32011        29       2864.35       0.00           16          857.84
#> 814      3021         8        418.74       0.00           28         1626.70
#> 815      3750        33        592.83       0.00           15         1968.87
#> 816      4055        14        383.39       0.00           16           49.57
#> 817       685         8        930.68       0.00           24          291.25
#> 818      1845        29        450.32       0.00           16          162.61
#> 819     26265        24       4640.61       0.00           16          823.84
#> 820      6277        15       4182.22       0.00           16          672.45
#> 821     28980        21       3508.33       0.00           21        15484.21
#> 822     29143        17       1691.33       0.00           16          421.63
#> 823      1567        16        694.99       0.00           21         3895.79
#> 824     23053        42       7459.52       0.00           13          162.04
#> 825     10131        62       1198.77       0.00           11        10535.06
#> 826     32672        51       8459.28       0.00           22          566.04
#> 827      3434        50       3269.26       0.00           25         9954.24
#> 828      1806        18       1560.98       0.00           30           15.71
#> 829      1101        21        629.37     504.11           30          217.01
#> 830     26069        25       1320.07    1420.00           22          250.00
#> 831      5083        13        625.58       0.00            9          176.56
#> 832     13399        17       1597.39     305.00            5          492.39
#> 833     71234        46       1901.19    1912.13            4          499.96
#> 834     11665        34        586.93       0.00           18         2619.41
#> 835     13383        13       2097.70       0.00           16          370.23
#> 836     18221        19       4537.88       0.00            7        24476.53
#> 837      2377        14       1125.33     732.09           24          286.20
#> 838    115040        22       3331.49       0.00           32         2171.33
#> 839      6431        31       5043.17    1526.84            8          396.50
#> 840     25643        19       1601.13       0.00           16          577.97
#> 841        87        24       3045.40       0.00           37         7502.28
#> 842     33552        17       2856.82       0.00           16          746.41
#> 843     11592        25       1000.71       0.00           16          361.28
#> 844      1898        23       1329.17       0.00           24         9749.14
#> 845      1025        26        678.50       0.00           16          139.14
#> 846      6731        31       1299.46       0.00            2         5094.56
#> 847     23621         6       4115.87       0.00           34        18336.70
#> 848      4325        20       1919.45       0.00            5         7461.08
#> 849      2081         6        646.46       0.00           35          458.36
#> 850     10446        19        405.99     358.09            7          173.74
#> 851     31376        33       6660.27       0.00            2        21240.19
#> 852      4131        41        609.28       0.00            4        14498.84
#> 853      9366        31       1238.24       0.00           11         7188.07
#> 854     39913        45       8193.31       0.00           21        29069.46
#> 855      6343        14       1933.50       0.00           29         7298.77
#> 856      6643        43        513.14       0.00           23        21645.52
#> 857     14565        50       1037.46       0.00           16          139.92
#> 858     22476        32       1200.04       0.00           37         8914.01
#> 859      5299        30       6432.61       0.00           16          760.38
#> 860      2187         9       7135.79       0.00           19        13132.35
#> 861     15592        40       1540.31       0.00           19         1735.43
#> 862      9562        26       4395.59       0.00           35         6964.04
#> 863     19727        20       3575.53    2171.54            2          483.93
#> 864      9591        28        344.16       0.00           34          575.60
#> 865     47098        71        964.26    6653.00           36          600.00
#> 866     15644         9       3276.16       0.00           24        13979.71
#> 867     26599        41       4466.47       0.00           19           93.38
#> 868      1779        20        471.68       0.00           14         3109.31
#> 869      3385        24        925.10     562.00           38          100.00
#> 870     10874        36       1632.79       0.00           14        14671.67
#> 871     30349        27       1106.10       0.00            4        25578.14
#> 872      5914        21        565.31       0.00           14         3720.46
#> 873     54554        24       5509.70       0.00           15        12354.30
#>     last_credit_pull_d last_fico_range_low tot_coll_amt open_acc_6m open_act_il
#> 1                   26                 560          722           2           2
#> 2                   26                 695            0           1           1
#> 3                   26                 700            0           0           1
#> 4                   25                 700            0           1           3
#> 5                   27                 755            0           0           1
#> 6                   26                 650            0           0           2
#> 7                   26                 670            0           0           3
#> 8                   26                 715         8341           2           1
#> 9                   31                 675            0           0           2
#> 10                  37                 700            0           1           3
#> 11                  30                 580           60           2           2
#> 12                  26                 760            0           0           2
#> 13                   6                 775            0           0           3
#> 14                   2                 720            0           0           3
#> 15                  26                 790            0           3           1
#> 16                  20                 635            0           6           1
#> 17                  15                 715            0           2           1
#> 18                  36                 660            0           1           2
#> 19                  26                 740            0           0           0
#> 20                  26                 655            0           0           1
#> 21                  26                 715            0           0           2
#> 22                   8                   0            0           0           1
#> 23                   6                 805            0           0           2
#> 24                  17                 675            0           2           2
#> 25                   6                 775            0           0          10
#> 26                  16                 690            0           0           0
#> 27                  26                 625         1830           2           2
#> 28                  25                   0            0           2           1
#> 29                  21                 750         5930           0           4
#> 30                  26                 600          171           3           3
#> 31                  26                 765            0           0           1
#> 32                  26                 695            0           1           1
#> 33                  17                 730            0           1           1
#> 34                  13                 735            0           2           1
#> 35                   9                 690            0           0           1
#> 36                  26                 655          264           2           3
#> 37                  12                 520            0           2           2
#> 38                  26                 795            0           2           2
#> 39                  31                 770            0           0           1
#> 40                  26                 715            0           0           7
#> 41                  12                 685            0           1          18
#> 42                   5                 705            0           1           1
#> 43                  13                 665            0           3           3
#> 44                   9                 700            0           0           3
#> 45                   9                 590            0           2           4
#> 46                  26                 690            0           1           2
#> 47                  26                 720          321           1          11
#> 48                  26                 630            0           0           2
#> 49                  26                 695         1036           1           1
#> 50                  27                   0            0           0          15
#> 51                   6                 770            0           1           1
#> 52                  13                 690            0           1           0
#> 53                  26                 500            0           0           3
#> 54                  26                 605            0           2           3
#> 55                   8                 510            0           0           2
#> 56                   6                 620            0           2           1
#> 57                  26                 715            0           2           6
#> 58                  12                 775            0           2           2
#> 59                  12                 740           25           3           2
#> 60                  36                 730            0           2           4
#> 61                  26                 735            0           3           2
#> 62                   9                 780            0           1           2
#> 63                  30                 560            0           0           4
#> 64                  11                 515            0           1           1
#> 65                  26                 700            0           0           7
#> 66                   9                 710            0           2           9
#> 67                  28                 730            0           0           2
#> 68                   7                 580         1105           1           3
#> 69                  28                 520            0           0           1
#> 70                  16                 705            0           0           1
#> 71                   9                 720            0           2           0
#> 72                   3                 555            0           2          14
#> 73                   5                 720            0           1           2
#> 74                  26                 655            0           0           1
#> 75                  16                 585            0           1           1
#> 76                  13                 690            0           0           2
#> 77                  26                 745            0           1           3
#> 78                  37                 795            0           1           1
#> 79                   9                   0            0           0           1
#> 80                  26                 765            0           2           2
#> 81                  13                 520          230           0           2
#> 82                  13                 525            0           3           6
#> 83                  16                 725          194           0           3
#> 84                  19                 545            0           2           0
#> 85                  16                 730            0           2           2
#> 86                   3                 740            0           0           6
#> 87                   3                 525            0           1           2
#> 88                  23                 695            0           0           1
#> 89                  13                 585            0           0           3
#> 90                  31                 670          500           2           0
#> 91                  30                 815            0           2           6
#> 92                  16                 680            0           1           2
#> 93                  28                 670            0           1           1
#> 94                  16                 705         6594           0           2
#> 95                  13                 645            0           0           3
#> 96                  36                 720            0           0           2
#> 97                  26                 790            0           0           2
#> 98                  26                 700            0           1           4
#> 99                  22                 630            0           2           6
#> 100                 28                 745            0           2           9
#> 101                 26                 645           95           4           4
#> 102                  6                 635          133           1           2
#> 103                 16                 700            0           2           1
#> 104                 21                 755            0           1           0
#> 105                 26                 660            0           3           0
#> 106                  7                 690            0           1           1
#> 107                  9                 730            0           0           1
#> 108                 19                 565           67           2           2
#> 109                 16                 720            0           3           2
#> 110                 13                 725            0           0           4
#> 111                 26                 800            0           3           3
#> 112                 33                 675            0           2           4
#> 113                  3                 700            0           2           3
#> 114                  3                 695            0           0           2
#> 115                 26                 790            0           2           4
#> 116                 18                 815            0           0           2
#> 117                 31                 685            0           4           4
#> 118                  9                 730            0           1           3
#> 119                 26                 705            0           3          15
#> 120                 26                 680            0           0           2
#> 121                 17                 750            0           2          12
#> 122                 26                 575            0           0           1
#> 123                 34                 620         1443           1           1
#> 124                 26                 750            0           0           0
#> 125                 26                 540            0           1           1
#> 126                  5                 710            0           0           2
#> 127                 17                 675            0           0           1
#> 128                 34                 590            0           1           3
#> 129                 14                 755            0           0           1
#> 130                 25                 775            0           2           3
#> 131                 26                 655            0           1           3
#> 132                 31                 790            0           2           2
#> 133                 37                 725            0           1           1
#> 134                  3                 795            0           0           1
#> 135                 26                 740            0           1           2
#> 136                 26                 725            0           2           3
#> 137                 21                 510         1082           2           0
#> 138                 26                 715            0           0           2
#> 139                 26                 510           70           0           0
#> 140                 32                 715            0           0           1
#> 141                 37                 720            0           1           5
#> 142                 26                 690            0           0           1
#> 143                 26                 710            0           3           1
#> 144                 36                 680            0           2           4
#> 145                 26                 700            0           2           2
#> 146                 26                 690          972           3           2
#> 147                 26                 685            0           1           1
#> 148                  9                 690            0           2           3
#> 149                 18                 500            0           5           1
#> 150                  6                 710            0           3           1
#> 151                 26                 715            0           2           4
#> 152                 36                 525         7233           1           0
#> 153                 25                 630            0           3           2
#> 154                 16                 670            0           0           1
#> 155                 26                 560            0           0           0
#> 156                 26                 795            0           0           0
#> 157                 26                 715            0           0           0
#> 158                 31                 610            0           1           4
#> 159                  9                 755            0           0           1
#> 160                 19                 690          428           0           4
#> 161                 26                 565          200           2           3
#> 162                 13                 550            0           2           1
#> 163                 37                 720            0           0           1
#> 164                 26                 785            0           3           4
#> 165                 36                 685            0           0           7
#> 166                 13                 670            0           0           2
#> 167                 19                 675            0           2          14
#> 168                 26                 715            0           3           4
#> 169                  9                 650            0           2           3
#> 170                 13                 610            0           2           2
#> 171                 16                 765            0           0           5
#> 172                 16                 660            0           0           3
#> 173                 20                 835            0           0           1
#> 174                  7                 740            0           2           5
#> 175                 26                 675          300           3           2
#> 176                 16                 600            0           1           4
#> 177                  3                 780            0           0           1
#> 178                  9                 795            0           0           3
#> 179                  2                 625          882           5           6
#> 180                 26                 660            0           2           3
#> 181                 21                 825            0           1           2
#> 182                 33                 750            0           0           5
#> 183                 26                 720            0           1           1
#> 184                 26                 720            0           0           0
#> 185                 26                 690            0           0           4
#> 186                 13                 555            0           4           7
#> 187                 16                 690            0           0           4
#> 188                  6                 585            0           2           7
#> 189                 27                 545            0           1           1
#> 190                 26                 660            0           0           5
#> 191                 15                 740            0           2           2
#> 192                 16                 595            0           1           1
#> 193                 13                 630            0           0           1
#> 194                 28                 760         1053           0           3
#> 195                 26                 715          200           1           4
#> 196                  9                 715            0           0           3
#> 197                 26                 675            0           2          11
#> 198                 15                 635          207           0           1
#> 199                  9                 615            0           1           1
#> 200                 25                 800            0           2           4
#> 201                 13                 695            0           3           2
#> 202                 16                 765            0           0           3
#> 203                 11                 575         2403           3          19
#> 204                 26                 770            0           0           6
#> 205                 16                 620            0           0           1
#> 206                 26                 710            0           0           3
#> 207                 26                 670            0           0           1
#> 208                 16                 600            0           0           4
#> 209                 26                 730            0           1           5
#> 210                 26                 705            0           1           1
#> 211                  9                 680            0           0           1
#> 212                 37                 575            0           3           1
#> 213                 33                 640            0           1           4
#> 214                 26                   0            0           3           1
#> 215                 33                 565            0           1           1
#> 216                 34                 725            0           3           1
#> 217                 17                 800            0           1           1
#> 218                 16                 765            0           0           2
#> 219                 26                 705            0           1           5
#> 220                  9                 615           50           1           1
#> 221                 26                 760            0           2           2
#> 222                 26                 720            0           1           3
#> 223                 26                 730            0           2           2
#> 224                 26                 705           72           0           3
#> 225                 13                 695            0           2           1
#> 226                 16                 795            0           1          14
#> 227                  8                 685            0           1           1
#> 228                 26                 740          418           2           3
#> 229                 31                 710            0           0          11
#> 230                 16                 740            0           0           4
#> 231                 26                 685            0           0           0
#> 232                  9                 710            0           2           0
#> 233                  9                 605          723           4           1
#> 234                 16                 600         3774           1           1
#> 235                 26                 670            0           2           2
#> 236                 26                 790            0           1           2
#> 237                 26                 725            0           0           2
#> 238                 26                 710           58           1           2
#> 239                 16                 720            0           0           5
#> 240                 13                 630            0           4           6
#> 241                 15                 555           60           0           1
#> 242                 25                 555            0           0           2
#> 243                  3                 810            0           2           1
#> 244                 27                 740            0           2           3
#> 245                  9                 740            0           1           2
#> 246                 11                 545            0           5           5
#> 247                 26                 740            0           0           3
#> 248                  9                 700            0           3           3
#> 249                 26                 695            0           4           2
#> 250                 13                 610            0           0           0
#> 251                  1                 730            0           0           1
#> 252                 13                 660            0           0           2
#> 253                 26                 780            0           1           1
#> 254                 26                 645            0           1           4
#> 255                 11                 570            0           3           2
#> 256                 37                 705            0           2           4
#> 257                  5                 540            0           0           2
#> 258                 16                 580           50           1           2
#> 259                 13                 550            0           0           3
#> 260                 21                   0            0           1           5
#> 261                 16                 785            0           1           6
#> 262                 26                 760            0           0           2
#> 263                 22                 710            0           1           2
#> 264                 13                 735            0           0           2
#> 265                 18                 740            0           0           1
#> 266                 26                 620            0           1           0
#> 267                 16                 640            0           5           2
#> 268                 26                 680            0           9           1
#> 269                 33                 510         2183           4           2
#> 270                 26                 585            0           1           1
#> 271                 26                 750          216           1           2
#> 272                 18                 630            0           1           3
#> 273                 13                 535            0           2           3
#> 274                 26                 680            0           6           5
#> 275                 31                 690          784           1           0
#> 276                 26                 680            0           1           3
#> 277                 26                 575           88           0           1
#> 278                 26                 645            0           1           0
#> 279                 28                 745            0           1           3
#> 280                 13                 745            0           1           5
#> 281                 18                 525            0           1           3
#> 282                 26                 550            0           3           1
#> 283                 26                 700            0           2           1
#> 284                 15                 700            0           1           7
#> 285                 26                 730            0           1           3
#> 286                 24                 660            0           4           5
#> 287                 13                 595            0           2           9
#> 288                 25                 745            0           1           1
#> 289                  9                 650          109           0           2
#> 290                 31                 665            0           3           0
#> 291                  8                   0            0           2           1
#> 292                 16                 655            0           2           4
#> 293                 12                 805            0           1           1
#> 294                 16                 715         1120           0           3
#> 295                 26                 555         1807           1           5
#> 296                 31                 645            0           2           3
#> 297                 26                 730            0           1           2
#> 298                 11                 775            0           0           0
#> 299                 26                 685         1615           5           2
#> 300                  6                 735            0           0           0
#> 301                 18                   0            0           1           2
#> 302                 26                 655            0           0           2
#> 303                 26                 780            0           2           2
#> 304                 19                 800            0           1           3
#> 305                 18                 725            0           0           3
#> 306                 21                 530            0           3           3
#> 307                 26                 700            0           1           1
#> 308                 26                 650          328           3           2
#> 309                 16                 705            0           0           1
#> 310                 14                 785            0           2           0
#> 311                  7                 505            0           2           7
#> 312                 26                 690            0           0           2
#> 313                  3                 660            0           0           1
#> 314                  2                 555          700           2           4
#> 315                 14                 790            0           3           1
#> 316                  2                 540            0           0           7
#> 317                 32                 595            0           4           4
#> 318                 13                 685          941           0           4
#> 319                 16                 695          397           1          10
#> 320                 26                 620         1912           1           3
#> 321                 26                 510         2129           3           0
#> 322                 16                 685          166           1           0
#> 323                 26                 765            0           1           0
#> 324                 26                 530            0           0           1
#> 325                 26                 750            0           3           8
#> 326                 11                 590           55           5           4
#> 327                 37                 745          158           0           1
#> 328                  9                 685            0           5           7
#> 329                  9                 715            0           0           1
#> 330                 16                 665            0           1           2
#> 331                  6                 610          155           1           2
#> 332                  9                 530            0           5           9
#> 333                 26                 680            0           3           4
#> 334                 15                 685            0           2           0
#> 335                 30                 700            0           3           4
#> 336                  6                 735            0           0           2
#> 337                 26                 555            0           1           3
#> 338                 26                   0          125           5           3
#> 339                 26                 590         3151           2           1
#> 340                  9                 815            0           3           1
#> 341                  9                 795            0           0           2
#> 342                 26                 690          638           2           2
#> 343                 26                 760            0           1           2
#> 344                 26                 725            0           1           4
#> 345                 25                 635            0           0           4
#> 346                  6                 640          310           0           2
#> 347                 19                 540            0           1           2
#> 348                 15                 785            0           1           2
#> 349                 30                 730         2961           1           2
#> 350                 34                 720            0           0           2
#> 351                 16                 715            0           1           2
#> 352                 26                 710            0           2          17
#> 353                 12                 795            0           0           2
#> 354                 14                 695            0           6           6
#> 355                 19                 715            0           1           1
#> 356                 26                 760            0           0           3
#> 357                 17                 700            0           0           3
#> 358                 26                 750            0           1           4
#> 359                 26                 650            0           2           3
#> 360                 19                   0            0           3           2
#> 361                 19                 730            0           0           1
#> 362                 26                 710       102841           0           0
#> 363                 25                 730            0           1           0
#> 364                 26                 725           75           3           3
#> 365                 13                 690            0           0           4
#> 366                  9                 710            0           0           2
#> 367                 34                 720            0           1           6
#> 368                 19                 685            0           2           3
#> 369                 26                 720            0           2           3
#> 370                 36                 710          219           0           1
#> 371                 28                 585            0           2           2
#> 372                  9                 810            0           0           1
#> 373                  9                 780            0           2           6
#> 374                 26                 715            0           4           0
#> 375                 37                 695            0           1          12
#> 376                 26                 615            0           1           3
#> 377                 26                 580            0           0           1
#> 378                 26                 690            0           2           0
#> 379                 16                 545            0           1           2
#> 380                 34                 630            0           0           2
#> 381                 16                 710         7259           1           4
#> 382                 13                 720            0           1           1
#> 383                 26                 690            0           0           3
#> 384                  9                 730            0           0           1
#> 385                  6                 760            0           0           5
#> 386                 16                 615            0           2           2
#> 387                  4                 800            0           1           1
#> 388                 12                 550            0           1           3
#> 389                 10                 765            0           0           0
#> 390                 26                 775            0           1           1
#> 391                  9                 585          192           1           2
#> 392                 26                   0            0           2          17
#> 393                  3                 590            0           7           1
#> 394                  6                 565            0           1           3
#> 395                 26                 730            0           1           3
#> 396                 16                 685            0           1           1
#> 397                 26                 630            0           0           1
#> 398                 31                 610            0           1           3
#> 399                 19                 640            0           1           1
#> 400                 12                 765            0           0           2
#> 401                 26                 720            0           0           3
#> 402                  9                 750            0           0           1
#> 403                 18                 715            0           1           2
#> 404                 31                 675            0           1           4
#> 405                 33                 705            0           1           2
#> 406                  8                 545            0           1           1
#> 407                 34                 680            0           2           2
#> 408                 26                 690          219           0           0
#> 409                  9                 660            0           1           1
#> 410                 25                 750           91           0          10
#> 411                  9                 805            0           1           3
#> 412                  9                 770            0           0           5
#> 413                  2                 760            0           1           2
#> 414                 25                 745          200           3           3
#> 415                 26                 700            0           2           2
#> 416                 13                 565            0           1           2
#> 417                 34                 665            0           1           1
#> 418                 29                 765            0           0           2
#> 419                 26                 715            0           0           4
#> 420                 26                   0            0           4           1
#> 421                  2                 670            0           4           2
#> 422                 34                 570          237           1           1
#> 423                 37                 725           60           1           2
#> 424                 26                 645            0           3           1
#> 425                 26                 795            0           3           1
#> 426                 13                 645            0           0           4
#> 427                 23                 715            0           2           4
#> 428                 26                 660            0           1           2
#> 429                  9                 645         2704           0           2
#> 430                  6                 730            0           2           0
#> 431                 26                 630            0           0           2
#> 432                 26                 715            0           1           0
#> 433                 26                 800            0           0           2
#> 434                 12                   0            0           1           0
#> 435                 16                 660            0           1          21
#> 436                 13                 650            0           3          10
#> 437                 13                 675            0           0           9
#> 438                  9                 705            0           2           0
#> 439                 13                 705            0           0           1
#> 440                  9                 715            0           0           1
#> 441                 26                 640            0           1           7
#> 442                  9                 690            0           1           3
#> 443                 26                 665            0           2           1
#> 444                 26                 740            0           1           2
#> 445                  8                   0            0           2           1
#> 446                 31                 730            0           1           3
#> 447                 24                 685            0           2           3
#> 448                 15                 550            0           3           4
#> 449                  9                 675            0           0           0
#> 450                 25                 545            0           0          18
#> 451                 25                 550            0           2           1
#> 452                 19                 665          156           0           6
#> 453                 26                 715            0           0           6
#> 454                 26                 580            0           1           6
#> 455                 26                 590          798           1           8
#> 456                  9                 660            0           0           2
#> 457                 26                 600          341           0           3
#> 458                 16                 770            0           0           0
#> 459                 16                 645            0           1           1
#> 460                 26                 570            0           1          10
#> 461                  6                 725          396           1           3
#> 462                 36                 830            0           0           3
#> 463                  5                 785            0           0           0
#> 464                 36                 685            0           0           5
#> 465                 16                 745          399           1           1
#> 466                  9                 725            0           1           3
#> 467                  9                 670          669           0           1
#> 468                  3                 685            0           1           4
#> 469                  9                 640            0           0          11
#> 470                 21                 785            0           0           3
#> 471                  6                 705            0           0           4
#> 472                 13                 685            0           2           6
#> 473                 25                 710            0           0           5
#> 474                 22                 775            0           1           1
#> 475                 26                 720            0           1           0
#> 476                 26                 740          192           0           1
#> 477                 37                 715            0           1           0
#> 478                 26                 755            0           0           1
#> 479                 24                 710           60           0           1
#> 480                 13                 660            0           1           2
#> 481                 26                 750            0           1           4
#> 482                 25                 790            0           0           2
#> 483                 21                 580         1376           6           7
#> 484                 26                 695            0           1           3
#> 485                 26                 770            0           0           4
#> 486                 26                 685            0           1           2
#> 487                 26                 715          564           3           2
#> 488                 15                 595            0           2           5
#> 489                 16                 745            0           1           7
#> 490                 27                 700            0           0           1
#> 491                 26                 570            0           0           2
#> 492                  9                 630            0           1           1
#> 493                 26                 740            0           4           5
#> 494                  6                 705            0           0           2
#> 495                 26                 815            0           0           2
#> 496                 26                 665            0           0           4
#> 497                  1                 690            0           2           4
#> 498                 28                 530            0           0           3
#> 499                 16                 710            0           5           1
#> 500                 26                 545            0           1           1
#> 501                  9                 670            0           2           5
#> 502                 13                 585            0           3           3
#> 503                 26                 595            0           1           1
#> 504                 26                 735            0           1           2
#> 505                 26                 730            0           1           4
#> 506                  2                 545           75           1           4
#> 507                 26                 755            0           0           3
#> 508                 35                 670            0           1           1
#> 509                 16                 775            0           1           2
#> 510                 26                 775            0           0           0
#> 511                  5                 565           99           2           6
#> 512                  8                 525            0           1           1
#> 513                  6                 720            0           1           4
#> 514                  9                 720            0           1           1
#> 515                 26                 760            0           3           6
#> 516                 26                 580          880           0           1
#> 517                 16                 670            0           1           2
#> 518                  5                 650           51           2           1
#> 519                 26                 605            0           4           3
#> 520                 26                 635            0           0           4
#> 521                 26                 660            0           1           2
#> 522                  9                 725            0           2           2
#> 523                 26                 710            0           2           4
#> 524                  6                 755            0           2           3
#> 525                 30                 605            0           2           4
#> 526                 26                 715            0           3           3
#> 527                 13                 635         2530           4           3
#> 528                 26                 680            0           0           0
#> 529                 19                 790            0           2           0
#> 530                 26                 805            0           0           1
#> 531                 26                 735            0           1          12
#> 532                 31                 700            0           0           3
#> 533                 26                 680            0           2           6
#> 534                 26                 575            0           2           1
#> 535                 18                 670          706           2           1
#> 536                  3                 730            0           3           4
#> 537                 26                 720            0           0           0
#> 538                 36                 595            0           4           0
#> 539                 26                 810            0           0           9
#> 540                 26                 580            0           3           6
#> 541                  9                 725            0           0           0
#> 542                 34                 575            0           2           9
#> 543                 26                 615            0           2           1
#> 544                 26                 780           65           1           5
#> 545                 31                 685            0           1           2
#> 546                  2                 530          590           0           1
#> 547                 26                 705            0           1           2
#> 548                 26                 600            0           0           6
#> 549                  9                 765         1077           0           0
#> 550                 19                 765            0           0           1
#> 551                 28                   0            0           0           8
#> 552                 26                 575            0           1           4
#> 553                 26                 700            0           2           2
#> 554                 16                 655            0           3           5
#> 555                 12                 555            0           0           2
#> 556                  9                 700            0           1           1
#> 557                 37                 710            0           1           6
#> 558                  9                 705            0           0           7
#> 559                 26                 750            0           2           5
#> 560                 13                 800            0           1           2
#> 561                 28                 645            0           0           5
#> 562                 19                   0            0           2           2
#> 563                 16                 740          450           0           3
#> 564                 33                 810            0           0           1
#> 565                 16                 735           57           3           5
#> 566                 16                 715            0           2           2
#> 567                 26                 790            0           1           2
#> 568                 26                 765          851           2          11
#> 569                 26                 720            0           1           7
#> 570                 26                 665            0           3           9
#> 571                  9                 670            0           3           2
#> 572                 16                 730            0           1           0
#> 573                 13                 815            0           1           3
#> 574                 33                 590            0           3           2
#> 575                 34                 685            0           1           2
#> 576                 26                 705            0           3           4
#> 577                 26                 670            0           4           1
#> 578                  3                 640            0           1           2
#> 579                  9                 790            0           0           5
#> 580                 16                 730            0           0           1
#> 581                 26                 600            0           1           3
#> 582                 15                 525            0           2           6
#> 583                 31                 645          592           3           2
#> 584                 26                 695          103           0           6
#> 585                  2                 715            0           0           0
#> 586                 26                 710            0           3           2
#> 587                  9                 715            0           1           3
#> 588                 32                 695            0           1           2
#> 589                 24                 715            0           2           2
#> 590                 16                 730            0           2           2
#> 591                 26                 645            0           1           1
#> 592                  9                 690            0           3           4
#> 593                 26                 705            0           1           7
#> 594                 31                 825            0           1           3
#> 595                 26                 700            0           1           2
#> 596                  9                 760          761           0           3
#> 597                  2                 795            0           2           2
#> 598                 16                 795            0           1           0
#> 599                 19                 740            0           0           0
#> 600                 16                 675            0           0           1
#> 601                 34                 745            0           0           0
#> 602                 16                 690            0           0           1
#> 603                 26                 690            0           0           6
#> 604                 37                 710            0           0           0
#> 605                 13                 700            0           0           4
#> 606                 26                 755            0           2           1
#> 607                  2                 705            0           0           9
#> 608                 18                 505            0           2           3
#> 609                 26                 790            0           2           1
#> 610                 31                 560            0           2           1
#> 611                 26                 690            0           0           0
#> 612                 26                 690            0           1           4
#> 613                 20                 795            0           1           2
#> 614                  9                 680            0           1           1
#> 615                 18                 555            0           3           2
#> 616                 26                 780            0           0           1
#> 617                 21                   0            0           1           1
#> 618                  9                 750            0           0           1
#> 619                 26                 775            0           0           0
#> 620                 26                 645          154           2           1
#> 621                 26                 710            0           0           3
#> 622                  3                 530            0           3           4
#> 623                 30                 720            0           2           3
#> 624                 11                 530            0           2           1
#> 625                 13                 650          800           0           1
#> 626                 18                 710            0           2           1
#> 627                 26                 765            0           1           1
#> 628                 26                 780            0           1           2
#> 629                 26                 665            0           0           1
#> 630                 16                 535          422           2           3
#> 631                  8                 555         3539           1           3
#> 632                 13                 595            0           5           3
#> 633                 26                 715            0           1           2
#> 634                 26                 745          170           1           2
#> 635                 26                 695            0           1           2
#> 636                 19                 565            0           0           1
#> 637                 16                 570            0           2           1
#> 638                 13                 700            0           1           2
#> 639                 26                 660            0           0           2
#> 640                 26                 695            0           2          19
#> 641                 18                 540            0           0           2
#> 642                 26                 705            0           0           3
#> 643                 13                 720            0           1           0
#> 644                 26                 690            0           0           2
#> 645                  9                 710            0           0           0
#> 646                 13                 635            0           2           2
#> 647                 26                 770            0           1           1
#> 648                 16                 720            0           2           3
#> 649                 26                 730            0           2           1
#> 650                 30                   0            0           1           4
#> 651                 26                 610            0           2           7
#> 652                 13                 555         1471           0           0
#> 653                 13                 695          458           3           1
#> 654                 26                 735            0           0           3
#> 655                 26                 720            0           1           3
#> 656                 31                 635            0           1           3
#> 657                 26                 790            0           1           4
#> 658                 20                 780            0           0           2
#> 659                 24                 560            0           1           4
#> 660                 26                 765           65           1           9
#> 661                  9                 765            0           0           0
#> 662                 26                 720            0           0           0
#> 663                 26                 650            0           2           2
#> 664                 30                 715            0           1           2
#> 665                 26                 740            0           1           2
#> 666                  5                 525            0           2           1
#> 667                 28                 535            0           0           1
#> 668                 13                 765            0           2           2
#> 669                 26                 615            0           3           3
#> 670                 26                 780            0           3           2
#> 671                 13                 760          371           3           1
#> 672                 26                 620            0           1          12
#> 673                 26                 740            0           1           5
#> 674                 26                 720            0           1           2
#> 675                 26                 745            0           0           0
#> 676                 26                 625            0           2           1
#> 677                 16                 690            0           2           2
#> 678                  9                 735            0           0          11
#> 679                 26                 820            0           4           1
#> 680                  5                 680            0           0           1
#> 681                 26                 720            0           1          11
#> 682                 19                 555            0           1           3
#> 683                 26                 795            0           1           1
#> 684                 37                 690            0           0           1
#> 685                 19                 725            0           1           2
#> 686                 26                 780            0           0           1
#> 687                 26                 635            0           2           4
#> 688                 26                 605            0           0           2
#> 689                  9                 700            0           1           2
#> 690                 26                 725            0           1           2
#> 691                 13                 530            0           2           5
#> 692                  9                 630            0           1           3
#> 693                 26                 670            0           2           2
#> 694                 28                 645         3026           3           2
#> 695                 26                 715            0           0           3
#> 696                 26                 770            0           0           0
#> 697                 36                 510            0           0           3
#> 698                 26                 655            0           2           5
#> 699                  9                 680            0           2           6
#> 700                 26                 700            0           0           1
#> 701                  9                 710            0           1           1
#> 702                 26                 560            0           4           3
#> 703                 11                 575            0           2           2
#> 704                 19                 615          941           2           7
#> 705                 26                 665            0           0           3
#> 706                 26                 720            0           0           2
#> 707                 37                 710            0           1           0
#> 708                  3                 685            0           8           0
#> 709                 26                 740            0           1           1
#> 710                  9                 790           63           1           1
#> 711                 34                 710            0           0           0
#> 712                  5                 665          111           0           1
#> 713                 13                 695            0           1           2
#> 714                 34                 835            0           0           0
#> 715                 26                 725            0           1           1
#> 716                 13                 810            0           0           1
#> 717                 20                 790            0           2           2
#> 718                 26                 575            0           0           2
#> 719                 26                 730            0           3           4
#> 720                 26                 740            0           3           3
#> 721                  1                 650            0           3           0
#> 722                 14                 540            0           1           9
#> 723                 26                 680            0           3           4
#> 724                 19                 825            0           0           3
#> 725                 26                 775            0           1           2
#> 726                 28                 505            0           3           7
#> 727                 32                 590            0           2           1
#> 728                 20                 740            0           1           2
#> 729                 13                 700            0           0           5
#> 730                 19                 695            0           3           1
#> 731                 36                 790            0           1           3
#> 732                 26                 655            0           1           5
#> 733                 26                   0            0           1           4
#> 734                 26                 695            0           1           2
#> 735                 26                 560            0           1           3
#> 736                 26                 585            0           2          17
#> 737                 26                 635            0           2           1
#> 738                 34                 760            0           0           1
#> 739                 34                 705            0           3           7
#> 740                  5                 740            0           2           2
#> 741                 26                 805            0           1           2
#> 742                 26                 730            0           1           1
#> 743                 26                 780            0           0           2
#> 744                 21                 550            0           3          14
#> 745                 31                 805            0           2           8
#> 746                 26                 660            0           1           3
#> 747                 37                 800            0           2           3
#> 748                 19                 735            0           0           2
#> 749                 11                   0            0           2           4
#> 750                 26                 720            0           0           1
#> 751                  1                 755            0           0           1
#> 752                 26                 685          202           0           5
#> 753                 26                 680          491           2           6
#> 754                 26                 715          203           4           9
#> 755                 13                 665            0           0           1
#> 756                 15                 700          296           4           2
#> 757                 36                 800            0           0           0
#> 758                 36                 785            0           1           3
#> 759                  6                 640            0           1           4
#> 760                 26                 530            0           4           5
#> 761                 33                 690          184           0           3
#> 762                 13                 700            0           3           2
#> 763                 13                 830            0           1           2
#> 764                 16                 780            0           0           6
#> 765                  2                 535          116           1           1
#> 766                 34                 775            0           2           0
#> 767                 26                 765            0           2           1
#> 768                 13                 720            0           1           3
#> 769                 26                 725            0           2           2
#> 770                 26                 515            0           1           3
#> 771                 15                 785            0           0           4
#> 772                 26                 755            0           3          19
#> 773                 26                 715            0           0           1
#> 774                 26                 575            0           0           3
#> 775                 13                 665           85           3           2
#> 776                 16                 685            0           0           0
#> 777                 25                 745            0           1           3
#> 778                 13                 675            0           3           1
#> 779                 26                 725            0           0           0
#> 780                 26                 725            0           1           1
#> 781                  9                 720            0           0           1
#> 782                  4                 645          216           2           2
#> 783                 34                   0            0           1           1
#> 784                 31                 680            0           1           0
#> 785                  1                 715          317           0           2
#> 786                 37                 740            0           0           2
#> 787                 19                 740            0           1           4
#> 788                  6                 770            0           0           1
#> 789                  6                 665         1436           0           2
#> 790                  9                 700            0           0           1
#> 791                 26                 665            0           2           0
#> 792                 26                 710            0           2           2
#> 793                 18                 735            0           6           2
#> 794                 26                 750            0           1           1
#> 795                 11                   0            0           1           4
#> 796                 37                 725            0           0           5
#> 797                 26                 780            0           0           9
#> 798                 26                 675            0           4           2
#> 799                 12                 720            0           1           3
#> 800                 26                 785            0           1           2
#> 801                 16                 795            0           1           3
#> 802                 13                 605            0           0           4
#> 803                 32                 765            0           1           2
#> 804                  9                 805            0           0           8
#> 805                  9                 750          249           0           0
#> 806                 26                 665            0           2           2
#> 807                  3                 595            0           3           5
#> 808                 26                 705          250           1           2
#> 809                 31                 740            0           0           1
#> 810                 19                 550            0           0           1
#> 811                 32                 725            0           1           0
#> 812                 26                 780            0           0           7
#> 813                 26                 800            0           0           1
#> 814                 26                 650            0           1           1
#> 815                 15                 675            0           1          16
#> 816                 26                 635            0           4           3
#> 817                 13                 640          643           2           0
#> 818                 26                 785            0           0           3
#> 819                 26                 680            0           1           3
#> 820                  9                 710            0           1           0
#> 821                 21                 745            0           0           1
#> 822                 26                 785            0           0           2
#> 823                 26                 650            0           0           2
#> 824                 26                 655            0           1           5
#> 825                 19                 695            0           3           9
#> 826                 26                 605            0           1           3
#> 827                 25                 685            0           2           1
#> 828                 34                 730            0           2           0
#> 829                  2                 565            0           4           3
#> 830                 36                 705            0           1           2
#> 831                 26                 780            0           1           0
#> 832                 26                 600            0           1           1
#> 833                 11                 580            0           1           2
#> 834                 18                 610           67           1           6
#> 835                 26                 700          600           0           1
#> 836                 26                 715            0           1           2
#> 837                  5                 595         1581           1           0
#> 838                 26                 720            0           0           3
#> 839                 19                 520          197           1          15
#> 840                  9                 690            0           0           6
#> 841                 26                 785            0           1           4
#> 842                  9                 800            0           0           0
#> 843                  9                 805            0           0           0
#> 844                 26                 655            0           1           2
#> 845                 26                 695            0           0           1
#> 846                 24                 705            0           0           6
#> 847                 33                 815            0           0           2
#> 848                 13                 700            0           0          13
#> 849                 37                 720           69           0           1
#> 850                 21                 510            0           1           3
#> 851                 27                 725            0           2          12
#> 852                 16                 715          833           1           1
#> 853                 26                 670            0           2           2
#> 854                  9                 625          433           2           5
#> 855                 26                 795            0           0           1
#> 856                 26                 570         1066           3           9
#> 857                  9                 645          329           3           5
#> 858                 13                 695            0           1           3
#> 859                 16                 700            0           1           7
#> 860                 26                 620            0           0           3
#> 861                  9                 715            0           2           1
#> 862                 16                 785            0           2           4
#> 863                 36                 610            0           2           2
#> 864                 16                 770            0           1           2
#> 865                  6                 660            0           4           2
#> 866                 26                 675            0           0           1
#> 867                 19                 705            0           2           4
#> 868                 36                 665            0           0           6
#> 869                  9                 540            0           3           3
#> 870                 26                 670            0           3           1
#> 871                 25                 785            0           0           2
#> 872                 14                 690            0           0           3
#> 873                 36                 745            0           0           1
#>     open_il_12m open_il_24m mths_since_rcnt_il total_bal_il il_util open_rv_12m
#> 1             0           1                 21         4981      36           3
#> 2             0           1                 19        18005      73           2
#> 3             0           4                 19        10827      73           0
#> 4             0           3                 14        73839      84           4
#> 5             0           0                338         3976      99           0
#> 6             0           2                 18        29433      63           2
#> 7             0           4                 13        27111      75           0
#> 8             0           0                 35        17493      57           2
#> 9             2           3                 10       106748      72           0
#> 10            1           2                  2        37430      67           0
#> 11            1           3                  2        22195      71           2
#> 12            0           0                 27        23413      55           0
#> 13            0           0                 47        47665      43           2
#> 14            0           1                 13        47194      58           0
#> 15            2           2                  3        25416      97           2
#> 16            0           1                 19         9358      51          12
#> 17            1           2                  9        10815      69           3
#> 18            1           2                  6        12221      91           1
#> 19            0           0                 64            0      74           0
#> 20            0           1                 19        26212      83           0
#> 21            0           0                 28        12525      35           0
#> 22            0           0                 47         3830      22           0
#> 23            0           1                 15        13175      44           0
#> 24            2           2                  3        20689      94           1
#> 25            1           4                  9        68644      74           0
#> 26            0           1                 19            0      74           0
#> 27            0           0                 27        34624      68           2
#> 28            0           1                 14        13676      75           2
#> 29            1           3                  9        82819      66           1
#> 30            2           4                  5        67459      62           4
#> 31            0           0                 45         5337      26           1
#> 32            1           1                  5       352033      74           0
#> 33            0           1                 21        10003      76           1
#> 34            0           1                 23        14807      66           3
#> 35            1           1                 10        16016      80           0
#> 36            0           0                 47        44379      84           6
#> 37            2           2                  6        32895      94           1
#> 38            2           3                  8        32532      84           3
#> 39            1           1                 11        24041      89           0
#> 40            0           2                 14        49908      97           0
#> 41            3           9                  5       269893     110           0
#> 42            0           0                 35          921      15           2
#> 43            2           4                  2       196437      95           1
#> 44            0           3                 17        25792      63           0
#> 45            1           4                  6        66773      88           1
#> 46            0           2                 22        13889      44           2
#> 47            0           0                 28        35367      49           0
#> 48            0           2                 14        18276      74           2
#> 49            0           0                 76        20690     101           1
#> 50            0           0                 52        71856     118           0
#> 51            0           0                 55        16094      49           1
#> 52            0           0                 91            0      74           2
#> 53            0           0                 35        16012      43           0
#> 54            1           2                  2         9935      43           2
#> 55            1           2                 10        58071      99           0
#> 56            0           0                 43         3656      37           2
#> 57            2           3                  3        32446      54           0
#> 58            0           1                 13        30388      83           3
#> 59            2           3                  5        36226      84           2
#> 60            2           4                  6       130740      84           0
#> 61            1           2                  3        14683      81           3
#> 62            0           0                 28        25447       6           1
#> 63            2           3                  7        34940      54           2
#> 64            0           1                 24         6234      30           1
#> 65            1           1                 10        71557      81           1
#> 66            2           3                  2        27723      94           0
#> 67            0           1                 14         8820      42           3
#> 68            0           0                 25        34454      43           3
#> 69            0           2                 16        29947      81           2
#> 70            1           2                  7        16182      91           2
#> 71            0           0                 12            0      74           2
#> 72            1           3                  5        59096      94           1
#> 73            1           2                 11        12681      71           2
#> 74            0           1                 13         6113      75           0
#> 75            0           0                 37         6600      40           1
#> 76            0           1                 17        22705      52           0
#> 77            1           4                  3        36812      79           1
#> 78            0           0                 54        55536      74           2
#> 79            0           2                 17         6468      22           0
#> 80            0           1                 13       136887      86           4
#> 81            1           1                 12        20907      64           1
#> 82            2           3                  6        94902      76           2
#> 83            1           3                  9        35380      82           0
#> 84            1           1                 12            0      74           3
#> 85            0           0                 45        96244      54           2
#> 86            0           0                 62        15913      68           1
#> 87            0           1                 13         3174      20           2
#> 88            0           0                 50        14477      74           1
#> 89            0           1                 18        21378      73           1
#> 90            0           0                 12            0      74           2
#> 91            2           3                  1        65001      61           1
#> 92            1           1                  5         7478      36           1
#> 93            1           1                  9        18047      84           2
#> 94            2           2                  7        49789      89           0
#> 95            0           1                 19        32262      60           0
#> 96            2           2                  8        40694      70           1
#> 97            0           1                 18        21661      68           1
#> 98            0           0                 28        15145      56           2
#> 99            2           5                  1        37571      89           1
#> 100           2           5                  3       223030     106           0
#> 101           2           3                  1       118276      88           3
#> 102           1           3                  3        36504      91           0
#> 103           0           1                 16         8268      64           2
#> 104           2           2                  9            0      74           5
#> 105           0           0                 46            0      74           7
#> 106           0           2                 13         1720      34           2
#> 107           0           1                 13        23747      85           1
#> 108           1           3                  5        30789      96           2
#> 109           1           2                  6        36551      92           3
#> 110           2           2                  8        59374      87           2
#> 111           1           1                  2        36538      59           2
#> 112           2           6                  3        52062      87           1
#> 113           2           3                  2        55161      92           0
#> 114           0           1                 21         6743      67           1
#> 115           2           3                  7       208715      80           5
#> 116           1           2                  9        70178      76           3
#> 117           2           4                  3        34276      81           3
#> 118           3           4                  1        17405      58           0
#> 119           5           8                  3        83999      88           2
#> 120           1           2                  8        17980      85           0
#> 121           4           6                  3        47073      86           0
#> 122           0           1                 21        22086      74           1
#> 123           0           0                 63         5927      74           3
#> 124           0           0                275            0      74           0
#> 125           1           1                 11        15051      88           2
#> 126           0           2                 14        17186      71           0
#> 127           0           0                 32         7110      55           1
#> 128           3           4                  1        47691      90           0
#> 129           0           0                141        42475      74           2
#> 130           1           3                  6        12923      62           2
#> 131           1           1                  9        13210      91           2
#> 132           0           1                 23        15756      45           2
#> 133           0           1                 24         9480      73           1
#> 134           0           1                 19        25604      74           0
#> 135           1           1                  4        33383      76           1
#> 136           1           1                  7        42009      65           2
#> 137           0           0                 59            0      74           4
#> 138           1           2                 12        10617      37           1
#> 139           0           1                 18        12162      68           0
#> 140           0           0                 29        10930      63           0
#> 141           0           1                 15        17850      64           1
#> 142           0           6                 13         8657      72           1
#> 143           1           2                  7        21499      86           4
#> 144           2           4                  6        96200      85           2
#> 145           2           2                  4        46985      99           0
#> 146           0           0                121        73862     126           4
#> 147           0           1                 17         9465      63           2
#> 148           1           2                  4        18315      63           1
#> 149           2           3                  2        31456      99           3
#> 150           1           2                  7        18450      92           5
#> 151           0           0                 38        10578      46           1
#> 152           0           0                 28            0       0           2
#> 153           1           1                  2        48949      99           2
#> 154           1           2                  8       101301      79           0
#> 155           0           0                 12            0      74           2
#> 156           0           0                 89            0      74           0
#> 157           0           0                 12            0      74           0
#> 158           2           4                  5        30775      70           0
#> 159           0           1                 14         8496      84           1
#> 160           1           1                 10        63294      74           1
#> 161           1           1                  5        28603      97           1
#> 162           1           1                  8         4237      85           3
#> 163           0           0                 27         1223      23           2
#> 164           1           2                  5        54795      57           4
#> 165           0           1                 18        29319      80           0
#> 166           1           1                  9        13968      82           1
#> 167           2           4                  4        74316      97           1
#> 168           3           5                  3        46381      38           1
#> 169           1           5                  1        30552      90           1
#> 170           1           1                  5        10128      85           5
#> 171           0           2                 15        40390      62           0
#> 172           0           0                 42        23951      54           0
#> 173           0           1                 15        32185      74           0
#> 174           0           0                 27        39393      82           1
#> 175           2           3                  2        59658      93           2
#> 176           1           3                  1        46878      62           1
#> 177           0           0                 31         1522      23           0
#> 178           0           0                 40        31335      47           0
#> 179           2           3                  1        42371      81           6
#> 180           3           3                  1        37204      90           2
#> 181           1           1                  2        50591      41           1
#> 182           0           0                 38         7215      42           1
#> 183           2           3                  2         8693      99           1
#> 184           0           0                100            0      74           0
#> 185           0           0                 53        33943      45           0
#> 186           2           3                  4       227438      64           3
#> 187           0           1                 14        23935      35           2
#> 188           0           0                103        70750      83           2
#> 189           0           0                 26         8504      57           1
#> 190           1           1                  9        23244      88           1
#> 191           1           5                  5        18420      85           4
#> 192           1           1                 12        37787      74           2
#> 193           1           1                  8        25470      78           1
#> 194           0           2                 16        25495      63           0
#> 195           3           4                  6        51359      81           0
#> 196           0           1                 24        25747      11           1
#> 197           1           1                  6        61900      84           2
#> 198           0           0                 36        11355      48           1
#> 199           1           1                  2        12929      98           0
#> 200           1           1                  5        48659      65           0
#> 201           1           1                 11        24545      59           4
#> 202           2           3                  8        30525      64           1
#> 203           2           5                  2       106263      97           2
#> 204           0           0                 33        22934      54           1
#> 205           0           0                 49         8223      39           0
#> 206           0           3                 14        34422      70           1
#> 207           0           1                 20        16772      58           1
#> 208           1           4                 11        40876      79           1
#> 209           2           3                  7        33927      58           1
#> 210           0           0                 31        11431      61           1
#> 211           0           1                 17        25810      85           1
#> 212           0           1                 24        58250      99           3
#> 213           1           2                  3        22760      50           0
#> 214           1           2                  5        14156      83           2
#> 215           0           0                118         7324      48           1
#> 216           0           0                 41        10939      49           3
#> 217           1           1                 12        24347      91           1
#> 218           0           1                 19         5568      45           0
#> 219           0           3                 14        57395      61           0
#> 220           0           0                124        35959      74           1
#> 221           1           3                  5        24808      74           2
#> 222           2           3                  2        13876      73           1
#> 223           2           5                  2        12205      69           1
#> 224           0           3                 17        11300      70           0
#> 225           0           1                 16         2955      59           2
#> 226           0           2                 16        70424      94           1
#> 227           0           1                 21        15895      80           2
#> 228           1           1                 12        72335      88           2
#> 229           1           1                  7        81500      61           2
#> 230           1           3                 12        79434      83           0
#> 231           0           0                 12            0      74           0
#> 232           0           0                 58            0      74           3
#> 233           0           0                 54       107052      74           7
#> 234           0           0                 36         6483      51           1
#> 235           1           2                  4         3624      35           1
#> 236           0           1                  4       109006      61           1
#> 237           1           2                  8        34351      85           0
#> 238           0           3                 13        32461      80           2
#> 239           1           1                  9        40194      88           1
#> 240           1           2                  8       105520      73           3
#> 241           0           0                 34         9170      68           1
#> 242           1           2                 12        43182      88           1
#> 243           0           1                 23        11138      45           2
#> 244           2           3                  9        70937      87           4
#> 245           0           0                 51        24073      74           2
#> 246           4           6                  1        64321      93           2
#> 247           1           1                  9       108609      87           0
#> 248           1           3                 10        24567      67           4
#> 249           2           2                  3         8749      89           2
#> 250           0           0                 98            0      74           2
#> 251           0           1                 20        31476      74           0
#> 252           0           0                 25        29444      63           2
#> 253           0           2                 19        36026      74           1
#> 254           2           2                 10        43906      82           3
#> 255           1           1                  5        49339      65           3
#> 256           3           4                  4        33182      79           2
#> 257           1           3                  9        11932      88           0
#> 258           1           2                  3        50834      65           3
#> 259           1           2                  9         5179      27           0
#> 260           2           3                  5        48624      63           1
#> 261           1           2                  6        11015      48           0
#> 262           0           1                 24         1713       9           1
#> 263           1           2                  7        22954      87           2
#> 264           1           1                  7        17141      82           0
#> 265           0           0                 28        12402     112           0
#> 266           0           0                 95            0       0           3
#> 267           1           2                  5        15828      85          10
#> 268           0           0                 27        13743      51          11
#> 269           2           3                  6        45670      88           2
#> 270           0           0                 29        30043      74           1
#> 271           0           1                 13        35281      63           1
#> 272           3           4                  7        33157      90           1
#> 273           1           1                  5        70998      98           5
#> 274           0           1                 14        11744      76           6
#> 275           0           0                168            0      74           2
#> 276           2           2                  7        25509      58           3
#> 277           0           0                 27         2847      29           1
#> 278           0           0                 77            0      74           0
#> 279           2           3                  3        42319      82           0
#> 280           2           3                  4        55721      87           0
#> 281           0           1                 23        37438      80           1
#> 282           0           1                 16        38183      74           4
#> 283           0           0                 28         6192      60           3
#> 284           2           5                  4        57530      89           2
#> 285           0           2                 16        27784      55           2
#> 286           3           5                  3        80191      85           2
#> 287           1           4                 12       249265      61           2
#> 288           0           0                 25         7239      66           3
#> 289           1           1                  9        75669      74           1
#> 290           2           2                  6            0      74           5
#> 291           0           1                 14         8159      68           7
#> 292           2           5                  6        66922      77           1
#> 293           1           2                 10        13511      84           1
#> 294           1           3                  9        28036      63           0
#> 295           1           2                  7        25921      54           1
#> 296           0           0                 29        12968      63           3
#> 297           2           2                 10        27595      75           0
#> 298           0           1                 21            0      74           0
#> 299           1           1                  7        33353      73           5
#> 300           0           0                 82            0      74           0
#> 301           1           4                  7        15347      73           2
#> 302           1           2                  9         9312      44           0
#> 303           1           1                  6        26237      82           2
#> 304           1           4                  2        75502      92           0
#> 305           0           1                 21        23084      55           0
#> 306           2           3                  1        20011      82           4
#> 307           0           0                 39         2440      24           1
#> 308           1           5                  1        58350      69           5
#> 309           1           2                  8         7489      83           0
#> 310           2           3                  6            0      74           1
#> 311           1           3                  6        60593      58           1
#> 312           1           1                 11        11227      83           0
#> 313           0           1                 22        20348      72           3
#> 314           1           6                  7        82446      91           3
#> 315           0           3                 19        42265      74           2
#> 316           0           3                 15        61222      98           1
#> 317           2           3                  4        56532      77           3
#> 318           0           4                 20        91643      64           0
#> 319           1           1                  7        66006     116           1
#> 320           1           2                  4        12940      40           0
#> 321           0           0                 61            0      74           5
#> 322           0           0                 77            0      74           0
#> 323           0           1                 18            0      74           1
#> 324           0           0                 25         5682      56           1
#> 325           3           4                  2       124312     104           0
#> 326           3           4                  4        15574      62           2
#> 327           1           1                  8         9172      87           2
#> 328           3           6                  3       115011      86           4
#> 329           0           1                 13        11413      71           0
#> 330           0           1                 15         3767      52           2
#> 331           1           1                  2        29429      85           2
#> 332           4           7                  4       105137      96           1
#> 333           3           6                  3        48247      74           1
#> 334           0           0                145            0      74           5
#> 335           4           5                  3       103519      92           1
#> 336           0           1                 18        35938      69           0
#> 337           2           2                  8       105731      94           1
#> 338           3           3                  3        26891      93           2
#> 339           0           0                 47        75416      74           3
#> 340           0           0                 41         3006      41           3
#> 341           0           0                 32        14395      49           0
#> 342           1           2                  5       289020      96           1
#> 343           0           2                 21        28315      61           1
#> 344           2           3                  4        47242      67           1
#> 345           1           2                 12        57960      83           2
#> 346           0           2                 19        30892      75           2
#> 347           0           1                 16        15833      63           3
#> 348           0           2                 15        21075      68           1
#> 349           0           4                 13        28362      84           1
#> 350           0           0                 28        31618      59           0
#> 351           2           2                  4        36461      85           0
#> 352           2           3                  3       113867     126           1
#> 353           1           2                 12        35638      70           0
#> 354           2           2                  4        67283      78           4
#> 355           1           1                  5        20847      93           0
#> 356           0           0                 26        17427      35           2
#> 357           1           3                  7        32138      82           1
#> 358           1           1                  9        80195      64           1
#> 359           1           1                  2        26748      66           1
#> 360           4           4                  1        19558      96           1
#> 361           0           0                 30         8238      59           0
#> 362           0           0                129            0      74           3
#> 363           0           0                119            0      74           1
#> 364           0           0                 43        10585     111           3
#> 365           1           3                 10        40022      76           0
#> 366           0           1                 20        12031      43           0
#> 367           1           1                  2        36884      86           0
#> 368           1           5                  6        11833      74           1
#> 369           2           3                  3        37722      93           0
#> 370           0           1                 15        22817      84           0
#> 371           0           1                 14        43148      81           3
#> 372           0           0                 31        27314      74           1
#> 373           0           1                 19        75987      42           2
#> 374           0           0                 12            0      74           4
#> 375           3           4                  7        98697     100           1
#> 376           0           2                 16        41822      78           2
#> 377           0           1                 22        11876      44           1
#> 378           0           0                 71            0      74           2
#> 379           0           0                 53        12607     126           1
#> 380           0           0                 25         9500      51           1
#> 381           1           1                 10        82665      88           4
#> 382           0           0                111         6680      35           1
#> 383           1           1                 11        15545      83           1
#> 384           0           1                 16         6862      86           0
#> 385           1           5                  8        87311      92           1
#> 386           1           1                  2        39170      97           4
#> 387           1           1                  6        15660      91           0
#> 388           1           2                  3        39278      76           0
#> 389           0           0                 30        22525      74           0
#> 390           0           0                 32         4403      18           1
#> 391           1           3                  1          989      56           0
#> 392           1           6                  4       180201     109           4
#> 393           1           1                  7        16055      92           9
#> 394           1           2                  5        25459      69           0
#> 395           1           3                 12        44909      59           1
#> 396           0           0                 51         3179      20           1
#> 397           1           1                 12        65600      74           0
#> 398           2           3                  8        24295      95           1
#> 399           0           1                 15        10152      77           3
#> 400           1           2                  7        13974      55           0
#> 401           2           5                  8        46030      91           2
#> 402           0           1                 22        12415      72           0
#> 403           0           1                 14        31882      60           1
#> 404           0           1                 13        91913      62           1
#> 405           1           3                 10        31059      78           2
#> 406           1           1                 12        31921      86           5
#> 407           2           2                  6        29427      84           1
#> 408           0           1                 13            0      74           0
#> 409           1           1                  5        12624      90           2
#> 410           2           2                  9        29806      61           0
#> 411           1           1                  3        25115      39           1
#> 412           0           0                 45        15873      38           0
#> 413           0           2                 16        39209      97           4
#> 414           2           3                  1        39782      74           2
#> 415           0           1                 13        54420      90           3
#> 416           1           2                 12        55563      74           0
#> 417           1           3                  8        38631      74           2
#> 418           1           1                  9        17635      39           0
#> 419           1           5                  9       120292      88           0
#> 420           2           2                  4        50103      74           6
#> 421           0           0                 25        21565      62           4
#> 422           0           1                 15         5516      63           2
#> 423           1           3                  6        47723      98           0
#> 424           1           2                  5        10550      93           2
#> 425           0           1                 22         4326      39           3
#> 426           0           3                 16        59321      60           1
#> 427           0           0                 55         9101      23           2
#> 428           0           3                 13        24876      71           3
#> 429           0           0                 32        24818      46           1
#> 430           0           0                 62            0      74           3
#> 431           0           3                 13        32092      72           2
#> 432           0           0                129            0      74           1
#> 433           0           1                 19        23613      71           4
#> 434           0           0                 12            0      74           1
#> 435           2           3                  3        83964     100           0
#> 436           0           2                 21        49515      95           3
#> 437           1           1                  7        41735      93           1
#> 438           0           0                115            0      74           3
#> 439           0           1                 17         7501      75           0
#> 440           1           3                  8        27168      92           0
#> 441           0           1                 14        62372      71           2
#> 442           0           1                 19        15136      46           2
#> 443           0           0                 29        15037      66           2
#> 444           1           3                  9        20117      60           0
#> 445           0           0                 31          602      10           4
#> 446           1           1                  3        62037     112           1
#> 447           2           4                  3        30750      68           2
#> 448           4           4                  1        41677      94           1
#> 449           0           1                 20            0      74           0
#> 450           0           0                 41       101538      91           0
#> 451           0           1                 14        24967      71           2
#> 452           2           5                 10        48582      75           0
#> 453           1           2                  9        71843      76           1
#> 454           1           1                  8        51228      70           2
#> 455           0           3                 15        76325      83           1
#> 456           0           0                 34        10614      38           2
#> 457           1           3                  9        34177      81           1
#> 458           0           0                 67            0      74           0
#> 459           0           0                 25        15709      68           3
#> 460           0           2                 16        45013     108           4
#> 461           0           1                 14       124311      42           1
#> 462           0           0                 36        33879      74           1
#> 463           0           0                 49            0      74           0
#> 464           0           1                 22        40150      53           0
#> 465           1           1                  1        15944     100           0
#> 466           0           2                 21       116281      18           2
#> 467           2           3                  9        21852      91           2
#> 468           1           2                  8        23309      55           2
#> 469           0           1                 20       203742      77           1
#> 470           0           0                 40        13377      56           0
#> 471           1           3                 10        43145      70           1
#> 472           2           3                  1        59222      76           1
#> 473           1           5                  9       131504      96           0
#> 474           2           3                  2         6356      98           0
#> 475           0           0                 93            0      74           1
#> 476           0           1                 14         2712      27           0
#> 477           0           0                151            0      74           1
#> 478           0           1                 20        20381      78           1
#> 479           2           2                 10         8564      43           3
#> 480           1           2                  9        16483      77           2
#> 481           0           0                 41        87425      48           1
#> 482           1           2                 12        26093      67           0
#> 483           3           4                  3        44567      81           4
#> 484           1           2                  1        18165      41           3
#> 485           0           1                 14        29012      65           1
#> 486           0           0                 44        27060      45           0
#> 487           0           1                 14         9645      31           3
#> 488           2           4                  1        39430      59           0
#> 489           1           2                  6       117716      98           0
#> 490           0           1                 18        53567      74           0
#> 491           0           1                 15        33222      73           0
#> 492           1           2                  9        21218      94           1
#> 493           1           2                  3        41293      95           3
#> 494           0           0                 28        12317      47           1
#> 495           0           1                 15        19793      57           3
#> 496           0           4                 15       103113      51           1
#> 497           2           3                  3        40904      59           0
#> 498           0           2                 15        41514      64           2
#> 499           1           1                  1         5500     100           3
#> 500           1           1                  5        45066      74           1
#> 501           2           5                 10        59950      83           6
#> 502           1           2                  6        13385      76           2
#> 503           0           0                 29        10432      54           2
#> 504           1           2                 11       150138      77           1
#> 505           2           3                  3       113723      92           0
#> 506           1           2                  1        48585      91           1
#> 507           0           2                 13        37359      53           0
#> 508           1           1                  2        11159      98           0
#> 509           0           0                 32       126780      49           4
#> 510           0           0                238            0      74           0
#> 511           1           1                  7        58269      99           3
#> 512           0           0                 37         5283      50           3
#> 513           1           3                  8        20477      48           1
#> 514           0           0                 31        16592      61           4
#> 515           0           1                 20        52452      68           5
#> 516           0           1                 16        11933      58           1
#> 517           0           1                 20        14819      53           1
#> 518           1           1                  2         6946      99           1
#> 519           2           5                  1        25363      71           5
#> 520           0           1                 24        86075      76           0
#> 521           0           1                 16         7706      41           1
#> 522           0           1                 15        16497      37           5
#> 523           3           4                  2        48554      91           1
#> 524           2           2                  3        70666      94           0
#> 525           2           2                 11        20386      47           2
#> 526           2           3                  6        24062      45           4
#> 527           2           3                  2        56058      98           6
#> 528           0           0                148            0      74           1
#> 529           0           0                170            0       0           2
#> 530           0           0                 28        13222      63           0
#> 531           1           2                  9        85270      98           1
#> 532           0           1                 15        30713      65           1
#> 533           3           5                  5        62286      81           0
#> 534           1           1                  5        20880      97           0
#> 535           0           1                 19        11243      72           3
#> 536           1           3                  7        70560      73           4
#> 537           0           0                 12            0      74           1
#> 538           1           3                  4            0      74           5
#> 539           0           2                 13       174034      86           0
#> 540           2           2                  1       108232      72           1
#> 541           0           0                104            0      74           1
#> 542           1           3                  1        64911      81           1
#> 543           0           1                 15         2583      19           7
#> 544           1           4                  2        27024      68           3
#> 545           1           1                 10       120507      94           1
#> 546           0           1                 18        18414      84           2
#> 547           2           3                 10        25903      85           2
#> 548           3           4                  7        33706      50           1
#> 549           0           0                 12            0      74           3
#> 550           0           1                 13        17907      72           0
#> 551           1           2                  8        33969      73           1
#> 552           0           2                 16        43501      65           3
#> 553           2           3                  4        20640      96           0
#> 554           6           9                  4       100689      87           4
#> 555           0           0                 32        10364      35           0
#> 556           1           1                  5         9777      95           0
#> 557           0           4                 15        54840     106           1
#> 558           0           1                 16        26208      48           2
#> 559           1           3                  6        67461      82           1
#> 560           1           2                 12        20892      80           1
#> 561           0           0                 41        46430     105           0
#> 562           1           1                 10        13725      74           2
#> 563           0           1                 17        36332      77           1
#> 564           0           0                 59         6950      23           0
#> 565           2           5                  4        31449      65           5
#> 566           1           1                  9        17168      59           2
#> 567           1           3                  6        49512      94           1
#> 568           2           5                  4       155212      94           0
#> 569           0           2                 15        69751      76           1
#> 570           2           6                  3        51865      79           1
#> 571           1           1                  2        43711      56           4
#> 572           0           0                 44            0      74           1
#> 573           0           0                 29        19656      33           2
#> 574           1           2                  7        21101      85           3
#> 575           1           1                 10        21383      75           0
#> 576           1           1                  2        28175      55           2
#> 577           0           0                 29         4850      43           4
#> 578           1           1                 10        50257      65           3
#> 579           1           2                 11        63399      69           1
#> 580           1           1                  9         4998      81           0
#> 581           0           1                 16         4681      42           4
#> 582           2           3                  3        92313      89           0
#> 583           2           2                  4        77217      98           3
#> 584           1           2                  7        49710      48           1
#> 585           0           0                 12            0      74           1
#> 586           2           2                  9        57478      89           4
#> 587           2           3                  6        39980      85           2
#> 588           1           3                 12        16676      61           1
#> 589           1           2                  4        31109      80           1
#> 590           0           0                 26         9326      26           2
#> 591           0           1                 13         3052      76           2
#> 592           2           6                  1       116813      44           1
#> 593           1           3                 10        46371      77           2
#> 594           0           0                 43        27511      53           1
#> 595           0           1                 14        12311      64           1
#> 596           2           2                  8       221434      86           1
#> 597           1           4                  3        64702      73           4
#> 598           0           0                 29            0      74           3
#> 599           0           0                 79            0      74           0
#> 600           1           1                 11        13182      78           0
#> 601           0           0                 57            0      74           1
#> 602           1           1                 11        14149      87           0
#> 603           0           0                 87        28194      54           0
#> 604           0           0                 12            0      74           2
#> 605           1           4                  9        40989      67           0
#> 606           0           0                 31        13450      68           2
#> 607           0           0                 32        98566      99           1
#> 608           2           4                  6        29507      67           2
#> 609           0           1                 14        21369      86           2
#> 610           1           3                  2         5070      95           2
#> 611           0           0                 74            0      74           1
#> 612           2           4                  8        76449      86           2
#> 613           0           1                 15        46086      71           3
#> 614           0           2                 13       103098      74           1
#> 615           2           2                  4        21814      92           3
#> 616           0           1                 16        10257      56           0
#> 617           0           1                 19        17336      78           0
#> 618           1           1                  9        12151      75           0
#> 619           0           0                 12            0      74           1
#> 620           1           1                  8         5711      87           3
#> 621           1           2                 10        42889      68           0
#> 622           4           4                  2        55636     104           3
#> 623           0           3                 13        12349      52           3
#> 624           0           3                 17        10963      43           6
#> 625           0           0                 47        15026      74           0
#> 626           1           2                  1        17000     100           1
#> 627           0           0                 31        12013      53           2
#> 628           2           2                  4        45642      90           0
#> 629           0           0                 25        19880      71           0
#> 630           1           1                  3        33434      82           6
#> 631           2           3                  2        31364      86           0
#> 632           2           3                  1        61634      95           2
#> 633           1           4                  4        53149      93           0
#> 634           1           2                  5        31707      83           2
#> 635           0           0                 26         5054      28           1
#> 636           1           2                 10        21042      89           2
#> 637           0           1                 19        24083      82           4
#> 638           1           3                 12        22239      82           1
#> 639           0           0                 34         1569      17           0
#> 640           2           2                  6        56681      99           1
#> 641           0           1                 23        14744      67           0
#> 642           1           3                  7        95752      47           1
#> 643           0           0                 12            0      74           1
#> 644           1           1                 11        27475      75           0
#> 645           0           0                101            0      74           0
#> 646           1           3                  1        53282      91           2
#> 647           1           1                  1        13372      92           2
#> 648           1           2                  1        35679      75           4
#> 649           1           3                 10        16113      74           1
#> 650           2           2                 11        71481     108           1
#> 651           0           0                 32        13676      48           2
#> 652           0           0                 12            0      74           1
#> 653           1           2                  5        33119      74           3
#> 654           0           1                 14        28320      60           0
#> 655           1           3                  9        20651      90           1
#> 656           0           1                 19        11288      15           1
#> 657           1           3                 12        41966      83           2
#> 658           0           0                 31        13349      44           0
#> 659           1           1                  3        17891      83           0
#> 660           1           1                  3        42674      83           1
#> 661           0           0                158            0      74           1
#> 662           0           0                 12            0      74           0
#> 663           0           1                 19        29156      76           3
#> 664           1           2                  1        36278      92           0
#> 665           0           0                 42        23183      68           3
#> 666           1           1                  8          630      50           4
#> 667           0           0                 36         1123       3           0
#> 668           0           1                 13        41492      94           3
#> 669           1           2                  7        51502      73           4
#> 670           3           3                  2        43780      96           0
#> 671           1           2                  3        31943      97           4
#> 672           0           4                 19        41368      93           1
#> 673           0           1                 13        28192      72           1
#> 674           0           0                 31        24960      54           1
#> 675           0           0                107            0      74           0
#> 676           1           1                  2         6897      98           2
#> 677           0           2                 17        36094      64           1
#> 678           0           1                 19        46644      56           0
#> 679           1           2                  3        21215      99           4
#> 680           1           2                  7        14495     100           2
#> 681           0           1                 22       129056     105           1
#> 682           1           2                  2        29094      87           0
#> 683           0           0                 34         2790      13           1
#> 684           1           1                  7        22056      92           0
#> 685           1           1                  6        14982      53           0
#> 686           0           0                 27         3249      33           0
#> 687           1           2                  4       120272      84           2
#> 688           0           1                 17        26761      78           0
#> 689           0           2                 19        24660      69           1
#> 690           1           1                  7        18023      50           1
#> 691           3           5                  4       117737      81           2
#> 692           0           1                 13        74722      68           2
#> 693           0           1                 13        39127     106           4
#> 694           1           2                  5         9922      65           3
#> 695           0           0                 33         9043      51           0
#> 696           0           0                 25            0       0           0
#> 697           1           2                 10        57545      77           2
#> 698           1           2                  6        24607      41           1
#> 699           3           8                  3       151723      86           1
#> 700           1           1                 12        10837      72           4
#> 701           0           0                 44        12286      43           3
#> 702           0           0                 67          923      39           4
#> 703           1           2                  7        44033      60           1
#> 704           0           0                 40        44821     114           3
#> 705           1           1                  9        75354      89           0
#> 706           0           1                 23        16542      55           0
#> 707           0           0                100            0      74           1
#> 708           1           1                  7            0      74           7
#> 709           2           4                  8        16809      84           2
#> 710           0           1                 24         2881      44           1
#> 711           0           0                101            0      74           0
#> 712           0           1                 14        26453      85           1
#> 713           2           2                  6        30129      92           1
#> 714           0           0                151            0      74           0
#> 715           0           0                 39         7586      39           2
#> 716           0           0                 78        12908      41           0
#> 717           1           2                  1        21632      62           3
#> 718           1           1                 10        38751      56           0
#> 719           2           4                  3        29558      72           1
#> 720           0           1                 18        39347      66           2
#> 721           0           0                 12            0      74           4
#> 722           0           0                 28        32319      98           1
#> 723           3           3                  2        42746      77           2
#> 724           1           3                 10        38283      85           0
#> 725           0           1                 14         3343      24           2
#> 726           1           2                  7        42106      60           7
#> 727           0           0                 43         5150      52           3
#> 728           1           1                  5        37366      96           1
#> 729           0           2                 17        63345      71           0
#> 730           1           1                  4        11155      86           2
#> 731           2           2                 12        51996      52           1
#> 732           3           3                  2        49739      90           0
#> 733           1           3                  4        39862      53           0
#> 734           1           3                  8        64509      85           1
#> 735           1           2                  5        53598      96           0
#> 736           5           5                  5        87664      90           0
#> 737           0           0                 25        15570      71           4
#> 738           0           0                230         8524      45           1
#> 739           1           1                  1        59960     110           4
#> 740           2           3                  4        24855      92           1
#> 741           1           1                  7        14773      67           1
#> 742           0           0                 25        19959      60           2
#> 743           1           1                 12        20675      79           2
#> 744           0           3                 16       141286     104           2
#> 745           0           0                 41       102386      91           2
#> 746           2           3                  5        50720      77           0
#> 747           1           2                 11        34765      59           2
#> 748           0           2                 20        23492      71           0
#> 749           2           3                  3        77582      57           3
#> 750           0           1                 13        29778      87           0
#> 751           0           1                 16        18140      83           1
#> 752           1           1                  8        18461      63           1
#> 753           2           7                  4       103331      91           1
#> 754           2           2                  7       124152     122           6
#> 755           1           1                  7        25038      95           0
#> 756           1           2                  2        31748      86           3
#> 757           0           0                133            0      74           1
#> 758           2           3                  4        48690      61           0
#> 759           1           1                  7        77534     101           2
#> 760           4           5                  3        33419      83           0
#> 761           0           2                 13        29962      79           1
#> 762           2           2                  5        51676      95           1
#> 763           1           3                  7        44291      67           2
#> 764           1           5                  9        85254      69           1
#> 765           1           2                  9         7914      94           1
#> 766           0           0                 45            0      74           1
#> 767           0           1                 16        19141      66           4
#> 768           2           3                  9        83177      70           2
#> 769           2           3                  3        28771      87           1
#> 770           2           5                  7        87919      89           3
#> 771           1           2                  7        57988      83           0
#> 772           1           2                  9       114233      89           3
#> 773           1           2                  7        16860      94           0
#> 774           0           1                 17        54133      53           0
#> 775           1           1                  5        34624      68           3
#> 776           0           0                 62            0      74           0
#> 777           1           1                  8        59790      57           3
#> 778           0           0                 42        13169      45           3
#> 779           0           0                136            0      74           0
#> 780           1           1                  3         8282      95           0
#> 781           0           0                 53         7567      74           0
#> 782           2           2                  5        16882      95           4
#> 783           0           0                 39        12891      54           1
#> 784           0           0                 34            0      74           1
#> 785           0           1                 13        10750      42           0
#> 786           0           1                 13        19989      65           1
#> 787           1           3                  8        67775      66           2
#> 788           0           1                 16        60489      66           0
#> 789           1           2                 12        38220      74           0
#> 790           0           0                 25        17773      63           0
#> 791           0           0                 89            0      74           2
#> 792           1           2                  9        53130      15           4
#> 793           1           2                 12        88733      87           6
#> 794           0           1                 22        32394      74           2
#> 795           3           4                  1        17964      87           0
#> 796           1           2                 12        28301      50           0
#> 797           0           0                 34       116294      95           1
#> 798           0           2                 20       314761      79           5
#> 799           0           0                 41       100282      64           1
#> 800           1           1                  2        32120      93           2
#> 801           1           1                  4        38390      50           0
#> 802           0           0                 26        34444      82           0
#> 803           1           2                 12        16934      76           1
#> 804           1           1                 11        90737     108           0
#> 805           0           0                 12            0      74           0
#> 806           0           2                 13        15006      55           5
#> 807           2           3                  3       130634      82           2
#> 808           0           1                 22        12595      46           2
#> 809           0           1                 19         3708      74           0
#> 810           0           1                 17         7536      63           1
#> 811           0           0                114            0      74           2
#> 812           1           2                 11        92061      83           1
#> 813           0           0                 46         6541      82           2
#> 814           1           1                  2        16223      99           0
#> 815           1           4                  4       126422     122           1
#> 816           3           5                  3        29521      92           1
#> 817           0           0                100            0      74           4
#> 818           0           1                 15        82665      68           0
#> 819           0           0                 28        50964      58           2
#> 820           0           0                 34            0      74           2
#> 821           0           0                 60        15785      65           2
#> 822           0           0                 25        27183      50           0
#> 823           0           0                 29        59240     104           1
#> 824           0           3                 16       111627      88           2
#> 825           4           7                  5       148335      69           0
#> 826           2           2                 10        44060      89           7
#> 827           1           1                 10        29668      92           3
#> 828           0           0                 90            0      74           6
#> 829           2           2                  1        87911      87           3
#> 830           0           1                 17        10030      54           1
#> 831           0           0                150            0       0           3
#> 832           0           1                 14        15279      85           1
#> 833           0           2                 15        53970      84           2
#> 834           2           4                  9       100216      63           1
#> 835           1           1                 12        14078      83           1
#> 836           1           2                  3       132360      99           0
#> 837           0           0                 61            0      74           3
#> 838           0           1                 15        11592      58           0
#> 839           2           3                  6       119253     108           0
#> 840           0           1                 16         8905      44           0
#> 841           2           3                  9        38563      72           1
#> 842           0           0                118            0      74           0
#> 843           0           0                 66            0      74           0
#> 844           1           1                  8         6644      31           2
#> 845           0           0                 27        15396      57           0
#> 846           0           0                 40       302393     132           0
#> 847           0           0                 29        19066      44           0
#> 848           0           0                 38       132367     132           1
#> 849           0           1                 13         3044      21           0
#> 850           1           2                  3        23458      46           0
#> 851           1           2                  6        29722      43           1
#> 852           0           1                 21        15865      65           3
#> 853           0           2                 16        23685      63           6
#> 854           2           2                 11       139765      74           3
#> 855           0           0                 61         5474      19           0
#> 856           1           2                 12        50595      92           5
#> 857           4           4                  3        68087     105           4
#> 858           0           2                 14        55539      69           3
#> 859           1           1                  2        46164      94           0
#> 860           1           2                  9        91960      98           0
#> 861           0           1                 16         6743      62           5
#> 862           1           3                  4        40297      65           1
#> 863           3           4                  4        25727      81           0
#> 864           1           2                  3        24109      97           1
#> 865           0           1                 19        48913      49           4
#> 866           1           1                 11        12127      77           0
#> 867           2           8                  1        81346      83           1
#> 868           0           1                 17       103796     101           1
#> 869           2           5                  2        19076      72           1
#> 870           2           2                  5         7050      94           4
#> 871           0           2                 21        14089      41           0
#> 872           0           0                 28        27944      77           0
#> 873           1           1                 11         7192      84           1
#>     open_rv_24m max_bal_bc all_util inq_fi total_cu_tl inq_last_12m avg_cur_bal
#> 1             3        722       34      3           1            4       20701
#> 2             3       6472       29      0           0            6        9733
#> 3             2       2081       65      2           5            1       31617
#> 4             7       9702       78      2           1            3       27644
#> 5             0       4522       76      0           0            0        2560
#> 6             3      13048       74      1           0            1       30030
#> 7             0        640       55      1           0            2       17700
#> 8             7       2524       46      2           0            1        1997
#> 9             2       4725       49      0           0            1       28528
#> 10            2       7386       67      0           2            0       19159
#> 11            4       1271       55      2           1            2        2014
#> 12            1       8752       61      0           1            0       13819
#> 13            2       4744       50      0           0            2        7912
#> 14            1       8937       57      1           0            1       33976
#> 15            3       7672       38      0           1            1       17089
#> 16           16        653       46      2           0            5        1051
#> 17            5      12971       51      0           4            1       15732
#> 18            4      14982       65      0           0            1        2848
#> 19            1       6881       63      0           0            1        3797
#> 20            0       4360       87      2           0            0       11884
#> 21            1       2260       45      0           1            1       12911
#> 22            0        367       22      0           0            0         815
#> 23            1       1448       15      2           0            1        1287
#> 24            3       8897       90      0           3            1       16631
#> 25            0       9347       63      2           6            1       12057
#> 26            1       5966       50      0           3            3        2429
#> 27            2        739       64      0           5            0        5683
#> 28           10       3138       61      1           1            3       17783
#> 29            2      12597       65      1           1            0       26756
#> 30            8       5447       28      2           6            3        8556
#> 31            2       6987       25      0           2            2        6136
#> 32            0       9358       87      1           0            2       74481
#> 33            3      12056       55      0           0            0       19107
#> 34            3       2445       53      1           0            2       29558
#> 35            1      14981       82      1           0            1        5356
#> 36            9       2957       61      0           0            3        2759
#> 37            1       6742       85      0           3            3       20578
#> 38            5       1600       34      0           2            0        2864
#> 39            3       3081       58      1           0            5       13788
#> 40            1       1097       89      1           3            1        5721
#> 41            3       3234      108      4           0            9       11493
#> 42            5       3818       56      0           2            3       11739
#> 43            2         48       87      6           3            5       28167
#> 44            2       4361       67      2           1            1       22864
#> 45            1       2180       71      0           0            3       23424
#> 46            6       7213       50      1           0            0        7176
#> 47            1       5020       40      1           4            0        8639
#> 48            4       3281       63      1           3            2       19881
#> 49            2       5037       78      0           0            1        6839
#> 50            1       1846       96      0           0            0        2886
#> 51            2      11556       54      0           1            0        5122
#> 52            5       4879       75      0           0            2        5032
#> 53            0       9701       60      0           4            0       37486
#> 54            3       4627       62      1           0            6       39020
#> 55            1       4952       89      1           0            5        6435
#> 56            3       4367       41      0           1            0        1835
#> 57            4       4797       60      1           9            2       14073
#> 58            6       2724       75      0           4            2        5146
#> 59            2       6833       53      1           0            1        2649
#> 60            1      12141       71      4           3            3       51015
#> 61            5       3382       52      0           1            1       15864
#> 62            1      19339       34      0           7            0       61551
#> 63            3       8284       51      2           2            1        4322
#> 64            1       9973       77      0           0            1        6406
#> 65            2       8415       76      0           1            1       12455
#> 66            2       4958       75      2           4            2        9350
#> 67            5      10538       42      2           0            3       20490
#> 68            5       4871       54      3           0            9        5505
#> 69            3       9413       82      2           2            3       18316
#> 70            6       2990       33      1           1            5        1470
#> 71            2       4917       75      0           0            0        2045
#> 72            1       4807       76      5           0            7        3432
#> 73            3       5431       63      0           0            2       13763
#> 74            4       2361       54      1           1            1        1666
#> 75            1       7392       44      1           0            2        8054
#> 76            2       3906       35      1           2            2        2927
#> 77            1       2776       62      1           0            3       25644
#> 78            6       2491       21      1           1            3       13773
#> 79            0          0       22      2           6            1       17827
#> 80            6       1984       49      2           2            0        5926
#> 81            2       3058       54      2           0            4        2128
#> 82            3       4829       65      1           3            3       25410
#> 83            1       4288       63      2           1            8       33919
#> 84            4       5509       33      2           0            3         913
#> 85            7       3046       36      0           0            5        9720
#> 86            2          0       65      0           3            1       12915
#> 87            3       2701       30      4           1            2        1150
#> 88            1      12992       52      1           0            5        3068
#> 89            2       9131       64      1           0            3        5394
#> 90            5       4440       27      0           0            0         812
#> 91            5       7909       42      0           0            0       10086
#> 92            2        502       24      0           1            1        1995
#> 93            2       5706       82      2           0            6        5265
#> 94            0      11535       69      0           1            1       64448
#> 95            2       1503       61      3           0            1       20321
#> 96            1       7045       42      0           3            2       44714
#> 97            2       2688       52      0           0            3       10214
#> 98            6       2715       47      0           0            1        1933
#> 99            2        173       73      3           3            3        3887
#> 100           2       5063       82      2           0            3       20010
#> 101           3       1086       41      1           0            7       28774
#> 102           1       1044       76      8           0            3        9616
#> 103           5       5200       47      1           0            3        1541
#> 104           9       5756       64      1           0            3        1682
#> 105           8       1179       36      0           0            3         409
#> 106           2       2417       61      2           0            2        1092
#> 107           1      10748       70      1           0            1       17954
#> 108           3       2106       96      0           0            0        4766
#> 109           6       1721       81      0           1            3        5121
#> 110           3       5904       85      1           3            5        7022
#> 111           5       6383       51      1           2            2        3384
#> 112           2       2739       77      1           7            1        4855
#> 113           2      11308       88      3           0            5       12954
#> 114           1        214       57      0           0            1        1561
#> 115           7      11599       55      1           0            5       40172
#> 116           3      24033       60      0           1            2       37250
#> 117           5       1309       69      3           0            5        2803
#> 118           2      12966       51      4          13            4        5619
#> 119           3       5496       76      5           0            9        4767
#> 120           3       2368       40      4           0            9        2045
#> 121           1       4265       73      1           1            0        2817
#> 122           3       9404       55      1           6            3       19371
#> 123           5       3690       47      0           2            0       17708
#> 124           1       4517       44      1           0            1       24258
#> 125           4       6822       82      0           0            2       25497
#> 126           0       6186       71      1           0            1        3625
#> 127           1       2891       61      0           1            1        2149
#> 128           0       7630       66      1           0            3        5728
#> 129           4       3402       12      0           0            0        2847
#> 130           3        791       61      0           4            4        2110
#> 131           5       2923       41      1           1            2        1590
#> 132           5       1600       21      0           0            0       27437
#> 133           1       4132       71      1           1            2       16099
#> 134           0       8369       58      0           0            3       54280
#> 135           6      10950       63      1           2            3        8432
#> 136           5      11037       53      0           8            1       11093
#> 137           5       2725       27      1           1            3       12995
#> 138           1       6692       59      1           0            1        2647
#> 139           2      12840       76      1           0            0       16813
#> 140           1      25471       82      0           0            0       10387
#> 141           1       2558       35      0           1            0        2236
#> 142           2       8479       68      0           0            0        2645
#> 143           4       9895       56      0           0            2       16910
#> 144           4       4651       76      0           0            0        8061
#> 145           1       5035       91      1           1            6       43498
#> 146           5       2799       82      0           0            0       10293
#> 147           2      13008       62      0           0            3        3939
#> 148           1       4960       61      3           1            3        8407
#> 149           6       2127       79      2           1           10       21458
#> 150          11       7622       47      0           3            1        7201
#> 151           2       3415       53      0           0            1        8926
#> 152          11        364       27      0           1            2         198
#> 153           3         64       68      0           4            2        6916
#> 154           0       2696       54      0           0            0       28824
#> 155           8       1946       28      0           0            2         493
#> 156           0       5323       16      0           0            0        8372
#> 157           0       6158       81      0           0            0        2002
#> 158           2       2256       73      2           1            1       24412
#> 159           2          0       38      0           1            1       40315
#> 160           1       7762       74      6           1            6       22858
#> 161           3       3619       65      3           0            5        4165
#> 162           5       1750       73      2           0            3       42637
#> 163           4       8082       18      0           1            0        1409
#> 164           6       9229       38      2           0            4        4987
#> 165           2      10007       75      0           0            0        3255
#> 166           2       1831       64      0           0            3        2858
#> 167           3       1547       86      1           4            0        3579
#> 168           3       8086       60      0           2            1        8783
#> 169           4       1955       83      3           6            3       27070
#> 170           9       2116       64      0           0            3        1558
#> 171           3       3948       54      1           9            0        6376
#> 172           0       5596       61      0           1            0        5327
#> 173           0       3050        7      0           0            3      113218
#> 174           4       7009       50      2          12            4        8705
#> 175           6       3415       73      6           5            9       24122
#> 176           2       7079       70      2           0            3        7237
#> 177           2       5080       20      0           0            0         663
#> 178           3       7628       51      0           3            0        8364
#> 179          11        255       68      3           3            5        2045
#> 180           7       3428       60      3           0            3        2846
#> 181           1       4286       43      3           0            6       38304
#> 182           1       6261       45      0           6            1        2929
#> 183           2       7710       63      0           0            2        2283
#> 184           0       6601       91      0           0            1       16394
#> 185           0      11918       58      2           0            2        5724
#> 186           3       4423       64      3           2            3       31139
#> 187           4       8892       50      2           2            1       12926
#> 188           2       9387       62      0           0            1        4782
#> 189           1       3452       53      5           1            6       12691
#> 190           2       5275       70      3           0            3        2134
#> 191           5       8002       41      3           2            7        5016
#> 192           2       6553       79      2           1            3        6722
#> 193           6       3043       57      6           0            5        4707
#> 194           1      10123       73      2           4            4       22471
#> 195           0       1274       79      4           0            8       29468
#> 196           4       3389       53      1           0            0        3898
#> 197           4       3692       76      0           0            4       18499
#> 198           1       7340       51      1           4            5       26098
#> 199           0      10054       85      4           1            0        4049
#> 200           0      15829       44      0           0            4       75372
#> 201           5       3107       49      0           0            2       11942
#> 202           6        880       51      1          10            2        2219
#> 203           6       2580       93      6           0            2        4210
#> 204           1       4461       58      0           2            1        3307
#> 205           0      11438       47      2           0            1       15147
#> 206           3      18952       36      0           0            0       26131
#> 207           3       1868       54      1           0            1        3348
#> 208           7       6591       73      1           0            2       18251
#> 209           1       3959       52      0           1            0        4716
#> 210           3       5987       70      0           0            0       17421
#> 211           3       4374       44      0           1            0       15943
#> 212           5       2867       31      0           0            1        5294
#> 213           1       5075       51      1           0            1        2725
#> 214           6       4500       11      2           2            5        1197
#> 215           5       6364       76      0           0            1        4240
#> 216           6       1135       14      0           0            1         796
#> 217           1      17645       61      1           5            1       15000
#> 218           1      25499       60      0           0            0       24292
#> 219           1       7865       60      3           3            2       33765
#> 220           2          0       44      0           0            2        7871
#> 221           3       7306       62      0           1            2       35834
#> 222           2       5130       48      1           3            0        2215
#> 223           2      12477       53      1           3            2        3183
#> 224           1       3761       80      0           0            0        2382
#> 225           3       4903       44      0           0            3        1621
#> 226           2       6489       90      1           0            3       11612
#> 227           3       2549       66      1           0            0        2412
#> 228           3      25225       91      0           0            0       23102
#> 229           2      12467       64      3           0            2       24443
#> 230           0      14495       78      1          10            1       18629
#> 231           0       8641       87      0           0            0        4570
#> 232           3       6706       40      2           2            7       12239
#> 233           7       3703       41      0           1            2       12664
#> 234           1        841       45      0           0            4        1934
#> 235           1       9943       68      0           7            1       23398
#> 236           1      22333       69      0           0            1       13190
#> 237           1       4727       83      0           1            2       13026
#> 238           3       5210       72      1           0            0        4677
#> 239           4       2330       46      0           0            1        4911
#> 240           7       4335       69      1           0            6        6835
#> 241           3       4621       64      0           0            2        2669
#> 242           2       4016       80      1           0            3       17893
#> 243           3       6299       23      0           2            0        2939
#> 244           8       3889       56      3           1           14       19661
#> 245           3       8548       68      0           0            2        4309
#> 246           4       1928       82      5           4           14        4198
#> 247           0       2055       81      2           0            2       19244
#> 248           4       2701       48      0           6            0        3304
#> 249           2       2699       49      1           0            3       44605
#> 250           2       9266       48      0           0            1        3146
#> 251           0       4320       17      0           3            0       58048
#> 252           2       7300       57      0           1            3        4113
#> 253           1       9374       26      2           0            1       91632
#> 254           4       5125       81      0           0            4        4558
#> 255           3       6863       57      0           0            2       15935
#> 256           3      21439       50      3           0            6       13293
#> 257           1       6960       92      1           2            2       15069
#> 258           7       3742       25      1           0            5       22028
#> 259           0       6094       52      2           0            1       28068
#> 260           1      18528       70      1           4            5        8269
#> 261           1      12721       30      0           1            3        1772
#> 262           1       7856       22      0           0            0       13586
#> 263           4       2871       72      1           4            5        2847
#> 264           1       1215       53      0           3            0       11811
#> 265           0       6873       85      0           0            0       53477
#> 266           4        258        7      1           0            2         119
#> 267          17       2541       56      1           0            6        1300
#> 268          24       2678       40      0           4            7        1370
#> 269           3       2919       83      1           4            3        8119
#> 270           3      10427       40      0           0            0       24190
#> 271           2       5906       52      0           6            0       14295
#> 272           4       1506       81      0           0            0        5439
#> 273           6       1814       42      0           0            2        7702
#> 274           6       1040       52      0           0            2        1345
#> 275           5       2593       20      1           0            2         484
#> 276           4        133       46      2           0            7        2150
#> 277           7       1930       44      0           2            1         885
#> 278           0       2976       20      6           2            8      219353
#> 279           0          0       87      1           0            2       60195
#> 280           1      11857       87      0           0            0       16528
#> 281           4        577       70      0           0            1        2753
#> 282           4       1559       74      4           1            2        4334
#> 283           4       6038       45      0          11            9       20668
#> 284           3       1693       84      0           0            4       15358
#> 285           3      10387       47      0           1            3       40057
#> 286           2       2472       57      0           1            2       10633
#> 287           3       7846       58      1           0            3       31279
#> 288           5       1579       65      1           0            2        9393
#> 289           2      12066       40      0           1            2       12855
#> 290           8       3095       30      3           4            3       28044
#> 291          17       4818       30      0           1            3        1355
#> 292           1      12854       72      0           8            0       14982
#> 293           3       8763       51      0           3            2       15270
#> 294           1       1447       34      0           0            0        2134
#> 295           7       3072       51      1           0           10        1774
#> 296           6       1151       40      1           0            9        1430
#> 297           2       3433       63      0           0            2       21702
#> 298           3       3275        7      4           2            4        1211
#> 299           6       1699       50      0           1            0        6451
#> 300           0      17577       97      0           0            0       11629
#> 301           9       1477       56      1           0            0       29050
#> 302           0       5980       42      0           1            0       42104
#> 303           2       3922       24      0           0            3       18001
#> 304           3       3852       57      2           0            2        7647
#> 305           1       2882       48      0           0            1       19899
#> 306           6      21644       64      3           1            7       23087
#> 307           2       1038       37      0           2            0        1049
#> 308          13       3737       51     16           2           16        4712
#> 309           0       9510       86      1           3            1       14976
#> 310           1       4365       20      0           2            1       18632
#> 311           2       5384       60      1           0            1        4651
#> 312           4       6082       67      0           0            0        3479
#> 313           4      15716       84      1           0            2       24659
#> 314           5          8       75      0           0            2        7972
#> 315           2       5700       25      2           4            3       76289
#> 316           2       9642       82      0           0            1       11314
#> 317           6       3966       59      3           0            6       10118
#> 318           2        896       72      1           6            1       19943
#> 319           1       1084      113      2           0            8       14346
#> 320           4       4055       49      0           1            1        2445
#> 321           9       2153       38      1           0            2         637
#> 322           0       4314       29      0           5            1       14581
#> 323           4       2635       27      0           0            1       34921
#> 324           3       3477       71      0           0            1       36228
#> 325           2      26355       94      1           6            2       10659
#> 326           3       1291       51      0           4            1        1954
#> 327           4          0       70      0           4            2        1840
#> 328           8       6005       80      0           0            4       18552
#> 329           0      34441       71      0           0            0       13019
#> 330           4       4639       43      0           0            2        1443
#> 331           5       1403       74      0           0            0        4529
#> 332           6       9925       87      1           5            1       10990
#> 333           1       6696       65      1          12            3       21343
#> 334          10       3809       20      1           0            6         534
#> 335           3       6609       62      1           3            2       20137
#> 336           1       4366       68      0           1            0       13268
#> 337           3       2738       90      0           0            6       22881
#> 338           2        605       76      2           4            5        4243
#> 339           6       3820       29      0           0            3       20142
#> 340           3       1140       19      0           0            0       14368
#> 341           0       9799       59      0           0            0        6311
#> 342           6       3059       63      1           0            1       24851
#> 343           4      13336       50      0           1            2        4095
#> 344           3       3676       67      3           8            3       25074
#> 345           4       8039       73      0           0            0        6757
#> 346           2      14075       74      3           1            2        4415
#> 347           8       2154       47      0           3            0        1425
#> 348           2       3998       34      0           0            0       10698
#> 349           2       1134       73      0           4            3       52600
#> 350           0      10121       81      0          10            0       38844
#> 351           1         55       57      1           1            5        3790
#> 352           2       3192      114      0           2            2        5023
#> 353           1       4701       57      0           1            0        5086
#> 354           5       4182       35      1           0            4       23113
#> 355           1       5824       86      0           0            3       30843
#> 356           8       5632       32      0           6            2        2726
#> 357           1       3926       76      0           5            2       32478
#> 358           2       3535       43      2           0            4       21145
#> 359           3        937       63      0           0            2       46942
#> 360           6       4279       61      0           1            1        2180
#> 361           1       3396       52      0           3            2        2659
#> 362           5       2781       10      0           3            3         528
#> 363           1      16476       62      0           0            1       10741
#> 364           3       1379       67      0           0            1        1440
#> 365           0       1522       74      1           0            0       11569
#> 366           1       6778       52      0           0            0        2758
#> 367           2       3392       75      0           0            6       18383
#> 368           3       7862       65      0           1            0        4340
#> 369           0       7429       53      3           0            5       21609
#> 370           4       1361       59      1           0            1        3468
#> 371           4      13754       55      2          14            1        5717
#> 372           4       7479        9      0           3            1        4952
#> 373           3      24698       50      0           4            0        7771
#> 374          10       1696       15      0           0            4        5022
#> 375           3       3573       87      2           5            6       10153
#> 376           4       5082       73      0           0            0       32362
#> 377           3       7796       45      4           0            4        1695
#> 378           2       2340       27      0           0            1         635
#> 379           2       1505       81      0           0            1        1694
#> 380           1       2020       59      0           1            0       13210
#> 381           6       1354       39      0           6            4       20214
#> 382           3      16167       39      0           1            2        6252
#> 383           3        967       74      2           0            4        2657
#> 384           0       2623       69      2           0            0        1717
#> 385           3       7705       74      3           0            3       10034
#> 386           7       4832       57      0           1            4        5274
#> 387           3       6911       44      0           3            0       16462
#> 388           2       1024       68      0           2            1        6056
#> 389           2       2839       78      3           1            2       25762
#> 390           3      18758       33      0           0            1       39221
#> 391           1      15568       89      0           0            0       32383
#> 392           4       2097       97      0           4            4        6187
#> 393          12       5804       23      1           0            6        1549
#> 394           1       2060       56      1           1            8       26129
#> 395           1       4734       48      0           4            1        6262
#> 396           1      15023       43      0           3            0       28481
#> 397           2       4368       75      0          18            3        8582
#> 398           2       6222       89      0           1            2       16468
#> 399           4      13797       82      0           0            0        4227
#> 400           1       6968       60      1           0            1        4519
#> 401           5       7107       74      3           4            2       14301
#> 402           2       6233       59      0           0            0        2823
#> 403           1      13028       58      0           1            0       21103
#> 404           4       4739       61      0           4            0        9348
#> 405           2       8045       75      1           7            3        5794
#> 406          11       3527       47      0           0            4        9221
#> 407           2        353       68      1           0            2       16018
#> 408           0       2776       26      0           3            5        1179
#> 409           3       1080       53      0           0            3        1348
#> 410           4       6712       67      0           1            1        3220
#> 411           2      13105       32      0           0            0       12834
#> 412           3       1193       47      1           0            3        1450
#> 413           7      12224       82      0           3            4        7112
#> 414           2        667       72      2           5            3        8142
#> 415           5       6755       67      1           0            0        3972
#> 416           3     100973       75      1           5            5       57587
#> 417           2       2878       20      2           4            6        5225
#> 418           1          0       25      2           7            2       12610
#> 419           1       5720       74      2          16            5       14009
#> 420           8       1324       28      5           0            8        5438
#> 421           6       8947       70      3           2            5       43792
#> 422           6       7075       53      1           0            4       22679
#> 423           1       8372       75      1           0            1       24666
#> 424           8       1463       77      2           4            4        1562
#> 425           3       3763       17      2           0            3       31713
#> 426           2       2816       60      0           2            0       36875
#> 427           4       9591       43      3           2            3       12429
#> 428           9       3201       56      1           1            0        2425
#> 429           3        786       48      4           0            4       51325
#> 430           4       9060       43      0           0            3        2249
#> 431           8       5911       48      0           4            2       12568
#> 432           1        974       47      0           1            1         910
#> 433           6       2819       20      3           0            2        1456
#> 434           2       4277       57      0           0            0        1257
#> 435           0       1427       91      1           1            1        6322
#> 436           4       3078       77      1           1            2        3173
#> 437           2        407       89      6           0            7        3681
#> 438           6       6940       24      0           0            1         605
#> 439           2      20374       87      0           0            0       29752
#> 440           1       9567       86      1           1            1        6209
#> 441           2       6478       62      5           0            6       28693
#> 442           4       9305       41      0           1            2        7514
#> 443           5       7972       51      0           0            1        3194
#> 444           0       5786       66      0           0            1        9881
#> 445           5       1510       31      2           0            6         685
#> 446           2      12319       88      1           2            3       18328
#> 447           3       9535       79      0           9            2       47158
#> 448           2       3667       66      3           0            5        3819
#> 449           4       1580       81      1           0            0         673
#> 450           4       4074       68      0           0            2        2942
#> 451           3      21740       84      1           0            3       33610
#> 452           1       5690       74      1           2            1       14017
#> 453           2      11562       66      0           1            1       17605
#> 454           2       6728       72      1           0            4        8543
#> 455           2        890       79      2           3            0        5402
#> 456           3       3623       50      1           2            3       28718
#> 457           4       7961       77      2           1            2        2178
#> 458           3       4267       53      0           0            0        1589
#> 459           3       2051       48      1           0            1       28477
#> 460           6       1372       57      1           0            6        2179
#> 461           6       4265       44      1           7            2       10234
#> 462           2       8550       70      0           0            0        6125
#> 463           0      22845       56      0           1            3      104652
#> 464           1       7944       65      0           0            1        6941
#> 465           0       4184       60      0           1            0        2502
#> 466           4      10606       50      0           0            1       11390
#> 467           2       3390       55      3           1            5        3242
#> 468           2       1401       62      0          16            7       17084
#> 469           1      19443       78      1           1            6       22277
#> 470           0        157       60      0           3            0        3337
#> 471           3       6210       64      2           0            5       16991
#> 472           1       4622       77      1           1            2        7006
#> 473           1          0       77      2          13            3       59448
#> 474           4       4100       20      1          14            0        1332
#> 475           1      22528       85      0           2            2       24717
#> 476           4       3281       19      0           0            1         682
#> 477           2       3469       78      0           0            0        8199
#> 478           2      18309       73      0           0            0       33360
#> 479           7       1367       24      3           0            8        1071
#> 480           3       1573       72      1           0            1        2355
#> 481           2       3691       48      0           0            1        7438
#> 482           1       2721       34      1           0            0       20085
#> 483           6       2532       64      2           1            3        5699
#> 484           7      10130       54      1           0            0        3958
#> 485           2      13855       69      1           0            0       19304
#> 486           0      12022       61      0           0            2       43828
#> 487           3       2623       29      0           0            0        1612
#> 488           3          0       54      1           4            2        5204
#> 489           3      11714       54      0           0            1       45878
#> 490           2      14159       73      0           2            0       31809
#> 491           0       6428       81      1           2            2        6034
#> 492           1       2077       86      0           0            3        4689
#> 493           3       2048       88      0           0            0       23913
#> 494           3       7124       65      0           0            0        4313
#> 495           3       4444       38      1           0            0        6493
#> 496           2       3335       43      1           0            0       49664
#> 497           1      20253       71      0           9            2       37822
#> 498           5       4951       56      1           3            1        4686
#> 499           3       1473       45      0           0            1        5763
#> 500           2       4369       69      2           2            3        6773
#> 501           6       5045       81      1           0            3        5022
#> 502           3       3377       58      0           4            1        2347
#> 503           2       2660       44      0           1            7       21167
#> 504           1      12499       64      1           3            1       50596
#> 505           2          0       53      0           2            3       13005
#> 506           7       1828       87      0           6            1       17713
#> 507           0      13783       39      0           8            0       30531
#> 508           1        504       81      1           0            2       52039
#> 509          10      10597       27      1           1            5       15010
#> 510           0       5758       77      0           0            1        2900
#> 511          10        978       82      0           0            2        3584
#> 512           8       6099       57      0           2            2        1651
#> 513           1       4017       53      2           0            1        4003
#> 514           6       3246       35      1           0            5        2548
#> 515           6       5818       68      0           0            2        4490
#> 516           2      12303       69      0           3            0       15474
#> 517           3       2986       54      3           0            0        2559
#> 518           1        327       80      2           0            5       50544
#> 519          13       5600       55      4           0            3        2953
#> 520           0       4963       74      0           1            0        7084
#> 521           4       4545       56      0           0            0        4714
#> 522           6       5432       37      2           5            7        2286
#> 523           4       4282       77      2           0            3        5030
#> 524           1       9499       67      0           0            6       31348
#> 525           5       3714       42      2           6            6       12265
#> 526           5       5863       40      1           4            1        2983
#> 527           8       2914       64      4           1            6        4863
#> 528           2       5625       52      1           0            1        1307
#> 529           5      21296       28      0           0            2       41294
#> 530           1      11275       38      1           0            1       25541
#> 531           1       5372       90      0           0            1        5224
#> 532           2       8951       69      0           0            1        5376
#> 533           0      13829       79      2           0            8       41133
#> 534           1       7663       55      0           0            1        5060
#> 535           3       8166       59      0           1            1       13403
#> 536           5       4870       60      0           6            3       28036
#> 537           3       2768       46      0           0            1        1626
#> 538           6       5773       46      3           3            6       11544
#> 539           0       4570       74      0           0            0       31685
#> 540           1       2717       34      0           0            2       10227
#> 541           2      18524       86      0           0            2       50231
#> 542           3       5882       81      0           1            2        3623
#> 543           8      12567       36      0           0            2        1721
#> 544           4       2513       67      0           5            4        3074
#> 545           3      10317       88      2           0            6       44779
#> 546           3         35       73      0           0            3       39479
#> 547           4       9177       75      2           6            5       22769
#> 548           1       9014       60      0           0            2        3818
#> 549           6       2294       39      0           0            4         445
#> 550           1      22352       45      0           0            0        3463
#> 551           3       9128       57      2           2            8        8646
#> 552           5       5497       70      2           4            5       26956
#> 553           1       2138       91      1           4            0       36783
#> 554           5      17187       81      7          11           14       16625
#> 555           1       2020       44      0           0            1       11501
#> 556           1      28431       92      0           0            2       36079
#> 557           3       8935       61      0           1            3        8522
#> 558           3      12994       56      0           0            0        3722
#> 559           2      22618       85      0           1            1       26144
#> 560           2       7050       58      0           2            8       11712
#> 561           2       1819       95      0           0            1       19030
#> 562           7       1587       60      0           1            2        1780
#> 563           1        699       78      0           1            3        5380
#> 564           1      11900       18      0           1            1       34625
#> 565           9       6668       69      2           0            8        3660
#> 566           5       5679       58      0           0            1        2692
#> 567           2       8401       40      0           0            1       11662
#> 568           1      10006       66      0           0            2       23815
#> 569           1       7206       65      0           1            0       19321
#> 570           5       4438       76      0           2            2       12789
#> 571           5      11720       82      0           0            2       30609
#> 572           1       8375       37      0           1            0      100422
#> 573           6      15346       29      1           0            2        7023
#> 574           3        922       43      3           1            3        3021
#> 575           0       7007       73      0           3            1       32161
#> 576           4       1243       42      2           6            4        5440
#> 577           6       6096       45      0           0            2        1498
#> 578           6      16897       71      1           0            3       17909
#> 579           5       5427       48      2           7            2       11264
#> 580           1       2186       59      1           0            3        1293
#> 581           6      12344       59      0           1            0        2582
#> 582           1       1458       89      4           0            8       10534
#> 583           4        716       71      4           0           20        7882
#> 584           1        928       50      0          12            0        5104
#> 585           4       8600       66      0           2            0        2040
#> 586           6       6788       45      0           0            2       19814
#> 587           4       6587       71      0           0            4       47214
#> 588           3       8478       54      0           0            0        3757
#> 589           3        550       68      1           2            3        4175
#> 590           7       6782       42      0           0            1       17326
#> 591           6       1920       50      0           0            0         818
#> 592           2       2935       59      5           6            6       25024
#> 593           3       4531       67      0           1            3        3805
#> 594           2       9183       50      0           3            0       11672
#> 595           3       1664       70      0           0            0        2161
#> 596           2       1531       46      3           0            3       13437
#> 597           5       5090       47      2           4            7       26571
#> 598           7       6579       45      2           0            2       13971
#> 599           0       2620       77      0           0            0       40618
#> 600           0       8120       85      0           0            0        9094
#> 601           1       9437       59      0           1            0       27158
#> 602           0       2397       91      0           3            1        5540
#> 603           1       8833       56      0           0            0        3392
#> 604           9       2864       24      0           1            3       12203
#> 605           0       6553       64      0           5            0       15129
#> 606           4        738       46      0           0            0        1857
#> 607           1      10296       91      1           1            0        6692
#> 608           7       3933       55      2           0            4        3547
#> 609           2        738       40      0           0            0        2098
#> 610           3       1263       48      1          12            1       13041
#> 611           2      16414       38      3           0            3       32147
#> 612           2       5377       85      1           0            4       46780
#> 613           3      11520       67      2           0            4       20813
#> 614           2       5617       16      0           0            0       12556
#> 615           5       3674       68      1           4            4       16970
#> 616           2       6077       61      2           0            1        3792
#> 617           0        166       67      2           0            8       18502
#> 618           2      11961       55      1           2            2        4745
#> 619           1       2348       17      0           0            0         870
#> 620           5       8390       39      0           8            2       12974
#> 621           2       3151       69      0           0            3       20917
#> 622           6       2842       94      3           0            5        6047
#> 623           4       6074       55      0           1            1        6065
#> 624           8       5044       21      1           3            1        2277
#> 625           1       1641       48      1           1            0       35086
#> 626           1       2004      100      0           8            3       70816
#> 627           2       1207       56      0          10            1        2428
#> 628           1      10780       69      0           1            0        9219
#> 629           0       4014       75      0           0            0       11947
#> 630           6       3653       75      1           1            8        2703
#> 631           5       5029       76      1           2            0        3273
#> 632           4       8638       49      0           0            6       37720
#> 633           0       3165       83      0           0            0       58023
#> 634           5       6010       29      1           0            2       13196
#> 635           2       1637       45      0           1            1       16399
#> 636           4       8281       68      0           2            8        4418
#> 637           4       1001       77      2           0            3        4295
#> 638           4       8571       66      1           3            0        3389
#> 639           1       2003       42      0           0            0        4334
#> 640           3       3503       78      0           0            3        2992
#> 641           0       6507       68      0           1            0        2737
#> 642           2      16327       55      0           0            1       23202
#> 643           1       4527       39      0           0            0        1213
#> 644           1       1991       45      0           0            0       21940
#> 645           1       9949       98      0           0            0       15013
#> 646           3       1596       77      7           0            7       44571
#> 647           6       4027       32      0           0            3        1448
#> 648           5       3689       44      0           5            1       11644
#> 649           1       4302       46      0           2            0       15145
#> 650           4       3161       92      4           0            7        6680
#> 651           2       3444       51      1           1            3       18723
#> 652           7        736       17      0           0            2         146
#> 653           5       5383       32      2           0            2       16831
#> 654           1       2522       50      0           2            0       20378
#> 655           2       5676       48      0           8            0       12478
#> 656           1      18160       46      0           0            2       27051
#> 657           2       4812       66      0           0            3        7885
#> 658           0       4613       40      0           2            3       15394
#> 659           1       6864       60      0           1            1        2058
#> 660           2       7257       52      1           0            4        8659
#> 661           1      10303       47      0           0            1        3802
#> 662           0      14883       34      0           1            0       49582
#> 663           6       8382       69      3           0            1       17954
#> 664           0      11173       49      1           1            3       18147
#> 665           7       3961       61      4           3            3        8158
#> 666           6       1247       55      2           0            7        7178
#> 667           0       8224       45      0           7            2       34582
#> 668           3       2412       92      1           0            1        5296
#> 669           6       5350       71      0           0            2        5777
#> 670           2       5887       50      1           8            2        2608
#> 671           6       3391       63      1           1            3        4757
#> 672           2       2915       85      7           1            1        2673
#> 673           1       2927       70      0           0            2       12447
#> 674           1      12419       71      0          10            4       52194
#> 675           0       9831       61      0           0            0        8361
#> 676           2       4713       35      1           0            2       14755
#> 677           2      21056       76      1           1            1       30024
#> 678           2       8925       53      1           0            0        3272
#> 679           4      14726       83      1           0            8       31190
#> 680           3       4880       79      2           3            5       42248
#> 681           2       1925      102      8           7            3       14033
#> 682           0       2845       71      6           1            3       37923
#> 683           1       7626       37      0           0            1       52354
#> 684           0       8142       59      0           6            1       14791
#> 685           2       5335       59      1           0            1        2258
#> 686           0       9040       18      0           0            0        2496
#> 687           3       9769       87      0           1            1       17797
#> 688           0       2987       78      0           2            0       18737
#> 689           2       7105       66      3           7            1        5874
#> 690           2       7070       39      0           0            0       17293
#> 691           6       9966       67      0           2            2       17621
#> 692           3          0       57      3           4            8       10675
#> 693           5       8459       84      0           0            1        6393
#> 694           8       4022       61      3           0            6        1567
#> 695           0       7754       72      0           2            0       35069
#> 696           1       5698       56      0           1            0        5977
#> 697          10      11357       45      0           6            0       10272
#> 698           1       6163       37      1           0            2        3035
#> 699           4       3979       81      0          17            7       13510
#> 700           6       7573       49      3           0            6       35448
#> 701           3      11207       61      1           0            3       37705
#> 702          13       1838       40      5           0            8         524
#> 703           2      11085       57      3           2            8       36720
#> 704           3       6383       79      2           2            5       21109
#> 705           0       2051       80      0           5            0       24903
#> 706           0      19563       77      1           2            0       53347
#> 707           3       4288       63      0           2            0       30380
#> 708           7        707       13      2           0            3       21417
#> 709           8       7202       51      0           1            1       25998
#> 710           2       1938       21      0           1            0         698
#> 711           1       3230       59      2           0            4       18871
#> 712           2       4172       70      2           6            2        4932
#> 713           4       4883       74      5           0            7        2812
#> 714           0       6870       26      0           0            1       24543
#> 715           4       3577       24      0           0            1        3423
#> 716           0      10220       41      0           1            0       15169
#> 717           8       1325       17      1           2            1       21325
#> 718           0       4927       27      3           1            3        6246
#> 719           1       1659       48      1           0            2        2267
#> 720           3       6007       55      0           8            1       16058
#> 721           7      14187       49      1           1            1        1643
#> 722           2        512       89      0           2            1        2985
#> 723           3       3170       61      1           2            4        3403
#> 724           0       1680       84      1           4            1        7170
#> 725           2       7774       28      0           5            2       17927
#> 726          10       2268       51      1           0            5        2710
#> 727           3        878       35      1           2            3         818
#> 728           3      21067       68      0           1            1       37060
#> 729           2      12806       71      0          12            0       15318
#> 730           4        990       57      1           0            3        1182
#> 731           1       8098       70      0           2            1       18281
#> 732           4        600       73      0          10            3        4757
#> 733           4       8566       77      0           2            3       20047
#> 734           1      18892       82      0           3            2       18040
#> 735           2       2493       78      0           2            2        6867
#> 736           1       8317       89      0           1            4        4487
#> 737           4       7259       59      1           0            3        9679
#> 738           2      10860       43      0           0            1        3994
#> 739           4       6155       63      0           0            1        4977
#> 740           2       5247       72      2           1            4        8067
#> 741           2       5513       43      1           0            4        2254
#> 742           7       4201       55      1           0            0        2883
#> 743           3      13241       49      0           6            1       16181
#> 744           4       2444       95      0           0            2       10347
#> 745           2      10120       52      0           3            2       11428
#> 746           2       7418       78      2           0            6        3900
#> 747           5       9270       48      1           1            3        4547
#> 748           0       3687       76      2           0            0        8417
#> 749           3       1825       50      0           0            1        5943
#> 750           1       5836       67      2           4            0       17108
#> 751           1       2227       34      0           0            0       49152
#> 752           1      12692       67      0           0            1        4022
#> 753           1       1484       73      4           4            5       10531
#> 754          11       6664       83      0           0            0        9051
#> 755           1       1564       79      4           0           11       27689
#> 756           4       2319       58      0           1            3        2434
#> 757           1      11061       77      0           3            1      157556
#> 758           2       3189       75      0           2            2        6267
#> 759           4        618       50      1           0            0       31555
#> 760           0       1464       80      0           7            1       12317
#> 761           2          0       73      0           2            0        4217
#> 762           2       5595       51      0           0            1        6990
#> 763           3      12330       23      0          15            2        9549
#> 764           1       4175       53      0           4            2        7985
#> 765           2       5944       37      0           0            1        1767
#> 766           1       1821       28      0           7            4       29012
#> 767           4       3578       53      0           6            1        3341
#> 768           4       6600       51      0           1            0       14675
#> 769           4       3053       77      3           0            3       14057
#> 770          11       5362       57      2           9            6        8574
#> 771           0       6384       79      2           2            5       18311
#> 772           3       4001       79      2           2            4        9275
#> 773           0       5932       91      0           0            0       16353
#> 774           0       2699       44      0           0            0       15377
#> 775           4       2749       56      0           0            2        3234
#> 776           1       2752       39      0           0            0        1203
#> 777           7       3290       29      0           1            3        7096
#> 778          13       4786       52      0           1            5       16283
#> 779           0          0       79      0           1            0        4997
#> 780           0       2764       42      0           1            1        4102
#> 781           2      15749       43      0           0            0        2922
#> 782           6       2052       54      1           3            4        8263
#> 783           1       1977       65      0           1            0        2090
#> 784           2       5214       79      2           3            2       39068
#> 785           0       7856       58      0           1            0        4299
#> 786           2       8802       60      0           0            1        5603
#> 787           3       9633       63      0           0            1       38825
#> 788           1      19308       67      0           1            1       10094
#> 789           4       6000       73      1           3            7        6141
#> 790           0      13953       65      0           0            0        8705
#> 791           2      13314       69      1           0            3       39782
#> 792           6       7592       29      0           1            4        5465
#> 793           9       3595       40      1           0            7       23010
#> 794           2       2410       78      2           0            1       20109
#> 795           3       2232       61      5           5            6        2323
#> 796           0       5811       60      0           6            0       15092
#> 797           2       7289       92      0           0            0        7517
#> 798           6       3118       66      0           1            1       40136
#> 799           1      14379       64      0           1            0       26800
#> 800           4       7254       72      0           0            4        3003
#> 801           1      15400       50      0           0            0       33227
#> 802           1       5621       71      0           6            0        3934
#> 803           1       1453       55      2           0            2       22265
#> 804           1      15450      100      4           1           12        8559
#> 805           0      10993       55      2           0            2       17348
#> 806           6       1946       61      8           2            4        3148
#> 807           4       3447       82      1           2            3       21532
#> 808           3       3008       52      1           1            3        3407
#> 809           0         94       15      2           0            6         760
#> 810           1       7058       61      0           0            1        3102
#> 811           5       4434       19      0           0            0         777
#> 812           1      22992       79      0           0            0       12607
#> 813           6       8175       61      0           0            1       25361
#> 814           0       1096       90      0           1            2        4811
#> 815           5       1040      118      0           0            1        6199
#> 816           1        649       82      0           0            4       10546
#> 817           6        552        9      0           0            3          98
#> 818           0        859       38      0           2            0        9390
#> 819           3       9514       49      0           1            2        7723
#> 820           2       3103       59      0           3            1        1255
#> 821           3       7752       45      0           0            0        4070
#> 822           0      18794       53      0           0            1       51803
#> 823           2       1132       92      0           0            1       12161
#> 824           4       4986       84      0           1            0       21522
#> 825           2       4443       47      0           2            1        7923
#> 826           8       5011       80      1           6            6       18785
#> 827           6       2650       71      3           1            6        3009
#> 828           9       1788        5      0           0            5         129
#> 829           3        165       79      1           0            1       11127
#> 830           2       8322       74      0           2            1        4011
#> 831           4       1912       31      0           0            2         635
#> 832           6       5015       46      1           1            2        2390
#> 833          10       9349       73      1           5            2        4472
#> 834           4       5620       62      1           5            1       14449
#> 835           2       7007       87      0           5            0       26543
#> 836           1       6598       84      0           0            2       37429
#> 837           5       1298       29      1           0            2         216
#> 838           1          0       52      0           0            0       34743
#> 839           3       3313       94      0           0            1        5465
#> 840           0      10271       64      0           0            0        3455
#> 841           3          3       60      5           7            5        9864
#> 842           0      15366       48      0           0            0       31491
#> 843           2       7380       18      0           0            0        1288
#> 844           3       1134       39      0           0            3        1708
#> 845           1       1025       60      0           0            0       18561
#> 846           1       6303      119      1           0            1       46081
#> 847           0      12670       56      0           0            1        7115
#> 848           1       2381      127      1           0            0        9113
#> 849           0       1559       68      0           0            2        1708
#> 850           0       5063       52      2           1            2        3390
#> 851           9       6896       59      0           0            2       14273
#> 852           8       3018       31      3           0            3         952
#> 853           7       2801       40      0           7            3       12890
#> 854           3      13130       78      3           8           13       22361
#> 855           1       5346       26      0           0            0       26061
#> 856           9       4205       71      2           0            4        4779
#> 857           7       3681       59      3           0            4        3444
#> 858           6       3077       28      0           0            3        4334
#> 859           1       5299       93      1           0            4        5718
#> 860           1       2187       34      1           0            3       23536
#> 861           6       2082       57      0           0            3       23059
#> 862           1       2870       61      4           4            9       41193
#> 863           0       4813       88      0           7            0        7576
#> 864           3       2101       37      0           0            4        5897
#> 865           7       3099       55      1           9            2       14053
#> 866           2       6687       77      0           0            0       26794
#> 867           2       1206       79      2          10            7        9813
#> 868           2        916       85      0           0            0       11730
#> 869           1       1613       73      0           0            1        4492
#> 870           4       2092       63      2          15            4       23640
#> 871           2      18978       63      1           5            2       76183
#> 872           0       2264       77      0           1            0        5643
#> 873           2      12445       59      0           0            0        4750
#>     bc_open_to_buy mo_sin_old_il_acct mo_sin_old_rev_tl_op
#> 1             1506              148.0                  128
#> 2            57830              113.0                  192
#> 3             2737              125.0                  184
#> 4             4567              128.0                  210
#> 5              844              338.0                   54
#> 6                0              142.0                  306
#> 7            13674              149.0                   55
#> 8             8182              164.0                  129
#> 9             9966              155.0                  253
#> 10            7940               46.0                  234
#> 11            5128              115.0                  112
#> 12           16623               82.0                  379
#> 13            4778              234.0                   91
#> 14           17538              142.0                  168
#> 15           60336              109.0                  265
#> 16            1375              129.0                   95
#> 17           18383              114.0                  139
#> 18          263953              131.0                  294
#> 19           10607              113.0                  187
#> 20               0               50.0                   36
#> 21             437              124.0                  259
#> 22            2233               47.0                   53
#> 23           73931               63.0                  295
#> 24               0              149.0                  164
#> 25           11397              160.0                  175
#> 26            7011              141.0                  162
#> 27            3684              256.0                  360
#> 28            9044               93.0                  104
#> 29            3884              147.0                  166
#> 30           57049              124.0                  174
#> 31           26118              124.0                  223
#> 32            2926               76.0                  131
#> 33           23969               21.0                  173
#> 34           10051              111.0                    7
#> 35             523               10.0                  102
#> 36           12279              131.0                  137
#> 37             287              106.0                  176
#> 38           48533              159.0                  283
#> 39           11142               73.0                  173
#> 40            4319              186.0                  194
#> 41            1521              111.0                  126
#> 42             445              102.0                  169
#> 43             652              149.0                  179
#> 44             463              127.0                  135
#> 45             433              104.0                  111
#> 46           23476              143.0                  158
#> 47           17248              206.0                  310
#> 48            9949              120.0                  137
#> 49            6420              124.0                  206
#> 50           14203              164.0                  120
#> 51           10318              140.0                  176
#> 52             578              138.0                  172
#> 53           12875              135.0                  164
#> 54            2822               79.0                  142
#> 55            2382               76.0                   71
#> 56           18527               46.0                  174
#> 57             983              138.0                  203
#> 58             884              151.0                  186
#> 59           12808               90.0                  107
#> 60           21569              118.0                  220
#> 61            8818              123.0                  336
#> 62           24661              166.0                  238
#> 63            9516              138.0                   74
#> 64            1249               24.0                  240
#> 65            4286              172.0                  171
#> 66            4617              134.0                  124
#> 67           10457              142.0                  150
#> 68            1106              159.0                  352
#> 69            3924              150.0                  157
#> 70           38700              134.0                  100
#> 71             347              131.5                  110
#> 72           15310              183.0                  162
#> 73           11617              154.0                  162
#> 74            5787               13.0                   46
#> 75            7982              165.0                  174
#> 76           13312              128.0                  121
#> 77            9924               87.0                  255
#> 78          114609              141.0                  272
#> 79            8500               99.0                   60
#> 80            1188              147.0                  215
#> 81           10671               64.0                   91
#> 82           12642              177.0                  175
#> 83           15532              260.0                  129
#> 84            9831              128.0                  152
#> 85            6962              143.0                  168
#> 86            5934              167.0                   43
#> 87            6924               39.0                   59
#> 88            8302              156.0                  181
#> 89           10013              106.0                  163
#> 90           20142              131.5                  181
#> 91           41055              127.0                  145
#> 92           11498              147.0                  142
#> 93            2420                9.0                   88
#> 94           22057              134.0                  216
#> 95            2953              103.0                  115
#> 96           23397              159.0                  169
#> 97            8209              110.0                  111
#> 98           12925               52.0                   76
#> 99            5422               73.0                   72
#> 100          11939              254.0                  115
#> 101           1063              143.0                  192
#> 102           8240               98.0                  253
#> 103           9037              136.0                  367
#> 104           2773              159.0                  132
#> 105           3408               81.0                  126
#> 106           1388              125.0                  198
#> 107          14061              125.0                  174
#> 108              0              101.0                  109
#> 109           5220               14.0                  115
#> 110            763              157.0                  107
#> 111          23691              140.0                  145
#> 112           1661              116.0                  167
#> 113           2292              229.0                  197
#> 114           1286               66.0                  190
#> 115          32399              138.0                  707
#> 116          37800               91.0                  114
#> 117           6662               33.0                  173
#> 118          22487               77.0                  496
#> 119          11169               67.0                   70
#> 120          18432              121.0                  104
#> 121           6100              157.0                  157
#> 122          29902              146.0                  192
#> 123          13593              100.0                  384
#> 124          15785              275.0                  182
#> 125           4714              122.0                  141
#> 126           1749              136.0                   72
#> 127            239               94.0                  208
#> 128          18097              101.0                  108
#> 129          38057              148.0                  289
#> 130            459              116.0                   94
#> 131          17888               53.0                   86
#> 132          19900              114.0                  133
#> 133            168               24.0                  250
#> 134           5688               92.0                  290
#> 135           9477              156.0                  246
#> 136          21529              147.0                  387
#> 137           4275              103.0                  259
#> 138            610               43.0                   95
#> 139           4376               18.0                  157
#> 140          15361              128.0                  246
#> 141          43081               36.0                  175
#> 142           5467              122.0                  245
#> 143            344              124.0                  187
#> 144          12941               89.0                  316
#> 145           3661               51.0                  182
#> 146          14592              121.0                   32
#> 147          14271               17.0                  180
#> 148           4473              228.0                  230
#> 149           5030              222.0                  100
#> 150          25465              122.0                  315
#> 151           3707              107.0                  139
#> 152            400               28.0                  175
#> 153           2736               52.0                   45
#> 154           7004              112.0                  159
#> 155            464              131.5                  128
#> 156          39646              146.0                  375
#> 157            754              131.5                  140
#> 158            826              121.0                  146
#> 159           8000               43.0                   33
#> 160           1024              143.0                  101
#> 161          14431               33.0                  414
#> 162             82                8.0                  269
#> 163          26324              184.0                  193
#> 164          15186              220.0                  134
#> 165           1200               70.0                  159
#> 166           5553               66.0                   72
#> 167           2651               94.0                  102
#> 168           3764              150.0                  209
#> 169           3268              126.0                  257
#> 170           3024               49.0                   63
#> 171          13178              125.0                  189
#> 172           6698              172.0                  138
#> 173          50957               15.0                  361
#> 174          35742              303.0                  139
#> 175           2156              162.0                  225
#> 176              0               35.0                   86
#> 177          22037              175.0                  183
#> 178           1501              141.0                  271
#> 179           3645              170.0                  186
#> 180          21461              122.0                  354
#> 181          12608              266.0                  208
#> 182          15394              135.0                  338
#> 183           4276              136.0                  220
#> 184            902              119.0                  211
#> 185          21237              244.0                  284
#> 186           2225              148.0                  160
#> 187           7833              190.0                   96
#> 188           4133              172.0                  168
#> 189           4198               78.0                   81
#> 190           7832               75.0                   74
#> 191          21013              170.0                  456
#> 192            955              121.0                  354
#> 193          12316               29.0                   50
#> 194           1899               72.0                  149
#> 195              0              132.0                  379
#> 196           2973              148.0                  134
#> 197           5708              140.0                   99
#> 198           5440              232.0                  158
#> 199           2965              136.0                  359
#> 200          33863              125.0                  130
#> 201          20272               52.0                  220
#> 202           5392              106.0                  392
#> 203            444              142.0                  127
#> 204           1039              183.0                  147
#> 205           6762               49.0                  319
#> 206         105395              160.0                  233
#> 207            362              145.0                  124
#> 208           2175              131.0                  140
#> 209           7520              172.0                  237
#> 210           2076              135.0                  177
#> 211          43838              167.0                  160
#> 212           1750              181.0                  199
#> 213           4484              142.0                  178
#> 214         152306               29.0                  151
#> 215              0              183.0                  177
#> 216          63965               41.0                  339
#> 217          32706              137.0                  296
#> 218          23734               66.0                  385
#> 219           2135              161.0                  206
#> 220           5934              124.0                  229
#> 221           5390               19.0                   45
#> 222           7632              132.0                  164
#> 223          17898              113.0                  172
#> 224            233               71.0                  167
#> 225           3700              132.0                  185
#> 226           4752              149.0                  149
#> 227           3383              134.0                  157
#> 228            529              147.0                  320
#> 229           4362              158.0                  162
#> 230           5976              148.0                  253
#> 231             51              131.5                  132
#> 232           6489              126.0                  337
#> 233           6385              120.0                  116
#> 234           3348               94.0                   61
#> 235             76              143.0                  185
#> 236          12111              164.0                  434
#> 237           2273              129.0                  159
#> 238           5690               41.0                   79
#> 239           2200              100.0                  100
#> 240           3516              194.0                  106
#> 241           3447              149.0                  119
#> 242           5224               92.0                   96
#> 243          61230              141.0                  332
#> 244          14821              125.0                  192
#> 245           4040              101.0                  101
#> 246           4962              167.0                  194
#> 247           3005              227.0                  204
#> 248           3535              120.0                  196
#> 249          10311                3.0                  151
#> 250          26254              125.0                  233
#> 251          28032              154.0                  169
#> 252           4570               83.0                   95
#> 253          18126              139.0                  222
#> 254           2843              143.0                  189
#> 255           4300              125.0                  346
#> 256          44399              148.0                  390
#> 257            236               81.0                  376
#> 258          23082              186.0                  235
#> 259           1624              113.0                  243
#> 260          11291              239.0                  119
#> 261          50464              242.0                  149
#> 262          24550              127.0                  179
#> 263            629              110.0                  112
#> 264           2685              105.0                  428
#> 265           4627              111.0                  148
#> 266           1567              135.0                  165
#> 267          12822              135.0                  129
#> 268          19824               88.0                  377
#> 269            956               56.0                   33
#> 270          40607               89.0                  163
#> 271          18785              160.0                  373
#> 272            471              101.0                  107
#> 273           3167              136.0                  214
#> 274          10486              102.0                   88
#> 275           9328              170.0                  341
#> 276          10909              121.0                  153
#> 277           1949               54.0                  111
#> 278          11624              142.0                  186
#> 279           5934              142.0                  331
#> 280           1418              175.0                  179
#> 281           6289              148.0                  198
#> 282           1741              106.0                  133
#> 283           8333              124.0                  136
#> 284           2092              118.0                  120
#> 285          20813               73.0                   98
#> 286           2167              117.0                  125
#> 287          12002              155.0                  250
#> 288           1916              125.0                  244
#> 289          46254              145.0                  209
#> 290          10863              122.0                  435
#> 291          30804              194.0                  137
#> 292          10129              135.0                  146
#> 293          22337              122.0                  144
#> 294          25558              148.0                  226
#> 295           6920              148.0                  121
#> 296           6049               54.0                  196
#> 297           7957              142.0                  113
#> 298          64458               21.0                  346
#> 299          12015              114.0                  130
#> 300           1114              134.0                  243
#> 301           5219              117.0                  145
#> 302            330              138.0                  228
#> 303           5850              153.0                  429
#> 304          26490              111.0                  100
#> 305           2099              149.0                  246
#> 306          31947               58.0                  170
#> 307           1125              128.0                  155
#> 308            957              111.0                  102
#> 309            749               98.0                  368
#> 310          31422              136.0                  146
#> 311           3051              156.0                   96
#> 312          12914              133.0                  175
#> 313              0              129.0                  284
#> 314           6291              216.0                   38
#> 315          42520              205.0                  163
#> 316           4893              103.0                  185
#> 317           3833              124.0                  238
#> 318            304              126.0                  113
#> 319           1175               88.0                   74
#> 320           6453              227.0                  291
#> 321           9052              132.0                  106
#> 322          25008               90.0                  204
#> 323          18802              115.0                  233
#> 324            182               61.0                  132
#> 325           8525              146.0                  124
#> 326            709              153.0                  105
#> 327           5934               34.0                   49
#> 328           3868              139.0                  356
#> 329            559               13.0                  331
#> 330          15907               25.0                  109
#> 331           4927               26.0                  130
#> 332           2667              139.0                  297
#> 333           9729              126.0                  342
#> 334           1691              145.0                  257
#> 335          19968              210.0                  184
#> 336           4660              136.0                  112
#> 337            771              176.0                  299
#> 338           5847              122.0                  308
#> 339           4743              155.0                  133
#> 340          15387               41.0                  125
#> 341           2352              144.0                  168
#> 342           9598              119.0                  129
#> 343           6669              126.0                  225
#> 344           2924              187.0                  133
#> 345           6092              183.0                  253
#> 346           4680              124.0                  115
#> 347          10590               64.0                   54
#> 348          37202               76.0                  373
#> 349           3866              113.0                  193
#> 350           3949              123.0                  145
#> 351          11145                9.0                   64
#> 352              8              190.0                  259
#> 353          12109              133.0                  155
#> 354           2067              132.0                  204
#> 355           4500              109.0                  174
#> 356          47492              130.0                  259
#> 357            270              134.0                   94
#> 358           3465              171.0                  184
#> 359           1627               97.0                   46
#> 360           8714               98.0                   78
#> 361          11213              117.0                  317
#> 362          41348              327.0                  145
#> 363          19576              119.0                  198
#> 364           3621              153.0                  139
#> 365           1867              140.0                  119
#> 366           4995              108.0                  170
#> 367           7500               99.0                   78
#> 368           1557              131.0                  228
#> 369          10634               42.0                  150
#> 370           7477               58.0                  105
#> 371          39059              137.0                  194
#> 372         124059               91.0                  362
#> 373           9950              126.0                  193
#> 374          36445              131.5                   48
#> 375           6174              141.0                  112
#> 376            163              125.0                  200
#> 377          16426              127.0                  170
#> 378           2180               98.0                  193
#> 379            457              159.0                  117
#> 380           1168               77.0                   97
#> 381          10817              261.0                  283
#> 382          55623              208.0                  442
#> 383             90               92.0                  150
#> 384           1293              135.0                  310
#> 385           6270               76.0                   77
#> 386           4500              148.0                  114
#> 387          88446              158.0                  281
#> 388           6815              199.0                  105
#> 389            295               99.0                  136
#> 390          73787              126.0                  135
#> 391           1232              160.0                  243
#> 392           4034              135.0                   90
#> 393           3272              165.0                   38
#> 394           3290              159.0                  337
#> 395           6432               99.0                   96
#> 396          32528              147.0                  197
#> 397            525              154.0                  263
#> 398           6180              131.0                  214
#> 399            968              105.0                  121
#> 400           3724               38.0                  116
#> 401           6080              125.0                   94
#> 402           8195               22.0                  116
#> 403          17569               57.0                  332
#> 404           1622              141.0                  117
#> 405           3442              133.0                  155
#> 406           3867               44.0                   34
#> 407           1013              163.0                  294
#> 408          10585              139.0                  142
#> 409           3507                5.0                   59
#> 410           2404              171.0                   79
#> 411          67219              147.0                  348
#> 412           1625               97.0                   96
#> 413          11113              122.0                  183
#> 414            333              122.0                   12
#> 415          24131              116.0                  189
#> 416          38277              130.0                  291
#> 417          44719              171.0                  513
#> 418          38100               98.0                  129
#> 419           9113              132.0                  149
#> 420           8820              162.0                  104
#> 421            194               54.0                   77
#> 422          10489               15.0                   90
#> 423          12592              137.0                  154
#> 424           2025              173.0                  129
#> 425          24837               22.0                  569
#> 426          13141              157.0                  253
#> 427          18080              147.0                  148
#> 428            299              124.0                  200
#> 429            294              133.0                  173
#> 430          34094              197.0                  243
#> 431          41515              146.0                  315
#> 432           2064              146.0                  177
#> 433          73367              138.0                  384
#> 434           6700              131.5                   55
#> 435           4532              152.0                  265
#> 436           5050              159.0                  132
#> 437           1093              136.0                  134
#> 438           9560              119.0                   62
#> 439           5377              144.0                  295
#> 440           4471              152.0                  312
#> 441          10359              203.0                  153
#> 442          19481              194.0                  192
#> 443          17185              122.0                  253
#> 444           3631              142.0                  147
#> 445           3990               82.0                   16
#> 446           1275              138.0                  264
#> 447           3176               59.0                   79
#> 448          10641               61.0                  243
#> 449              0              105.0                   39
#> 450          21501              162.0                  198
#> 451           3203               14.0                  326
#> 452            357               52.0                  209
#> 453          11085              162.0                  103
#> 454              0              140.0                  237
#> 455            212              148.0                  152
#> 456           5052              146.0                  405
#> 457            574              159.0                  260
#> 458           4338               67.0                  107
#> 459          14560               50.0                   89
#> 460          32453               77.0                  151
#> 461          11578              146.0                  188
#> 462           1450              150.0                  341
#> 463          33017              138.0                  141
#> 464           4794              158.0                  112
#> 465           7347                1.0                  415
#> 466          21611               57.0                   66
#> 467          24028              134.0                  247
#> 468            160              131.0                  197
#> 469           5036              183.0                  192
#> 470            343               76.0                   88
#> 471           6328              158.0                   71
#> 472           1898              137.0                  110
#> 473           5934              175.0                  124
#> 474          39062              111.0                  245
#> 475          11669              126.0                  203
#> 476          21239               67.0                  146
#> 477           2538              151.0                  169
#> 478              0              151.0                  261
#> 479          14382              149.0                  161
#> 480            876              168.0                  169
#> 481           5446              148.0                  162
#> 482          36279              204.0                  382
#> 483          13592              130.0                  133
#> 484          24194               68.0                  223
#> 485           9080               76.0                   74
#> 486           4437              137.0                  181
#> 487           5895              127.0                  140
#> 488           5934              135.0                  163
#> 489           8375              151.0                  427
#> 490          10885              121.0                  173
#> 491            660              154.0                   68
#> 492            505              124.0                  114
#> 493           2273               71.0                   93
#> 494            166              184.0                  119
#> 495          13078              112.0                  368
#> 496          14957              132.0                  269
#> 497           4353              130.0                  277
#> 498          21921               61.0                  217
#> 499           2295              168.0                   49
#> 500           6051              134.0                  204
#> 501           5680               58.0                  117
#> 502           1289              102.0                  294
#> 503           9602               37.0                  107
#> 504          21431              152.0                  340
#> 505           5934              173.0                  183
#> 506            260              175.0                  122
#> 507          29408              142.0                  180
#> 508           2517              142.0                  156
#> 509          71597              146.0                  303
#> 510           4701              238.0                  279
#> 511           6231              187.0                   28
#> 512           4996              128.0                  121
#> 513           2456              116.0                  335
#> 514           3680              135.0                  271
#> 515           4101              112.0                  111
#> 516           6517              170.0                  210
#> 517           2514              112.0                  111
#> 518           1773               71.0                  301
#> 519          17059              141.0                  209
#> 520           2008              126.0                  170
#> 521          17759              244.0                  118
#> 522          19466               73.0                  115
#> 523           6498              158.0                  167
#> 524           3500              110.0                  122
#> 525           6086              235.0                  168
#> 526          25202               57.0                   97
#> 527           6453              254.0                  153
#> 528           4318              166.0                  146
#> 529          10757              170.0                  344
#> 530          41448              149.0                  177
#> 531             54              169.0                  155
#> 532            339              206.0                  144
#> 533           7341              122.0                  253
#> 534           5000              118.0                  262
#> 535          11092              258.0                  231
#> 536          24875              152.0                  172
#> 537           3358              131.5                   37
#> 538            407              199.0                  198
#> 539          24818              129.0                  366
#> 540           1450              134.0                  328
#> 541           3826              104.0                  234
#> 542           3088              220.0                  113
#> 543           6134              140.0                  129
#> 544           5593              110.0                  156
#> 545           3172               98.0                   67
#> 546           1965              134.0                  131
#> 547           4795              112.0                  212
#> 548           2615              156.0                  188
#> 549           3953              131.5                   50
#> 550          27744              142.0                  377
#> 551          18172              133.0                  215
#> 552           5649              119.0                  174
#> 553           1476               16.0                  355
#> 554           7598              121.0                  135
#> 555           1383              143.0                   91
#> 556          12480              125.0                  320
#> 557          34750              125.0                  237
#> 558          19008              128.0                  175
#> 559           7270              144.0                  406
#> 560          15148              101.0                  152
#> 561            236               53.0                  192
#> 562           4780              111.0                  116
#> 563            310              189.0                  179
#> 564          77176              149.0                  255
#> 565           2097              159.0                  162
#> 566           4473               41.0                  170
#> 567          33924              149.0                  462
#> 568          60874               70.0                  286
#> 569          24975              148.0                  311
#> 570           2282               99.0                   92
#> 571           1314               83.0                  219
#> 572          31508              143.0                  196
#> 573          68105              154.0                  263
#> 574           1100               25.0                  193
#> 575           5358              134.0                  208
#> 576           9913              130.0                  225
#> 577          15141              112.0                  167
#> 578           3541              144.0                  269
#> 579          64863              134.0                  312
#> 580           2873              125.0                  267
#> 581            470              158.0                  153
#> 582             88              208.0                   54
#> 583           4925              107.0                  153
#> 584          14772              164.0                  396
#> 585           5739              131.5                  206
#> 586           4612              104.0                   96
#> 587           3781              117.0                  257
#> 588           3373              104.0                  204
#> 589           2584              202.0                  144
#> 590           3381              144.0                  165
#> 591           1492               54.0                   52
#> 592           4565              168.0                  159
#> 593           5998               47.0                  172
#> 594          17628              136.0                  253
#> 595            196              129.0                  272
#> 596           8902              242.0                  250
#> 597          20239              131.0                  132
#> 598          31798               60.0                  301
#> 599             80               79.0                  120
#> 600           1200               11.0                  232
#> 601          36581              169.0                  424
#> 602           1725              187.0                  379
#> 603           1164              112.0                  169
#> 604          36981              131.5                   41
#> 605           9659               72.0                  106
#> 606           7165              142.0                  365
#> 607           6204              116.0                  146
#> 608          12017              140.0                  208
#> 609          21343               99.0                  162
#> 610            237              125.0                  272
#> 611          61112               74.0                  176
#> 612           2821               33.0                  212
#> 613           8861              120.0                  148
#> 614          50398              159.0                  174
#> 615          15088              137.0                  287
#> 616           6606               16.0                  188
#> 617           2334               40.0                   57
#> 618          18735              142.0                  130
#> 619          28808              131.5                  228
#> 620          28242              156.0                  307
#> 621           1542              140.0                  188
#> 622           3818              114.0                  171
#> 623           1545              126.0                  158
#> 624          35456               38.0                  296
#> 625            859              110.0                  115
#> 626              0              125.0                  305
#> 627             95              134.0                  247
#> 628          23807              160.0                  235
#> 629              0               25.0                  260
#> 630           2653              160.0                  223
#> 631           5982              160.0                  349
#> 632           7317              135.0                  149
#> 633           2878              115.0                  196
#> 634          28073              127.0                  255
#> 635           1041              150.0                  237
#> 636           2664              223.0                  103
#> 637            999              147.0                  103
#> 638          14653              114.0                  471
#> 639              0               48.0                  142
#> 640           2875               87.0                  313
#> 641           1405               49.0                  130
#> 642          26174              118.0                  194
#> 643           6092              131.5                   89
#> 644          28690               98.0                  203
#> 645            145              147.0                  153
#> 646          11369              151.0                  151
#> 647          28676              128.0                  194
#> 648          15840              136.0                  175
#> 649          14788              155.0                  158
#> 650           1103              286.0                   49
#> 651           1555              135.0                  137
#> 652           1289              131.5                  117
#> 653          15998              129.0                  185
#> 654            574              141.0                  343
#> 655          15565              117.0                  133
#> 656            513              103.0                  302
#> 657          15159              244.0                  221
#> 658           4984               41.0                  109
#> 659           5644               60.0                   48
#> 660          26151              138.0                  361
#> 661          21288              158.0                  171
#> 662          37162              131.5                  342
#> 663          11992               88.0                  181
#> 664          65137               57.0                  370
#> 665           2953              117.0                  212
#> 666           2046              120.0                   32
#> 667           5276               91.0                  213
#> 668            977              136.0                  132
#> 669           8125              190.0                  110
#> 670          18485              164.0                  241
#> 671          16370              105.0                  173
#> 672           4265              102.0                  184
#> 673            173              182.0                  229
#> 674            181              132.0                  386
#> 675          22285              107.0                  296
#> 676           4338              184.0                   90
#> 677           5939              132.0                  289
#> 678          12908              130.0                  146
#> 679           1363               36.0                  253
#> 680           3207              148.0                  152
#> 681           1892              127.0                   51
#> 682           1293              160.0                  163
#> 683           8774              146.0                  163
#> 684           3276               91.0                  326
#> 685           3715               58.0                  253
#> 686          48959              174.0                  377
#> 687            525              171.0                  255
#> 688            115               79.0                  113
#> 689            395              148.0                  199
#> 690          15187               49.0                  135
#> 691          10492              184.0                  250
#> 692           5200               86.0                  111
#> 693           8246              171.0                  117
#> 694           3053               85.0                  134
#> 695           1186              105.0                  197
#> 696           3700              152.0                  476
#> 697          46859              141.0                  144
#> 698          15227              165.0                  230
#> 699           1195              151.0                  222
#> 700          14290               12.0                   40
#> 701              0              148.0                  148
#> 702           4267               69.0                   92
#> 703           6924              132.0                  129
#> 704          13084               86.0                  119
#> 705          15249              168.0                  196
#> 706           1954               98.0                  236
#> 707           3392              101.0                  168
#> 708          18293              111.0                  222
#> 709          19442              127.0                  340
#> 710          20430              177.0                  488
#> 711           1073              101.0                  346
#> 712           7427              147.0                  180
#> 713           7774              122.0                  160
#> 714          33315              151.0                  248
#> 715          50523              140.0                  348
#> 716          29725              154.0                  461
#> 717          70088               66.0                  108
#> 718          27131               92.0                  308
#> 719          17361              140.0                  236
#> 720          35394              172.0                  234
#> 721          11775              131.5                  254
#> 722              0               75.0                   13
#> 723          11800              139.0                  140
#> 724            420               42.0                  286
#> 725          26295               85.0                  154
#> 726          13739              208.0                   47
#> 727           5297              111.0                  214
#> 728          31984              153.0                  361
#> 729           7168              156.0                  207
#> 730           3169              142.0                  136
#> 731           2817              195.0                  324
#> 732           5400              158.0                  178
#> 733           5586              127.0                  253
#> 734          11447              174.0                  359
#> 735            234              155.0                  141
#> 736            379              147.0                  108
#> 737           5000              140.0                   35
#> 738          19375              230.0                  119
#> 739          22939              126.0                  107
#> 740            200               76.0                  445
#> 741          16487              133.0                  123
#> 742          12117               26.0                  165
#> 743          48369              179.0                  268
#> 744           5934              147.0                  112
#> 745          18680              145.0                  244
#> 746           4261              116.0                  310
#> 747          41155               48.0                  108
#> 748           1313              124.0                   89
#> 749          15669               63.0                  106
#> 750          12933              139.0                  134
#> 751          35873              124.0                  152
#> 752           4597              134.0                  148
#> 753             26              131.0                  139
#> 754          15756              178.0                  179
#> 755           1936              239.0                  257
#> 756           7509              177.0                  105
#> 757           5439              133.0                  340
#> 758            199              109.0                  117
#> 759          12382              159.0                  143
#> 760           2913              130.0                   95
#> 761            700               76.0                  157
#> 762           3175               35.0                  123
#> 763          77391              156.0                  315
#> 764          28685              113.0                  193
#> 765          15958              100.0                  179
#> 766           9336              156.0                  239
#> 767          11647              157.0                  199
#> 768          20162              140.0                  164
#> 769           2576              125.0                  180
#> 770          12837              118.0                  148
#> 771            314              171.0                  301
#> 772           9550              153.0                  134
#> 773             92              137.0                  269
#> 774          20934              172.0                  262
#> 775           9379              131.0                   73
#> 776           7259              235.0                   45
#> 777          27774               64.0                   98
#> 778           2767              115.0                  216
#> 779           5934              136.0                  160
#> 780           4636              244.0                  186
#> 781          44182              133.0                  218
#> 782           1567              125.0                  388
#> 783            642               39.0                   50
#> 784           2371               86.0                  301
#> 785            409              169.0                  211
#> 786          11673              163.0                  225
#> 787          18102              254.0                  324
#> 788           7659              195.0                  193
#> 789           2936              150.0                  182
#> 790           8853              133.0                  166
#> 791             39              151.0                  239
#> 792          56759              148.0                  164
#> 793          33784              135.0                  152
#> 794            662              154.0                  196
#> 795           1667              116.0                  103
#> 796           1797              172.0                  161
#> 797           1415               76.0                  168
#> 798           2523              164.0                  234
#> 799          10428              170.0                  177
#> 800           7568               56.0                   55
#> 801          19534               47.0                  308
#> 802           4473              115.0                  147
#> 803           8947               13.0                  169
#> 804           1905              159.0                  125
#> 805          30219              131.5                  527
#> 806            808               35.0                  142
#> 807            115              167.0                  159
#> 808            492              160.0                  143
#> 809          17406               95.0                  247
#> 810           3663              253.0                  158
#> 811          39961              114.0                  254
#> 812          10155               99.0                   99
#> 813           6508              193.0                  187
#> 814            113               65.0                  138
#> 815            960              242.0                  225
#> 816            731               25.0                  329
#> 817           3248              100.0                   81
#> 818          40545              162.0                  349
#> 819          26235              158.0                  159
#> 820           3067              121.0                  143
#> 821          30179              188.0                  161
#> 822          19702              152.0                  296
#> 823           4968              129.0                   58
#> 824            758              364.0                  162
#> 825          22609              135.0                  138
#> 826           4140              143.0                  311
#> 827           8546              120.0                  412
#> 828          36594               90.0                  124
#> 829           2318              174.0                  117
#> 830           2715              139.0                  335
#> 831           4075              150.0                  446
#> 832          11355              155.0                   51
#> 833          21773               81.0                  243
#> 834           3813              158.0                  217
#> 835           1102               12.0                  279
#> 836           4190              123.0                  236
#> 837           5324               61.0                   98
#> 838           3500              164.0                  206
#> 839          12799               78.0                   41
#> 840           8157              174.0                  212
#> 841            297              119.0                  133
#> 842          35664              124.0                  302
#> 843          50612              243.0                  266
#> 844           1850              134.0                  256
#> 845           1700              150.0                  225
#> 846           5897              137.0                  120
#> 847           9379               36.0                  134
#> 848           2575              119.0                  155
#> 849           1525              126.0                  159
#> 850            537               95.0                  122
#> 851           2267              156.0                  148
#> 852          23643              115.0                  184
#> 853          22845              126.0                  258
#> 854           3761              165.0                  404
#> 855           9654              156.0                  110
#> 856          10772              111.0                  131
#> 857           9553              148.0                  139
#> 858          93469              139.0                  354
#> 859           1001              134.0                  127
#> 860           6500              241.0                  131
#> 861           1667              133.0                  182
#> 862           7631              132.0                   66
#> 863            187               53.0                  140
#> 864          51378               98.0                  104
#> 865          27254              155.0                  185
#> 866            856               65.0                   73
#> 867             94              159.0                  124
#> 868            700              145.0                  113
#> 869            687               41.0                  313
#> 870           2510              120.0                  188
#> 871           6551              117.0                  260
#> 872             36              152.0                  158
#> 873          63584              117.0                  189
#>     mo_sin_rcnt_rev_tl_op mo_sin_rcnt_tl mort_acc mths_since_recent_bc
#> 1                       3              3        1                    4
#> 2                       2              2        4                    2
#> 3                      14             14        5                  101
#> 4                       4              4        6                    4
#> 5                      32             32        0                   36
#> 6                      10             10        4                   12
#> 7                      32             13        3                   32
#> 8                       1              1        1                    4
#> 9                      15             10        1                   50
#> 10                     18              2        4                   28
#> 11                      1              1        2                    9
#> 12                     19             19        2                   19
#> 13                      9              9        0                   11
#> 14                     13             13        3                   13
#> 15                      1              1        4                    1
#> 16                      0              0        0                    8
#> 17                      5              5        2                    5
#> 18                     11              6        2                   11
#> 19                     22             22        0                   22
#> 20                     32             19        0                   36
#> 21                     18             18        2                   36
#> 22                     26             26        0                   26
#> 23                     19             15        0                   19
#> 24                      3              3        1                   13
#> 25                     37              9        5                   37
#> 26                     13             13        2                   34
#> 27                      1              1        0                    1
#> 28                      2              2        2                    2
#> 29                      9              9        8                    9
#> 30                      1              1        6                    2
#> 31                     10             10        3                   34
#> 32                     85              5        0                   85
#> 33                      3              3        1                    3
#> 34                      2              2        2                    2
#> 35                     16             10        0                   16
#> 36                      2              2        1                    2
#> 37                      7              6        6                   51
#> 38                      4              4        1                    4
#> 39                     16             11        1                   20
#> 40                     17             14        2                   17
#> 41                     15              5        0                   15
#> 42                      6              6        2                    9
#> 43                      3              2        0                    3
#> 44                     13             13        2                   13
#> 45                      9              6        3                    9
#> 46                      4              4        2                   10
#> 47                     19              3        5                   99
#> 48                     11              9        3                   11
#> 49                      1              1        0                   24
#> 50                     16             16        0                   16
#> 51                      6              6        0                    6
#> 52                      2              2        3                    2
#> 53                     38             35        4                   38
#> 54                      4              2        5                   27
#> 55                     23             10        0                   23
#> 56                      5              5        0                    6
#> 57                     13              3        7                   23
#> 58                      1              1        3                    3
#> 59                      2              2        0                   10
#> 60                     16              6        3                   16
#> 61                      6              3        4                    6
#> 62                      4              4        7                   73
#> 63                      7              7        0                   16
#> 64                      4              4        3                   52
#> 65                     12             10        1                   12
#> 66                     16              2        2                   16
#> 67                      7              7        4                   18
#> 68                      3              3        2                    3
#> 69                     11              9        2                   17
#> 70                      7              7        5                   10
#> 71                      3              3        0                    3
#> 72                      3              3        0                    3
#> 73                      3              3        2                    3
#> 74                     16             13        0                   17
#> 75                      4              4        2                    4
#> 76                     18             17        0                   45
#> 77                      8              3        1                    8
#> 78                      2              2        2                    2
#> 79                     35              9        2                   37
#> 80                      4              4        2                   13
#> 81                     10             10        0                   10
#> 82                      6              6        1                    6
#> 83                     16              9        1                   16
#> 84                      5              5        0                    5
#> 85                      3              3        1                    4
#> 86                      7              7        2                   12
#> 87                      5              5        0                    5
#> 88                      8              8        1                    8
#> 89                     11             11        0                   11
#> 90                      5              5        0                    5
#> 91                      4              1        2                    4
#> 92                      9              5        3                    9
#> 93                      6              6        0                    6
#> 94                     54              7        2                   54
#> 95                     19             19        1                   83
#> 96                     12              8        6                   12
#> 97                     12             12        2                   40
#> 98                      5              5        0                    5
#> 99                      6              1        0                    6
#> 100                    17              3        3                   17
#> 101                     4              1        2                    7
#> 102                    15              3        0                   15
#> 103                     1              1        3                   14
#> 104                     1              1        2                    1
#> 105                     1              1        0                    1
#> 106                     1              1        1                    1
#> 107                     7              7        6                   85
#> 108                     4              4        0                   28
#> 109                     5              5        0                    5
#> 110                    12              8        0                   21
#> 111                     2              2        0                    2
#> 112                     1              1        0                    1
#> 113                    18              2        3                   18
#> 114                     7              7        0                   62
#> 115                     4              4        5                    4
#> 116                     9              9        8                    9
#> 117                     1              1        0                    3
#> 118                    19              1        3                   19
#> 119                     6              3        0                   24
#> 120                    18              8        0                   20
#> 121                    15              3        0                   15
#> 122                    12             12        6                   12
#> 123                     2              2        2                    2
#> 124                    18             18        2                   27
#> 125                     4              4        1                    4
#> 126                    30             14        0                   49
#> 127                     8              8        0                   28
#> 128                    31              1        0                   42
#> 129                     8              8        0                    8
#> 130                     6              6        0                    6
#> 131                     4              4        0                   12
#> 132                     2              2        1                   17
#> 133                     3              3        3                   33
#> 134                    27              7        3                   27
#> 135                    12              4        1                   12
#> 136                     2              2        3                    3
#> 137                     0              0        1                   26
#> 138                    12             12        0                   12
#> 139                    23             18        1                   23
#> 140                    17             17        1                   17
#> 141                     1              1        0                   36
#> 142                    12             12        1                   12
#> 143                     5              5        2                    5
#> 144                     2              2        4                    2
#> 145                    15              4        3                   15
#> 146                     1              1        0                    2
#> 147                     4              4        0                    4
#> 148                     5              4        1                    5
#> 149                     4              2        8                   29
#> 150                     1              1        2                    1
#> 151                     1              1        5                    1
#> 152                     5              5        1                   12
#> 153                     2              2        0                   30
#> 154                    26              8        1                   37
#> 155                     9              9        0                    9
#> 156                    34             34        3                   34
#> 157                    26             26        0                  133
#> 158                    13              5        2                   18
#> 159                     9              9        1                   18
#> 160                     8              8        3                   27
#> 161                     6              5        0                   15
#> 162                     0              0        4                    3
#> 163                     7              7        0                   13
#> 164                     3              3        0                    3
#> 165                    18             18        1                   28
#> 166                    10              9        0                   10
#> 167                     6              4        0                    6
#> 168                     0              0        4                   22
#> 169                     1              1        2                    1
#> 170                     2              2        0                    2
#> 171                    19             15        2                   54
#> 172                    49             42        0                   49
#> 173                    38             15        2                   38
#> 174                     1              1        6                   16
#> 175                     4              2        5                   18
#> 176                    12              1        0                   12
#> 177                    17             17        0                   21
#> 178                    13             13        1                   19
#> 179                     3              1        1                    9
#> 180                    10              1        2                   10
#> 181                     8              2        1                    8
#> 182                     8              8        5                    8
#> 183                     8              2        1                    8
#> 184                    27             27        2                   27
#> 185                    46             46        7                   71
#> 186                     3              3        1                   25
#> 187                     8              8        2                    8
#> 188                     2              2        0                    2
#> 189                     8              4        3                    8
#> 190                     7              7        0                   14
#> 191                     4              4        3                   11
#> 192                     3              3        0                   10
#> 193                     7              7        0                    7
#> 194                    13             13        5                   56
#> 195                    28              6        3                  100
#> 196                    12             12        0                   16
#> 197                     5              5        2                   21
#> 198                    12             12        5                   12
#> 199                    41              2        5                   41
#> 200                    33              5        2                   33
#> 201                     2              2        3                    2
#> 202                    12              8        0                   13
#> 203                     0              0        2                   28
#> 204                     7              7        0                  114
#> 205                    31             31        2                  122
#> 206                    10             10       12                   10
#> 207                     7              7        2                   29
#> 208                    10             10        2                   10
#> 209                     3              3        0                    3
#> 210                     1              1        1                   15
#> 211                     9              9        4                   14
#> 212                     5              5        0                    5
#> 213                    20              3        2                   25
#> 214                     4              4        0                    4
#> 215                     6              6        0                   14
#> 216                     0              0        2                    0
#> 217                     3              3        3                   31
#> 218                    22             19        3                   22
#> 219                    14              3        7                   14
#> 220                     3              3        0                   12
#> 221                     5              5        1                    5
#> 222                     8              2        4                   36
#> 223                     8              2        2                    8
#> 224                    22             17        0                   22
#> 225                     0              0        1                    0
#> 226                     3              3        1                   17
#> 227                     2              2        0                   10
#> 228                     3              3        2                    3
#> 229                    11              7        5                   12
#> 230                    35             12        2                   42
#> 231                    29             29        0                   31
#> 232                     3              3        4                    4
#> 233                     3              3        0                    3
#> 234                     5              5        0                    5
#> 235                     4              4        1                   25
#> 236                     6              4        1                   38
#> 237                    16              8        1                   55
#> 238                     2              2        0                   17
#> 239                     7              7        0                    7
#> 240                     2              2        0                   23
#> 241                    11             11        0                   11
#> 242                     9              9        1                   21
#> 243                     0              0        0                    2
#> 244                     4              4        2                    4
#> 245                     6              6        0                   17
#> 246                     2              1        2                    2
#> 247                    33              9        2                   33
#> 248                     1              1        5                   62
#> 249                     3              3        2                    3
#> 250                     8              8        0                    8
#> 251                    31             20        2                   31
#> 252                     9              9        1                    9
#> 253                    11              3        2                  102
#> 254                     4              4        2                    4
#> 255                     1              1        5                    9
#> 256                     4              4        4                    4
#> 257                    18              9        4                   18
#> 258                     7              3        8                    7
#> 259                    28              9        5                   28
#> 260                    12              5        1                   25
#> 261                    15              6        0                   25
#> 262                    11             11        1                   52
#> 263                     4              4        1                   29
#> 264                    24              7        2                   52
#> 265                    99             28        2                  115
#> 266                     5              5        0                   12
#> 267                     1              1        0                    2
#> 268                     1              1        0                   10
#> 269                     5              5        0                    5
#> 270                     4              4        4                    4
#> 271                     5              5        3                    5
#> 272                     6              6        0                    6
#> 273                     4              4        3                    9
#> 274                     2              2        0                    2
#> 275                     1              1        0                   12
#> 276                     0              0        2                    0
#> 277                    12             12        0                   13
#> 278                   139              5        1                  183
#> 279                   105              3        2                   12
#> 280                    24              4        4                   24
#> 281                     0              0        0                   18
#> 282                     1              1        2                    2
#> 283                     1              1        4                    9
#> 284                     8              4        2                    8
#> 285                     1              1        4                    8
#> 286                     4              3        2                    6
#> 287                     1              1        5                    5
#> 288                     5              5        5                   14
#> 289                     8              8        1                    8
#> 290                     1              1        6                    1
#> 291                     2              2        0                    2
#> 292                     5              5        1                    5
#> 293                     0              0        3                   15
#> 294                    17              9        0                   29
#> 295                     5              5        0                    5
#> 296                     5              5        0                    5
#> 297                    17              4        2                   30
#> 298                    18             18        7                   18
#> 299                     0              0        1                    0
#> 300                   110             82        0                  110
#> 301                     2              2        4                   21
#> 302                    59              9        3                   61
#> 303                     3              3        2                   10
#> 304                    17              2        0                   17
#> 305                    18             18        4                   38
#> 306                     6              1        4                    7
#> 307                     6              6        2                    6
#> 308                     0              0        0                    2
#> 309                    30              8        5                   32
#> 310                     6              6        3                    6
#> 311                     1              1        0                    1
#> 312                    13             11        0                   13
#> 313                     9              9        5                    9
#> 314                     4              4        0                   11
#> 315                     3              3        4                  116
#> 316                     8              8        1                   33
#> 317                     2              2        7                   26
#> 318                    14             14        5                  109
#> 319                     5              5        1                    5
#> 320                    14              4        2                   19
#> 321                     2              2        0                    2
#> 322                    25              3        6                   25
#> 323                    12              6        2                   12
#> 324                     8              8        1                   18
#> 325                    15              2        0                   15
#> 326                     0              0        0                   42
#> 327                     9              8        0                   12
#> 328                     3              3        6                   11
#> 329                   169             13        0                  297
#> 330                     1              1        0                    7
#> 331                    10              2        1                   10
#> 332                     2              2        2                    2
#> 333                     9              3        3                    9
#> 334                     0              0        0                    6
#> 335                     5              3        4                    5
#> 336                    18             18        2                   42
#> 337                     2              2        3                   17
#> 338                     2              2        0                    2
#> 339                     4              4        1                    4
#> 340                     2              2        2                    2
#> 341                   124             32        0                  124
#> 342                     6              5        1                   13
#> 343                     1              1        0                    1
#> 344                     7              4        8                   24
#> 345                     9              9        1                    9
#> 346                    12             12        0                   12
#> 347                     5              5        0                    5
#> 348                     3              3        0                    3
#> 349                    11              2        5                   22
#> 350                    48             28        4                   79
#> 351                    17              4        0                   48
#> 352                    12              3        2                   14
#> 353                    20             12        1                   30
#> 354                     4              4        7                    4
#> 355                    15              5        2                   15
#> 356                    11              8        5                   11
#> 357                     8              7        3                   44
#> 358                     2              2        2                   19
#> 359                    11              2        3                   11
#> 360                     1              1        0                   15
#> 361                    17             17        0                   17
#> 362                     8              8        0                    8
#> 363                     4              4        0                    4
#> 364                     1              1        2                    4
#> 365                    25             10        4                   25
#> 366                    18             18        0                   34
#> 367                    15              2        2                   15
#> 368                     6              6        1                   38
#> 369                    25              3        1                   25
#> 370                    16             15        0                   22
#> 371                     1              1        0                    8
#> 372                    11             11        1                   11
#> 373                     2              2        5                    2
#> 374                     0              0        1                    2
#> 375                     0              0        1                    0
#> 376                     0              0        3                   17
#> 377                    12             12        5                   18
#> 378                     1              1        0                    3
#> 379                     2              2        0                   24
#> 380                     8              8        1                   25
#> 381                     6              6        4                   10
#> 382                     3              3        3                    3
#> 383                     7              7        1                    7
#> 384                    26             16        0                   51
#> 385                    11              8        0                   11
#> 386                     0              0        3                   10
#> 387                    20              6        4                   20
#> 388                    20              3        0                   22
#> 389                    13             13        1                   28
#> 390                     2              2        1                    2
#> 391                    14              1        4                  204
#> 392                     6              4        0                    6
#> 393                     0              0        0                    2
#> 394                    15              5        1                   89
#> 395                     4              4        0                    4
#> 396                     4              4        2                    4
#> 397                    19             12        2                   36
#> 398                     3              3        8                   17
#> 399                     4              4        1                    7
#> 400                    19              7        1                   19
#> 401                    11              8        2                   20
#> 402                    16             16        0                   21
#> 403                     5              5        5                    5
#> 404                     3              3        3                   17
#> 405                     4              4        0                   11
#> 406                     3              3        1                   15
#> 407                     2              2        2                   15
#> 408                    34             13        2                   34
#> 409                     8              5        0                    8
#> 410                    18              9        0                   33
#> 411                    10              3        1                   10
#> 412                    15             15        0                   23
#> 413                     5              5        0                    5
#> 414                     5              1        0                   12
#> 415                     2              2        5                    2
#> 416                    14              5        6                   14
#> 417                     4              4        2                   32
#> 418                    13              9        7                   33
#> 419                    19              9        4                   29
#> 420                     2              2        0                    2
#> 421                     4              4        5                    4
#> 422                     6              6        1                    6
#> 423                    13              6        2                   46
#> 424                     0              0        0                    3
#> 425                     2              2        4                    6
#> 426                     9              9        3                    9
#> 427                     3              3        4                    3
#> 428                     1              1        0                   46
#> 429                     7              7        3                   18
#> 430                     1              1        1                    5
#> 431                     9              9        5                    9
#> 432                     2              2        0                   58
#> 433                     7              7        0                   16
#> 434                     2              2        0                    2
#> 435                    25              3        2                   38
#> 436                     0              0        0                    0
#> 437                     7              7        0                   19
#> 438                     1              1        0                    1
#> 439                    20             17        4                   20
#> 440                    21              8        1                   21
#> 441                     4              4        2                   11
#> 442                     3              3        3                    8
#> 443                     1              1        1                    1
#> 444                    37              5        4                   37
#> 445                     3              3        0                    3
#> 446                    11              3        3                   11
#> 447                     9              3        2                    9
#> 448                    12              1        0                   16
#> 449                    14             14        0                   14
#> 450                    20             20        0                   21
#> 451                     6              3        6                   13
#> 452                    13             10        1                   26
#> 453                    12              9        4                   12
#> 454                     6              6        1                   63
#> 455                     4              4        0                    4
#> 456                     7              7        7                   44
#> 457                     9              9        2                   15
#> 458                    19             19        0                   19
#> 459                     8              4        1                   10
#> 460                     4              4        0                    4
#> 461                     1              1        4                    1
#> 462                    12             12        1                   85
#> 463                    51              8        2                   51
#> 464                    14             14        0                   14
#> 465                    35              1        0                   35
#> 466                     5              5        0                    5
#> 467                     7              7        0                   11
#> 468                     4              4        1                   11
#> 469                    11             11        5                   11
#> 470                    51             40        0                   80
#> 471                    11             10        2                   19
#> 472                     9              1        3                    9
#> 473                    17              9        6                   12
#> 474                    18              2        5                   20
#> 475                     1              1        5                   30
#> 476                    14             14        0                   14
#> 477                     1              1        2                    1
#> 478                    12             12        9                   16
#> 479                     9              9        3                   10
#> 480                     2              2        1                   25
#> 481                     2              2        0                   34
#> 482                    16             12        4                   32
#> 483                     1              1        1                    1
#> 484                     7              1        0                    7
#> 485                     8              8        2                    8
#> 486                    27              6        2                   27
#> 487                     1              1        0                    1
#> 488                    14              1        0                   12
#> 489                    16              6        6                   16
#> 490                    14             14        3                   14
#> 491                    26             15        0                   26
#> 492                     1              1        1                   71
#> 493                     3              3        1                    4
#> 494                     8              8        0                    8
#> 495                    11             11        2                   11
#> 496                    10             10        1                   10
#> 497                    23              3        5                   23
#> 498                    10             10        0                   11
#> 499                     1              1        2                   33
#> 500                     7              5        0                   25
#> 501                     4              4        1                    4
#> 502                     1              1        0                    1
#> 503                     1              1        1                    1
#> 504                     2              2       10                   34
#> 505                    18              3        1                   12
#> 506                    10              1        3                   10
#> 507                    69             13        4                   78
#> 508                    16              2        6                   16
#> 509                     4              4        5                    4
#> 510                   180            180        0                  184
#> 511                     4              4        0                    4
#> 512                     3              3        0                    7
#> 513                     4              4        1                    4
#> 514                     0              0        2                   15
#> 515                     1              1        0                    1
#> 516                     7              7        2                    7
#> 517                     6              6        1                  105
#> 518                     2              2        6                    2
#> 519                     1              1        0                    6
#> 520                    35             24        0                   35
#> 521                     4              4        5                   13
#> 522                     3              3        0                    3
#> 523                     8              2        0                    8
#> 524                    14              3        3                   14
#> 525                     1              1        1                    7
#> 526                     6              6        0                    6
#> 527                     0              0        0                    2
#> 528                    12             12        3                   16
#> 529                     3              3        2                    3
#> 530                    16              7        4                   16
#> 531                     6              6        0                   53
#> 532                    12             12        0                   86
#> 533                    63              5        2                   73
#> 534                    17              5        1                   43
#> 535                     1              1        2                    1
#> 536                     3              3        7                    6
#> 537                    10             10        0                   10
#> 538                     0              0        1                   42
#> 539                    40             13        6                   40
#> 540                     1              1        1                    1
#> 541                     7              7        5                    7
#> 542                     5              1        1                    5
#> 543                     4              4        0                    8
#> 544                     7              2        0                   10
#> 545                     6              6        1                    6
#> 546                     7              7        2                   23
#> 547                     5              5        3                    5
#> 548                     8              7        0                  125
#> 549                     7              7        0                    7
#> 550                    24             13        0                   26
#> 551                    10              8        2                   16
#> 552                     6              6        6                    6
#> 553                    19              4        3                   19
#> 554                     6              4        1                    7
#> 555                    21             16        0                   21
#> 556                    20              5        4                   20
#> 557                     8              5        1                    8
#> 558                    11             11        0                   11
#> 559                     4              4        5                   18
#> 560                     5              5        3                   14
#> 561                    19             19        2                   20
#> 562                     0              0        0                    0
#> 563                    10             10        2                   94
#> 564                    22             22        2                   22
#> 565                     2              2        1                    4
#> 566                     0              0        0                    2
#> 567                     7              6        6                   22
#> 568                    19              4        0                   19
#> 569                     2              2        7                    2
#> 570                     4              3        2                    4
#> 571                     4              2        6                    4
#> 572                     1              1        2                    1
#> 573                     4              4        4                    4
#> 574                     0              0        0                    0
#> 575                    44              5        8                   90
#> 576                     1              1        4                    1
#> 577                     1              1        3                    5
#> 578                     1              1        1                    1
#> 579                    12             11        3                   12
#> 580                    17              9        0                   56
#> 581                     0              0        0                   12
#> 582                    14              3        0                   14
#> 583                     2              2        1                    2
#> 584                     6              6        3                    6
#> 585                     9              9        5                    9
#> 586                     1              1        1                    1
#> 587                     8              6        6                   10
#> 588                     4              4        0                   23
#> 589                     6              4        0                    6
#> 590                     0              0        6                    4
#> 591                     5              5        2                   12
#> 592                     3              1        6                   37
#> 593                     4              4        2                   12
#> 594                     5              5        1                    5
#> 595                     1              1        0                   18
#> 596                    12              8        1                   12
#> 597                     2              2        4                    7
#> 598                     6              6        4                    9
#> 599                    36             36        1                   36
#> 600                   108             11        0                  127
#> 601                     8              8        4                    8
#> 602                    28             11        0                   28
#> 603                    24             24        0                   43
#> 604                     8              7        2                    8
#> 605                    68              9        2                   68
#> 606                     2              2        0                    2
#> 607                     7              7        0                   85
#> 608                     4              4        2                    4
#> 609                     4              4        0                    4
#> 610                     2              2        4                   16
#> 611                     8              8        5                   15
#> 612                     4              4        1                    4
#> 613                     3              3        3                   11
#> 614                     3              3        0                    3
#> 615                     1              1        4                    1
#> 616                    17             16        1                   17
#> 617                    32              0        1                   57
#> 618                    17              9        1                   17
#> 619                    11             11        0                   11
#> 620                     1              1        4                    1
#> 621                    14             10        3                   14
#> 622                     1              1        0                    1
#> 623                     1              1        1                    1
#> 624                     2              2        0                   11
#> 625                    19             19        1                   27
#> 626                     5              1        6                   50
#> 627                     4              4        3                   32
#> 628                    19              4        1                   19
#> 629                    50             25        0                   50
#> 630                     0              0        6                   10
#> 631                    14              2        0                   14
#> 632                     0              0        6                    0
#> 633                    43              4        5                   54
#> 634                     7              5        2                    7
#> 635                     5              5        2                   15
#> 636                    10             10        5                   19
#> 637                     4              4        0                   12
#> 638                     1              1        2                    1
#> 639                    17             17        1                   49
#> 640                     1              1        0                   16
#> 641                    25             23        0                   27
#> 642                    12              7        2                   12
#> 643                     1              1        0                    1
#> 644                    16             11        3                  105
#> 645                    21             21        1                   21
#> 646                     2              1        2                    2
#> 647                     9              1        0                   18
#> 648                     1              1        3                    1
#> 649                     4              3        3                    4
#> 650                     2              2        0                    2
#> 651                     2              2        3                    2
#> 652                     7              7        0                    7
#> 653                     4              4        2                    4
#> 654                    17             14        4                   62
#> 655                     4              4        2                   43
#> 656                     5              5        2                    5
#> 657                     2              2        0                    2
#> 658                    31             31        3                   31
#> 659                    15              3        0                   15
#> 660                     7              3        1                   90
#> 661                     8              8        1                    8
#> 662                   101             56        3                  101
#> 663                     6              6        4                    6
#> 664                    51              1        2                   51
#> 665                     2              2        3                    2
#> 666                     3              3        1                    3
#> 667                    28              8        4                   50
#> 668                     4              4        0                    4
#> 669                     4              4        0                    4
#> 670                    21              2        0                   21
#> 671                     3              3        0                    5
#> 672                     1              1        1                    1
#> 673                     5              5        2                   43
#> 674                     6              6        5                   54
#> 675                    34             34        2                   34
#> 676                     6              2        1                   41
#> 677                     4              4        2                    4
#> 678                    14             14        0                   14
#> 679                     2              2        1                    2
#> 680                    10              7        5                   10
#> 681                     2              2        1                    2
#> 682                    26              2        1                   26
#> 683                     1              1        2                  163
#> 684                    26              7        4                   26
#> 685                    13              6        0                   13
#> 686                    46             27        0                  120
#> 687                     0              0        1                   51
#> 688                    45             17        1                   45
#> 689                     1              1        0                   36
#> 690                     3              3        1                   25
#> 691                     0              0        2                    7
#> 692                     6              6        0                    6
#> 693                     6              6        0                    6
#> 694                     5              5        0                    5
#> 695                    53              7        6                   53
#> 696                    13             13        2                   25
#> 697                     8              8        2                    8
#> 698                     5              5        1                    5
#> 699                    10              3        2                   14
#> 700                    10             10        2                   10
#> 701                     5              5        5                    8
#> 702                     2              2        0                    2
#> 703                     4              4        4                    4
#> 704                     5              5        1                    7
#> 705                    31              9        1                   42
#> 706                    38             23        3                   38
#> 707                     2              2        1                    2
#> 708                     2              2        5                    2
#> 709                     1              1        6                    1
#> 710                     5              5        0                    5
#> 711                    18             11        4                   33
#> 712                     8              8        0                    8
#> 713                    12              6        1                   16
#> 714                    38             38        1                   79
#> 715                     1              1        3                    1
#> 716                    50             50        4                   50
#> 717                     1              1        2                    7
#> 718                    56             10        2                  137
#> 719                     5              3        3                   30
#> 720                     4              4        4                    4
#> 721                     3              3        0                    3
#> 722                     1              1        0                   13
#> 723                     6              2        8                   16
#> 724                    46             10        1                  153
#> 725                     6              6        1                    6
#> 726                     3              3        0                    3
#> 727                     1              1        0                    1
#> 728                     8              5        5                    8
#> 729                    13             13        5                   13
#> 730                     6              4        0                    6
#> 731                     2              2        3                   33
#> 732                    14              2        0                   19
#> 733                    17              4        1                   17
#> 734                     6              6        1                  107
#> 735                    15              5        0                   49
#> 736                    16              3        0                   16
#> 737                     2              2        3                    6
#> 738                    12             12        0                   24
#> 739                     1              1        0                    1
#> 740                     6              4        2                   31
#> 741                     6              6        0                    6
#> 742                     2              2        1                    2
#> 743                     7              7        4                    7
#> 744                     3              3        2                    5
#> 745                     3              3        5                    3
#> 746                    16              5        5                   16
#> 747                     2              2        0                    2
#> 748                    36             20        0                   73
#> 749                     3              3        0                    3
#> 750                    16             13        1                   42
#> 751                     9              9        2                    9
#> 752                     8              8        0                    8
#> 753                    11              4        0                  116
#> 754                     1              1        1                    1
#> 755                    14              7        5                   26
#> 756                     1              1        1                    1
#> 757                     9              9        5                    9
#> 758                    19              4        0                   19
#> 759                     4              4        1                    4
#> 760                    34              3        1                   34
#> 761                     8              8        1                   45
#> 762                     5              5        0                    5
#> 763                     5              5        2                    5
#> 764                    10              9        1                   26
#> 765                     5              5        0                    5
#> 766                     2              2        5                   56
#> 767                     4              4        2                    9
#> 768                     2              2        5                    2
#> 769                     6              3        1                    6
#> 770                     1              1        5                    1
#> 771                    31              7        2                   48
#> 772                     1              1        4                    1
#> 773                    31              7        3                   40
#> 774                    99             17        3                  113
#> 775                     1              1        0                    2
#> 776                    22             22        0                   25
#> 777                     5              5        1                    5
#> 778                     0              0        3                    0
#> 779                    41             41        0                   12
#> 780                    36              3        2                  106
#> 781                    13             13        0                   40
#> 782                     5              5        2                    5
#> 783                     1              1        0                    1
#> 784                     2              2        1                   36
#> 785                    38             13        0                   38
#> 786                     8              8        0                    8
#> 787                     4              4        3                    4
#> 788                    20             16        0                   48
#> 789                    18             12        1                   18
#> 790                   110             25        1                  110
#> 791                     3              3        2                    3
#> 792                     4              4        0                    4
#> 793                     2              2        6                    3
#> 794                     3              3        1                    8
#> 795                    18              1        0                   18
#> 796                    54             12        2                  127
#> 797                    11             11        0                   11
#> 798                     0              0        2                    2
#> 799                     3              3        7                    3
#> 800                     8              2        0                    9
#> 801                    19              4        5                   19
#> 802                    24             24        0                   30
#> 803                    12              5        4                   28
#> 804                    24             11        0                   34
#> 805                    48             48        2                   48
#> 806                     1              1        0                    1
#> 807                     0              0        3                   14
#> 808                     2              2        0                   52
#> 809                    36             19        0                   36
#> 810                    11             11        0                   11
#> 811                     3              3        4                    9
#> 812                    11             11        0                   11
#> 813                     9              9        5                    9
#> 814                    28              2        0                   28
#> 815                    11              4        0                   11
#> 816                     5              3        1                   44
#> 817                     0              0        0                   13
#> 818                    31             15        0                  123
#> 819                     1              1        0                    1
#> 820                     1              1        0                    1
#> 821                     7              7        0                   11
#> 822                    70             25        3                   70
#> 823                     9              9        5                    9
#> 824                     3              3        6                    3
#> 825                    20              5        0                   20
#> 826                     4              4        9                    4
#> 827                     1              1        3                    3
#> 828                     5              5        0                    5
#> 829                     1              1        0                   38
#> 830                     4              4        0                    4
#> 831                     5              5        0                  102
#> 832                     5              5        0                    5
#> 833                     3              3        0                    3
#> 834                     5              5        1                   16
#> 835                     9              8        5                    9
#> 836                    16              3        1                   16
#> 837                     5              5        1                    5
#> 838                    18             15        1                   18
#> 839                    14              6        0                   19
#> 840                    27             16        0                   27
#> 841                     6              6        5                   61
#> 842                    25             25        4                   25
#> 843                    24             24        0                   24
#> 844                     5              5        2                   12
#> 845                    19             19        2                   19
#> 846                    17             17        1                   32
#> 847                    55             29        0                   55
#> 848                     7              7        1                    7
#> 849                    48             13        0                   48
#> 850                    26              3        0                   26
#> 851                     4              4        1                    4
#> 852                     2              2        0                   13
#> 853                     0              0        4                    0
#> 854                     0              0        7                   29
#> 855                    17             14        3                   44
#> 856                     2              2        1                    3
#> 857                     1              1        0                    2
#> 858                     2              2        1                    2
#> 859                    14              2        0                   14
#> 860                    22              9        1                   22
#> 861                     4              4        6                   40
#> 862                     3              3        4                   25
#> 863                    29              4        0                   30
#> 864                    10              3        1                   10
#> 865                     1              1        6                    1
#> 866                    13             11        1                   20
#> 867                     5              1        5                   19
#> 868                    11             11        2                   11
#> 869                     2              2        0                    2
#> 870                     3              3        5                    4
#> 871                    17              8        6                   17
#> 872                    32             28        0                   61
#> 873                     7              7        0                    7
#>     mths_since_recent_inq num_accts_ever_120_pd num_actv_bc_tl num_bc_tl
#> 1                       4                     2              2         5
#> 2                       0                     0              5        17
#> 3                      10                     0              2         4
#> 4                       1                     0              4         9
#> 5                       5                     0              2         2
#> 6                      10                     0              4         5
#> 7                       8                     1              2         3
#> 8                       1                     0              6        10
#> 9                      10                     1              3         6
#> 10                     18                     0              7        11
#> 11                      2                     0              6         8
#> 12                      5                     0              7        11
#> 13                      9                     0              3         3
#> 14                      0                     0              3         6
#> 15                     10                     0              5        12
#> 16                      0                     0              2         2
#> 17                      9                     0              2         9
#> 18                     11                     2              6         8
#> 19                     10                     0              4        10
#> 20                     19                     0              1         1
#> 21                      6                     0              2         6
#> 22                     14                     0              1         3
#> 23                      4                     0              3        11
#> 24                      1                     0              3        13
#> 25                     10                     0              2         7
#> 26                      5                     0              2         4
#> 27                     14                     1              3         3
#> 28                      7                     0             10        12
#> 29                     15                     4              3         6
#> 30                      2                     0              6        21
#> 31                      1                     0              2         5
#> 32                      0                     0              3         4
#> 33                      5                     0              6         9
#> 34                      0                     0              3         3
#> 35                      9                     0              3         5
#> 36                      9                     5              8         9
#> 37                      6                     0              3         4
#> 38                     19                     2              5         8
#> 39                     11                     0              3        10
#> 40                      0                     0              2        14
#> 41                      5                     0              2         5
#> 42                      6                     0              3        11
#> 43                      0                     3              1         1
#> 44                      1                     0              4         5
#> 45                      8                     0              3         6
#> 46                     16                     0             14        18
#> 47                     15                     1              3         6
#> 48                     10                     0              6        13
#> 49                      1                     0              2         2
#> 50                      5                     0              7        20
#> 51                      5                     0              5         5
#> 52                      2                     0              3         3
#> 53                      5                     0              3         6
#> 54                      1                     0              4         8
#> 55                      0                     0              7         8
#> 56                     13                     1              6        19
#> 57                      7                     0              2         7
#> 58                      1                     1              4         6
#> 59                      5                     0              2         4
#> 60                      0                     0              2         7
#> 61                     12                     0              1        15
#> 62                      5                     0              1         3
#> 63                      9                     0              1         6
#> 64                      4                     0              5         7
#> 65                     10                     0              6        12
#> 66                      7                     0              2         4
#> 67                     11                     0              5        10
#> 68                      0                     1              5        11
#> 69                      8                     0              7        17
#> 70                      3                     1              3        13
#> 71                     21                     0              4         4
#> 72                      1                     1              3         5
#> 73                      2                     1              6        11
#> 74                     11                     0              4         5
#> 75                      0                     0              2         5
#> 76                      0                     0              3        10
#> 77                      3                     0              1         9
#> 78                      1                     0              7        19
#> 79                     11                     0              0         3
#> 80                     13                     1              3         6
#> 81                      2                     0              7        11
#> 82                      6                     0              2        15
#> 83                      1                     0              3        10
#> 84                      0                     0              3        10
#> 85                      1                     0              5        10
#> 86                     12                     0              0         0
#> 87                      0                     0              3         5
#> 88                      1                     1              5        12
#> 89                     11                     0              2         5
#> 90                     19                     0              6        15
#> 91                     14                     0              2        10
#> 92                      9                     0              1         5
#> 93                      4                     2              3         7
#> 94                      7                     0              2         3
#> 95                      6                     0              3         3
#> 96                      4                     0              3         9
#> 97                      3                     0              4         6
#> 98                      2                     0              6        10
#> 99                      0                     1              2         2
#> 100                     6                     0              2         9
#> 101                     0                     0              4         5
#> 102                     3                     0              2         2
#> 103                     1                     0              4        11
#> 104                     7                     0              2        23
#> 105                     1                     1              3         5
#> 106                     0                     1              6         9
#> 107                     1                     0              4        11
#> 108                    15                     0              3         4
#> 109                     5                     0              3         5
#> 110                     9                     0              4        13
#> 111                     2                     0              8        10
#> 112                     1                     0              1         5
#> 113                     3                     0              2         9
#> 114                     7                     0              1         6
#> 115                     5                     0             10        13
#> 116                     9                     0              3         8
#> 117                     3                     1              7         8
#> 118                     1                     0              1        10
#> 119                     2                     0              3         7
#> 120                     0                     0              1         6
#> 121                    16                     0              2        14
#> 122                     0                     0              8         9
#> 123                    13                     0              6        12
#> 124                     5                     0              2        11
#> 125                     4                     0              4        13
#> 126                     9                     0              3         6
#> 127                    11                     0              3         6
#> 128                     4                     0              5        10
#> 129                    16                     0              6        11
#> 130                     6                     0              2         4
#> 131                     8                     0              4        11
#> 132                     5                     0              1         6
#> 133                     3                     0              1         5
#> 134                     7                     0              2         8
#> 135                     5                     0              6         7
#> 136                     4                     0              3        17
#> 137                     0                     1              1         6
#> 138                    12                     0              6         8
#> 139                    18                     0              3         3
#> 140                    17                     0              8        11
#> 141                     5                     0              6         9
#> 142                     5                     0              5         6
#> 143                     5                     0              3         9
#> 144                    23                     0              6         7
#> 145                     4                     0              2         5
#> 146                    15                     0              5         5
#> 147                     4                     0              5        13
#> 148                     4                     0              4        11
#> 149                     2                     0              2         4
#> 150                    12                     0              2        11
#> 151                     5                     0              2         5
#> 152                     2                     0              3         5
#> 153                     4                     0              1         1
#> 154                     5                     0              1        12
#> 155                     9                     1              3         5
#> 156                     5                     0              4        11
#> 157                     5                     0              2         2
#> 158                     1                     0              6        10
#> 159                     3                     0              0         2
#> 160                     0                     0              4         5
#> 161                     5                     0              3         5
#> 162                     0                     3              2         9
#> 163                    13                     0              4         7
#> 164                     1                     0              3         7
#> 165                    18                     0              5         9
#> 166                     9                     0              3         4
#> 167                    19                     0              2         5
#> 168                     1                     1              8        15
#> 169                     1                     2              6         6
#> 170                     7                     0              3         5
#> 171                    23                     0              2         6
#> 172                     5                     0              4         5
#> 173                     0                     0              2         6
#> 174                     4                     0              3         9
#> 175                     4                     0              4         5
#> 176                     1                     0              5         8
#> 177                     5                     0              2         9
#> 178                     5                     0              5        12
#> 179                     0                     1              1        13
#> 180                     5                     0              4        19
#> 181                     2                     0              4         4
#> 182                     8                     2              4        13
#> 183                     8                     1              4        14
#> 184                     8                     0              7         7
#> 185                     0                     0              5         8
#> 186                     4                     0              6         8
#> 187                    11                     0              9        12
#> 188                     2                     0              6         8
#> 189                     2                     0              3         3
#> 190                    10                     0              3         7
#> 191                     1                     0              5        19
#> 192                     6                     0              4         6
#> 193                     7                     0              4         8
#> 194                     0                     0              5         6
#> 195                     7                     7              1         8
#> 196                    16                     0              3         4
#> 197                     0                     0              1         3
#> 198                     0                     1              2        10
#> 199                    20                     2              5         8
#> 200                     5                     0              2         5
#> 201                     3                     0             10        13
#> 202                    11                     0              2         3
#> 203                     5                     0              2         2
#> 204                     7                     0              1         2
#> 205                     7                     0              1         3
#> 206                    19                     0              5         9
#> 207                     4                     0              2         4
#> 208                    10                     1              3         7
#> 209                     5                     0              2         6
#> 210                    15                     0              4         5
#> 211                    17                     0              8        21
#> 212                     5                     0              6         8
#> 213                    11                     0              5         6
#> 214                     0                     0              4        13
#> 215                     6                     0              4         9
#> 216                     0                     1              1        14
#> 217                     3                     0              4        12
#> 218                     5                     0              4         6
#> 219                     1                     0              1         8
#> 220                     2                     2              0         4
#> 221                     5                     0              2         4
#> 222                    21                     0              5        11
#> 223                     2                     2              3         9
#> 224                    20                     0              5         9
#> 225                     0                     0              3        11
#> 226                     3                     0              4         6
#> 227                    16                     1              3         7
#> 228                     5                     0              5        10
#> 229                     7                     0              3         5
#> 230                    12                     0              2        10
#> 231                    20                     0              4         4
#> 232                     0                     8              3        12
#> 233                     5                     3              3        15
#> 234                     0                     0              3         4
#> 235                     4                     0              2         5
#> 236                     6                     0              9        11
#> 237                     8                     0              1         7
#> 238                    13                     0              5         7
#> 239                     7                     0              4         6
#> 240                     2                     3              3         4
#> 241                     6                     0              2         4
#> 242                     9                     0              3         6
#> 243                     5                     0              3         9
#> 244                     6                     0              3         9
#> 245                     6                     0              5         6
#> 246                     0                     1              6        14
#> 247                     9                     0              3         7
#> 248                    23                     1              2         4
#> 249                     3                     1              3         5
#> 250                     8                     0              6        11
#> 251                     5                     0              3         7
#> 252                     4                     0              2        11
#> 253                     5                     0              1         5
#> 254                     4                     0              6        15
#> 255                     5                     0              8        10
#> 256                     4                     0              5        11
#> 257                     3                     5              4        18
#> 258                     3                     2              3        11
#> 259                     9                     0              5        10
#> 260                     0                     0              7        13
#> 261                     6                     0              3        13
#> 262                     5                     0              2         9
#> 263                     0                     0              1         7
#> 264                     5                     0              1         6
#> 265                     5                     0              1         4
#> 266                     0                     0              3         6
#> 267                     1                     0             10        14
#> 268                     0                     0              3        44
#> 269                     6                     0              2         3
#> 270                    16                     0              5        11
#> 271                    23                     0              4        10
#> 272                    18                     0              3         4
#> 273                     5                     0              3         8
#> 274                    12                     0              5         6
#> 275                     8                     0              3         4
#> 276                     5                     0              5         8
#> 277                     2                     2              6         9
#> 278                     1                     0              1         2
#> 279                     0                     2              0         1
#> 280                     5                     0              2         5
#> 281                     0                     1              2        18
#> 282                    10                     0              1         2
#> 283                     0                     0              3         5
#> 284                     4                     0              6        10
#> 285                     1                     0              1         6
#> 286                     6                     0              3         5
#> 287                     1                     0              6        15
#> 288                     8                     1              2         6
#> 289                     9                     0              4        10
#> 290                     9                     0              2        13
#> 291                     2                     0              6        27
#> 292                     5                     0              3         8
#> 293                     8                     1              4        10
#> 294                     5                     0              4        12
#> 295                     5                     0              8        21
#> 296                     1                     0              1         4
#> 297                     5                     1              3         4
#> 298                     4                     0              3        18
#> 299                    21                     1              6        15
#> 300                     5                     0              3         3
#> 301                    15                     3              3        10
#> 302                    19                     0              2        13
#> 303                     3                     0              2         7
#> 304                     2                     0              4         8
#> 305                     6                     0              3        16
#> 306                     1                     0              5         8
#> 307                     5                     0              2         2
#> 308                     0                     0              5        13
#> 309                     0                     0              3         4
#> 310                     6                     0              5        11
#> 311                     3                     0              8        10
#> 312                    23                     1              7         7
#> 313                     5                     0              5        14
#> 314                     8                     2              2         2
#> 315                     3                     0              4         4
#> 316                     8                     1              3         9
#> 317                     1                     0              3        11
#> 318                     4                     0              1         1
#> 319                     5                     0              2         2
#> 320                     4                     0              4         6
#> 321                     4                     1              7         9
#> 322                     5                     0              3         7
#> 323                     7                     0              5         9
#> 324                    10                     1              4         5
#> 325                     4                     0              3         5
#> 326                     3                     0              1         2
#> 327                     8                     0              0         0
#> 328                     3                     0              4         8
#> 329                     5                     0              1         3
#> 330                     1                     0              7        16
#> 331                    13                     0              3         3
#> 332                     2                     3              7        18
#> 333                     4                     0              2         6
#> 334                     0                     1              1         6
#> 335                     3                     0              3         6
#> 336                    18                     0              4         5
#> 337                     2                     3              3         6
#> 338                     0                     0              2         5
#> 339                     4                     1              3         5
#> 340                     5                     0              3         7
#> 341                     5                     0              2         3
#> 342                     6                     0              5         9
#> 343                     2                     0             16        21
#> 344                     4                     1              2         6
#> 345                    23                     0              5         9
#> 346                    12                     1              5         8
#> 347                    16                     0              4        10
#> 348                    19                     0              1         5
#> 349                     3                     0              1         5
#> 350                     5                     0              3         3
#> 351                     9                     0              1         4
#> 352                    12                     6              1         5
#> 353                    20                     0              4         5
#> 354                     4                     0              6        10
#> 355                     3                     0              2         9
#> 356                     9                     0              4        17
#> 357                     1                     0              3         5
#> 358                     2                     0              1         3
#> 359                     6                     0              2         3
#> 360                     2                     0              3         9
#> 361                     0                     0              3         7
#> 362                     8                     4              4        11
#> 363                    11                     0              3         3
#> 364                     2                     0              1         7
#> 365                    15                     2              4         5
#> 366                     5                     0              5        10
#> 367                     2                     0              3         6
#> 368                    13                     0              3        10
#> 369                     4                     0              2         9
#> 370                     4                     1              4         6
#> 371                     1                     0              5        16
#> 372                    11                     0              6        17
#> 373                     5                     1              4         8
#> 374                     0                     0              6        11
#> 375                     0                     0              2        10
#> 376                    23                     5              3         8
#> 377                     0                     3              6        15
#> 378                     3                     0              3        11
#> 379                     4                     1              4         4
#> 380                     5                     0              4         9
#> 381                    10                     4              5         7
#> 382                     0                     0              6        21
#> 383                     7                     2              2         6
#> 384                    17                     0              4         5
#> 385                     8                     0              2         2
#> 386                     2                     1              2         3
#> 387                     5                     0              6         9
#> 388                     6                     0              2         4
#> 389                    11                     0              2         3
#> 390                     3                     0              7        18
#> 391                    14                     0              1         4
#> 392                     1                     0              3         8
#> 393                     0                     0              9         9
#> 394                     0                     0              3        12
#> 395                    12                     0              3         3
#> 396                     5                     0              4         7
#> 397                     2                     0              2         5
#> 398                     3                     0              4        10
#> 399                    15                     0              4         8
#> 400                     7                     1              2         3
#> 401                     9                     1              2         3
#> 402                    22                     0              3         7
#> 403                     5                     0              4        12
#> 404                    13                     0              3         5
#> 405                     1                     0              2         4
#> 406                     7                     0              3         3
#> 407                     2                     0              2        10
#> 408                     8                     2              2         7
#> 409                    11                     0              5         6
#> 410                     9                     0              3         5
#> 411                     5                     0              6        21
#> 412                     2                     0              2         4
#> 413                     3                     0              4        12
#> 414                     1                     0              1         1
#> 415                    20                     3              7        18
#> 416                     3                     0              4        10
#> 417                     1                     2              3        13
#> 418                     9                     0              0         5
#> 419                     0                     0              2         5
#> 420                     3                     0              2         5
#> 421                     4                     0              3         9
#> 422                     0                     1              4         6
#> 423                     6                     0              3         5
#> 424                     0                     0              6        14
#> 425                     2                     0              1        13
#> 426                    22                     0              3         6
#> 427                     1                     0              5        12
#> 428                    15                     0              1        13
#> 429                     7                     1              2         4
#> 430                     1                     0              7        19
#> 431                     9                     0              7        25
#> 432                     2                     2              2         6
#> 433                     8                     0              6        17
#> 434                     5                     0              6         7
#> 435                     2                     0              2         8
#> 436                     0                     0              6        12
#> 437                     2                     6              1         4
#> 438                     1                     0              1         2
#> 439                     5                     0              5         8
#> 440                     8                     1              3         6
#> 441                     0                     0              3         5
#> 442                     3                     0              3         9
#> 443                     1                     0              7        13
#> 444                     7                     0              2         8
#> 445                     0                     4              4         5
#> 446                     3                     0              3         3
#> 447                     1                     0              2         5
#> 448                     1                     0              4         6
#> 449                    15                     0              4         4
#> 450                     0                     0              7        18
#> 451                     3                     0              6         8
#> 452                    12                     0              2         4
#> 453                    11                     0              6         7
#> 454                     8                     1              2         9
#> 455                    21                     0              2         7
#> 456                     2                     1              4         9
#> 457                     5                     2             10        20
#> 458                     5                     0              3         5
#> 459                    10                     1              3         5
#> 460                     1                     1              6         9
#> 461                     1                     0              5         9
#> 462                    22                     0              1         4
#> 463                     8                     0              3         5
#> 464                    11                     0              5        10
#> 465                     5                     0              4         5
#> 466                     2                     0              6        11
#> 467                     3                     1              8        11
#> 468                     4                     0              2         4
#> 469                     2                     1              5         7
#> 470                     5                     0              1         1
#> 471                     6                     1              2         4
#> 472                     4                     0              3         3
#> 473                     8                     0              0         4
#> 474                    15                     0              4        11
#> 475                     1                     0             11        18
#> 476                    10                     0              7         8
#> 477                     5                     0              7         8
#> 478                     5                     0              2         6
#> 479                     3                     0              3        19
#> 480                     8                     1              2         5
#> 481                     2                     0              5         6
#> 482                    18                     0              1         7
#> 483                     5                     1              7         9
#> 484                    18                     0              7        14
#> 485                    15                     0              4         7
#> 486                     9                     0              4         5
#> 487                     5                     0              3         4
#> 488                     3                     0              0         5
#> 489                     6                     0              4         8
#> 490                    17                     0              3         8
#> 491                     8                     0              3         3
#> 492                     1                     1              3         5
#> 493                     5                     0              3         3
#> 494                     5                     0              5         8
#> 495                    15                     0              2        11
#> 496                    15                     0              4         9
#> 497                     3                     0              4         6
#> 498                    10                     0              7        13
#> 499                     1                     0              2         2
#> 500                     5                     0              4         7
#> 501                    10                     0              9         9
#> 502                     4                     0              2         3
#> 503                     0                     0              2         7
#> 504                     5                    17              6        15
#> 505                     9                     0              0         0
#> 506                     3                     2              4         5
#> 507                    13                     0              3        10
#> 508                     0                     1              2         9
#> 509                     2                     0              7        12
#> 510                     5                     1              3         5
#> 511                     0                     3              2         4
#> 512                     7                     1              3         6
#> 513                     5                     1              3        10
#> 514                     1                     0              5        19
#> 515                    11                     0              6         8
#> 516                     5                     0              5         7
#> 517                    13                     0              1         2
#> 518                     0                    12              1        20
#> 519                     4                     1              3        13
#> 520                     5                     0              5         9
#> 521                     5                     0             18        27
#> 522                     4                     0              4        11
#> 523                     2                     0              3        10
#> 524                     0                     0              5         7
#> 525                     1                     1              1         6
#> 526                     0                     0              5         9
#> 527                     3                     4              6        12
#> 528                     6                     0              2         4
#> 529                     3                     0              8        15
#> 530                     9                     0              3         8
#> 531                     6                     0              2         9
#> 532                    12                     0              3         3
#> 533                     4                     0              2        10
#> 534                     0                     0              3        13
#> 535                     8                     0              4         6
#> 536                     4                     3              3        11
#> 537                    10                     0              2         3
#> 538                     1                     0              2         6
#> 539                     5                     0              3         5
#> 540                     0                     1              4         5
#> 541                     7                     1              5         5
#> 542                     5                     0             11        18
#> 543                     2                     0              3        10
#> 544                     8                     1              3        10
#> 545                     0                     0              9        10
#> 546                     1                     2              1         3
#> 547                     4                     0              4        14
#> 548                     3                     0              8        11
#> 549                     0                     0              5         7
#> 550                     5                     0              2        10
#> 551                     3                     0              1         6
#> 552                     1                     1              7        17
#> 553                    23                     0              2         8
#> 554                     0                     0              3         8
#> 555                     9                     1              3         3
#> 556                     5                     0              5        10
#> 557                     8                     0              5        17
#> 558                    22                     0              5         9
#> 559                     4                     0              5        10
#> 560                     0                     0              3        13
#> 561                     4                     0              2         6
#> 562                     0                     0              5         5
#> 563                     0                     0              3         6
#> 564                    10                     0              5         7
#> 565                     1                     0              6        10
#> 566                     9                     0              3         8
#> 567                     7                     0              5        13
#> 568                     5                     0              5         7
#> 569                    15                     0              7        12
#> 570                     1                     0              5         9
#> 571                     5                     0             10        14
#> 572                    16                     0              4        10
#> 573                     4                     0              8        22
#> 574                     0                     0              3         7
#> 575                     5                     0              4         6
#> 576                     1                     0              3         9
#> 577                     1                     2              8        11
#> 578                     4                     0              7        14
#> 579                    11                     0             10        19
#> 580                     4                     0              2         5
#> 581                     5                     1              3         4
#> 582                     3                     0              2         2
#> 583                     1                     1              2         8
#> 584                    13                     0              1         8
#> 585                    22                     3              6        12
#> 586                     0                     0              5         8
#> 587                     4                     0              4         8
#> 588                     5                     0              2         4
#> 589                     6                     0              2         5
#> 590                     0                     0              5        14
#> 591                     5                     0              4         4
#> 592                     1                     0              1         5
#> 593                     9                     0              4         7
#> 594                    19                     0              4         8
#> 595                    22                     0              4         6
#> 596                     9                     1             10        13
#> 597                     3                     1              4         7
#> 598                     6                     0              7        13
#> 599                    13                     0              1         1
#> 600                     5                     0              2         5
#> 601                     5                     0              9        11
#> 602                     2                     0              2         7
#> 603                     5                     0              3         9
#> 604                     8                     0             10        11
#> 605                     5                     0              3         6
#> 606                     5                     0              3         5
#> 607                    16                     0              2         9
#> 608                     0                     0              5         7
#> 609                    15                     0              7        10
#> 610                    10                     2              1         7
#> 611                     8                     3              5         8
#> 612                     4                     0              2         3
#> 613                     0                     0              3         8
#> 614                     5                     0              4        14
#> 615                     1                     0              7        13
#> 616                    12                     0              5         6
#> 617                     1                     0              1         1
#> 618                     9                     1              3         5
#> 619                     5                     0              6         6
#> 620                     1                     0              4        13
#> 621                    10                     2              3         4
#> 622                     0                     2              2        10
#> 623                     6                     0              4         7
#> 624                     2                     0              1         6
#> 625                    21                     0              1         3
#> 626                     1                     0              1         4
#> 627                     7                     0              2         3
#> 628                    14                     0              4         8
#> 629                     5                     0              1         5
#> 630                     1                     0              8        13
#> 631                    14                     7              4        14
#> 632                     1                     0              6        11
#> 633                     5                     0              2         3
#> 634                     8                     0              5        14
#> 635                     5                     0              5         5
#> 636                     3                     0              2         5
#> 637                     8                     0              1         1
#> 638                    21                     0              6        21
#> 639                     5                     0              1         7
#> 640                     4                     0              2         3
#> 641                    23                     0              6         7
#> 642                     7                     0              7         9
#> 643                     5                     0              2         4
#> 644                    16                     0              3         7
#> 645                     5                     0              4         6
#> 646                     0                     0              3         4
#> 647                     1                     0              6        14
#> 648                     7                     4              3        22
#> 649                     5                     0              4         8
#> 650                     7                     1              2         3
#> 651                     8                     0              2         5
#> 652                     1                     4              3        11
#> 653                     6                     0              5        17
#> 654                    13                     0              2         3
#> 655                    20                     0              2         7
#> 656                     5                     0              5         8
#> 657                     2                     0              2        12
#> 658                     0                     0              3         8
#> 659                     3                     0              3         5
#> 660                     0                     0              3         6
#> 661                     9                     1              5         6
#> 662                     5                     0              2         4
#> 663                    12                     2              8        14
#> 664                     0                     0              3        10
#> 665                     2                     0              3        10
#> 666                     6                     0              4         4
#> 667                     9                     0              1         4
#> 668                     8                     0              4        12
#> 669                     4                     1              7         9
#> 670                     5                     0              5        16
#> 671                     3                     1              3         5
#> 672                    10                     1              4         6
#> 673                    11                     0              4        15
#> 674                     6                     0              1         1
#> 675                     5                     0              8        13
#> 676                     2                     0              2         5
#> 677                     6                     0              5        11
#> 678                    14                     0              6         9
#> 679                     2                     0              3         9
#> 680                     1                     3              2         6
#> 681                     3                     0              3         5
#> 682                     2                     2              4         6
#> 683                     1                     0              1         6
#> 684                     7                     0              2         5
#> 685                     6                     0              6        12
#> 686                     5                     0              2        12
#> 687                     0                     0              3         3
#> 688                     5                     0              5         6
#> 689                    10                     0              1         7
#> 690                     5                     0              3         4
#> 691                     4                     1              8        17
#> 692                     0                     4              0         7
#> 693                    11                     0              2         6
#> 694                     0                     4              4        10
#> 695                    22                     0              3         8
#> 696                     5                     0              4         4
#> 697                    13                     0              5        21
#> 698                     5                     0              2         6
#> 699                     0                     0              3         6
#> 700                     0                     0              5         5
#> 701                     5                     0              4         6
#> 702                     0                     3              9        15
#> 703                     5                     0              3        12
#> 704                     5                     0              3         8
#> 705                     5                     0              1         7
#> 706                    15                     0              3         9
#> 707                     5                     0              3         4
#> 708                     6                     0              1        10
#> 709                     7                     0              4        14
#> 710                     5                     0              6        10
#> 711                     1                     0              3        14
#> 712                     1                     0              4         5
#> 713                     1                     0              6        19
#> 714                     1                     0              5         7
#> 715                     1                     0              8        21
#> 716                     5                     0              4        10
#> 717                    13                     0              3        19
#> 718                    10                     2              3        10
#> 719                     4                     0              2        10
#> 720                     7                     0              5        12
#> 721                    10                     1              4         4
#> 722                     1                     0              1         1
#> 723                     2                     1              5        10
#> 724                    11                     0              1         2
#> 725                     0                     0              5         8
#> 726                     3                     4              5         6
#> 727                     5                     0              2        10
#> 728                     0                     0              3        10
#> 729                    16                     0              5        10
#> 730                     4                     2              8        11
#> 731                     2                     0              4         6
#> 732                     7                     0              1         3
#> 733                     1                     0              9        10
#> 734                     7                     2              4         8
#> 735                     4                     1              2         3
#> 736                     5                     0              3         3
#> 737                     2                     0              2         3
#> 738                    12                     0              3         7
#> 739                     9                     0              4        11
#> 740                     6                     1              3        13
#> 741                     0                     1              1        12
#> 742                    13                     0              7         8
#> 743                    10                     0              7        16
#> 744                     5                     0              4         7
#> 745                     3                     1              2        10
#> 746                     5                     1             12        14
#> 747                     6                     0              6        11
#> 748                    20                     0              1         8
#> 749                    10                     1              9        10
#> 750                    13                     2              5         7
#> 751                    13                     0              1         7
#> 752                     8                     0              4        11
#> 753                     0                     1              2         7
#> 754                    13                     1              5        10
#> 755                     3                     2              1        15
#> 756                     2                     0              5         8
#> 757                     9                     0              1         6
#> 758                     4                     0              3         5
#> 759                    20                     0              1         6
#> 760                     8                     0              2         2
#> 761                    13                     1              0         3
#> 762                     5                     0              7        11
#> 763                     3                     0              4        15
#> 764                    10                     0              2         6
#> 765                     5                     1              3         8
#> 766                     6                     3              2         7
#> 767                     6                     0              4        10
#> 768                    20                     0              4        14
#> 769                     3                     0              2         7
#> 770                     0                     1              2        18
#> 771                     7                     1              2         4
#> 772                     1                     1              4         9
#> 773                     5                     0              3         9
#> 774                    20                     1              6         9
#> 775                     0                     0              5         7
#> 776                    22                     0              4         4
#> 777                     3                     0              3        14
#> 778                     3                     0              6         7
#> 779                    20                     0              0         9
#> 780                    12                     0              1         5
#> 781                     5                     0              8        12
#> 782                     5                     0              7        13
#> 783                    15                     0              5         7
#> 784                     4                     0              2         5
#> 785                    14                     0              4         6
#> 786                     1                     2              4        10
#> 787                     7                     4              5        16
#> 788                     4                     0              5         9
#> 789                     3                     1              5        13
#> 790                     5                     0              3         5
#> 791                     0                     3              3         9
#> 792                     0                     0              6        10
#> 793                     5                     0              3        16
#> 794                     2                     0              3         4
#> 795                     1                     0              3         7
#> 796                    23                     0              3         4
#> 797                    14                     0              3         4
#> 798                     7                     2              4         8
#> 799                     5                     0              5         6
#> 800                     2                     0              4         9
#> 801                     5                     0              3         4
#> 802                     5                     0              3         5
#> 803                     7                     0              1        10
#> 804                     8                     0              5         9
#> 805                     7                     0              5         5
#> 806                     1                     2              2         6
#> 807                     0                     2              2         9
#> 808                     1                     0              1         2
#> 809                     7                     0              1         3
#> 810                    11                     0              5         7
#> 811                    13                     1              6        11
#> 812                    15                     0              3         6
#> 813                     8                     0              6        10
#> 814                     3                     0              2         3
#> 815                    11                    12              1         2
#> 816                     1                     1              4         4
#> 817                     6                     0              1         2
#> 818                     5                     0              3         5
#> 819                     1                     0              5        10
#> 820                    10                     1              2         6
#> 821                    17                     0              5         6
#> 822                    12                     0              3         5
#> 823                     9                     2              1         2
#> 824                    16                     4              4         9
#> 825                    10                     0              2        14
#> 826                     4                     0              5        18
#> 827                     1                     0              3        21
#> 828                     2                     0              4        15
#> 829                     1                     4              2         5
#> 830                     5                     0              4        12
#> 831                    12                     0              2         3
#> 832                    10                     0              3         5
#> 833                     3                     0             10        21
#> 834                    10                     4              4        10
#> 835                    14                     0              2         4
#> 836                     3                     0              5        10
#> 837                     8                     1              5        11
#> 838                    20                     4              0         7
#> 839                     1                     0              2         6
#> 840                    16                     0              4         7
#> 841                     4                     1              1         2
#> 842                     5                     0              4         7
#> 843                     5                     0              4        11
#> 844                     5                     5              2         8
#> 845                     5                     0              1         7
#> 846                     0                     0              1         2
#> 847                     8                     0              3         4
#> 848                    19                     1              2         4
#> 849                     5                     0              2         4
#> 850                     3                     0              2         6
#> 851                     4                     0              8        11
#> 852                     0                     0              3        19
#> 853                     0                     0              3         9
#> 854                     0                     1              9        15
#> 855                    14                     0              1         3
#> 856                    11                     0              4         7
#> 857                     2                     0              4         7
#> 858                     0                     0             10        19
#> 859                     2                    13              1         8
#> 860                     0                     0              1         3
#> 861                     4                     0              3        13
#> 862                     0                     0              5         6
#> 863                     5                     0              1         4
#> 864                     3                     0             11        15
#> 865                     4                     0              7        28
#> 866                     5                     0              4         4
#> 867                     1                     2              1         8
#> 868                    13                     1              3         4
#> 869                     5                     3              1         9
#> 870                     7                     0              2         9
#> 871                     9                     0              3         5
#> 872                     5                     0              1         3
#> 873                     5                     0              6        15
#>     num_il_tl num_op_rev_tl num_rev_tl_bal_gt_0 pct_tl_nvr_dlq percent_bc_gt_75
#> 1           3             4                   4           76.9              0.0
#> 2           6            20                   5           97.4              7.7
#> 3           6             4                   3          100.0             50.0
#> 4          10             7                   6           96.6             60.0
#> 5           2             4                   3          100.0            100.0
#> 6           7             9                   6           96.3            100.0
#> 7           9             3                   2           93.3              0.0
#> 8           3            13                   9           95.7             28.6
#> 9           5             5                   3           94.4             33.3
#> 10          3            12                  11          100.0             75.0
#> 11          6            16                  13          100.0             14.3
#> 12          4            13                   9           91.7             22.2
#> 13          5             4                   3          100.0             66.7
#> 14          7             5                   3          100.0             20.0
#> 15          6            15                   6          100.0              0.0
#> 16          2            17                  13          100.0             50.0
#> 17          8            13                   5          100.0             25.0
#> 18          6             9                   9           78.9              0.0
#> 19          3             6                   5          100.0             40.0
#> 20          3             2                   2          100.0            100.0
#> 21          4             5                   4           93.7             50.0
#> 22          1             5                   2           66.7              0.0
#> 23          4            10                   3          100.0              0.0
#> 24         10             8                   6          100.0            100.0
#> 25         24             8                   2          100.0              0.0
#> 26          5             7                   6           90.5              0.0
#> 27         12             5                   5           80.0              0.0
#> 28          2            15                  12          100.0             63.6
#> 29         17             8                   5           87.2             66.7
#> 30         15            29                  16          100.0              0.0
#> 31         14             8                   4           97.1              0.0
#> 32         11             4                   3          100.0             75.0
#> 33          1            12                  10          100.0             57.1
#> 34          2             3                   3          100.0              0.0
#> 35          1             6                   4          100.0            100.0
#> 36         39            21                  17           92.1              0.0
#> 37          6            11                   9          100.0            100.0
#> 38         11            12                  10           92.3              0.0
#> 39          5            13                   3          100.0              0.0
#> 40         25             2                   2          100.0             50.0
#> 41         35             6                   3           97.7             50.0
#> 42          6             6                   5           87.5            100.0
#> 43         14             4                   4           68.4              0.0
#> 44          8             5                   4           94.4             75.0
#> 45         19             7                   5          100.0            100.0
#> 46          9            24                  20          100.0             50.0
#> 47         26             5                   4           91.3             33.3
#> 48          7             9                   6           96.0             37.5
#> 49          3             3                   3          100.0              0.0
#> 50         25            14                   8           97.9             30.8
#> 51          3             8                   5          100.0             60.0
#> 52          4            11                  11          100.0             66.7
#> 53         13             4                   4          100.0              0.0
#> 54          5             7                   6          100.0             75.0
#> 55         13             9                   8          100.0             87.5
#> 56          2            19                  11           94.3             44.4
#> 57         14             6                   6          100.0            100.0
#> 58         13            12                  10           94.1            100.0
#> 59          7            16                   4          100.0              0.0
#> 60          9             4                   2           95.7             33.3
#> 61          8             9                   3           97.2              0.0
#> 62          4             4                   1          100.0              0.0
#> 63          7             6                   1          100.0             25.0
#> 64          1             8                   7          100.0            100.0
#> 65         14            10                   7          100.0             71.4
#> 66         16             8                   5          100.0             33.3
#> 67          3            13                   8          100.0             20.0
#> 68         10             8                   6           87.5             85.7
#> 69          5            11                   9          100.0             85.7
#> 70          6            15                   4           97.0              0.0
#> 71          0             7                   6          100.0            100.0
#> 72         28             4                   4           96.9              0.0
#> 73          4             6                   6           89.5             16.7
#> 74          1             6                   5          100.0              0.0
#> 75          3             3                   2          100.0             33.3
#> 76          8            10                   3          100.0              0.0
#> 77          8             5                   3           95.7              0.0
#> 78          6            28                  12           97.9              0.0
#> 79         10             5                   3           94.7              0.0
#> 80         21            25                  12           97.1            100.0
#> 81          3            14                   8           90.5             20.0
#> 82         24             5                   2           97.8             50.0
#> 83          8             8                   4          100.0             28.6
#> 84          2             8                   4           94.1              0.0
#> 85         12            14                   8          100.0             40.0
#> 86         14             4                   3          100.0             33.3
#> 87          2             7                   5          100.0             60.0
#> 88          7            13                  10           96.8             33.3
#> 89          6             4                   2           84.6              0.0
#> 90          0            12                   7          100.0             12.5
#> 91         13            11                   4          100.0              0.0
#> 92         11             2                   1          100.0              0.0
#> 93          1             4                   3           80.0             50.0
#> 94          6             3                   3          100.0              0.0
#> 95          5             5                   5          100.0              0.0
#> 96         15             4                   3          100.0              0.0
#> 97          4            15                   6           96.0             40.0
#> 98          4             8                   7          100.0             33.3
#> 99          8             4                   4           87.5              0.0
#> 100        28            13                   5           98.1              0.0
#> 101        17             6                   5           78.0             75.0
#> 102         7             2                   2          100.0              0.0
#> 103         3            13                   5          100.0             40.0
#> 104         4             5                   3          100.0             33.3
#> 105         3            10                   8           87.5             25.0
#> 106         4            10                   9           59.1             66.7
#> 107         4             7                   5          100.0             50.0
#> 108         9             7                   6          100.0            100.0
#> 109         2             6                   4          100.0             25.0
#> 110        12             7                   6          100.0            100.0
#> 111        12            15                  10          100.0             11.1
#> 112        14            13                   4           97.0             33.3
#> 113        10             4                   3          100.0            100.0
#> 114         6             3                   2           92.3              0.0
#> 115        28            17                  13          100.0             30.0
#> 116         6            10                   3          100.0             42.9
#> 117         5            12                  11           88.9             28.6
#> 118        11             9                   4          100.0             50.0
#> 119        19             5                   3          100.0              0.0
#> 120         5             8                   2          100.0              0.0
#> 121        36             9                   3           96.8              0.0
#> 122        10            12                  11          100.0             50.0
#> 123         3            17                  13           96.9              0.0
#> 124         1             9                   4           85.0              0.0
#> 125         2             5                   4          100.0             75.0
#> 126         5             5                   4           92.9             50.0
#> 127         2             6                   6          100.0             66.7
#> 128         7             8                   6          100.0             50.0
#> 129         2            17                  11           96.4              0.0
#> 130         8             4                   3          100.0             50.0
#> 131         4            13                   7          100.0              0.0
#> 132         3             7                   4          100.0              0.0
#> 133         1             4                   3          100.0            100.0
#> 134         3             9                   4          100.0             66.7
#> 135         3            15                   9          100.0             57.1
#> 136        11            17                   5           95.9             22.2
#> 137         3            17                   7           92.3              0.0
#> 138         3            11                   8          100.0            100.0
#> 139         1             3                   3          100.0             66.7
#> 140         3            10                   8          100.0             75.0
#> 141         5             8                   7          100.0              0.0
#> 142        59             7                   6           95.5             40.0
#> 143         8             9                   6           84.0            100.0
#> 144         6            10                   6          100.0             42.9
#> 145         4             2                   2          100.0             50.0
#> 146         6             6                   6          100.0              0.0
#> 147         1             9                   5          100.0             28.6
#> 148         7            13                  11           93.3             75.0
#> 149         5             9                   4           96.2              0.0
#> 150         7            16                  11          100.0              0.0
#> 151         5             6                   5           84.2              0.0
#> 152         1            12                   8          100.0             33.3
#> 153         4             6                   5          100.0              0.0
#> 154         8             8                   3          100.0              0.0
#> 155         0            12                   7           86.7             33.3
#> 156         4            13                   4          100.0              0.0
#> 157         0             5                   4          100.0            100.0
#> 158         8             9                   8          100.0            100.0
#> 159         2             3                   1          100.0              0.0
#> 160         8            10                  10           95.7             75.0
#> 161         3             6                   4           91.7              0.0
#> 162         1             8                   5           80.8            100.0
#> 163         2            12                  10          100.0              0.0
#> 164        31            12                   4           98.0             40.0
#> 165        16            10                   7          100.0            100.0
#> 166         2             7                   6          100.0              0.0
#> 167        27             9                   3          100.0             33.3
#> 168         8            14                  12           97.4             75.0
#> 169        11             7                   7           90.9             16.7
#> 170         2            11                   8          100.0             50.0
#> 171        10            13                   8          100.0              0.0
#> 172        12             5                   5          100.0             50.0
#> 173         1             5                   3          100.0              0.0
#> 174        10            16                   7           97.3             16.7
#> 175        12            12                  11          100.0             80.0
#> 176         4             5                   5          100.0            100.0
#> 177         3            13                   2          100.0             16.7
#> 178         8            16                  12           82.1            100.0
#> 179        16            18                   7           95.1              0.0
#> 180        11            16                   4           97.3             12.5
#> 181         9             5                   5          100.0              0.0
#> 182         9             5                   4           90.6             50.0
#> 183         9            13                   4           92.1             50.0
#> 184         2            11                   8           90.9            100.0
#> 185        11             8                   6           97.0             71.4
#> 186        17            10                  10          100.0             66.7
#> 187         8            14                  11           92.0             45.5
#> 188         9            12                  12           96.0             50.0
#> 189         4            11                  10          100.0             33.3
#> 190        10            11                   3          100.0             42.9
#> 191        21            16                   6          100.0             22.2
#> 192         5             8                   8           83.3             75.0
#> 193         2             7                   6          100.0              0.0
#> 194         6            11                  10           92.0            100.0
#> 195        12             4                   3           61.3            100.0
#> 196         4             6                   4          100.0             50.0
#> 197        16             5                   4          100.0              0.0
#> 198         6             6                   2           85.2              0.0
#> 199         2            11                   8           72.7             66.7
#> 200        29             4                   2          100.0              0.0
#> 201         4            13                  13           95.5              0.0
#> 202        11            15                  10          100.0              0.0
#> 203        27             8                   6          100.0             50.0
#> 204        12             5                   3          100.0            100.0
#> 205         1            10                   7           83.3              0.0
#> 206         6             9                   5          100.0             25.0
#> 207         3             5                   3          100.0            100.0
#> 208         9             8                   7           88.0             33.3
#> 209         7             4                   3          100.0              0.0
#> 210         2             5                   5          100.0             75.0
#> 211         7            16                   9          100.0              0.0
#> 212        17            11                   8          100.0             16.7
#> 213         6             9                   7          100.0             40.0
#> 214         3            15                   4          100.0              0.0
#> 215         5             5                   5           94.1            100.0
#> 216         1            18                   4           83.9              0.0
#> 217         8             8                   5          100.0             33.3
#> 218         4             4                   4          100.0             25.0
#> 219        11             3                   2          100.0            100.0
#> 220         6             5                   4           86.4             33.3
#> 221         3             7                   3          100.0             50.0
#> 222         9             8                   7          100.0             20.0
#> 223         7             7                   5           90.5             25.0
#> 224         4             6                   6           93.3            100.0
#> 225        17             4                   2           80.0              0.0
#> 226        16             6                   6           96.2             60.0
#> 227         6             8                   5          100.0             25.0
#> 228         9             9                   7          100.0            100.0
#> 229        25             5                   4          100.0             66.7
#> 230        22             8                   3          100.0             50.0
#> 231         0             5                   4          100.0            100.0
#> 232         3             4                   3           50.0             33.3
#> 233         6             9                   6           90.0              0.0
#> 234         3             3                   3           85.7              0.0
#> 235         8             7                   5           88.0            100.0
#> 236         7            12                   9          100.0             44.4
#> 237         9             1                   1          100.0              0.0
#> 238         5            10                   6          100.0             60.0
#> 239         7             4                   3          100.0             25.0
#> 240        25            10                  10           93.9              0.0
#> 241         3             5                   4           81.8             66.7
#> 242         6             7                   5          100.0             60.0
#> 243         5             7                   3          100.0              0.0
#> 244        12            13                   7           97.4              0.0
#> 245        11             9                   8          100.0             60.0
#> 246        13            12                  10           97.1             42.9
#> 247        26             3                   3          100.0             66.7
#> 248         9            11                   7           96.6              0.0
#> 249         2             5                   4           72.7              0.0
#> 250         5             9                   7          100.0             42.9
#> 251         5             5                   4           89.5              0.0
#> 252         7             8                   6           90.3             33.3
#> 253         6             5                   1          100.0              0.0
#> 254        11             9                   8           96.3             57.1
#> 255         7            15                  11           97.0             87.5
#> 256        11            10                   8           94.1              0.0
#> 257         6             6                   4           75.0            100.0
#> 258         8            10                   4           88.6              0.0
#> 259        13             6                   5           90.0            100.0
#> 260        11            12                   8          100.0             44.4
#> 261        18            10                   3          100.0             11.1
#> 262         3            10                   4          100.0              0.0
#> 263         6             9                   7          100.0             50.0
#> 264         7             7                   5          100.0              0.0
#> 265         3             2                   1          100.0             50.0
#> 266         5             4                   2           77.0              0.0
#> 267         4            19                  16           96.7             33.3
#> 268         6            26                   5          100.0             22.2
#> 269         6             4                   2          100.0             50.0
#> 270         2            10                   7          100.0             42.9
#> 271         6             7                   6           96.2              0.0
#> 272         6             4                   4           90.9             66.7
#> 273         9             7                   5           81.0             33.3
#> 274         7             6                   6          100.0              0.0
#> 275         2            17                  10          100.0             33.3
#> 276         5             9                   5          100.0              0.0
#> 277         2             8                   8           76.9             16.7
#> 278         7             2                   1           83.3              0.0
#> 279         6             2                   2           81.2             33.3
#> 280        12             4                   3           96.8            100.0
#> 281         9            12                   5           87.5             14.3
#> 282         4            16                  15           92.3              0.0
#> 283        15            10                   4          100.0             25.0
#> 284        16             8                   8           93.1             33.3
#> 285         7             4                   2          100.0              0.0
#> 286        27             5                   4          100.0             33.3
#> 287        28            10                   8           79.3              0.0
#> 288         3            11                   9           80.0             50.0
#> 289         6             9                   5          100.0              0.0
#> 290         8             9                   4           97.6             25.0
#> 291         7            17                   8          100.0             10.0
#> 292        22            10                   5           97.1             40.0
#> 293         7            10                   5           96.7             25.0
#> 294         8            14                   9           97.1              0.0
#> 295         6            18                  13           94.1             30.8
#> 296         3             7                   3          100.0              0.0
#> 297        10             6                   4           95.2             25.0
#> 298         1             4                   3          100.0              0.0
#> 299         4            16                   7           97.0             12.5
#> 300         2             4                   3          100.0            100.0
#> 301         6            11                   9           82.9              0.0
#> 302         5             6                   4           96.4            100.0
#> 303         5             5                   3          100.0              0.0
#> 304        22             9                   4          100.0             28.6
#> 305         9            13                   4           97.8             50.0
#> 306         4            10                   6          100.0             28.6
#> 307         4             5                   4           57.0            100.0
#> 308        19            12                   6           95.5             60.0
#> 309         6             7                   5           81.0            100.0
#> 310         7             9                   7          100.0              0.0
#> 311        13            11                  10          100.0             75.0
#> 312         3             8                   7           92.3             14.3
#> 313         2            13                   8          100.0             83.3
#> 314        13             7                   4           90.9              0.0
#> 315        14             7                   6           92.6              0.0
#> 316        14            10                   7           97.5             25.0
#> 317        12             9                   7           74.0            100.0
#> 318        13             8                   7           96.7              0.0
#> 319        20             2                   2           85.7             50.0
#> 320         4            10                   7           94.4             80.0
#> 321         4            11                  11          100.0             14.3
#> 322         2            13                   9          100.0             40.0
#> 323         5             9                   6          100.0              0.0
#> 324         5             5                   5           85.7            100.0
#> 325        23             6                   4          100.0             50.0
#> 326         8             7                   4          100.0              0.0
#> 327         2             6                   4          100.0             33.3
#> 328        17             9                   8           95.3             75.0
#> 329         1             2                   1          100.0            100.0
#> 330         2            10                   8          100.0             37.5
#> 331         2             5                   5          100.0              0.0
#> 332        17            12                   8           74.4             55.6
#> 333        27             7                   4           95.9              0.0
#> 334         1            11                   5           90.5             50.0
#> 335        10            13                   7           97.4             20.0
#> 336        14             6                   5          100.0             25.0
#> 337        14             8                   6           62.5            100.0
#> 338        10             5                   4           90.5             33.3
#> 339        20             9                   4           93.1              0.0
#> 340         1             7                   3          100.0              0.0
#> 341         6             2                   2          100.0             50.0
#> 342         9             9                   7          100.0             28.6
#> 343         6            18                  15          100.0             20.0
#> 344        18             4                   2           97.4             50.0
#> 345        18             9                   7           94.3             40.0
#> 346         5            12                   6           87.5             33.3
#> 347         5            17                  11          100.0             28.6
#> 348         3             7                   2          100.0              0.0
#> 349         7             3                   2           83.3              0.0
#> 350        10             7                   6          100.0             66.7
#> 351         2             9                   2           94.1              0.0
#> 352        20             9                   7           82.9            100.0
#> 353         5             7                   6          100.0             20.0
#> 354         8             9                   8           76.0             33.3
#> 355         7             2                   2           92.0            100.0
#> 356         4            21                  12          100.0              0.0
#> 357        10             7                   6           96.2            100.0
#> 358         6             5                   3          100.0              0.0
#> 359        19             3                   2           91.7             50.0
#> 360        44            10                   5          100.0             25.0
#> 361         3             7                   5           88.9              0.0
#> 362         6             9                   4           77.3              0.0
#> 363         1             3                   3          100.0             66.7
#> 364         9             7                   4           95.2              0.0
#> 365        13             6                   5           92.0             50.0
#> 366         3            10                   7           91.3             80.0
#> 367        12             6                   3           95.2              0.0
#> 368        24             9                   6          100.0            100.0
#> 369         4             8                   4           91.3             25.0
#> 370         2             7                   5           90.9              0.0
#> 371        17            19                   9          100.0             22.2
#> 372         3            13                   8          100.0              0.0
#> 373        13            11                   8           97.2             40.0
#> 374         0            16                   9          100.0              0.0
#> 375        28             7                   3          100.0             25.0
#> 376         6             9                   8           73.1            100.0
#> 377         7            22                   9           86.0             22.2
#> 378         2            10                  10          100.0             66.7
#> 379         8            11                  11           90.9             75.0
#> 380         5             7                   6          100.0            100.0
#> 381        11             7                   7           80.8              0.0
#> 382         8            22                   7           97.9             21.4
#> 383         5             6                   5           88.9            100.0
#> 384         2             9                   7          100.0             25.0
#> 385        20             5                   5          100.0              0.0
#> 386         8             7                   5           79.0             50.0
#> 387         3            12                   8          100.0              0.0
#> 388         9             4                   3          100.0              0.0
#> 389         4             7                   7           92.3            100.0
#> 390         7            10                   7          100.0             12.5
#> 391         9             1                   1           78.9            100.0
#> 392        25            16                  12          100.0              0.0
#> 393         4            16                   8           95.0             11.1
#> 394         8             7                   4           96.3             33.3
#> 395         9             6                   6          100.0              0.0
#> 396         6             7                   5          100.0             40.0
#> 397        26            10                   7           96.0             50.0
#> 398         4             7                   8          100.0             50.0
#> 399         2             6                   5           80.0             75.0
#> 400         3             3                   3           87.5             50.0
#> 401        26             8                   3           94.9             33.3
#> 402         1             9                   6           91.7              0.0
#> 403         2             7                   5          100.0             50.0
#> 404        11             7                   5           92.0            100.0
#> 405         9             8                   6          100.0             50.0
#> 406         3            13                  12          100.0             33.3
#> 407        10             8                   6           97.1              0.0
#> 408         4             4                   2           75.0              0.0
#> 409         1            12                   9           93.7              0.0
#> 410        20             6                   6          100.0             66.7
#> 411         6            13                   6          100.0             10.0
#> 412         8             8                   3           90.0             50.0
#> 413        17             7                   6          100.0             40.0
#> 414         9             2                   2          100.0              0.0
#> 415         7            23                  14           93.9             30.0
#> 416         9            12                   7          100.0             37.5
#> 417        21             8                   4          100.0              0.0
#> 418         3             9                   2          100.0              0.0
#> 419        28             6                   2          100.0             25.0
#> 420         5             9                   3           93.7              0.0
#> 421         3             7                   7          100.0            100.0
#> 422         1            10                   5           94.4             20.0
#> 423         7             5                   3           87.5              0.0
#> 424        14            11                   8           96.8             85.7
#> 425         1             9                   6          100.0             33.3
#> 426         9            10                   6           88.0             25.0
#> 427         5             8                   5          100.0             50.0
#> 428        11            14                   8          100.0            100.0
#> 429         4             3                   3           76.9            100.0
#> 430         9            18                  13           97.3             22.2
#> 431        11            22                  14           96.9             10.0
#> 432         2            10                   7           55.6             50.0
#> 433         4            24                  12          100.0              0.0
#> 434         0             8                   6          100.0             71.4
#> 435        45             7                   3           95.4              0.0
#> 436        19            14                  11           91.7             57.1
#> 437        16             3                   3           76.7              0.0
#> 438         3            12                   2          100.0              0.0
#> 439         4             7                   5          100.0             66.7
#> 440         7             7                   4           63.2             66.7
#> 441        18             6                   4          100.0             33.3
#> 442         9             8                   5           92.6              0.0
#> 443         9            11                   9           96.6             12.5
#> 444         8             4                   3           77.8             50.0
#> 445         5             5                   4           60.0              0.0
#> 446         6            10                   5          100.0            100.0
#> 447         9             5                   5          100.0            100.0
#> 448         7            11                   6          100.0              0.0
#> 449         2             6                   5          100.0            100.0
#> 450        39            28                  20           98.8             12.5
#> 451         1             8                   8          100.0            100.0
#> 452         9             6                   5          100.0            100.0
#> 453        16            10                   8           90.6             50.0
#> 454        16             6                   5           94.1            100.0
#> 455        17             9                   6           94.1            100.0
#> 456         6            10                   9           96.7             75.0
#> 457        15            24                  18           93.2             80.0
#> 458         1             8                   5          100.0              0.0
#> 459         2             5                   4           88.9              0.0
#> 460        11            14                  10           92.3              0.0
#> 461        15            14                  10          100.0             42.9
#> 462         7             4                   3          100.0            100.0
#> 463         6             3                   3           92.9             33.3
#> 464        20             6                   5          100.0             83.3
#> 465         1             9                   5          100.0             40.0
#> 466         3            10                   7          100.0              0.0
#> 467         6            10                   8           94.1             62.5
#> 468        16             5                   5           96.3            100.0
#> 469        22             6                   6           94.7             80.0
#> 470         4             3                   2          100.0              0.0
#> 471         8             8                   5           95.7             66.7
#> 472        12             3                   3           88.9            100.0
#> 473        22             2                   2          100.0             33.3
#> 474        13            10                   5          100.0             14.3
#> 475         2            13                  12          100.0             58.3
#> 476         5            11                   9           95.0              0.0
#> 477         1             8                   8          100.0             85.7
#> 478         4             6                   4           95.7            100.0
#> 479         6            10                   3           94.9              0.0
#> 480         6            11                   8           95.8             50.0
#> 481         7            12                   8           95.8             40.0
#> 482         8             5                   1          100.0              0.0
#> 483        28            17                  14           92.3             25.0
#> 484         5            12                   7          100.0             36.4
#> 485        13             4                   4          100.0             50.0
#> 486         4             4                   4           92.3             75.0
#> 487         5            10                   8          100.0             33.3
#> 488        16             3                   2           84.6             33.3
#> 489        17             7                   4           93.0             50.0
#> 490        19             6                   3           93.7             50.0
#> 491        11             5                   5           90.0            100.0
#> 492         7             5                   4           93.7             66.7
#> 493         9             6                   4          100.0             66.7
#> 494        11             8                   6          100.0            100.0
#> 495         4            12                   3          100.0              0.0
#> 496        13             5                   4          100.0              0.0
#> 497        13             9                   8           96.9            100.0
#> 498         4            11                   9           95.0             11.1
#> 499        13             6                   6           91.0              0.0
#> 500         9             8                   6          100.0             80.0
#> 501         7            11                  11          100.0             44.4
#> 502         8             7                   5           95.7             50.0
#> 503         2             5                   3          100.0              0.0
#> 504         5             7                   7           53.8             33.3
#> 505        17             6                   3           89.3             33.3
#> 506        12            13                  12           93.5            100.0
#> 507        12            15                   5           94.6              0.0
#> 508         5             2                   2           91.3              0.0
#> 509        12            16                  11          100.0              0.0
#> 510         1             3                   3           83.3             33.3
#> 511        11            11                   4           81.8              0.0
#> 512         4            12                   8           96.0             75.0
#> 513         9             4                   3           94.1             33.3
#> 514         8             9                   9           80.0              0.0
#> 515        16             9                   6           96.2             57.1
#> 516        10             9                   6          100.0             60.0
#> 517         4             6                   5           80.0              0.0
#> 518         3             3                   1           44.7              0.0
#> 519        14            10                   6           95.9              0.0
#> 520        15            10                   9           93.3             40.0
#> 521        12            30                  28          100.0             20.0
#> 522         4            12                   6           88.9             37.5
#> 523         6             7                   4          100.0              0.0
#> 524        11             7                   5          100.0             60.0
#> 525        14             8                   3           96.3             25.0
#> 526         4            11                   5           94.7             14.3
#> 527        20            10                   8           94.1             16.7
#> 528         2             8                   4          100.0             66.7
#> 529         1             9                   5           96.0             28.6
#> 530         9             7                   5          100.0              0.0
#> 531        31             6                   5          100.0            100.0
#> 532         6             6                   5           91.7            100.0
#> 533        17             4                   2          100.0             50.0
#> 534         3             5                   3           50.0             66.7
#> 535        15            10                   7           93.7             40.0
#> 536        14             9                   6           92.1             20.0
#> 537         0             3                   3          100.0              0.0
#> 538         8            12                   8           93.7            100.0
#> 539        12             6                   5          100.0              0.0
#> 540        19             5                   4           59.0             50.0
#> 541         1             9                   7           87.5             80.0
#> 542        17            18                  15          100.0             83.3
#> 543         7            11                   7           93.1             20.0
#> 544        12             8                   6           96.3             50.0
#> 545         8            11                  11          100.0             66.7
#> 546         4             3                   1           83.3              0.0
#> 547         9             7                   6          100.0             25.0
#> 548        15            10                   9          100.0             77.8
#> 549         0            10                   7          100.0             14.3
#> 550         3            11                   6          100.0              0.0
#> 551        10            10                   5           85.2              0.0
#> 552        13            11                  10           93.9             57.1
#> 553         3             5                   5           83.3             50.0
#> 554        23             7                   5          100.0             20.0
#> 555         4             5                   5           90.0             66.7
#> 556         5             6                   5           83.0             60.0
#> 557         8            16                   7           97.1             33.3
#> 558         7            10                   8          100.0             14.3
#> 559         8             9                   6          100.0             83.3
#> 560         8             9                   4          100.0              0.0
#> 561         5            13                   7           81.2            100.0
#> 562         3             8                   8          100.0             20.0
#> 563         7             5                   5          100.0             66.7
#> 564         6            11                   6          100.0             14.3
#> 565         9            11                   9          100.0            100.0
#> 566         2            13                  10           95.8             33.3
#> 567        15            15                   8           97.8             28.6
#> 568        17             9                   7          100.0             14.3
#> 569        10             7                   7           90.0             28.6
#> 570        11             9                   9          100.0             60.0
#> 571         3            18                  16          100.0            100.0
#> 572         2             4                   4           94.1             25.0
#> 573         5            25                  10           98.0             14.3
#> 574         2             6                   6           75.0             33.3
#> 575         7             9                   8          100.0             50.0
#> 576        10            13                   9           97.8              0.0
#> 577         7            13                  10           86.7             22.2
#> 578         9            15                  14          100.0             57.1
#> 579        14            17                  10          100.0              0.0
#> 580         4             7                   6           78.6             33.3
#> 581        11            12                   6           88.5            100.0
#> 582         9             2                   2          100.0            100.0
#> 583         4             8                   3           85.7              0.0
#> 584        17             9                   3           85.4              0.0
#> 585         0            14                  12           90.0             66.7
#> 586         7             8                   6          100.0             20.0
#> 587        15            10                   9           91.0             75.0
#> 588         7             6                   3           92.9             50.0
#> 589         9             7                   4           78.9              0.0
#> 590         6            10                   7           97.1             80.0
#> 591         2             9                   7           86.7             50.0
#> 592        17             7                   4           84.6             33.3
#> 593         8             8                   6          100.0             60.0
#> 594         7             8                   6           95.2              0.0
#> 595         4             8                   6          100.0            100.0
#> 596        30            14                  11           90.0              9.1
#> 597        11            12                   6           96.9             16.7
#> 598         2            12                  11          100.0             14.3
#> 599         1             3                   2          100.0            100.0
#> 600         1             2                   2           90.0            100.0
#> 601         6            16                  15           90.3             22.2
#> 602         4             5                  10          100.0            100.0
#> 603        12             8                   4           96.3             75.0
#> 604         0            14                  14          100.0              0.0
#> 605         7             4                   3          100.0             25.0
#> 606         3             7                   5          100.0              0.0
#> 607        13            10                   2           96.2             50.0
#> 608         7             8                   6           95.5              0.0
#> 609         4            10                   8          100.0              0.0
#> 610         8             5                   4           87.5            100.0
#> 611         1             8                   6           88.0              0.0
#> 612         7             3                   3          100.0             50.0
#> 613         8             9                   3          100.0             50.0
#> 614        16             8                   4           88.9             14.3
#> 615        14            13                  10           86.5             55.6
#> 616         1             5                   5          100.0             20.0
#> 617         3             2                   1          100.0              0.0
#> 618        10             8                   3           96.3              0.0
#> 619         0             8                   6          100.0              0.0
#> 620         5            14                   8           91.9             33.3
#> 621        10             5                   3           76.2             66.7
#> 622        10             8                   6           86.0             33.3
#> 623         6             9                   7          100.0             75.0
#> 624         5            14                   2           90.0              0.0
#> 625         3             3                   1          100.0             50.0
#> 626         8             3                   3          100.0            100.0
#> 627         3             6                   5           93.7            100.0
#> 628         6             5                   4          100.0             20.0
#> 629         1             1                   1          100.0            100.0
#> 630         6            18                  12          100.0             77.8
#> 631         7            10                   7           82.5             80.0
#> 632        12             7                   6           97.0             33.3
#> 633        14             4                   2           84.0              0.0
#> 634        19            18                   7          100.0             11.1
#> 635         6             7                   7           88.9             80.0
#> 636         5             7                   4          100.0             50.0
#> 637         2             5                   4           80.0              0.0
#> 638         4            12                   9          100.0             50.0
#> 639         3             3                   2           92.3            100.0
#> 640        22             4                   4           96.0            100.0
#> 641         3             8                   8          100.0             50.0
#> 642         6            11                  10           96.2             28.6
#> 643         0             4                   3          100.0             33.3
#> 644         5             6                   5           72.2             25.0
#> 645         3             9                   6           94.1            100.0
#> 646         5             5                   3          100.0              0.0
#> 647         6            21                  11          100.0              0.0
#> 648         7             9                   7           86.4             25.0
#> 649        12             8                   6          100.0             20.0
#> 650        23             9                   7           96.9            100.0
#> 651        11            11                   3           96.4             66.7
#> 652         0             7                   4           68.7             25.0
#> 653         9            24                   8           98.1             14.3
#> 654         7            10                   5          100.0             50.0
#> 655        13            10                   4          100.0              0.0
#> 656         5             8                   8          100.0            100.0
#> 657         8             2                   2          100.0              0.0
#> 658         3             6                   4          100.0             25.0
#> 659         6            10                   5          100.0             40.0
#> 660        20            10                   5          100.0             25.0
#> 661         1             6                   5           77.8             20.0
#> 662         0             3                   2          100.0             33.3
#> 663         5            16                  13           91.2             33.3
#> 664         3             9                   4          100.0             37.5
#> 665         9            10                   8          100.0             33.3
#> 666         3             6                   6          100.0             25.0
#> 667         3             7                   5           95.2             50.0
#> 668        17             9                   6           90.0             80.0
#> 669         6             9                   8           88.2             37.5
#> 670        11            22                  10          100.0              0.0
#> 671         5             7                   4           86.7              0.0
#> 672        15             7                   5           96.4             50.0
#> 673         8             8                   8           94.3            100.0
#> 674         5             4                   3          100.0            100.0
#> 675         1            10                   8           94.1             30.0
#> 676         4             9                   4          100.0             25.0
#> 677         5             9                   7          100.0             83.3
#> 678        23            11                   6          100.0             33.3
#> 679         3            11                   8          100.0            100.0
#> 680        10             4                   3           84.6             66.7
#> 681        20             4                   4           96.0             33.3
#> 682         8             8                   8          100.0             75.0
#> 683         3             1                   1          100.0              0.0
#> 684         4             5                   5           95.2             50.0
#> 685         3            14                   8          100.0             55.6
#> 686         7             4                   3          100.0              0.0
#> 687        12            12                   8           89.3            100.0
#> 688         3             6                   6          100.0            100.0
#> 689        10             4                   3          100.0            100.0
#> 690         2             6                   4          100.0             33.3
#> 691        15            16                  11           87.8             62.5
#> 692         9             4                   0           75.0              0.0
#> 693         7             6                   4           82.4              0.0
#> 694         3            10                   6           93.7             20.0
#> 695        10             4                   4           89.3            100.0
#> 696         4            10                   8          100.0             75.0
#> 697        29            19                   8          100.0             27.3
#> 698        11             5                   2           91.7              0.0
#> 699        19            10                   7           97.0             66.7
#> 700         1             7                   6          100.0             40.0
#> 701         9            10                   8          100.0             66.7
#> 702         8            27                  17           76.2             60.0
#> 703        16             4                   3          100.0             33.3
#> 704        10             7                   5          100.0             50.0
#> 705         7             8                   4           85.7             25.0
#> 706         4             5                   4          100.0             75.0
#> 707         3             5                   4          100.0             66.7
#> 708         2            12                   4          100.0              0.0
#> 709        17             9                   5           98.2              0.0
#> 710         2            11                   7           90.9              0.0
#> 711         1             8                   6           96.2            100.0
#> 712         5             6                   4          100.0             50.0
#> 713        12            14                   7           95.3             62.5
#> 714         1             5                   5          100.0              0.0
#> 715         4            18                  12          100.0              0.0
#> 716         6             5                   4          100.0              0.0
#> 717         3            12                   3          100.0              0.0
#> 718         9             6                   5           83.3             66.7
#> 719        10             9                   2           89.7              0.0
#> 720        14            12                   7          100.0              0.0
#> 721         0            11                   4           76.9             50.0
#> 722        17             2                   1          100.0            100.0
#> 723        11            12                   9           97.6             16.7
#> 724         4             3                   3           91.7            100.0
#> 725         5            10                   6          100.0             16.7
#> 726         7            11                   5           77.8              0.0
#> 727         4             8                   4           93.1              0.0
#> 728         5             8                   4          100.0             40.0
#> 729        23             8                   6           97.6             50.0
#> 730        10            14                  14           92.9             12.5
#> 731        10             5                   5          100.0             75.0
#> 732        26             7                   4           94.7              0.0
#> 733         5            11                  10           95.0             66.7
#> 734         8             5                   5           88.2             75.0
#> 735         8             6                   4           86.7            100.0
#> 736        27             6                   6          100.0            100.0
#> 737         4             5                   5           88.0             50.0
#> 738         1             5                   4          100.0              0.0
#> 739         7             8                   5           90.9             14.3
#> 740        10            15                  10           93.5            100.0
#> 741        10             7                   1           96.2             20.0
#> 742         2            12                  10          100.0             25.0
#> 743         9            13                  11          100.0             33.3
#> 744        19            10                   8           88.6             50.0
#> 745        38            12                   4           98.5             50.0
#> 746         7            17                  17           78.8             58.3
#> 747         6            10                   6          100.0             20.0
#> 748         7             3                   2          100.0              0.0
#> 749         6            11                  11           77.8             66.7
#> 750         5             9                   7           90.0             60.0
#> 751         3             6                   1           93.7              0.0
#> 752         8             5                   5          100.0             50.0
#> 753        41             3                   2           71.1            100.0
#> 754        22            13                   7           82.9             57.1
#> 755        10             9                   3           67.5              0.0
#> 756         6            14                   8           87.0             33.3
#> 757         1             2                   2          100.0              0.0
#> 758         9             7                   7          100.0            100.0
#> 759        18             4                   2          100.0              0.0
#> 760        11             4                   3          100.0              0.0
#> 761         5             7                   4           77.8              0.0
#> 762         3             7                   5          100.0             42.9
#> 763        20            17                   8          100.0              0.0
#> 764        11             6                   4          100.0             33.3
#> 765         5             9                   5           95.0             50.0
#> 766         6            12                   6           88.0              0.0
#> 767         8             7                   6          100.0             25.0
#> 768        11            13                   7          100.0             14.3
#> 769         4             6                   4          100.0              0.0
#> 770        22             9                   3           98.3             25.0
#> 771        11             7                   5           95.8            100.0
#> 772        34            15                   6           98.4             50.0
#> 773         4             5                   5          100.0            100.0
#> 774         7             6                   6           95.0             33.3
#> 775         9            11                   5           87.0             33.3
#> 776         4             6                   5          100.0              0.0
#> 777         4            18                   3          100.0             16.7
#> 778         2            14                  10           93.0             50.0
#> 779         1             2                   1          100.0             33.3
#> 780         5             2                   2          100.0              0.0
#> 781         2            19                  11          100.0             18.2
#> 782         6            11                  10           89.0             66.7
#> 783         1            11                   8          100.0             83.3
#> 784         2             4                   4           92.3              0.0
#> 785         6             8                   6           94.1            100.0
#> 786         9             4                   4           90.6             75.0
#> 787        15             6                   6           90.5             20.0
#> 788         6             9                   5          100.0             57.1
#> 789        11             6                   5           93.1             60.0
#> 790         4             3                   3           80.0             33.3
#> 791         6             6                   5           80.0            100.0
#> 792         8            12                   9           96.0              0.0
#> 793        16            10                   7          100.0              0.0
#> 794        21             4                   4           92.9             66.7
#> 795        11             6                   5           82.0             66.7
#> 796        12             5                   5          100.0            100.0
#> 797        20             5                   4          100.0            100.0
#> 798        35             8                   7           78.4             25.0
#> 799        27             6                   6          100.0             20.0
#> 800         2            13                   4          100.0             37.5
#> 801         3             6                   3          100.0             33.3
#> 802        13             9                   5          100.0             75.0
#> 803         2             4                   2          100.0              0.0
#> 804         9             6                   5          100.0            100.0
#> 805         0             6                   5          100.0             20.0
#> 806         3             5                   5          100.0             50.0
#> 807         9             5                   3           59.3            100.0
#> 808         5             5                   3          100.0            100.0
#> 809         5             4                   1           93.3              0.0
#> 810         3             9                   7           92.9             60.0
#> 811         1            14                   6           90.3             12.5
#> 812        19             5                   4          100.0             50.0
#> 813         7            10                  10          100.0             66.7
#> 814         2             4                   3           87.5            100.0
#> 815        26             5                   5           59.4              0.0
#> 816         6             6                   6           92.3             50.0
#> 817         1             7                   2           75.0              0.0
#> 818        14             6                   4           84.6              0.0
#> 819        11             7                   5           95.7             16.7
#> 820         3             5                   3           93.3             50.0
#> 821        10            10                   9          100.0             20.0
#> 822         3             5                   4          100.0              0.0
#> 823         8             3                   2           87.5              0.0
#> 824        21             7                   7           75.6            100.0
#> 825        40            11                   4          100.0              0.0
#> 826        11            17                   7           92.2             71.4
#> 827        14            12                   4          100.0             16.7
#> 828         1            14                   4          100.0              0.0
#> 829        13             5                   5           75.0              0.0
#> 830         3             9                   6          100.0             25.0
#> 831         1             8                   7          100.0             50.0
#> 832         4            11                   6           94.1              0.0
#> 833         9            28                  19          100.0             28.6
#> 834        15             9                   6           85.3             50.0
#> 835         1             3                   3           92.3            100.0
#> 836         5             6                   6           95.0             60.0
#> 837         1            11                   6           92.3             20.0
#> 838         9             4                   2           68.2              0.0
#> 839        20             8                   4          100.0              0.0
#> 840         9             5                   4          100.0             75.0
#> 841         9             6                   2           79.2              0.0
#> 842         2             6                   5          100.0             20.0
#> 843         5            12                   6          100.0              0.0
#> 844         3             3                   4           39.0              0.0
#> 845        12             1                   1           54.0              0.0
#> 846        17             5                   2           96.7              0.0
#> 847         2             4                   3          100.0             50.0
#> 848        15             2                   2           94.7              0.0
#> 849         2             2                   2           67.0             50.0
#> 850         4             7                   4           88.9            100.0
#> 851        14            12                  11          100.0             87.5
#> 852         5            19                   5           97.6              0.0
#> 853         8            11                   4          100.0              0.0
#> 854         9            18                  16           84.4             80.0
#> 855         3             3                   2           92.9              0.0
#> 856        19            16                   5           69.2             28.6
#> 857        26            19                  13           98.0              0.0
#> 858         5            15                  12          100.0              0.0
#> 859        22             2                   1           44.8             50.0
#> 860         4             1                   1           67.0              0.0
#> 861         7            12                  12           83.0             66.7
#> 862        13             9                   9          100.0             16.7
#> 863        12             4                   4          100.0            100.0
#> 864         6            17                  12          100.0              0.0
#> 865        20            19                  14          100.0             11.1
#> 866         2             6                   4          100.0            100.0
#> 867        16             7                   4           95.1            100.0
#> 868        12             3                   3           80.0             66.7
#> 869         8             2                   2           75.0              0.0
#> 870         8             8                   3          100.0             50.0
#> 871         7             3                   3          100.0             66.7
#> 872        14             4                   3          100.0            100.0
#> 873         2            13                   7          100.0             36.4
#>     pub_rec_bankruptcies
#> 1                      0
#> 2                      0
#> 3                      0
#> 4                      0
#> 5                      0
#> 6                      0
#> 7                      0
#> 8                      1
#> 9                      0
#> 10                     0
#> 11                     1
#> 12                     0
#> 13                     0
#> 14                     0
#> 15                     0
#> 16                     1
#> 17                     0
#> 18                     0
#> 19                     0
#> 20                     0
#> 21                     0
#> 22                     0
#> 23                     0
#> 24                     0
#> 25                     0
#> 26                     0
#> 27                     0
#> 28                     0
#> 29                     0
#> 30                     0
#> 31                     0
#> 32                     0
#> 33                     0
#> 34                     0
#> 35                     0
#> 36                     0
#> 37                     0
#> 38                     0
#> 39                     0
#> 40                     1
#> 41                     0
#> 42                     0
#> 43                     0
#> 44                     0
#> 45                     0
#> 46                     0
#> 47                     0
#> 48                     0
#> 49                     0
#> 50                     0
#> 51                     0
#> 52                     0
#> 53                     0
#> 54                     0
#> 55                     0
#> 56                     0
#> 57                     0
#> 58                     0
#> 59                     0
#> 60                     0
#> 61                     0
#> 62                     0
#> 63                     1
#> 64                     0
#> 65                     0
#> 66                     0
#> 67                     1
#> 68                     0
#> 69                     0
#> 70                     0
#> 71                     0
#> 72                     1
#> 73                     0
#> 74                     1
#> 75                     0
#> 76                     0
#> 77                     0
#> 78                     0
#> 79                     0
#> 80                     0
#> 81                     0
#> 82                     0
#> 83                     1
#> 84                     0
#> 85                     1
#> 86                     1
#> 87                     0
#> 88                     0
#> 89                     0
#> 90                     0
#> 91                     0
#> 92                     1
#> 93                     0
#> 94                     0
#> 95                     0
#> 96                     1
#> 97                     0
#> 98                     0
#> 99                     0
#> 100                    0
#> 101                    0
#> 102                    0
#> 103                    1
#> 104                    0
#> 105                    0
#> 106                    0
#> 107                    0
#> 108                    0
#> 109                    0
#> 110                    0
#> 111                    0
#> 112                    0
#> 113                    0
#> 114                    0
#> 115                    0
#> 116                    0
#> 117                    0
#> 118                    0
#> 119                    0
#> 120                    0
#> 121                    0
#> 122                    0
#> 123                    0
#> 124                    0
#> 125                    0
#> 126                    2
#> 127                    1
#> 128                    0
#> 129                    0
#> 130                    0
#> 131                    0
#> 132                    0
#> 133                    0
#> 134                    0
#> 135                    0
#> 136                    0
#> 137                    0
#> 138                    0
#> 139                    0
#> 140                    0
#> 141                    0
#> 142                    0
#> 143                    0
#> 144                    1
#> 145                    0
#> 146                    0
#> 147                    0
#> 148                    0
#> 149                    0
#> 150                    0
#> 151                    0
#> 152                    0
#> 153                    0
#> 154                    0
#> 155                    0
#> 156                    0
#> 157                    0
#> 158                    0
#> 159                    0
#> 160                    0
#> 161                    0
#> 162                    0
#> 163                    0
#> 164                    0
#> 165                    0
#> 166                    0
#> 167                    0
#> 168                    0
#> 169                    0
#> 170                    0
#> 171                    0
#> 172                    0
#> 173                    0
#> 174                    0
#> 175                    1
#> 176                    0
#> 177                    0
#> 178                    0
#> 179                    1
#> 180                    1
#> 181                    0
#> 182                    0
#> 183                    0
#> 184                    0
#> 185                    0
#> 186                    0
#> 187                    0
#> 188                    0
#> 189                    0
#> 190                    0
#> 191                    0
#> 192                    0
#> 193                    0
#> 194                    0
#> 195                    0
#> 196                    0
#> 197                    0
#> 198                    0
#> 199                    0
#> 200                    0
#> 201                    1
#> 202                    0
#> 203                    1
#> 204                    0
#> 205                    0
#> 206                    0
#> 207                    1
#> 208                    0
#> 209                    0
#> 210                    0
#> 211                    0
#> 212                    0
#> 213                    1
#> 214                    0
#> 215                    0
#> 216                    0
#> 217                    0
#> 218                    0
#> 219                    1
#> 220                    0
#> 221                    0
#> 222                    1
#> 223                    0
#> 224                    0
#> 225                    1
#> 226                    0
#> 227                    0
#> 228                    0
#> 229                    0
#> 230                    0
#> 231                    0
#> 232                    0
#> 233                    0
#> 234                    0
#> 235                    0
#> 236                    0
#> 237                    0
#> 238                    0
#> 239                    0
#> 240                    0
#> 241                    0
#> 242                    0
#> 243                    0
#> 244                    0
#> 245                    0
#> 246                    1
#> 247                    0
#> 248                    0
#> 249                    0
#> 250                    0
#> 251                    0
#> 252                    0
#> 253                    0
#> 254                    1
#> 255                    0
#> 256                    0
#> 257                    0
#> 258                    1
#> 259                    0
#> 260                    0
#> 261                    0
#> 262                    0
#> 263                    1
#> 264                    0
#> 265                    0
#> 266                    1
#> 267                    0
#> 268                    1
#> 269                    0
#> 270                    0
#> 271                    0
#> 272                    0
#> 273                    1
#> 274                    0
#> 275                    1
#> 276                    1
#> 277                    0
#> 278                    0
#> 279                    0
#> 280                    0
#> 281                    1
#> 282                    1
#> 283                    0
#> 284                    1
#> 285                    0
#> 286                    1
#> 287                    0
#> 288                    0
#> 289                    0
#> 290                    0
#> 291                    0
#> 292                    0
#> 293                    0
#> 294                    0
#> 295                    1
#> 296                    0
#> 297                    0
#> 298                    0
#> 299                    0
#> 300                    0
#> 301                    0
#> 302                    0
#> 303                    0
#> 304                    0
#> 305                    0
#> 306                    0
#> 307                    1
#> 308                    0
#> 309                    0
#> 310                    0
#> 311                    0
#> 312                    0
#> 313                    0
#> 314                    0
#> 315                    0
#> 316                    0
#> 317                    1
#> 318                    0
#> 319                    0
#> 320                    0
#> 321                    0
#> 322                    0
#> 323                    0
#> 324                    0
#> 325                    0
#> 326                    1
#> 327                    0
#> 328                    0
#> 329                    0
#> 330                    0
#> 331                    0
#> 332                    0
#> 333                    0
#> 334                    1
#> 335                    0
#> 336                    0
#> 337                    0
#> 338                    0
#> 339                    0
#> 340                    0
#> 341                    0
#> 342                    1
#> 343                    0
#> 344                    1
#> 345                    0
#> 346                    0
#> 347                    0
#> 348                    0
#> 349                    0
#> 350                    0
#> 351                    0
#> 352                    0
#> 353                    0
#> 354                    1
#> 355                    0
#> 356                    0
#> 357                    0
#> 358                    1
#> 359                    0
#> 360                    0
#> 361                    0
#> 362                    1
#> 363                    0
#> 364                    1
#> 365                    0
#> 366                    0
#> 367                    0
#> 368                    0
#> 369                    0
#> 370                    0
#> 371                    0
#> 372                    0
#> 373                    0
#> 374                    0
#> 375                    0
#> 376                    0
#> 377                    0
#> 378                    0
#> 379                    0
#> 380                    0
#> 381                    0
#> 382                    0
#> 383                    1
#> 384                    1
#> 385                    0
#> 386                    1
#> 387                    0
#> 388                    1
#> 389                    0
#> 390                    0
#> 391                    0
#> 392                    0
#> 393                    1
#> 394                    0
#> 395                    1
#> 396                    0
#> 397                    0
#> 398                    0
#> 399                    0
#> 400                    0
#> 401                    0
#> 402                    1
#> 403                    0
#> 404                    0
#> 405                    0
#> 406                    0
#> 407                    0
#> 408                    0
#> 409                    0
#> 410                    0
#> 411                    0
#> 412                    0
#> 413                    0
#> 414                    0
#> 415                    0
#> 416                    0
#> 417                    0
#> 418                    0
#> 419                    1
#> 420                    0
#> 421                    0
#> 422                    0
#> 423                    0
#> 424                    1
#> 425                    0
#> 426                    0
#> 427                    0
#> 428                    1
#> 429                    0
#> 430                    0
#> 431                    0
#> 432                    0
#> 433                    0
#> 434                    0
#> 435                    0
#> 436                    1
#> 437                    0
#> 438                    0
#> 439                    0
#> 440                    0
#> 441                    0
#> 442                    0
#> 443                    0
#> 444                    0
#> 445                    0
#> 446                    0
#> 447                    0
#> 448                    1
#> 449                    0
#> 450                    0
#> 451                    0
#> 452                    1
#> 453                    0
#> 454                    0
#> 455                    0
#> 456                    0
#> 457                    0
#> 458                    1
#> 459                    0
#> 460                    0
#> 461                    0
#> 462                    0
#> 463                    0
#> 464                    0
#> 465                    0
#> 466                    0
#> 467                    1
#> 468                    0
#> 469                    0
#> 470                    0
#> 471                    0
#> 472                    0
#> 473                    0
#> 474                    0
#> 475                    0
#> 476                    0
#> 477                    0
#> 478                    0
#> 479                    0
#> 480                    1
#> 481                    0
#> 482                    0
#> 483                    0
#> 484                    0
#> 485                    0
#> 486                    0
#> 487                    0
#> 488                    0
#> 489                    0
#> 490                    0
#> 491                    0
#> 492                    0
#> 493                    0
#> 494                    0
#> 495                    0
#> 496                    1
#> 497                    0
#> 498                    0
#> 499                    1
#> 500                    0
#> 501                    0
#> 502                    0
#> 503                    0
#> 504                    0
#> 505                    0
#> 506                    0
#> 507                    0
#> 508                    0
#> 509                    0
#> 510                    0
#> 511                    1
#> 512                    0
#> 513                    1
#> 514                    1
#> 515                    0
#> 516                    0
#> 517                    0
#> 518                    0
#> 519                    0
#> 520                    0
#> 521                    0
#> 522                    0
#> 523                    1
#> 524                    0
#> 525                    1
#> 526                    0
#> 527                    1
#> 528                    1
#> 529                    0
#> 530                    0
#> 531                    0
#> 532                    0
#> 533                    1
#> 534                    1
#> 535                    0
#> 536                    0
#> 537                    0
#> 538                    0
#> 539                    0
#> 540                    1
#> 541                    0
#> 542                    1
#> 543                    0
#> 544                    0
#> 545                    0
#> 546                    0
#> 547                    0
#> 548                    0
#> 549                    0
#> 550                    0
#> 551                    0
#> 552                    0
#> 553                    0
#> 554                    0
#> 555                    1
#> 556                    0
#> 557                    0
#> 558                    0
#> 559                    0
#> 560                    0
#> 561                    0
#> 562                    0
#> 563                    0
#> 564                    0
#> 565                    0
#> 566                    0
#> 567                    0
#> 568                    0
#> 569                    0
#> 570                    0
#> 571                    0
#> 572                    0
#> 573                    0
#> 574                    1
#> 575                    0
#> 576                    0
#> 577                    0
#> 578                    0
#> 579                    0
#> 580                    0
#> 581                    0
#> 582                    0
#> 583                    0
#> 584                    0
#> 585                    0
#> 586                    1
#> 587                    0
#> 588                    0
#> 589                    0
#> 590                    0
#> 591                    1
#> 592                    0
#> 593                    1
#> 594                    0
#> 595                    0
#> 596                    0
#> 597                    0
#> 598                    0
#> 599                    1
#> 600                    0
#> 601                    0
#> 602                    0
#> 603                    0
#> 604                    0
#> 605                    0
#> 606                    0
#> 607                    0
#> 608                    2
#> 609                    0
#> 610                    0
#> 611                    0
#> 612                    0
#> 613                    0
#> 614                    0
#> 615                    0
#> 616                    0
#> 617                    0
#> 618                    0
#> 619                    0
#> 620                    0
#> 621                    0
#> 622                    0
#> 623                    0
#> 624                    0
#> 625                    0
#> 626                    0
#> 627                    1
#> 628                    1
#> 629                    0
#> 630                    2
#> 631                    0
#> 632                    0
#> 633                    0
#> 634                    0
#> 635                    1
#> 636                    0
#> 637                    0
#> 638                    1
#> 639                    0
#> 640                    0
#> 641                    0
#> 642                    0
#> 643                    0
#> 644                    0
#> 645                    0
#> 646                    0
#> 647                    0
#> 648                    0
#> 649                    0
#> 650                    0
#> 651                    0
#> 652                    0
#> 653                    0
#> 654                    1
#> 655                    0
#> 656                    0
#> 657                    0
#> 658                    0
#> 659                    0
#> 660                    0
#> 661                    0
#> 662                    0
#> 663                    0
#> 664                    0
#> 665                    1
#> 666                    0
#> 667                    0
#> 668                    0
#> 669                    0
#> 670                    0
#> 671                    0
#> 672                    1
#> 673                    0
#> 674                    0
#> 675                    0
#> 676                    0
#> 677                    0
#> 678                    0
#> 679                    0
#> 680                    0
#> 681                    1
#> 682                    0
#> 683                    0
#> 684                    0
#> 685                    0
#> 686                    0
#> 687                    0
#> 688                    0
#> 689                    1
#> 690                    0
#> 691                    0
#> 692                    0
#> 693                    0
#> 694                    0
#> 695                    0
#> 696                    0
#> 697                    0
#> 698                    0
#> 699                    1
#> 700                    0
#> 701                    0
#> 702                    1
#> 703                    0
#> 704                    0
#> 705                    0
#> 706                    0
#> 707                    0
#> 708                    0
#> 709                    0
#> 710                    0
#> 711                    2
#> 712                    0
#> 713                    0
#> 714                    0
#> 715                    0
#> 716                    0
#> 717                    0
#> 718                    0
#> 719                    0
#> 720                    0
#> 721                    0
#> 722                    0
#> 723                    1
#> 724                    0
#> 725                    0
#> 726                    0
#> 727                    0
#> 728                    0
#> 729                    0
#> 730                    0
#> 731                    0
#> 732                    1
#> 733                    0
#> 734                    0
#> 735                    0
#> 736                    0
#> 737                    0
#> 738                    0
#> 739                    0
#> 740                    0
#> 741                    0
#> 742                    0
#> 743                    0
#> 744                    0
#> 745                    0
#> 746                    0
#> 747                    0
#> 748                    0
#> 749                    0
#> 750                    0
#> 751                    0
#> 752                    0
#> 753                    0
#> 754                    0
#> 755                    0
#> 756                    0
#> 757                    0
#> 758                    0
#> 759                    0
#> 760                    0
#> 761                    0
#> 762                    0
#> 763                    0
#> 764                    0
#> 765                    0
#> 766                    0
#> 767                    1
#> 768                    0
#> 769                    1
#> 770                    1
#> 771                    0
#> 772                    0
#> 773                    0
#> 774                    0
#> 775                    0
#> 776                    0
#> 777                    0
#> 778                    1
#> 779                    0
#> 780                    0
#> 781                    0
#> 782                    0
#> 783                    0
#> 784                    0
#> 785                    0
#> 786                    0
#> 787                    0
#> 788                    0
#> 789                    0
#> 790                    0
#> 791                    0
#> 792                    0
#> 793                    0
#> 794                    0
#> 795                    1
#> 796                    0
#> 797                    0
#> 798                    0
#> 799                    0
#> 800                    0
#> 801                    0
#> 802                    0
#> 803                    0
#> 804                    0
#> 805                    0
#> 806                    0
#> 807                    0
#> 808                    1
#> 809                    0
#> 810                    0
#> 811                    0
#> 812                    0
#> 813                    0
#> 814                    0
#> 815                    0
#> 816                    0
#> 817                    0
#> 818                    0
#> 819                    0
#> 820                    0
#> 821                    0
#> 822                    0
#> 823                    0
#> 824                    0
#> 825                    0
#> 826                    0
#> 827                    1
#> 828                    0
#> 829                    0
#> 830                    0
#> 831                    0
#> 832                    0
#> 833                    0
#> 834                    0
#> 835                    0
#> 836                    0
#> 837                    0
#> 838                    0
#> 839                    0
#> 840                    0
#> 841                    0
#> 842                    0
#> 843                    0
#> 844                    0
#> 845                    1
#> 846                    0
#> 847                    0
#> 848                    0
#> 849                    0
#> 850                    0
#> 851                    0
#> 852                    0
#> 853                    1
#> 854                    0
#> 855                    0
#> 856                    0
#> 857                    0
#> 858                    0
#> 859                    0
#> 860                    1
#> 861                    0
#> 862                    0
#> 863                    0
#> 864                    0
#> 865                    0
#> 866                    1
#> 867                    0
#> 868                    0
#> 869                    0
#> 870                    1
#> 871                    0
#> 872                    0
#> 873                    0
```



