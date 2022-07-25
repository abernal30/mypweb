# Rational agent and behavioral finance in investment



The test in this chapter and some of the text is  based on Wooldridge (2012) with my own codes.

##   Shorts samples test of efficient markets hypothesis (EMH) for one asset

Let yt be the daily price of the S&P500. A strict form of the efficient markets hypothesis states that information observable to the market prior to day t should not help to predict the price. If we use only past information on y, the EMH is stated as:

$$E(y_t/ y_{t-1} ,y_{t-1},.... )=E(y_t) $$

If the previous equation is false, we could use the information on the past to predict the current price. 

The EMH presumes that such investment opportunities will be noticed and will disappear almost instantaneously.

One simple way that equation is to specify the AR(1) model as the alternative model. 

$$y_t= \beta_0 +\beta_1\ y_{t-1}+u_t, $$


A significant beta1 coefficient would reject EMH; then, we could use the information on the past to predict the current price. However, it is a common practice to make the test using more lags.  

$$y_t= \beta_0 +\beta_1\ y_{t-1},...,y_{t-n}+u_t$$




Now, to make the lags of the price, we make a Loop For, creating the lag and storing the created lags into the xts object where the S&P500 close price is stored.

Note. Compute a lagged version of a time series, shifting the time base back by a given number of observations.




```r
dji<-all[,4]
dji<-Delt(all[,4])
la2<-stats::lag(dji,2)
la3<-stats::lag(dji,3)
dji<-cbind(dji,la2,la3)
dji<-na.omit(dji)
colnames(dji)<-c("SP500","SP500_lag2","SP500_lag3")
head(dji)
#>                  SP500   SP500_lag2   SP500_lag3
#> 2007-01-08  0.01412499 -0.014953248 -0.012687405
#> 2007-01-09 -0.01652505 -0.009250474 -0.014953248
#> 2007-01-10  0.01032165  0.014124994 -0.009250474
#> 2007-01-11  0.04110240 -0.016525047  0.014124994
#> 2007-01-12  0.01437702  0.010321651 -0.016525047
#> 2007-01-15  0.02137235  0.041102400  0.010321651
```


Run the regression

```r
summary(lm(SP500~.,data =dji))
#> 
#> Call:
#> lm(formula = SP500 ~ ., data = dji)
#> 
#> Residuals:
#>       Min        1Q    Median        3Q       Max 
#> -0.170951 -0.009607 -0.000202  0.009729  0.136626 
#> 
#> Coefficients:
#>               Estimate Std. Error t value Pr(>|t|)  
#> (Intercept)  0.0007028  0.0003145   2.235   0.0255 *
#> SP500_lag2   0.0272148  0.0160414   1.697   0.0899 .
#> SP500_lag3  -0.0120675  0.0160421  -0.752   0.4520  
#> ---
#> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
#> 
#> Residual standard error: 0.0196 on 3890 degrees of freedom
#> Multiple R-squared:  0.0008625,	Adjusted R-squared:  0.0003488 
#> F-statistic: 1.679 on 2 and 3890 DF,  p-value: 0.1867
```


Remember, a significant beta1 coefficient would reject EMH; then, we could use the past information to predict the a price, in this case the DJI. 


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




