# Rational agent and behavioral finance in investment


In this chapter we will use the following libraries:


```r
library(quantmod)
library(lmtest)
library(openxlsx)
library(dplyr)
```


Around 1970, economists argued that an efficient market should instantaneously reflect all the available information of a particular financial security, called the Efficient Market Hypothesis EMH [@FAMA]. Then, arbitrage opportunities were difficult to exist, or they used to argue that those markets needed to be more predictable. Academics were reasonably content with the EMH until in1987 stock market behavior in 1987 was bizarre. The year began with the Dow Jones Industrial Average's historic collapse. What is interesting about 1987 is that trading folklore and the activities of leading academic economists fit the behavioral finance point of view, not the EMH point of view. Economists actively discussing and acting in financial markets seemed to believe that markets were predictable, a key principle of modern behavioral finance [@BF].

By designing systematic trading platforms, some traders and trading systems aim to generate trading signals that consistently produce positive outcomes over many trades. Usually, the trades test successfully of trading systems on large amounts of past historical data. A more scientific method for analyzing a particular financial security may lie in determining whether the security price changes are random or not. If the price changes are random, the probability of detecting a consistently profitable trading opportunity for that particular security is small. On the other hand, if the price changes are nonrandom, the financial security has persistent predictability and should be analyzed further. Then, it is possible to measure the relative availability of trading opportunities with the market inefficiency tests [@HFT]. In summary, if the tests detect that new information takes in slowly in the asset prices, arbitrage opportunities exist, and the market is inefficient. 

In this chapter, we apply a test proposed by [@wooldridge] to identify arbitrage opportunities or to find inefficient markets. 


##   EMH test on historical returns for one asset

Suppose $y_{t}$ is the daily price of the S&P500. A strict form of the efficient markets hypothesis establishes that the historical information on the index before day *t* should not help predict the index. If we use only past information on $y_{t}$, a market is efficient if the following is true:

$$y_t= \beta_0 +\beta_1\ y_{t-1} + \beta_2\ y_{t-2}+u_t$$

Where the term on the right is the expected value of $y_{t}$, given the historical information of the index $y_{t-1} ,y_{t-2},....$. In other words, the expected value does not depend on its own historical information. However, if the previous equation is false, it implies that we could use the information to predict the current price. If the previous equation is false, we could use the information on the past to predict the current price. 


Suppose that we want to make the EMH test on the returns of the S&P 500 index.  In the next code, we download the S&P 500 index from "2019-10-20" to "2022-10-20". Remember that by yahoo finance, the ticker name is "^GSPC". 

```r
ticker<-"^GSPC"
getSymbols(ticker, from="2019-10-20", to="2022-10-20")
#> [1] "^GSPC"
```

We take the close price of the index, apply the function *Delt* to estimate the returns. 

```r
close<-GSPC[,4]
ret<-Delt(close)
```



$$y_t= \beta_0 +\beta_1\ y_{t-1} + \beta_2\ y_{t-2}+u_t$$

To run the previous model, we need to create the variables and store them in a data frame. We apply the function *lag* to create the lagged variables-

```r
lag1<-stats::lag(ret[,1],1)
lag2<-stats::lag(ret[,1],2)
ret<-cbind(ret, lag1,lag2)
```

For convenience, we change the column names.


```r
colnames(ret)<-c("SP500","SP500_lag1","SP500_lag2")
```


We apply an OLS regression:

```r
model<-lm(SP500 ~.,data =ret)
summary(model)
#> 
#> Call:
#> lm(formula = SP500 ~ ., data = ret)
#> 
#> Residuals:
#>       Min        1Q    Median        3Q       Max 
#> -0.111441 -0.005786  0.000537  0.007425  0.092978 
#> 
#> Coefficients:
#>               Estimate Std. Error t value Pr(>|t|)    
#> (Intercept)  0.0004309  0.0005521   0.780 0.435371    
#> SP500_lag1  -0.1990963  0.0362312  -5.495 5.35e-08 ***
#> SP500_lag2   0.1248616  0.0362416   3.445 0.000602 ***
#> ---
#> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
#> 
#> Residual standard error: 0.01514 on 750 degrees of freedom
#>   (3 observations deleted due to missingness)
#> Multiple R-squared:  0.06649,	Adjusted R-squared:  0.06401 
#> F-statistic: 26.71 on 2 and 750 DF,  p-value: 6.219e-12
```


Remember, a significant beta1 coefficient would reject EMH. 


For example, we could use past information on the price or a moving average to forecast the Biance price.  

