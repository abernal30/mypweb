# Portfolio management algorithms






For this chapter, we will take the filtered stocks from “Rational agents theory and behavioral finance theories”. As you remember, in the the previous chapter we applied the momentum strategy. 

In the file df_merge.xlsx you will find those estimations for the in_sample and out_sample. 


```r
df_merge<-read.csv("https://raw.githubusercontent.com/abernal30/BookAFP/main/data/df_merge.csv")
```


As we mention in the previous chapter, If the performance on out-sample data is pretty like in-sample data, we assume the parameters and rule set have good generalization power and can be used for live trading. In this session, we filter for those stocks that have similar out-sample and in-sample data. For that purpose, we took the difference between Sharpe ratios in the in_sample and out_sample. We also need to define a threshold of tolerance for that difference. For example, we only take stocks for which the difference is less than 20% in absolute value.  
df   %>%
  filter(Sharpe_diff < n & Sharpe_diff > -n)
n is the threshold


```r
treh<-0.2

  df_merge2<- df_merge   %>%
  filter(Sharpe_diff < treh & Sharpe_diff > -treh)
df_merge2
#>          ticker    in_return     in_sd in_Sharpe   out_return     out_sd
#> 1    AAPL.Close -0.054771548 0.2747365   -0.1994 -0.082373325 0.23299813
#> 2    AMZN.Close -0.016509915 0.2788159   -0.0592 -0.035770361 0.29049652
#> 3    TSLA.Close  0.564098218 0.5666686    0.9955  0.532664333 0.45951725
#> 4     TSM.Close -0.135791198 0.3021663   -0.4494 -0.121677232 0.22938977
#> 5     BAC.Close  0.027169757 0.3355819    0.0810 -0.011570730 0.20220638
#> 6   NSRGF.Close -0.223354678 0.1760606   -1.2686 -0.210133138 0.15186225
#> 7   LVMUY.Close -0.108547037 0.2736889   -0.3966 -0.101687079 0.24380399
#> 8  BAC.PK.Close  0.017871194 0.1272181    0.1405  0.001189315 0.06644728
#> 9   RHHBF.Close -0.667692583 0.3264243   -2.0455 -0.660057680 0.33908716
#> 10     KO.Close -0.038464295 0.2080766   -0.1849 -0.023579715 0.12021278
#> 11     TM.Close -0.091717493 0.2144692   -0.4276 -0.118928664 0.21121348
#> 12  RYDAF.Close  0.017644987 0.3660373    0.0482  0.001445751 0.20941036
#> 13   SHEL.Close -0.010895257 0.3851020   -0.0283 -0.003778604 0.17708269
#> 14    BHP.Close -0.028648261 0.3246614   -0.0882 -0.029134810 0.25596044
#> 15     VZ.Close -0.119001044 0.1588481   -0.7491 -0.102977092 0.12531984
#> 16    AZN.Close -0.207450467 0.2140546   -0.9691 -0.193951418 0.17725594
#> 17  AZNCF.Close -0.372264173 0.2180492   -1.7072 -0.346987471 0.21398241
#> 18   ADBE.Close -0.044939517 0.2979994   -0.1508 -0.049712045 0.24744804
#> 19  NVSEF.Close -0.095764445 0.2272564   -0.4214 -0.034737903 0.11387070
#> 20    DIS.Close -0.045685926 0.2775004   -0.1646 -0.079449603 0.21852916
#> 21   ORCL.Close -0.121939597 0.2788517   -0.4373 -0.095171013 0.25138906
#> 22 WFC.PR.Close -0.002421536 0.1492000   -0.0162  0.002284474 0.05670749
#> 23   TMUS.Close -0.216877202 0.2408982   -0.9003 -0.195021363 0.18670398
#>    out_Sharpe Sharpe_diff
#> 1     -0.3535      0.1541
#> 2     -0.1231      0.0639
#> 3      1.1592     -0.1637
#> 4     -0.5304      0.0810
#> 5     -0.0572      0.1382
#> 6     -1.3837      0.1151
#> 7     -0.4171      0.0205
#> 8      0.0179      0.1226
#> 9     -1.9466     -0.0989
#> 10    -0.1961      0.0112
#> 11    -0.5631      0.1355
#> 12     0.0069      0.0413
#> 13    -0.0213     -0.0070
#> 14    -0.1138      0.0256
#> 15    -0.8217      0.0726
#> 16    -1.0942      0.1251
#> 17    -1.6216     -0.0856
#> 18    -0.2009      0.0501
#> 19    -0.3051     -0.1163
#> 20    -0.3636      0.1990
#> 21    -0.3786     -0.0587
#> 22     0.0403     -0.0565
#> 23    -1.0445      0.1442
```



