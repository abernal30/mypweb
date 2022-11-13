# Credit analysis


In this chapter we will use the following libraries:


```r
library(openxlsx)
library(caret)
library(MASS)
```



In this chapter, we will cover credit allocation analysis (loan origination). The database credit.xlsx has historical information
on Lendingclub, <https://www.lendingclub.com/> fintech marketplace bank
at scale. On the spreadsheets, you will find the variable description.
The original data set has at least 2 million observations and 150
variables. Inside the file "credit.xlsx," you will find only 873
observations (rows) and 71 columns. Each row represents a
Lendingclub client. We previously made the data cleaning (missing
values, correlated variables, Zero- and Near Zero-Variance Predictors).
To know more about data cleaning, see the 2nd chapter of this book.

In the next output, we see the variables of Lendingclub's customers when
they granted the loan. For example, the variable term is the term, in
years, of the loan, "annual_inc," which is the customer's annual income
when she got the loan.


```r
data<-openxlsx::read.xlsx("data/credit.xlsx")
str(data[,1:10])
#> 'data.frame':	873 obs. of  10 variables:
#>  $ Default            : chr  "Fully Paid" "Fully Paid" "Fully Paid" "Fully Paid" ...
#>  $ term               : num  1 1 2 2 1 1 1 1 1 1 ...
#>  $ installment        : num  123 820 433 290 405 ...
#>  $ grade              : num  3 3 2 6 3 2 2 1 2 3 ...
#>  $ emp_title          : num  299 209 623 126 633 636 481 540 631 314 ...
#>  $ emp_length         : num  3 3 3 5 6 3 3 8 3 5 ...
#>  $ home_ownership     : num  1 1 1 1 3 1 1 3 1 1 ...
#>  $ annual_inc         : num  55000 65000 63000 104433 34000 ...
#>  $ verification_status: num  1 1 1 2 2 1 1 1 1 1 ...
#>  $ purpose            : num  3 10 4 6 3 3 6 2 2 9 ...
```

The variable "Default", winch originally has the name "loan_status", it
has two labels:


```r
table(data[,"Default"])
#> 
#> Charged Off  Fully Paid 
#>         145         728
```

"Charge off" means that the credit grantor wrote your account off of
their receivables as a loss and is closed to future charges. When an
account displays a status of "charge off," it is closed to future use,
although the customer still owns the debt. For this example, we will
consider Charged Off equivalent to Default and Fully Paid as no default.

In a previous output, we show that the "Default" variable class is
"character," and a function we will apply below does only accept numeric
or factor variables. We transform that variable into "factor."


```r
data[,"Default"]<-factor(data[,"Default"])
```

### Prediction with the Logit model

First, we split the data set into training and test, 80% of the training
and 20% of the test data set. For an explanation of this procedure, see
chapter [Machine learning with market direction prediction: Logit].

We use the function sample when the data set is not a time series. Which
randomly generates dim[1]\*n numbers of the full data set. Where dim[1]
is the number of rows of the full data set, and n is a %, in this case,
80%.


```r
set.seed (1)
dim<-dim(data)
train_sample<-sample(dim[1],dim[1]*0.8)

train <- data[train_sample, ]
test  <- data[-train_sample, ]
```

Because the function "sample" generates random numbers, we use
"set.seed" to specify seeds to control the results; in other words, we
will always get the same results, even when we generate random numbers.

We will run the following logit model:

$$Default=\alpha_{0}\ +\beta_{1}\ term_{1}+\beta_{2}\ grade_{2}+...+\beta_{n}\ variable_{n}+e$$
Where $e$ is the error term.

The next code is to run the logit model using the train set:


```r
model<-glm(Default ~ term+grade ,data= train ,family=binomial())
summary(model)
#> 
#> Call:
#> glm(formula = Default ~ term + grade, family = binomial(), data = train)
#> 
#> Deviance Residuals: 
#>     Min       1Q   Median       3Q      Max  
#> -2.3183   0.3755   0.4647   0.5723   1.3079  
#> 
#> Coefficients:
#>             Estimate Std. Error z value Pr(>|z|)    
#> (Intercept)  3.75411    0.34435  10.902  < 2e-16 ***
#> term        -0.69219    0.24816  -2.789  0.00528 ** 
#> grade       -0.44522    0.09073  -4.907 9.25e-07 ***
#> ---
#> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
#> 
#> (Dispersion parameter for binomial family taken to be 1)
#> 
#>     Null deviance: 631.12  on 697  degrees of freedom
#> Residual deviance: 572.50  on 695  degrees of freedom
#> AIC: 578.5
#> 
#> Number of Fisher Scoring iterations: 4
```

The next code is to run the logit model with all the variables, using
the train set is:

$$Default=\alpha_{0}\ +\beta\ X+e$$ Where $X$ represents all the
variables inside the data bases, except "Default".


```r
model_all<-glm(Default ~. ,data=train ,family=binomial())
```

The next step is to make the prediction on the test data set, based on
the the "model_all".


