---
output:
  html_document: default
  word_document: default
---


# Rational agent and behavioral finance in investment


```r
library(quantmod)
library(FinTS)
library(tseries)
library(rugarch)
library(dplyr) 
```

Around 1970, economists argued that an efficient market (Fama, 1970) should instantaneously reflect all the available information of a particular financial security, Efficient market hypothesis EMH. 

Then, arbitrage opportunities were difficult to exist, or they used to argue that those markets were not predictable.

Academics were reasonably content with the efficient market hypothesis (EMH) until sometime toward the end of the twentieth century. The year 1987 was critical in undermining faith in the EMH. 


U.S. stock market behavior in 1987 was bizarre. The year began with the Dow Jones Industrial Average historic collapse. 

What is interesting about 1987 is that trading folklore and the activities of leading academic economists fit the behavioral finance point of view, not the EMH point of view. 


•Economists who were actively discussing and acting in financial markets seemed to believe that markets were predictable, a key tenet of modern behavioral finance.
The test in this chapter and some of the text is  based on Wooldridge (2012) with my own codes.

In summary, trading strategies perform best in non-efficient markets, where abundant arbitrage opportunities exist. Perfectly efficient markets instantaneously incorporate all available market information, then no trading opportunities. 

•In this session, we will measure whether a market is efficient. We could build algorithms to predict the stock markets if the market is inefficient. 

##   Shorts samples test of efficient markets hypothesis (EMH) for one asset

Suppose $y_{t}$ is the daily price of the S&P500. A strict form of the efficient markets hypothesis establishes that the observable information to the index prior to day *t* should not help predict the index. If we use only past information on $y_{t}$, the EMH is could be presented as:

$$E(y_t/ y_{t-1} ,y_{t-1},.... )=E(y_t) $$

Where the term on the right is the expected value of $y_{t}$, given the historical information of the index $y_{t-1} ,y_{t-1},....$. In other words, the expected value does not depend on its own historical information. However, if the previous equation is false, it implies that we could use the information to predict the current price. If the previous equation is false, we could use the information on the past to predict the current price. 

The EMH presumes that such investment opportunities will be noticed and will disappear almost instantaneously.

One simple way to make the EMH test is to specify the AR(1): 

$$y_t= \beta_0 +\beta_1\ y_{t-1}+u_t, $$


A significant $\beta_1$ coefficient would reject EMH; then, we could use the information on the past to predict the current price. However, it is a common practice to make the test using more lags.  

$$y_t= \beta_0 +\beta_1\ y_{t-1},...,y_{t-n}+u_t$$

Suppose that we want to make the EMH test on the returns of the S&P 500 index.  In the next code, we download the S&P 500 index from "2019-10-20" to "2022-10-20". Remember that by yahoo finance, the ticker name is "^GSPC". 

```r
ticker<-"^GSPC"
getSymbols(ticker, from="2019-10-20", to="2022-10-20")
#> [1] "^GSPC"
```

As you see in the environment, the object that stores the S&P information is only called GSPC. We take the close price of the index, column 4, and we apply the function *Delt* to estimate the returns. 

```r
all<-GSPC
dji<-all[,4]
dji<-Delt(all[,4])
```

The historical information we download is daily, then we could start testing if we could use the information of the two previous two days to predict the current index price.  

 

$$y_t= \beta_0 +\beta_1\ y_{t-1},y_{t-2}+u_t$$

To run the previous model, we need to create the variables and store them in a data frame. We apply the function *lag* to create the lagged variables and store them in the object *dji* with the close price. Remember that there are other libraries whit a function *lag*, and in writing *stats::lag* we are calling the lag function from the stats library. For convenience, we change the column names and delete missing values rows by applying the function  *na.omit*.


```r
la2<-stats::lag(dji,1)
la3<-stats::lag(dji,2)
dji<-cbind(dji,la2,la3)
dji<-na.omit(dji)
colnames(dji)<-c("SP500","SP500_lag1","SP500_lag2")
head(dji)
#>                    SP500    SP500_lag1    SP500_lag2
#> 2019-10-24  0.0019204462  0.0028471490 -0.0035686666
#> 2019-10-25  0.0040727006  0.0019204462  0.0028471490
#> 2019-10-28  0.0055813379  0.0040727006  0.0019204462
#> 2019-10-29 -0.0008324052  0.0055813379  0.0040727006
#> 2019-10-30  0.0032533702 -0.0008324052  0.0055813379
#> 2019-10-31 -0.0030228606  0.0032533702 -0.0008324052
```


We apply an OLS regression:

```r
summary(lm(SP500~.,data =dji))
#> 
#> Call:
#> lm(formula = SP500 ~ ., data = dji)
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
#> Multiple R-squared:  0.06649,	Adjusted R-squared:  0.06401 
#> F-statistic: 26.71 on 2 and 750 DF,  p-value: 6.219e-12
```


Remember, a significant beta1 coefficient would reject EMH; then, in this case, it implies that we could use the previous two days to predict the current price. 


## Long samples test of efficient markets hypothesis (EMH) for one asset



Although the EMH states that the expected return given past observable information should be constant, it says nothing about the conditional variance. It could be tested using a 
heteroskedasticity, such as Breusch-Pagan, for example. However, this heteroskedasticity is better characterized by the ARCH model.

Suppose we have the dependent variable, y(t), a contemporary exogenous variable, z(t).