$$bnb_{t}=\alpha\ +\beta1\ bnb_{t-1}+\beta2\ bnb_{t-2} +\beta3\ sma +\ e$$


## EMH test for variance for one asset

For some financial time series, such as stock returns, the expected returns do not depend on past returns (the Market is efficient), but the variance of returns does. For example, in the model:

$$r_t= \beta_0 +\beta_1\ r_{t-1}+u_t$$




We could apply a test to verify if the variance of returns have an effect o the returns: 


$$u^2_{t}= \delta_0 +\delta_1\ r_{t-1}+e_t$$

The previous model is heteroskedasticity test, then we could applya Breusch-Pagan test for heteroskedasticity, using the function bptest(model)

```r
lmtest::bptest(model)
#> 
#> 	studentized Breusch-Pagan test
#> 
#> data:  model
#> BP = 27.216, df = 2, p-value = 1.23e-06
```

Suppose the test is significant in the former result (p-value <10%). Then there is no EMH, and we could use the variance or standard deviation to make a prediction. 


For example, we could use past information and a standard deviation moving average, $sdma$,  to forecast the Biance price.

$$bnb_{t}=\alpha\ +\beta1\ bnb_{t-1}+\beta2\ bnb_{t-2} +\beta3\ sma +\beta4\ sdma +\ e$$

See chapter 5 of the book for an example. "Big data and machine learning"
https://www.arturo-bernal.com/book/AFP/big-data-and-machine-learning.html


We show another example in the next section.

## EMH tests for the variance for many assets to build a portfolio

In the next session, we will cover the subject of market anomalies. We will cover the momentum anomalies. However, that strategy is for constructing a portfolio of stocks. However, first, we need to filter those stocks for which we can make a prediction. We filter for the variance test of efficient markets hypothesis.

First we read the data, and for convenience, we transform the data frame into xts. 


```r
df<-read.csv("https://raw.githubusercontent.com/abernal30/BookAFP/main/data/df_dates.csv")
df<-as.data.frame(df)
date<-as.Date(df[,1],format="%m/%d/%Y") 
datax<- xts(df[,-1],
         order.by = date)
```


We estimated the returns, Delt. 
apply(data, 2, func)

```r
return_all<-apply(datax, 2, Delt)
```



Xts again: becasue we loos the xts property:

```r
return_all_xts<-xts(return_all,
         order.by = as.Date(date))
head(return_all_xts[,1:4]) 
#>              AAPL.Close   MSFT.Close    GOOG.Close  GOOGL.Close
#> 2020-01-02           NA           NA            NA           NA
#> 2020-01-03 -0.009722044 -0.012451750 -0.0049072022 -0.005231342
#> 2020-01-06  0.007968248  0.002584819  0.0246570974  0.026654062
#> 2020-01-07 -0.004703042 -0.009117758 -0.0006240057 -0.001931646
#> 2020-01-08  0.016086289  0.015928379  0.0078803309  0.007117757
#> 2020-01-09  0.021240806  0.012492973  0.0110444988  0.010497921
```

The following code has the same test we made for one stock on the previous second section of this chapter:  

```r
i<-1

#For the next chunk:

#---------------copy from here------- 
data<-return_all_xts[, i]
lag1 <- stats::lag(data,1)
lag2 <- stats::lag(data,2)
ret <- data.frame(cbind(data, lag1,lag2))
colnames(ret)<-c("SP500","SP500_lag1","SP500_lag2")

#-------------to here ------------

model <- lm(SP500 ~.,data =ret)
bp <- bptest(model)
bp <- bp[["p.value"]]
bp
#>         BP 
#> 0.00134487
```


The next chunk shows a "Loop For" to show a similar procedure as the previous chunk. The knowledge it requires is over the level of this course's contents. You could expect to develop the process for the final exam following the procedure of the previous chunk, building an R function together with the "apply" function, not creating a "Loop For."