```r
library(quantmod)
#library(xml2) # this are for the  code 
#library(rvest) # in 154 to 177 lines, just instaal it if you are running that code
library(openxlsx)
library(FinTS)
library(tseries)
library(rugarch)
```


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
col_name
#>   [1] "AAPL.Close"   "MSFT.Close"   "GOOG.Close"   "GOOGL.Close"  "AMZN.Close"  
#>   [6] "TSLA.Close"   "BRK.A.Close"  "BRK.B.Close"  "FB.Close"     "TSM.Close"   
#>  [11] "JNJ.Close"    "UNH.Close"    "NVDA.Close"   "V.Close"      "JPM.Close"   
#>  [16] "TCEHY.Close"  "TCTZF.Close"  "XOM.Close"    "BAC.Close"    "PG.Close"    
#>  [21] "WMT.Close"    "MA.Close"     "CVX.Close"    "JPM.PC.Close" "JPM.PD.Close"
#>  [26] "NSRGY.Close"  "NSRGF.Close"  "LVMUY.Close"  "BAC.PK.Close" "HD.Close"    
#>  [31] "LVMHF.Close"  "BAC.PL.Close" "LLY.Close"    "PFE.Close"    "RHHBF.Close" 
#>  [36] "RHHBY.Close"  "RHHVF.Close"  "KO.Close"     "BML.PL.Close" "BAC.PE.Close"
#>  [41] "ABBV.Close"   "BML.PG.Close" "BAC.PB.Close" "BML.PH.Close" "BML.PJ.Close"
#>  [46] "BABA.Close"   "BABAF.Close"  "AVGO.Close"   "NVO.Close"    "NONOF.Close" 
#>  [51] "IDCBF.Close"  "IDCBY.Close"  "PEP.Close"    "MRK.Close"    "ASML.Close"  
#>  [56] "ASMLF.Close"  "TM.Close"     "TOYOF.Close"  "RYDAF.Close"  "SHEL.Close"  
#>  [61] "TMO.Close"    "BHP.Close"    "BHPLF.Close"  "VZ.Close"     "AZN.Close"   
#>  [66] "COST.Close"   "AZNCF.Close"  "ABT.Close"    "NVS.Close"    "WFC.PY.Close"
#>  [71] "ADBE.Close"   "NVSEF.Close"  "DIS.Close"    "WFC.PL.Close" "CMCSA.Close" 
#>  [76] "ORCL.Close"   "DHR.Close"    "WFC.PR.Close" "WFC.PQ.Close" "ACN.Close"   
#>  [81] "CSCO.Close"   "LRLCF.Close"  "LRLCY.Close"  "CICHF.Close"  "CICHY.Close" 
#>  [86] "MCD.Close"    "NKE.Close"    "INTC.Close"   "WFC.Close"    "C.PJ.Close"  
#>  [91] "TMUS.Close"   "PM.Close"     "AMD.Close"    "LIN.Close"    "TXN.Close"   
#>  [96] "CRM.Close"    "BMY.Close"    "UPS.Close"    "RLLCF.Close"  "QCOM.Close"
# add the ticker na,me
rownames(ar)<-col_name
ar
#>                        ar
#> AAPL.Close   3.731438e-17
#> MSFT.Close   6.033001e-39
#> GOOG.Close   3.224007e-16
#> GOOGL.Close  2.570860e-16
#> AMZN.Close   5.093709e-04
#> TSLA.Close   2.275599e-05
#> BRK.A.Close  9.677118e-32
#> BRK.B.Close  1.239809e-42
#> FB.Close     9.999469e-01
#> TSM.Close    1.023778e-17
#> JNJ.Close    3.822835e-55
#> UNH.Close    4.686889e-41
#> NVDA.Close   5.391420e-16
#> V.Close      4.215814e-25
#> JPM.Close    2.665692e-25
#> TCEHY.Close  9.959273e-01
#> TCTZF.Close  9.621425e-01
#> XOM.Close    1.416110e-15
#> BAC.Close    2.022251e-25
#> PG.Close     3.813798e-46
#> WMT.Close    6.208759e-18
#> MA.Close     1.311943e-25
#> CVX.Close    2.928436e-36
#> JPM.PC.Close 1.177236e-31
#> JPM.PD.Close 1.802175e-39
#> NSRGY.Close  3.838131e-34
#> NSRGF.Close  9.786318e-17
#> LVMUY.Close  3.131397e-30
#> BAC.PK.Close 1.334235e-44
#> HD.Close     1.407075e-28
#> LVMHF.Close  4.802603e-23
#> BAC.PL.Close 5.073425e-39
#> LLY.Close    4.523789e-01
#> PFE.Close    4.057538e-10
#> RHHBF.Close  3.191309e-10
#> RHHBY.Close  4.145604e-21
#> RHHVF.Close  4.923693e-22
#> KO.Close     7.387011e-37
#> BML.PL.Close 3.373921e-38
#> BAC.PE.Close 4.353408e-40
#> ABBV.Close   8.142840e-15
#> BML.PG.Close 5.972559e-27
#> BAC.PB.Close 1.512724e-39
#> BML.PH.Close 1.148495e-37
#> BML.PJ.Close 5.720375e-32
#> BABA.Close   9.584327e-01
#> BABAF.Close  9.915286e-01
#> AVGO.Close   6.211754e-36
#> NVO.Close    2.608217e-17
#> NONOF.Close  6.222725e-03
#> IDCBF.Close  6.026186e-06
#> IDCBY.Close  5.635984e-05
#> PEP.Close    9.403916e-42
#> MRK.Close    4.164260e-16
#> ASML.Close   2.102782e-21
#> ASMLF.Close  7.137022e-18
#> TM.Close     4.283555e-10
#> TOYOF.Close  1.000000e+00
#> RYDAF.Close  2.468392e-29
#> SHEL.Close   6.660111e-28
#> TMO.Close    1.081827e-17
#> BHP.Close    2.567945e-27
#> BHPLF.Close  4.454357e-06
#> VZ.Close     2.605012e-23
#> AZN.Close    1.370948e-12
#> COST.Close   6.450225e-13
#> AZNCF.Close  3.414503e-05
#> ABT.Close    2.771004e-30
#> NVS.Close    2.446595e-34
#> WFC.PY.Close 6.843455e-20
#> ADBE.Close   3.030550e-27
#> NVSEF.Close  2.347245e-28
#> DIS.Close    3.796324e-12
#> WFC.PL.Close 1.137063e-29
#> CMCSA.Close  1.099444e-25
#> ORCL.Close   1.339600e-13
#> DHR.Close    4.817773e-25
#> WFC.PR.Close 3.355589e-40
#> WFC.PQ.Close 1.902654e-26
#> ACN.Close    5.024203e-30
#> CSCO.Close   5.216193e-16
#> LRLCF.Close  2.238256e-27
#> LRLCY.Close  2.921135e-35
#> CICHF.Close  4.538511e-03
#> CICHY.Close  7.367629e-03
#> MCD.Close    4.305474e-27
#> NKE.Close    4.117376e-07
#> INTC.Close   4.792427e-23
#> WFC.Close    1.886590e-30
#> C.PJ.Close   7.192938e-31
#> TMUS.Close   4.916386e-08
#> PM.Close     1.502492e-46
#> AMD.Close    3.990568e-08
#> LIN.Close    1.617731e-22
#> TXN.Close    2.126171e-36
#> CRM.Close    9.968976e-01
#> BMY.Close    2.116336e-17
#> UPS.Close    2.309662e-02
#> RLLCF.Close  3.103137e-01
#> QCOM.Close   8.319828e-08
```

The following code counts the number of tickers that we could use to make a prediction, applyinf the ifelse function and combing it with the object that contains the EMH test.

```r
library(dplyr)