```r
predict<-predict(model_all, newdata = test,type = "response")
head(predict)
#>           10           18           21           23           24           26 
#> 1.000000e+00 2.220446e-16 1.000000e+00 1.000000e+00 1.000000e+00 1.000000e+00
```

The "type = response" argument is to transform the results into
probability.

In the previous output, we see that our prediction is not of the type
"Fully Paid" or "Charged Off," then we transform our forecast in that
categories. We set as threshold 0.5; if our prediction is higher than
0.5, then we transform it into "Fully Paid," and otherwise "Charged
Off." It is a common practice if you wonder why our threshold is 0.5.
Still, more importantly, after estimating the prediction accuracy of our
model, we could change this threshold to improve the prediction
performance.


```r
predicf_char<-ifelse(predict>.5,"Fully Paid","Charged Off")
head(predicf_char)
#>            10            18            21            23            24 
#>  "Fully Paid" "Charged Off"  "Fully Paid"  "Fully Paid"  "Fully Paid" 
#>            26 
#>  "Fully Paid"
```

## Measuring model performance

To measure the performance of our prediction, we will use the confusion
Matrix. Before that, we need to transform our prediction into a factor.


```r

predict_factor<-factor(predicf_char)
caret::confusionMatrix(predict_factor,test[,"Default"])$table
#>              Reference
#> Prediction    Charged Off Fully Paid
#>   Charged Off          27          8
#>   Fully Paid            1        139
```

The confusion Matrix categorizes our predictions according to whether
they match the actual value. One of the table's dimensions indicates the
possible categories of predicted values, while the other shows the same
for real (reference) values.

There are other measures that the "confusionMatrix" functions show. For
this chapter, we are only concerned about Accuracy and Sensitivity.


```r
confusionMatrix(predict_factor,test[,"Default"])
#> Confusion Matrix and Statistics
#> 
#>              Reference
#> Prediction    Charged Off Fully Paid
#>   Charged Off          27          8
#>   Fully Paid            1        139
#>                                           
#>                Accuracy : 0.9486          
#>                  95% CI : (0.9046, 0.9762)
#>     No Information Rate : 0.84            
#>     P-Value [Acc > NIR] : 8.743e-06       
#>                                           
#>                   Kappa : 0.8263          
#>                                           
#>  Mcnemar's Test P-Value : 0.0455          
#>                                           
#>             Sensitivity : 0.9643          
#>             Specificity : 0.9456          
#>          Pos Pred Value : 0.7714          
#>          Neg Pred Value : 0.9929          
#>              Prevalence : 0.1600          
#>          Detection Rate : 0.1543          
#>    Detection Prevalence : 0.2000          
#>       Balanced Accuracy : 0.9549          
#>                                           
#>        'Positive' Class : Charged Off     
#> 
```



The accuracy is, therefore, a proportion representing the number of true
positives and negatives divided by the total number of predictions. In
this case, our mode Accuracy is 0.9485714

The sensitivity of a model (also called the true positive rate) measures
the proportion of positive examples correctly classified. For this
example, at the end of the "confusionMatrix" output we see that the
'Positive' Class is "Charged Off". Our mode Sensitivity is 0.9642857.

## Prediction with Linear Discriminant Analysis (LDA)

Why do we need another method when we already have the logistic model?
There are several reasons [@statistical_lerarning]:

• When the classes are well-separated, the parameter estimates for the
logistic regression model are surprisingly unstable. The linear
Discriminant Analysis method does not suffer from this problem.

• If the number of observations is small and the distribution of the
independent variables is approximately normal in each class, the linear
discriminant model is again more stable than the logistic regression
model.

For this chapter, we will use the LDA model to compare its Accuracy
against the Logit model.

The next code estimates the LDA model and makes the prediction:


```r
model_lda<-MASS::lda(Default~.,data=train)
pred_lda<-predict(model_lda, newdata = test)
```

The object "pred_lda" is an R-list, which contains the prediction, but
also many other statistics, then to apply the "confusionMatrix" we need
to get the prediction results:


```r
confusionMatrix(pred_lda[["class"]],test[,"Default"])
#> Confusion Matrix and Statistics
#> 
#>              Reference
#> Prediction    Charged Off Fully Paid
#>   Charged Off          24          4
#>   Fully Paid            4        143
#>                                           
#>                Accuracy : 0.9543          
#>                  95% CI : (0.9119, 0.9801)
#>     No Information Rate : 0.84            
#>     P-Value [Acc > NIR] : 2.372e-06       
#>                                           
#>                   Kappa : 0.8299          
#>                                           
#>  Mcnemar's Test P-Value : 1               
#>                                           
#>             Sensitivity : 0.8571          
#>             Specificity : 0.9728          
#>          Pos Pred Value : 0.8571          
#>          Neg Pred Value : 0.9728          
#>              Prevalence : 0.1600          
#>          Detection Rate : 0.1371          
#>    Detection Prevalence : 0.1600          
#>       Balanced Accuracy : 0.9150          
#>                                           
#>        'Positive' Class : Charged Off     
#> 
```



The Accuracy of the LDA model is 0.9542857, and its Sensitivity is 0.8571429.

