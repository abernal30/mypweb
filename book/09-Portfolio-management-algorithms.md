# Portfolio management algorithms






For this chapter, we will take the filtered stocks from “Rational agents theory and behavioral finance theories”. As you remember, in the the previous chapter we applied the momentum strategy. 

In the file df_merge.xlsx you will find those estimations for the in_sample and out_sample. 


```r
df_merge<-read.xlsx("df_merge.xlsx",rowNames=T)
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
#>                 in_return     in_sd in_Sharpe   out_return     out_sd
#> AAPL.Close   -0.054771548 0.2747365   -0.1994 -0.082373325 0.23299812
#> AMZN.Close   -0.016509915 0.2788159   -0.0592 -0.035770361 0.29049652
#> TSLA.Close    0.564098218 0.5666686    0.9955  0.532664333 0.45951725
#> TSM.Close    -0.135791198 0.3021663   -0.4494 -0.121677232 0.22938977
#> BAC.Close     0.027169757 0.3355819    0.0810 -0.011570730 0.20220638
#> NSRGF.Close  -0.223354678 0.1760606   -1.2686 -0.210133138 0.15186225
#> LVMUY.Close  -0.108547037 0.2736889   -0.3966 -0.101687079 0.24380399
#> BAC.PK.Close  0.017871194 0.1272181    0.1405  0.001189315 0.06644728
#> RHHBF.Close  -0.667692583 0.3264243   -2.0455 -0.660057680 0.33908716
#> KO.Close     -0.038464295 0.2080766   -0.1849 -0.023579715 0.12021278
#> TM.Close     -0.091717493 0.2144692   -0.4276 -0.118928664 0.21121348
#> RYDAF.Close   0.017644987 0.3660373    0.0482  0.001445751 0.20941036
#> SHEL.Close   -0.010895257 0.3851020   -0.0283 -0.003778604 0.17708269
#> BHP.Close    -0.028648261 0.3246614   -0.0882 -0.029134810 0.25596044
#> VZ.Close     -0.119001044 0.1588481   -0.7491 -0.102977092 0.12531984
#> AZN.Close    -0.207450467 0.2140546   -0.9691 -0.193951418 0.17725594
#> AZNCF.Close  -0.372264173 0.2180492   -1.7072 -0.346987471 0.21398241
#> ADBE.Close   -0.044939517 0.2979994   -0.1508 -0.049712045 0.24744804
#> NVSEF.Close  -0.095764445 0.2272564   -0.4214 -0.034737903 0.11387070
#> DIS.Close    -0.045685926 0.2775004   -0.1646 -0.079449603 0.21852916
#> ORCL.Close   -0.121939597 0.2788517   -0.4373 -0.095171013 0.25138906
#> WFC.PR.Close -0.002421536 0.1492000   -0.0162  0.002284474 0.05670749
#> TMUS.Close   -0.216877202 0.2408982   -0.9003 -0.195021363 0.18670398
#>              out_Sharpe Sharpe_diff
#> AAPL.Close      -0.3535      0.1541
#> AMZN.Close      -0.1231      0.0639
#> TSLA.Close       1.1592     -0.1637
#> TSM.Close       -0.5304      0.0810
#> BAC.Close       -0.0572      0.1382
#> NSRGF.Close     -1.3837      0.1151
#> LVMUY.Close     -0.4171      0.0205
#> BAC.PK.Close     0.0179      0.1226
#> RHHBF.Close     -1.9466     -0.0989
#> KO.Close        -0.1961      0.0112
#> TM.Close        -0.5631      0.1355
#> RYDAF.Close      0.0069      0.0413
#> SHEL.Close      -0.0213     -0.0070
#> BHP.Close       -0.1138      0.0256
#> VZ.Close        -0.8217      0.0726
#> AZN.Close       -1.0942      0.1251
#> AZNCF.Close     -1.6216     -0.0856
#> ADBE.Close      -0.2009      0.0501
#> NVSEF.Close     -0.3051     -0.1163
#> DIS.Close       -0.3636      0.1990
#> ORCL.Close      -0.3786     -0.0587
#> WFC.PR.Close     0.0403     -0.0565
#> TMUS.Close      -1.0445      0.1442
```