The momentum strategy consists of buying stocks when the instrument is trending up or selling when is down. For this case, we will order the sample by the in_Sharpe, and split the sample into 3 tranches. The first will be the stocks for taking long positions and the 3rd the one of shorts positions.    

df %>% arrange(desc(col))


```r
df_filtered<- df_merge2 %>% arrange(desc(in_Sharpe))
# to gt the names of those stocks
```
The next code is to make the split for winners


```r
co<-df_filtered[,1]
le<-length(co)
n<-round(le/3,0)  
win<-co[1:n] # long positions
n
#> [1] 8
```


The next code is to make the split for losers

```r
loss<-co[(le-n):le]
loss # short positions
#> [1] "TM.Close"    "ORCL.Close"  "TSM.Close"   "VZ.Close"    "TMUS.Close" 
#> [6] "AZN.Close"   "NSRGF.Close" "AZNCF.Close" "RHHBF.Close"
```

Finally, we combine the tranches 1 and 3 in one single object


```r
co_all<-c(win,loss)
co_all
#>  [1] "TSLA.Close"   "BAC.PK.Close" "BAC.Close"    "RYDAF.Close"  "WFC.PR.Close"
#>  [6] "SHEL.Close"   "AMZN.Close"   "BHP.Close"    "TM.Close"     "ORCL.Close"  
#> [11] "TSM.Close"    "VZ.Close"     "TMUS.Close"   "AZN.Close"    "NSRGF.Close" 
#> [16] "AZNCF.Close"  "RHHBF.Close"
```


We will generate thousands of simulations of the portfolio weights, and we need to generate aleatory numbers for the weights. For the long position, the weights must be positive and for the short position, the weight must be negative. Then, for the first tranche

The function runif will create random numbers if we apply that function to the first tranche, runif(n, 0, 1), n the number of simulations we want. We need to generate the number of random weights that are in the rend_win object.

Regarding the set.seed(42), because runif generate aleatory numbers. Then it is useful to take out the # before set.seed(42) and get the same result. After everyone gets the same results, insert the # again.

For simplicity, we will generate one portfolio weights simulation, and late we will generate more.
runif. 

```r
w<- 1.2 # long position weight
w_short<- 1-w 
set.seed(42)
#runif 
ru<-runif(n , 0, 1)
# weigths sum
su<-sum(ru)
# runif/sum and trasnsforming into data frame
we_win<-data.frame(ru*w/su)
#colnanmes weigth
colnames(we_win)<-"we"
# row names from win
rownames(we_win)<-win
```


For the short position the weights must be negative. Then, for the 3rd tranche:

```r
ru<-runif(length(loss), 0, 1)
set.seed(42)
su<-sum(ru)
# runif/sum and trasnsforming into data frame
we_loss<-data.frame(ru*w_short/su) 
#colnanmes weigth
colnames(we_loss)<-"we"
# row names from loss
rownames(we_loss)<-loss
sum(we_loss) # set.seed(42)
#> [1] -0.2
we_loss
#>                      we
#> TM.Close    -0.02150707
#> ORCL.Close  -0.02308076
#> TSM.Close   -0.01498448
#> VZ.Close    -0.02354061
#> TMUS.Close  -0.03059711
#> AZN.Close   -0.00836163
#> NSRGF.Close -0.01513346
#> AZNCF.Close -0.03077199
#> RHHBF.Close -0.03202288
```