In this case, the LDA Accuracy is higher than the logit model; the
sensitivity is the opposite. Depending on what we are interested in, the
model better predicts performance. In "credit allocation," we are
usually more concerned with the 'Positive' cases, in this case, "Charged
Off," because of the default risk.

### 3 Cross validation.

Cross-validation is a "resampling" method. It involves repeatedly
drawing samples from a training set and refitting a model of interest on each sample to obtain additional information about the model. Such an approach may allow us to get information that would not be available from fitting the model only once using the original training sample.

Instead of dividing the sample only once, this approach involves
randomly k-fold CV splits the set of observations into k groups, or
folds, of approximately equal size.

In other words, this procedure would validate it our Accuracy/sensitivity will be stable when we split the sample in training and test it several times. The Caret function "train" can optimize for Accuracy.

We start by applying it to the logit model:


```r
# similar to the previous models
gbmFit1 <- train(Default ~ ., data = train,
                  
                 method = "glm",
                 
# in here we have the tuning parameters, in this case we use the "cv" method for cross, which is the number of times the model split the sample.                   
                            trControl = trainControl(method = "cv", number = 10),
                        trace=0,   metric="Accuracy")
gbmFit1
#> Generalized Linear Model 
#> 
#> 698 samples
#>  70 predictor
#>   2 classes: 'Charged Off', 'Fully Paid' 
#> 
#> No pre-processing
#> Resampling: Cross-Validated (10 fold) 
#> Summary of sample sizes: 627, 628, 628, 629, 628, 629, ... 
#> Resampling results:
#> 
#>   Accuracy   Kappa    
#>   0.9470376  0.8144761
```

And for the LDA:


```r
# similar to the previous models
gbmFit1 <- train(Default ~ ., data = train,
                  
                 method = "lda",
                 
# in here we have the tuning parameters, in this case we use the "cv" method for cross, which is the number of times the model split the sample.                   
                            trControl = trainControl(method = "cv", number = 10),
                        trace=0,   metric="Accuracy")
gbmFit1
#> Linear Discriminant Analysis 
#> 
#> 698 samples
#>  70 predictor
#>   2 classes: 'Charged Off', 'Fully Paid' 
#> 
#> No pre-processing
#> Resampling: Cross-Validated (10 fold) 
#> Summary of sample sizes: 628, 629, 629, 628, 628, 628, ... 
#> Resampling results:
#> 
#>   Accuracy   Kappa    
#>   0.9527329  0.8275475
```

Our results are consistent with the previous result; the Accuracy of the
LDA is higher than the Logit one.

To improve the accuracy, we could also apply variable selection methods,
such as glmStepAIC.

***Warning*****:** The following code takes 20 minutes to run, depending
on the processor.


```r
gbmFit1 <- train(Default ~ ., data = train, method = "glmStepAIC",
              
                 trControl = trainControl(method = "cv", number = 10),
                        trace=0,   metric="Accuracy")
    gbmFit1

```

    ## Generalized Linear Model with Stepwise Feature Selection
    ##
    ## 698 samples 
    ## 70 predictor 
    ## 2 classes: 'Charged Off', 'Fully Paid'
    ##
    ## No pre-processing Resampling: Cross-Validated (10 fold)
    ## Summary of sample sizes: 628, 629, 628, 628, 628, 628, ... 
    ## Resampling results:
    ##
    ## Accuracy   Kappa
    ## 0.9599149 0.8585923


The accuracy of 0.9599 is higher than the Logit model, which includes all variables 0.9470. 

To get the final model or the final variables, we apply the following:


```r
step_var<-rownames(data.frame(gbmFit1$finalModel$coefficients))[-1] 
step_var
```


```
#>  [1] "term"                  "purpose"               "revol_bal"            
#>  [4] "total_rec_int"         "recoveries"            "last_pymnt_amnt"      
#>  [7] "last_fico_range_high"  "open_act_il"           "total_cu_tl"          
#> [10] "num_accts_ever_120_pd" "num_sats"              "num_tl_op_past_12m"   
#> [13] "total_bc_limit"
```

If we want to use those variables to further improve the model:


```r
train_step<-cbind(train[,"Default"],train[,step_var])
colnames(train_step)[1]<-"Default"
test_step<-cbind(test[,"Default"],test[,step_var])
colnames(test_step)[1]<-"Default"
```

For example, if we run the LDA model with those variables again:

```r
gbmFit1 <- train(Default ~ ., data = train_step,
                  
                 method = "lda",
                 
# in here we have the tuning parameters, in this case we use the "cv" method for cross, which is the number of times the model split the sample.                   
                            trControl = trainControl(method = "cv", number = 10),
                        trace=0,   metric="Accuracy")
gbmFit1
#> Linear Discriminant Analysis 
#> 
#> 698 samples
#>  13 predictor
#>   2 classes: 'Charged Off', 'Fully Paid' 
#> 
#> No pre-processing
#> Resampling: Cross-Validated (10 fold) 
#> Summary of sample sizes: 628, 628, 627, 629, 628, 628, ... 
#> Resampling results:
#> 
#>   Accuracy   Kappa    
#>   0.9612612  0.8568619
```

We got a higher accuracy.