The momentum strategy consists of buying stocks when the instrument is trending up or selling when is down. For this case, we will order the sample by the in_Sharpe, and split the sample into 3 tranches. The first will be the stocks for taking long positions and the 3rd the one of shorts positions.    

df %>% arrange(desc(col))


```r
df_filtered<- df_merge2 %>% arrange(desc(in_Sharpe))
# to gt the names of those stocks
```
The next code is to make the split for winners


```r
co<-rownames(df_filtered)
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
data<-read.xlsx("dfx_2.xlsx")
date<-data[,1]
data<-data[,-1]
datax<- xts(data,
         order.by = as.Date(date))
datax<-na.omit(datax)
ret<-apply(datax,2,Delt)
retx<- xts(ret,
         order.by = as.Date(date))
retx<-na.omit(retx)
head(retx)
#>              AAPL.Close   MSFT.Close    GOOG.Close  GOOGL.Close   AMZN.Close
#> 2020-01-03 -0.009722044 -0.012451750 -0.0049072022 -0.005231342 -0.012139050
#> 2020-01-06  0.007968248  0.002584819  0.0246570974  0.026654062  0.014885590
#> 2020-01-07 -0.004703042 -0.009117758 -0.0006240057 -0.001931646  0.002091556
#> 2020-01-08  0.016086289  0.015928379  0.0078803309  0.007117757 -0.007808656
#> 2020-01-09  0.021240806  0.012492973  0.0110444988  0.010497921  0.004799272
#> 2020-01-10  0.002260711 -0.004627059  0.0069726829  0.006458647 -0.009410597
#>              TSLA.Close  BRK.A.Close   BRK.B.Close    TSM.Close     JNJ.Close
#> 2020-01-03  0.029633186 -0.009074946 -0.0096764570 -0.032978014 -0.0115777351
#> 2020-01-06  0.019254668  0.003110672  0.0035812717 -0.011539821 -0.0012475257
#> 2020-01-07  0.038800516 -0.003847624 -0.0047138948  0.016204931  0.0061068006
#> 2020-01-08  0.049204848  0.000846855  0.0003098752  0.007373114 -0.0001378742
#> 2020-01-09 -0.021945005  0.012013986  0.0117703834  0.008170213  0.0029662802
#> 2020-01-10 -0.006627343 -0.008966885 -0.0088781940 -0.006246817 -0.0022697641
#>               UNH.Close   NVDA.Close      V.Close     JPM.Close    XOM.Close
#> 2020-01-03 -0.010119627 -0.016005985 -0.007953061 -0.0194911055 -0.008039492
#> 2020-01-06  0.006941973  0.004193620 -0.002162468 -0.0007951424  0.007678103
#> 2020-01-07 -0.006036629  0.012106623 -0.002642846 -0.0170005865 -0.008184027
#> 2020-01-08  0.021084181  0.001875597  0.017118003  0.0078009785 -0.015080353
#> 2020-01-09 -0.005677570  0.010982611  0.006929981  0.0036512341  0.007655626
#> 2020-01-10  0.003092937  0.005349370  0.002690836 -0.0099679495 -0.008887657
#>               BAC.Close      PG.Close    WMT.Close     MA.Close    CVX.Close
#> 2020-01-03 -0.020763104 -0.0067255650 -0.008828006 -0.009756491 -0.003458766
#> 2020-01-06 -0.001432779  0.0013868331 -0.002035771  0.002662910 -0.003388183
#> 2020-01-07 -0.006599685 -0.0061914623 -0.009264802 -0.003386183 -0.012769486
#> 2020-01-08  0.010109821  0.0042626773 -0.003431658  0.016288648 -0.011422821
#> 2020-01-09  0.001715699  0.0109378498  0.010330552  0.013110455 -0.001614231
#> 2020-01-10 -0.008278533  0.0009689382 -0.008350409  0.006696884 -0.009105608
#>             JPM.PC.Close  JPM.PD.Close  NSRGY.Close  NSRGF.Close   LVMUY.Close
#> 2020-01-03  0.0035087719  0.0007236252  0.009343229  0.008883944 -0.0089095992
#> 2020-01-06 -0.0020978671 -0.0028922632  0.008065228  0.004035975 -0.0002140839
#> 2020-01-07 -0.0049054308 -0.0029006526 -0.018638022 -0.017083894 -0.0094197925
#> 2020-01-08  0.0031690141  0.0018181455 -0.014545108 -0.015057116  0.0082126863
#> 2020-01-09 -0.0007020358 -0.0003629038 -0.004324518 -0.004435227  0.0128617038
#> 2020-01-10 -0.0007024236  0.0007261438 -0.003965688  0.009573479 -0.0044444233
#>             BAC.PK.Close     HD.Close  LVMHF.Close BAC.PL.Close    PFE.Close
#> 2020-01-03  0.0003654971 -0.003323368 -0.006602660  0.006245253 -0.005365275
#> 2020-01-06 -0.0014614906  0.004704764 -0.006219153  0.002806236 -0.001284402
#> 2020-01-07 -0.0062202708 -0.006546658 -0.004860237  0.004126138 -0.003343631
#> 2020-01-08  0.0055227909  0.014964255  0.014608657  0.002705518  0.007999982
#> 2020-01-09  0.0007323691  0.015329858  0.008519702  0.002197802 -0.004352270
#> 2020-01-10  0.0021953531 -0.004307478 -0.002829981  0.006358529  0.015428132
#>             RHHBF.Close   RHHBY.Close  RHHVF.Close      KO.Close  BML.PL.Close
#> 2020-01-03 -0.016002498 -0.0058693325 -0.005822844 -0.0054555917  0.0028829900
#> 2020-01-06  0.000000000  0.0091020414  0.009093749 -0.0003657159  0.0043121150
#> 2020-01-07  0.007193082 -0.0078010483 -0.004246265 -0.0076824221 -0.0079738295
#> 2020-01-08  0.000000000  0.0034397788 -0.002822377  0.0018432811  0.0057708162
#> 2020-01-09  0.000000000  0.0004897405  0.001261392  0.0182153089  0.0028688115
#> 2020-01-10  0.000000000  0.0041604992  0.005500009  0.0034333032  0.0008173682
#>             BAC.PE.Close   ABBV.Close BML.PG.Close  BAC.PB.Close  BML.PH.Close
#> 2020-01-03  0.0036023613 -0.009491971 -0.002793296 -0.0003606563  0.0084745763
#> 2020-01-06 -0.0041828862  0.007891827  0.002334220 -0.0018037158  0.0001353408
#> 2020-01-07  0.0005265521 -0.005704731 -0.001895622 -0.0079508132 -0.0052700617
#> 2020-01-08 -0.0028559364  0.007087389  0.023365206  0.0036429143 -0.0001313937
#> 2020-01-09  0.0022504091  0.007707797  0.002416781  0.0025409075  0.0062327291
#> 2020-01-10  0.0010042457 -0.012748044  0.003502661 -0.0007241491 -0.0032649252
#>             BML.PJ.Close   AVGO.Close    NVO.Close  NONOF.Close IDCBF.Close
#> 2020-01-03 -0.0004245435 -0.025435071 -0.021418780 -0.008495163  0.00000000
#> 2020-01-06 -0.0004248088 -0.001495913 -0.001751059 -0.009617066 -0.01282051
#> 2020-01-07 -0.0029750105 -0.003442516 -0.001052394 -0.001059269 -0.01298701
#> 2020-01-08  0.0080989347 -0.012474491 -0.001755979 -0.006362690  0.00000000
#> 2020-01-09  0.0080380127 -0.008032584  0.010378206  0.026147243  0.01315789
#> 2020-01-10 -0.0002558295 -0.022987033  0.021587762  0.012133819 -0.02597403
#>              IDCBY.Close     PEP.Close    MRK.Close   ASML.Close  ASMLF.Close
#> 2020-01-03 -0.0217530390 -0.0013989250 -0.008583205 -0.016114570 -0.008484406
#> 2020-01-06 -0.0052321779  0.0038338788  0.004273991 -0.007954438 -0.018624121
#> 2020-01-07 -0.0065746220 -0.0157179515 -0.026625958  0.010860079  0.033202228
#> 2020-01-08 -0.0006618134  0.0051488846 -0.006726433  0.010375200  0.000000000
#> 2020-01-09  0.0079470199  0.0006681218  0.008803649 -0.001689390  0.000000000
#> 2020-01-10  0.0045992116 -0.0019288821  0.001678195 -0.017254005 -0.008604732
#>                 TM.Close  RYDAF.Close    SHEL.Close    TMO.Close    BHP.Close
#> 2020-01-03 -0.0104752879  0.006000000  0.0078673750 -0.010877189 -0.006737054
#> 2020-01-06  0.0001421243  0.009608979  0.0124564028  0.007186692 -0.001099853
#> 2020-01-07  0.0052567378  0.002625533 -0.0091863026  0.005659088  0.000000000
#> 2020-01-08 -0.0024732599 -0.013420622 -0.0117550162  0.001590278  0.004954964
#> 2020-01-09 -0.0046047675 -0.010948905 -0.0001675155  0.006442702 -0.009495910
#> 2020-01-10 -0.0055511994 -0.010063704 -0.0112265751 -0.002669743  0.002765431
#>             BHPLF.Close     VZ.Close    AZN.Close    COST.Close AZNCF.Close
#> 2020-01-03  0.000000000 -0.010646962 -0.005953542  0.0008234279  0.01500000
#> 2020-01-06  0.025641063 -0.002152351 -0.004192434  0.0002741816 -0.01773402
#> 2020-01-07  0.000000000 -0.011116675  0.003809122 -0.0015763408  0.00000000
#> 2020-01-08  0.000000000  0.001845654 -0.002396625  0.0114638611  0.00000000
#> 2020-01-09 -0.007720625 -0.014402965  0.002602623  0.0160508024  0.01263793
#> 2020-01-10  0.007410189  0.002378998 -0.004592731 -0.0072808193  0.00000000
#>               ABT.Close    NVS.Close  WFC.PY.Close    ADBE.Close  NVSEF.Close
#> 2020-01-03 -0.012190892 -0.001685055 -0.0007443617 -0.0078342106 -0.016350242
#> 2020-01-06  0.005239225  0.006751756 -0.0003724395  0.0057261475  0.011796225
#> 2020-01-07 -0.005559359 -0.009954909 -0.0100596125 -0.0009588445  0.006889264
#> 2020-01-08  0.004076380  0.000000000  0.0003763643  0.0134376550 -0.002947358
#> 2020-01-09  0.002667939  0.003069369  0.0011287058  0.0076361235  0.003483974
#> 2020-01-10 -0.012494227 -0.009602153  0.0030063885 -0.0018799059  0.000000000
#>                DIS.Close  WFC.PL.Close  CMCSA.Close    ORCL.Close    DHR.Close
#> 2020-01-03 -0.0114709651  0.0030911938 -0.007934781 -0.0035218350 -0.006189201
#> 2020-01-06 -0.0058020887  0.0071859787 -0.007553877  0.0052083893  0.002984152
#> 2020-01-07  0.0003433093  0.0041091328  0.005820506  0.0022205588  0.009831214
#> 2020-01-08 -0.0020590460  0.0044112657  0.010238126 -0.0005538959  0.001024710
#> 2020-01-09 -0.0039201652  0.0006756757  0.002643732  0.0046185109  0.008829772
#> 2020-01-10 -0.0014500241 -0.0020256583 -0.011645749  0.0012872379  0.002854043
#>             WFC.PR.Close WFC.PQ.Close    ACN.Close    CSCO.Close  LRLCF.Close
#> 2020-01-03  0.0031002410  0.005428954 -0.001665434 -0.0163155108  0.012540442
#> 2020-01-06 -0.0024039148 -0.003239741 -0.006530076  0.0035691370 -0.013899656
#> 2020-01-07 -0.0089500175 -0.006861719 -0.021589983 -0.0064852930 -0.015187754
#> 2020-01-08  0.0006946162  0.002181782  0.001961502  0.0006316698 -0.003638843
#> 2020-01-09  0.0024297120 -0.004716945  0.008907121 -0.0042087542  0.007826087
#> 2020-01-10  0.0000000000  0.004739300  0.007324812 -0.0040151944  0.005625557
#>             LRLCY.Close CICHF.Close  CICHY.Close    MCD.Close     NKE.Close
#> 2020-01-03 -0.006258441 -0.03448276 -0.023282225 -0.003535988 -0.0027397163
#> 2020-01-06  0.003914894  0.00000000 -0.004651163  0.011245502 -0.0008830063
#> 2020-01-07 -0.023228196  0.04761905 -0.011682301  0.001482741 -0.0004910439
#> 2020-01-08  0.007811144  0.00000000  0.002364007  0.016187134 -0.0022597367
#> 2020-01-09  0.005167068  0.01136364  0.010023585  0.011849847 -0.0006893156
#> 2020-01-10  0.002570202 -0.02247191  0.009924110 -0.005183595 -0.0057154216
#>               INTC.Close    WFC.Close    C.PJ.Close   TMUS.Close     PM.Close
#> 2020-01-03 -0.0121630835 -0.006139572  0.0052502275 -0.005344166 -0.001878190
#> 2020-01-06 -0.0028285858 -0.005990266  0.0000000000  0.005756748  0.011642926
#> 2020-01-07 -0.0166861338 -0.008286215 -0.0031337048  0.003815759  0.004417636
#> 2020-01-08  0.0006787884  0.003038359  0.0027942718  0.006335530  0.018981469
#> 2020-01-09  0.0055960318 -0.001703900 -0.0003482410  0.004910602  0.002726011
#> 2020-01-10 -0.0060708264 -0.004361843  0.0003483624 -0.010149079 -0.009288627
#>               AMD.Close    LIN.Close    TXN.Close    BMY.Close     UPS.Close
#> 2020-01-03 -0.010183300 -0.026003653 -0.013274747 -0.008841190 -0.0005993664
#> 2020-01-06 -0.004320967 -0.004238507 -0.006961275  0.003185744 -0.0044551405
#> 2020-01-07 -0.002893139  0.002152762  0.019297456  0.015084154 -0.0017211446
#> 2020-01-08 -0.008704622  0.012498154  0.002704513 -0.001094932  0.0056896897
#> 2020-01-09  0.023834392  0.007039910  0.012099315  0.024898152  0.0023143836
#> 2020-01-10 -0.016336593  0.001101250 -0.010127176 -0.003361253 -0.0090652271
#>              QCOM.Close
#> 2020-01-03 -0.018829687
#> 2020-01-06 -0.005860664
#> 2020-01-07  0.028436007
#> 2020-01-08 -0.002922356
#> 2020-01-09  0.013527280
#> 2020-01-10  0.003892759
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
ret_a<-df_merge2[co_all,1]

ret_a_f<-twe %*%ret_a 
ret_a_f
#>           [,1]
#> [1,] 0.1818727
```


### Graphs of the results

Pending

## Bibliography
Cervantes, M., Montoya, M. Á., & Bernal Ponce, L. A. (2016). Effect of the Business Cycle on Investment Strategies: Evidence from Mexico. Revista Mexicana de Economía y Finanzas, 11(2).

Hilpisch (2019). Python for Finance, 2nd Edition. O’Reilly Media, Inc. Sebastopol, CA.