pred<-ifelse(ar[,1]<0.1,"Predict","No Predict")
# merge with the ar object 
ar<-cbind(ar,pred)
ar
#>                        ar       pred
#> AAPL.Close   3.731438e-17    Predict
#> MSFT.Close   6.033001e-39    Predict
#> GOOG.Close   3.224007e-16    Predict
#> GOOGL.Close  2.570860e-16    Predict
#> AMZN.Close   5.093709e-04    Predict
#> TSLA.Close   2.275599e-05    Predict
#> BRK.A.Close  9.677118e-32    Predict
#> BRK.B.Close  1.239809e-42    Predict
#> FB.Close     9.999469e-01 No Predict
#> TSM.Close    1.023778e-17    Predict
#> JNJ.Close    3.822835e-55    Predict
#> UNH.Close    4.686889e-41    Predict
#> NVDA.Close   5.391420e-16    Predict
#> V.Close      4.215814e-25    Predict
#> JPM.Close    2.665692e-25    Predict
#> TCEHY.Close  9.959273e-01 No Predict
#> TCTZF.Close  9.621425e-01 No Predict
#> XOM.Close    1.416110e-15    Predict
#> BAC.Close    2.022251e-25    Predict
#> PG.Close     3.813798e-46    Predict
#> WMT.Close    6.208759e-18    Predict
#> MA.Close     1.311943e-25    Predict
#> CVX.Close    2.928436e-36    Predict
#> JPM.PC.Close 1.177236e-31    Predict
#> JPM.PD.Close 1.802175e-39    Predict
#> NSRGY.Close  3.838131e-34    Predict
#> NSRGF.Close  9.786318e-17    Predict
#> LVMUY.Close  3.131397e-30    Predict
#> BAC.PK.Close 1.334235e-44    Predict
#> HD.Close     1.407075e-28    Predict
#> LVMHF.Close  4.802603e-23    Predict
#> BAC.PL.Close 5.073425e-39    Predict
#> LLY.Close    4.523789e-01 No Predict
#> PFE.Close    4.057538e-10    Predict
#> RHHBF.Close  3.191309e-10    Predict
#> RHHBY.Close  4.145604e-21    Predict
#> RHHVF.Close  4.923693e-22    Predict
#> KO.Close     7.387011e-37    Predict
#> BML.PL.Close 3.373921e-38    Predict
#> BAC.PE.Close 4.353408e-40    Predict
#> ABBV.Close   8.142840e-15    Predict
#> BML.PG.Close 5.972559e-27    Predict
#> BAC.PB.Close 1.512724e-39    Predict
#> BML.PH.Close 1.148495e-37    Predict
#> BML.PJ.Close 5.720375e-32    Predict
#> BABA.Close   9.584327e-01 No Predict
#> BABAF.Close  9.915286e-01 No Predict
#> AVGO.Close   6.211754e-36    Predict
#> NVO.Close    2.608217e-17    Predict
#> NONOF.Close  6.222725e-03    Predict
#> IDCBF.Close  6.026186e-06    Predict
#> IDCBY.Close  5.635984e-05    Predict
#> PEP.Close    9.403916e-42    Predict
#> MRK.Close    4.164260e-16    Predict
#> ASML.Close   2.102782e-21    Predict
#> ASMLF.Close  7.137022e-18    Predict
#> TM.Close     4.283555e-10    Predict
#> TOYOF.Close  1.000000e+00 No Predict
#> RYDAF.Close  2.468392e-29    Predict
#> SHEL.Close   6.660111e-28    Predict
#> TMO.Close    1.081827e-17    Predict
#> BHP.Close    2.567945e-27    Predict
#> BHPLF.Close  4.454357e-06    Predict
#> VZ.Close     2.605012e-23    Predict
#> AZN.Close    1.370948e-12    Predict
#> COST.Close   6.450225e-13    Predict
#> AZNCF.Close  3.414503e-05    Predict
#> ABT.Close    2.771004e-30    Predict
#> NVS.Close    2.446595e-34    Predict
#> WFC.PY.Close 6.843455e-20    Predict
#> ADBE.Close   3.030550e-27    Predict
#> NVSEF.Close  2.347245e-28    Predict
#> DIS.Close    3.796324e-12    Predict
#> WFC.PL.Close 1.137063e-29    Predict
#> CMCSA.Close  1.099444e-25    Predict
#> ORCL.Close   1.339600e-13    Predict
#> DHR.Close    4.817773e-25    Predict
#> WFC.PR.Close 3.355589e-40    Predict
#> WFC.PQ.Close 1.902654e-26    Predict
#> ACN.Close    5.024203e-30    Predict
#> CSCO.Close   5.216193e-16    Predict
#> LRLCF.Close  2.238256e-27    Predict
#> LRLCY.Close  2.921135e-35    Predict
#> CICHF.Close  4.538511e-03    Predict
#> CICHY.Close  7.367629e-03    Predict
#> MCD.Close    4.305474e-27    Predict
#> NKE.Close    4.117376e-07    Predict
#> INTC.Close   4.792427e-23    Predict
#> WFC.Close    1.886590e-30    Predict
#> C.PJ.Close   7.192938e-31    Predict
#> TMUS.Close   4.916386e-08    Predict
#> PM.Close     1.502492e-46    Predict
#> AMD.Close    3.990568e-08    Predict
#> LIN.Close    1.617731e-22    Predict
#> TXN.Close    2.126171e-36    Predict
#> CRM.Close    9.968976e-01 No Predict
#> BMY.Close    2.116336e-17    Predict
#> UPS.Close    2.309662e-02    Predict
#> RLLCF.Close  3.103137e-01 No Predict
#> QCOM.Close   8.319828e-08    Predict
```

Finally, we could filter to get only those tickers with category predict. 

arf<- df %>%
  filter(coll== "category")

```r
library(dplyr) 
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