Finally, we combine both weights.

```r
we_all<-rbind(we_win,we_loss)
we_all
#>                       we
#> TSLA.Close    0.21952864
#> BAC.PK.Close  0.22487269
#> BAC.Close     0.06866573
#> RYDAF.Close   0.19928491
#> WFC.PR.Close  0.15400152
#> SHEL.Close    0.12456895
#> AMZN.Close    0.17676122
#> BHP.Close     0.03231633
#> TM.Close     -0.02150707
#> ORCL.Close   -0.02308076
#> TSM.Close    -0.01498448
#> VZ.Close     -0.02354061
#> TMUS.Close   -0.03059711
#> AZN.Close    -0.00836163
#> NSRGF.Close  -0.01513346
#> AZNCF.Close  -0.03077199
#> RHHBF.Close  -0.03202288
```

The portfolio standard deviation is the result of the covariance multiplied by the portfolio weights. 

We estimate the covariance matrix, only for the tickers in tranches 1 and 3. For that covariance we need the returs of that tickers only.

Once we have the filtered stocks, get the returns of those stocks, taking the returns that we estimated the last session, from:


```r
data<-read.csv("https://raw.githubusercontent.com/abernal30/BookAFP/main/data/dfx_2.csv")
date<-data[,1]
data<-data[,-1]
datax<- xts(data,
         order.by = as.Date(date))
datax<-na.omit(datax)
ret<-apply(datax,2,Delt)
retx<- xts(ret,
         order.by = as.Date(date))
retx<-na.omit(retx)
head(retx[,1:5])
#>              AAPL.Close   MSFT.Close    GOOG.Close  GOOGL.Close   AMZN.Close
#> 2020-01-03 -0.009722044 -0.012451750 -0.0049072022 -0.005231342 -0.012139050
#> 2020-01-06  0.007968248  0.002584819  0.0246570974  0.026654062  0.014885590
#> 2020-01-07 -0.004703042 -0.009117758 -0.0006240057 -0.001931646  0.002091556
#> 2020-01-08  0.016086289  0.015928379  0.0078803309  0.007117757 -0.007808656
#> 2020-01-09  0.021240806  0.012492973  0.0110444988  0.010497921  0.004799272
#> 2020-01-10  0.002260711 -0.004627059  0.0069726829  0.006458647 -0.009410597
```

Also, we filter to get only the filtered stocks, from co_all


```r
retx_all<-retx[,co_all]
```

portfolio covariance
cov(df,use="complete.obs")

```r
covar<-cov(retx_all,use="complete.obs")
```

portfolio_std =cov%*% weigths
%*% para multiplicar matrices

```r
portfolio_std =covar %*% we_all[,1]
portfolio_std
#>                      [,1]
#> TSLA.Close   7.294191e-04
#> BAC.PK.Close 1.071041e-04
#> BAC.Close    3.251108e-04
#> RYDAF.Close  4.231321e-04
#> WFC.PR.Close 1.213717e-04
#> SHEL.Close   4.451261e-04
#> AMZN.Close   2.318799e-04
#> BHP.Close    3.398317e-04
#> TM.Close     1.541582e-04
#> ORCL.Close   1.497664e-04
#> TSM.Close    2.662127e-04
#> VZ.Close     6.751342e-05
#> TMUS.Close   2.029555e-04
#> AZN.Close    1.142982e-04
#> NSRGF.Close  9.421821e-05
#> AZNCF.Close  7.306948e-05
#> RHHBF.Close  8.447179e-05
```



But we need annualized the portfolio_std
twe<-t(weigths)
portfolio_std_1=(twe%*%portfolio_std*252)^.5

```r
twe<-t(we_all[,1])
portfolio_std_1=(twe%*%portfolio_std*252)^.5
```

The following code has the annualized returns of the in_sample data, only for the momentum portfolio.


```r
rownames(df_merge2)<-df_merge2[,1]
ret_a<-df_merge2[co_all,2]

ret_a_f<-twe %*%ret_a 
ret_a_f
#>           [,1]
#> [1,] 0.1818727
```


# References