```r
bp_all_Loop<-data.frame()
dim<-dim(return_all_xts)[2] 
for (i in 1:dim){
 
#----copy here -----

  data<-return_all_xts[, i]
lag1 <- stats::lag(data,1)
lag2 <- stats::lag(data,2)
ret <- data.frame(cbind(data, lag1,lag2))
colnames(ret)<-c("SP500","SP500_lag1","SP500_lag2")

# ----to Here----------------

model <- lm(SP500 ~.,data =ret)
  bp <- bptest(model)
  bp <- bp[["p.value"]]
  bp_all_Loop[i,1]<-colnames(return_all_xts)[i]
  bp_all_Loop[i,2]<-bp
}
colnames(bp_all_Loop)<-c("Ticker","p-value")
head(bp_all_Loop)
#>        Ticker     p-value
#> 1  AAPL.Close 0.001344870
#> 2  MSFT.Close 0.002359318
#> 3  GOOG.Close 0.007415325
#> 4 GOOGL.Close 0.009713821
#> 5  AMZN.Close 0.542408713
#> 6  TSLA.Close 0.086492404
tail(bp_all_Loop)
#>          Ticker      p-value
#> 95    TXN.Close 0.0002357208
#> 96    CRM.Close 0.8711756478
#> 97    BMY.Close 0.1610277136
#> 98    UPS.Close 0.9590241181
#> 99  RLLCF.Close 0.8083656314
#> 100  QCOM.Close 0.0539758548
```


The following code helps us count the number of tickers we could use to make a prediction, applying the "ifelse" function and combing it with the object containing the EMH test.


```r
pred<-ifelse(bp_all_Loop[,"p-value"]<0.1,"Predict","No Predict" )

bp_all_Loop_1<- cbind(bp_all_Loop,pred)

head(bp_all_Loop_1)
#>        Ticker     p-value       pred
#> 1  AAPL.Close 0.001344870    Predict
#> 2  MSFT.Close 0.002359318    Predict
#> 3  GOOG.Close 0.007415325    Predict
#> 4 GOOGL.Close 0.009713821    Predict
#> 5  AMZN.Close 0.542408713 No Predict
#> 6  TSLA.Close 0.086492404    Predict
tail(bp_all_Loop_1)
#>          Ticker      p-value       pred
#> 95    TXN.Close 0.0002357208    Predict
#> 96    CRM.Close 0.8711756478 No Predict
#> 97    BMY.Close 0.1610277136 No Predict
#> 98    UPS.Close 0.9590241181 No Predict
#> 99  RLLCF.Close 0.8083656314 No Predict
#> 100  QCOM.Close 0.0539758548    Predict
```

Finally, we could filter to get only those tickers with category predict. 

```r
bp_all_Loop_f<- bp_all_Loop_1 %>%
  dplyr::filter(pred == "Predict") 
head(bp_all_Loop_f)
#>        Ticker      p-value    pred
#> 1  AAPL.Close 1.344870e-03 Predict
#> 2  MSFT.Close 2.359318e-03 Predict
#> 3  GOOG.Close 7.415325e-03 Predict
#> 4 GOOGL.Close 9.713821e-03 Predict
#> 5  TSLA.Close 8.649240e-02 Predict
#> 6 BRK.A.Close 3.950538e-06 Predict
tail(bp_all_Loop_f)  
#>        Ticker      p-value    pred
#> 71  WFC.Close 6.487655e-07 Predict
#> 72 C.PJ.Close 3.292293e-07 Predict
#> 73   PM.Close 2.047688e-07 Predict
#> 74  LIN.Close 3.799114e-02 Predict
#> 75  TXN.Close 2.357208e-04 Predict
#> 76 QCOM.Close 5.397585e-02 Predict
```


Also we take the historical information of the filtered stocks

```r
ticker<-bp_all_Loop_f[,"Ticker"]
emh<-return_all_xts[,ticker]
head(emh[,1:4])
#>              AAPL.Close   MSFT.Close    GOOG.Close  GOOGL.Close
#> 2020-01-02           NA           NA            NA           NA
#> 2020-01-03 -0.009722044 -0.012451750 -0.0049072022 -0.005231342
#> 2020-01-06  0.007968248  0.002584819  0.0246570974  0.026654062
#> 2020-01-07 -0.004703042 -0.009117758 -0.0006240057 -0.001931646
#> 2020-01-08  0.016086289  0.015928379  0.0078803309  0.007117757
#> 2020-01-09  0.021240806  0.012492973  0.0110444988  0.010497921
tail(emh[,1:4])
#>              AAPL.Close   MSFT.Close    GOOG.Close  GOOGL.Close
#> 2022-05-19 -0.024641392 -0.003699634 -0.0147285646 -0.013543429
#> 2022-05-20  0.001747288 -0.002291226 -0.0129350191 -0.013371513
#> 2022-05-23  0.040119232  0.032031977  0.0215299497  0.023689766
#> 2022-05-24 -0.019215988 -0.003951656 -0.0514075636 -0.049494164
#> 2022-05-25  0.001139947  0.011170149 -0.0008165988 -0.001556952
#> 2022-05-26  0.023199508  0.012875229  0.0232096155  0.018784556
```