$$E(y_t/z_t,y_{t-1},z_{t-1},y_{t-2},..)= \beta_0 +\beta_1\ z_t+\beta_2\ y_{t-1}+\beta_3\ z_{t-1}. $$


The typical approach is to assume that: 

$$Var(y_t/z_t,y_{t-1},z_{t-1},y_{t-2},..)= \sigma, $$
is a constant. But this variance could follow an ARCH model:

$$Var(y_t/z_t,y_{t-1},z_{t-1},y_{t-2},..)=Var(u_t/z_t,y_{t-1},z_{t-1},y_{t-2},..)=\\ \alpha_0 +\alpha_1\ u^2_{t-2}.$$
Where

$$u_t=y_t-E(y_t/z_t,y_{t-1},z_{t-1},y_{t-2},..)$$

We can check for ARCH effects by using the ArchTest() function from the FinTS package. Lagrange Multiplier (LM) test for autoregressive conditional heteroscedasticity (ARCH). Computes the Lagrange multiplier test for conditional heteroscedasticity. Equivalent to the test by OLS: 

$$u_t=\alpha_0 +\alpha_1\ u^2_{t-2}$$
We look to verify the significance of alpha 1. If alpha 1 significant, we reject the null hypothesis and conclude the presence of ARCH(1) effects. Then we could use past information to predict the future.
ArchTest(object,lags=n), usually lags=1


```r
ticker<-"^GSPC"
getSymbols(ticker,from="2021-05-01",to="2022-05-01")
#> [1] "^GSPC"
dji_long<-GSPC[,4]
```




```r
ar<-ArchTest(dji_long,lags=1)
data.frame(ar$p.value)
#>               ar.p.value
#> Chi-squared 9.801502e-53
```

If p-value is <10%, in this case, we conclude the presence of ARCH(1) effects, then we could make a forecast of the time series using past information.




## Long samples test of efficient markets hypothesis (EMH) applyied to portfolio 

In the next session, we will cover the subject of market anomalies. We will cover the momentum anomalies. However, that strategy is for constructing a portfolio of stocks. However, first, we need to filter those stocks for which we can make a prediction. We will assume that is going to be a long-term horizon portfolio. Then, we need to apply the long samples test of the efficient markets hypothesis. 



```r
df<-read.xlsx("data/df_dates.xlsx", detectDates = T)

date<-df[,1]

dim<-dim(df)

# important to takeout the data before transforming to xts, other wise does not transform into numeric. 
data<-df[,2:dim[2]]

datax<- xts(data,
         order.by = as.Date(date))

dfx<-na.omit(datax)
```


The following code makes the long samples test of the efficient markets hypothesis (EMH) applied to many assets of the dfx object. 

We start by estimating the returns for each time series and deleteing missig valies. 

```r
return<-Delt(dfx[,1])

# we are going to apply the Delt function to the 100 stocks

# function apply()
return_all<-apply(dfx, 2, Delt)
# es 1 for rows o 2for columns
```


The next loops knowledge is over the level of this course contents requires, so its application could be covered in the final exam, but the topic of loops will not be covered.




```r
dfr<-return_all
m<-26
#---de aqui es la creación del Loop for, esto rebasa el nivel de este curso
ar<-c()
n<-dim(dfr)[2]
for (i in 1:n){
ar1<-ArchTest(dfr[,i],lags=m)$p.value
ar<-c(ar,ar1)
}
#----- 
ar<-data.frame(ar)
# it has the p value of the EMH test, if the p-value is lees than 10%, then we could make predictions 

# add the name of the ticker
col_name<-colnames(return_all)
head(col_name)
#> [1] "AAPL.Close"  "MSFT.Close"  "GOOG.Close"  "GOOGL.Close" "AMZN.Close" 
#> [6] "TSLA.Close"
# add the ticker na,me
rownames(ar)<-col_name
head(ar)
#>                       ar
#> AAPL.Close  3.731438e-17
#> MSFT.Close  6.033001e-39
#> GOOG.Close  3.224007e-16
#> GOOGL.Close 2.570860e-16
#> AMZN.Close  5.093709e-04
#> TSLA.Close  2.275599e-05
```

The following code counts the number of tickers that we could use to make a prediction, applyinf the ifelse function and combing it with the object that contains the EMH test.

```r
library(dplyr)

pred<-ifelse(ar[,1]<0.1,"Predict","No Predict")
# merge with the ar object 
ar<-cbind(ar,pred)
head(ar)
#>                       ar    pred
#> AAPL.Close  3.731438e-17 Predict
#> MSFT.Close  6.033001e-39 Predict
#> GOOG.Close  3.224007e-16 Predict
#> GOOGL.Close 2.570860e-16 Predict
#> AMZN.Close  5.093709e-04 Predict
#> TSLA.Close  2.275599e-05 Predict
```

Finally, we could filter to get only those tickers with category predict. 

arf<- df %>%
  filter(coll== "category")

```r

arf<- ar %>%
  filter(pred == "Predict")
```


Also we take the historical information of the filtered stocks

```r
# this code takes the names of the filtered tickers
col_filterd<-rownames(arf)
dfx_2<-dfx[,col_filterd]
```




```r
dfx_3<-data.frame(dfx_2)
date<-rownames(dfx_3)
dfx_4<-cbind(date,dfx_3)
```


## Bibliography

Wooldridge, J. M. (2012). Introductory econometrics: A modern approach. Mason, OH: Thomson/South-Western.

