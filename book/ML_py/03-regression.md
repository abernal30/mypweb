# Training and evaluating regression models








Machine learning models such as regression aim to predict the dependent variable, "y." This chapter describes the machine learning processes to predict a time series. We will predict the ASURB.MX close price. As independent variables, we will use some macroeconomic variables. 

$$ASURB.MX\_price_{t}=\beta_{0}+\beta_{1}\ ER_{t-1} + \beta_{2}\ inflation_{t-1} + \beta_{3}\ srate_{t-1}+ \beta_{4}\ growth_{t-1}+ \beta_{5}\ mrate_{t-1}+ \beta_{6}\ inv_{t-1} + \beta_{7}\ MR_{t-1} + e_{t}$$
Where $ER$ is the exchange rate USD-MXN, $inflation$ is the Mexican inflation, $srate$ is the short term Mexican treasury bills (28 days), $growth$ is the monthly Mexican economic growth indicator, $mrate$ is the 1 year Mexican treasury bills, $inv$is the Mexican gross fixed investment, $MR$ is the Mexican stock market index and $e$ is the error term. 

## Data collection and cleaning. 

In this section, we use the APIs we cover in Chapter 1. Also, we will use other sources of information. In some cases, we will detect and clean missing values. First we download the yahoo finance information.


```python
import numpy as np
import pandas as pd
import yfinance as yf
from sie_banxico import SIEBanxico
from sklearn.metrics import mean_squared_error

import matplotlib.pyplot as plt
```




```python
# This function download market data from Yahoo! Finance's
# It returns a data frame with a specific format date. 
# Parameters:
# tickers: a list with the Yahoo Finance ticker symbol
# inter: intervals: 1m,2m,5m,15m,30m,60m,90m,1h,1d,5d,1wk,1mo,3mo
# price_type: Open', 'High', 'Low', 'Close', 'Volume', 'Dividends', 'Stock Splits
# format_date:  index format date
# d_ini: initial date if the subset
# d_fin: final date of the subset

def my_yahoo_tickers(tickers,inter,price_type,format_date,d_ini,d_fin):
    def my_yahoo(ticker,inter,price_type,format_date,d_ini,d_fin):
        x = yf.Ticker(ticker)
        hist = x.history(start=d_ini,end=d_fin,interval=inter)
        date=list(hist.index)
        hist_date=[date.strftime(format_date) for date in date]
        price=list(hist[price_type])
        hist={ticker:price}
        hist=pd.DataFrame(hist,index=hist_date)
        return hist.loc[d_ini:d_fin]
    if len(tickers)==1:
        s1=my_yahoo(tickers[0],inter,price_type,format_date,d_ini,d_fin)
    else:
        s1=my_yahoo(tickers[0],inter,price_type,format_date,d_ini,d_fin)
        for ticker in range(1,len(tickers)):
            s2=my_yahoo(tickers[ticker],inter,price_type,format_date,d_ini,d_fin)
            s1=pd.concat([s1,s2],axis=1)
    return s1
inter='1mo'
price_type="Close"
format_date="%Y-%m-%d"
d_ini="2020-02-01"
d_fin="2023-03-01"
tickers = [ticker,"^MXX"]
```


```{}
df=my_yahoo_tickers(tickers,inter,price_type,format_date,d_ini,d_fin)
df.head()
```


```{=html}
<style type="text/css">
</style>
<table id="T_c2448">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_c2448_level0_col0" class="col_heading level0 col0" >ASURB.MX</th>
      <th id="T_c2448_level0_col1" class="col_heading level0 col1" >^MXX</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_c2448_level0_row0" class="row_heading level0 row0" >2020-02-01</th>
      <td id="T_c2448_row0_col0" class="data row0 col0" >303.918579</td>
      <td id="T_c2448_row0_col1" class="data row0 col1" >41324.308594</td>
    </tr>
    <tr>
      <th id="T_c2448_level0_row1" class="row_heading level0 row1" >2020-03-01</th>
      <td id="T_c2448_row1_col0" class="data row1 col0" >208.942261</td>
      <td id="T_c2448_row1_col1" class="data row1 col1" >34554.531250</td>
    </tr>
    <tr>
      <th id="T_c2448_level0_row2" class="row_heading level0 row2" >2020-04-01</th>
      <td id="T_c2448_row2_col0" class="data row2 col0" >225.808441</td>
      <td id="T_c2448_row2_col1" class="data row2 col1" >36470.109375</td>
    </tr>
    <tr>
      <th id="T_c2448_level0_row3" class="row_heading level0 row3" >2020-05-01</th>
      <td id="T_c2448_row3_col0" class="data row3 col0" >213.797256</td>
      <td id="T_c2448_row3_col1" class="data row3 col1" >36122.730469</td>
    </tr>
    <tr>
      <th id="T_c2448_level0_row4" class="row_heading level0 row4" >2020-06-01</th>
      <td id="T_c2448_row4_col0" class="data row4 col0" >240.083420</td>
      <td id="T_c2448_row4_col1" class="data row4 col1" >37716.429688</td>
    </tr>
  </tbody>
</table>

```


Also, we download the macroeconomic variables using the Banxico API we used in chapter 1.

```{=html}
<style type="text/css">
</style>
<table id="T_2480a">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_2480a_level0_col0" class="col_heading level0 col0" >TC</th>
      <th id="T_2480a_level0_col1" class="col_heading level0 col1" >Cetes_28</th>
      <th id="T_2480a_level0_col2" class="col_heading level0 col2" >infla</th>
      <th id="T_2480a_level0_col3" class="col_heading level0 col3" >igae</th>
      <th id="T_2480a_level0_col4" class="col_heading level0 col4" >Cetes_1a</th>
      <th id="T_2480a_level0_col5" class="col_heading level0 col5" >inv_fija</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_2480a_level0_row0" class="row_heading level0 row0" >2020-02-01</th>
      <td id="T_2480a_row0_col0" class="data row0 col0" >18.844300</td>
      <td id="T_2480a_row0_col1" class="data row0 col1" >6.960000</td>
      <td id="T_2480a_row0_col2" class="data row0 col2" >0.360000</td>
      <td id="T_2480a_row0_col3" class="data row0 col3" >108.799200</td>
      <td id="T_2480a_row0_col4" class="data row0 col4" >6.680000</td>
      <td id="T_2480a_row0_col5" class="data row0 col5" >95.730000</td>
    </tr>
    <tr>
      <th id="T_2480a_level0_row1" class="row_heading level0 row1" >2020-03-01</th>
      <td id="T_2480a_row1_col0" class="data row1 col0" >22.378400</td>
      <td id="T_2480a_row1_col1" class="data row1 col1" >6.810000</td>
      <td id="T_2480a_row1_col2" class="data row1 col2" >0.290000</td>
      <td id="T_2480a_row1_col3" class="data row1 col3" >109.990100</td>
      <td id="T_2480a_row1_col4" class="data row1 col4" >6.610000</td>
      <td id="T_2480a_row1_col5" class="data row1 col5" >93.140000</td>
    </tr>
    <tr>
      <th id="T_2480a_level0_row2" class="row_heading level0 row2" >2020-04-01</th>
      <td id="T_2480a_row2_col0" class="data row2 col0" >24.265800</td>
      <td id="T_2480a_row2_col1" class="data row2 col1" >6.090000</td>
      <td id="T_2480a_row2_col2" class="data row2 col2" >0.360000</td>
      <td id="T_2480a_row2_col3" class="data row2 col3" >88.291500</td>
      <td id="T_2480a_row2_col4" class="data row2 col4" >5.800000</td>
      <td id="T_2480a_row2_col5" class="data row2 col5" >63.160000</td>
    </tr>
    <tr>
      <th id="T_2480a_level0_row3" class="row_heading level0 row3" >2020-05-01</th>
      <td id="T_2480a_row3_col0" class="data row3 col0" >23.423000</td>
      <td id="T_2480a_row3_col1" class="data row3 col1" >5.470000</td>
      <td id="T_2480a_row3_col2" class="data row3 col2" >0.300000</td>
      <td id="T_2480a_row3_col3" class="data row3 col3" >89.023800</td>
      <td id="T_2480a_row3_col4" class="data row3 col4" >5.030000</td>
      <td id="T_2480a_row3_col5" class="data row3 col5" >64.900000</td>
    </tr>
    <tr>
      <th id="T_2480a_level0_row4" class="row_heading level0 row4" >2020-06-01</th>
      <td id="T_2480a_row4_col0" class="data row4 col0" >22.299000</td>
      <td id="T_2480a_row4_col1" class="data row4 col1" >5.060000</td>
      <td id="T_2480a_row4_col2" class="data row4 col2" >0.370000</td>
      <td id="T_2480a_row4_col3" class="data row4 col3" >97.956700</td>
      <td id="T_2480a_row4_col4" class="data row4 col4" >4.910000</td>
      <td id="T_2480a_row4_col5" class="data row4 col5" >78.690000</td>
    </tr>
  </tbody>
</table>

```


```{}

#---- Do not change anything from here ----

def my_banxico_py(token,my_series,my_series_name,d_in,d_fin,format_date):
    le=len(my_series)
    ser=0
    if(le==1):
        ser=0
        api = SIEBanxico(token = token, id_series = my_series[ser])
        timeseries_range=api.get_timeseries_range(init_date=d_in, end_date=d_fin)
        timeseries_range=timeseries_range['bmx']['series'][0]['datos']
        data=pd.DataFrame(timeseries_range)
        dates=[pd.Timestamp(date).strftime(format_date) for date in list(data["fecha"])]
        data=pd.DataFrame({my_series_name[ser]:list(data["dato"])},index=dates)
    else:
        ser=0
        api = SIEBanxico(token = token, id_series = my_series[ser])
        timeseries_range=api.get_timeseries_range(init_date=d_in, end_date=d_fin)
        timeseries_range=timeseries_range['bmx']['series'][0]['datos']
        data=pd.DataFrame(timeseries_range)
        dates=[pd.Timestamp(date).strftime(format_date) for date in list(data["fecha"])]
        data=pd.DataFrame({my_series_name[ser]:list(data["dato"])},index=dates)
        for ser in range(1,le):
            api = SIEBanxico(token = token, id_series = my_series[ser])
            timeseries_range=api.get_timeseries_range(init_date=d_in, end_date=d_fin)
            timeseries_range=timeseries_range['bmx']['series'][0]['datos']
            data2=pd.DataFrame(timeseries_range)
            dates2=[pd.Timestamp(date).strftime(format_date) for date in list(data2["fecha"])]
            data2=pd.DataFrame({my_series_name[ser]:list(data2["dato"])},index=dates2)
            data=pd.concat([data,data2],axis=1)
    ban_names=list(data.columns)
    for col_i in range(data.shape[1]):
        cel_num=[float(cel) for cel in data[ban_names[col_i]]]
        data[ban_names[col_i]]=cel_num
    return data
#----- To here ------------
    
token = "776cab8e243c5b661cee8571d7e3e5d573395471011cf093ab5684d233a80d67"

my_series=['SF17908' ,'SF282',"SP74660","SR16734","SF3367","SR16525"]
my_series_name=["TC","Cetes_28","infla","igae","Cetes_1a","inv_fija"]
d_ini="2020-02-01"
d_fin="2023-02-01"
format_date="%Y-%d-%m"
ban=my_banxico_py(token,my_series,my_series_name,d_ini,d_fin,format_date)
ban.head()
```

Here we concatenate the Banxico data and the Yahoo Finance one.
```{}
dataset=pd.concat([ban,df],axis=1)
dataset.head()
```



```{=html}
<style type="text/css">
</style>
<table id="T_79936">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_79936_level0_col0" class="col_heading level0 col0" >TC</th>
      <th id="T_79936_level0_col1" class="col_heading level0 col1" >Cetes_28</th>
      <th id="T_79936_level0_col2" class="col_heading level0 col2" >infla</th>
      <th id="T_79936_level0_col3" class="col_heading level0 col3" >igae</th>
      <th id="T_79936_level0_col4" class="col_heading level0 col4" >Cetes_1a</th>
      <th id="T_79936_level0_col5" class="col_heading level0 col5" >inv_fija</th>
      <th id="T_79936_level0_col6" class="col_heading level0 col6" >ASURB.MX</th>
      <th id="T_79936_level0_col7" class="col_heading level0 col7" >^MXX</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_79936_level0_row0" class="row_heading level0 row0" >2020-02-01</th>
      <td id="T_79936_row0_col0" class="data row0 col0" >18.844300</td>
      <td id="T_79936_row0_col1" class="data row0 col1" >6.960000</td>
      <td id="T_79936_row0_col2" class="data row0 col2" >0.360000</td>
      <td id="T_79936_row0_col3" class="data row0 col3" >108.799200</td>
      <td id="T_79936_row0_col4" class="data row0 col4" >6.680000</td>
      <td id="T_79936_row0_col5" class="data row0 col5" >95.730000</td>
      <td id="T_79936_row0_col6" class="data row0 col6" >303.918579</td>
      <td id="T_79936_row0_col7" class="data row0 col7" >41324.308594</td>
    </tr>
    <tr>
      <th id="T_79936_level0_row1" class="row_heading level0 row1" >2020-03-01</th>
      <td id="T_79936_row1_col0" class="data row1 col0" >22.378400</td>
      <td id="T_79936_row1_col1" class="data row1 col1" >6.810000</td>
      <td id="T_79936_row1_col2" class="data row1 col2" >0.290000</td>
      <td id="T_79936_row1_col3" class="data row1 col3" >109.990100</td>
      <td id="T_79936_row1_col4" class="data row1 col4" >6.610000</td>
      <td id="T_79936_row1_col5" class="data row1 col5" >93.140000</td>
      <td id="T_79936_row1_col6" class="data row1 col6" >208.942261</td>
      <td id="T_79936_row1_col7" class="data row1 col7" >34554.531250</td>
    </tr>
    <tr>
      <th id="T_79936_level0_row2" class="row_heading level0 row2" >2020-04-01</th>
      <td id="T_79936_row2_col0" class="data row2 col0" >24.265800</td>
      <td id="T_79936_row2_col1" class="data row2 col1" >6.090000</td>
      <td id="T_79936_row2_col2" class="data row2 col2" >0.360000</td>
      <td id="T_79936_row2_col3" class="data row2 col3" >88.291500</td>
      <td id="T_79936_row2_col4" class="data row2 col4" >5.800000</td>
      <td id="T_79936_row2_col5" class="data row2 col5" >63.160000</td>
      <td id="T_79936_row2_col6" class="data row2 col6" >225.808441</td>
      <td id="T_79936_row2_col7" class="data row2 col7" >36470.109375</td>
    </tr>
    <tr>
      <th id="T_79936_level0_row3" class="row_heading level0 row3" >2020-05-01</th>
      <td id="T_79936_row3_col0" class="data row3 col0" >23.423000</td>
      <td id="T_79936_row3_col1" class="data row3 col1" >5.470000</td>
      <td id="T_79936_row3_col2" class="data row3 col2" >0.300000</td>
      <td id="T_79936_row3_col3" class="data row3 col3" >89.023800</td>
      <td id="T_79936_row3_col4" class="data row3 col4" >5.030000</td>
      <td id="T_79936_row3_col5" class="data row3 col5" >64.900000</td>
      <td id="T_79936_row3_col6" class="data row3 col6" >213.797256</td>
      <td id="T_79936_row3_col7" class="data row3 col7" >36122.730469</td>
    </tr>
    <tr>
      <th id="T_79936_level0_row4" class="row_heading level0 row4" >2020-06-01</th>
      <td id="T_79936_row4_col0" class="data row4 col0" >22.299000</td>
      <td id="T_79936_row4_col1" class="data row4 col1" >5.060000</td>
      <td id="T_79936_row4_col2" class="data row4 col2" >0.370000</td>
      <td id="T_79936_row4_col3" class="data row4 col3" >97.956700</td>
      <td id="T_79936_row4_col4" class="data row4 col4" >4.910000</td>
      <td id="T_79936_row4_col5" class="data row4 col5" >78.690000</td>
      <td id="T_79936_row4_col6" class="data row4 col6" >240.083420</td>
      <td id="T_79936_row4_col7" class="data row4 col7" >37716.429688</td>
    </tr>
  </tbody>
</table>

```

### Definition of the independent variable

Here we define the independent variable and do some tests. 

As we mention below, we want to predict the `r `ticker` price in one month. Then, our model state that we have to lag one period, one month in this case, the dependent variable 
$ASURB.MX\_price_{t}$. Remember that our model is:

$$ASURB.MX\_price_{t}=\beta_{0}+\beta_{1}\ ER_{t-1} + \beta_{2}\ inflation_{t-1} + \beta_{3}\ srate_{t-1}+ \beta_{4}\ growth_{t-1}+ \beta_{5}\ mrate_{t-1}+ \beta_{6}\ inv_{t-1} + \beta_{7}\ MR_{t-1} + e_{t}$$
Where the suffix $t-1$ indicates the variable one period before period $t$.


To automate the procedure, we define:

```python
var=ticker # name of the ticker we want to predict
lag=-1 # number of periods we want to lag
```

We use the function shift():
```{}
y_lag=dataset[var].shift(lag) 
y_lag.head()
```



```
#> 2020-02-01    208.942261
#> 2020-03-01    225.808441
#> 2020-04-01    213.797256
#> 2020-05-01    240.083420
#> 2020-06-01    206.921692
#> Name: ASURB.MX, dtype: float64
```

Then we insert the lagged variable in the data set, adding the text "_lag":

```{}
dataset[var+"_lag"]= y_lag
dataset.head() #
```



```{=html}
<style type="text/css">
</style>
<table id="T_9583c">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_9583c_level0_col0" class="col_heading level0 col0" >TC</th>
      <th id="T_9583c_level0_col1" class="col_heading level0 col1" >Cetes_28</th>
      <th id="T_9583c_level0_col2" class="col_heading level0 col2" >infla</th>
      <th id="T_9583c_level0_col3" class="col_heading level0 col3" >igae</th>
      <th id="T_9583c_level0_col4" class="col_heading level0 col4" >Cetes_1a</th>
      <th id="T_9583c_level0_col5" class="col_heading level0 col5" >inv_fija</th>
      <th id="T_9583c_level0_col6" class="col_heading level0 col6" >ASURB.MX</th>
      <th id="T_9583c_level0_col7" class="col_heading level0 col7" >^MXX</th>
      <th id="T_9583c_level0_col8" class="col_heading level0 col8" >ASURB.MX_lag</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_9583c_level0_row0" class="row_heading level0 row0" >2020-02-01</th>
      <td id="T_9583c_row0_col0" class="data row0 col0" >18.844300</td>
      <td id="T_9583c_row0_col1" class="data row0 col1" >6.960000</td>
      <td id="T_9583c_row0_col2" class="data row0 col2" >0.360000</td>
      <td id="T_9583c_row0_col3" class="data row0 col3" >108.799200</td>
      <td id="T_9583c_row0_col4" class="data row0 col4" >6.680000</td>
      <td id="T_9583c_row0_col5" class="data row0 col5" >95.730000</td>
      <td id="T_9583c_row0_col6" class="data row0 col6" >303.918579</td>
      <td id="T_9583c_row0_col7" class="data row0 col7" >41324.308594</td>
      <td id="T_9583c_row0_col8" class="data row0 col8" >208.942261</td>
    </tr>
    <tr>
      <th id="T_9583c_level0_row1" class="row_heading level0 row1" >2020-03-01</th>
      <td id="T_9583c_row1_col0" class="data row1 col0" >22.378400</td>
      <td id="T_9583c_row1_col1" class="data row1 col1" >6.810000</td>
      <td id="T_9583c_row1_col2" class="data row1 col2" >0.290000</td>
      <td id="T_9583c_row1_col3" class="data row1 col3" >109.990100</td>
      <td id="T_9583c_row1_col4" class="data row1 col4" >6.610000</td>
      <td id="T_9583c_row1_col5" class="data row1 col5" >93.140000</td>
      <td id="T_9583c_row1_col6" class="data row1 col6" >208.942261</td>
      <td id="T_9583c_row1_col7" class="data row1 col7" >34554.531250</td>
      <td id="T_9583c_row1_col8" class="data row1 col8" >225.808441</td>
    </tr>
    <tr>
      <th id="T_9583c_level0_row2" class="row_heading level0 row2" >2020-04-01</th>
      <td id="T_9583c_row2_col0" class="data row2 col0" >24.265800</td>
      <td id="T_9583c_row2_col1" class="data row2 col1" >6.090000</td>
      <td id="T_9583c_row2_col2" class="data row2 col2" >0.360000</td>
      <td id="T_9583c_row2_col3" class="data row2 col3" >88.291500</td>
      <td id="T_9583c_row2_col4" class="data row2 col4" >5.800000</td>
      <td id="T_9583c_row2_col5" class="data row2 col5" >63.160000</td>
      <td id="T_9583c_row2_col6" class="data row2 col6" >225.808441</td>
      <td id="T_9583c_row2_col7" class="data row2 col7" >36470.109375</td>
      <td id="T_9583c_row2_col8" class="data row2 col8" >213.797256</td>
    </tr>
    <tr>
      <th id="T_9583c_level0_row3" class="row_heading level0 row3" >2020-05-01</th>
      <td id="T_9583c_row3_col0" class="data row3 col0" >23.423000</td>
      <td id="T_9583c_row3_col1" class="data row3 col1" >5.470000</td>
      <td id="T_9583c_row3_col2" class="data row3 col2" >0.300000</td>
      <td id="T_9583c_row3_col3" class="data row3 col3" >89.023800</td>
      <td id="T_9583c_row3_col4" class="data row3 col4" >5.030000</td>
      <td id="T_9583c_row3_col5" class="data row3 col5" >64.900000</td>
      <td id="T_9583c_row3_col6" class="data row3 col6" >213.797256</td>
      <td id="T_9583c_row3_col7" class="data row3 col7" >36122.730469</td>
      <td id="T_9583c_row3_col8" class="data row3 col8" >240.083420</td>
    </tr>
    <tr>
      <th id="T_9583c_level0_row4" class="row_heading level0 row4" >2020-06-01</th>
      <td id="T_9583c_row4_col0" class="data row4 col0" >22.299000</td>
      <td id="T_9583c_row4_col1" class="data row4 col1" >5.060000</td>
      <td id="T_9583c_row4_col2" class="data row4 col2" >0.370000</td>
      <td id="T_9583c_row4_col3" class="data row4 col3" >97.956700</td>
      <td id="T_9583c_row4_col4" class="data row4 col4" >4.910000</td>
      <td id="T_9583c_row4_col5" class="data row4 col5" >78.690000</td>
      <td id="T_9583c_row4_col6" class="data row4 col6" >240.083420</td>
      <td id="T_9583c_row4_col7" class="data row4 col7" >37716.429688</td>
      <td id="T_9583c_row4_col8" class="data row4 col8" >206.921692</td>
    </tr>
  </tbody>
</table>

```

As we see, the last row has a missing value, the one we lost by doing the lag. 
```{}
dataset.tail()
```


```{=html}
<style type="text/css">
</style>
<table id="T_4ddb7">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_4ddb7_level0_col0" class="col_heading level0 col0" >TC</th>
      <th id="T_4ddb7_level0_col1" class="col_heading level0 col1" >Cetes_28</th>
      <th id="T_4ddb7_level0_col2" class="col_heading level0 col2" >infla</th>
      <th id="T_4ddb7_level0_col3" class="col_heading level0 col3" >igae</th>
      <th id="T_4ddb7_level0_col4" class="col_heading level0 col4" >Cetes_1a</th>
      <th id="T_4ddb7_level0_col5" class="col_heading level0 col5" >inv_fija</th>
      <th id="T_4ddb7_level0_col6" class="col_heading level0 col6" >ASURB.MX</th>
      <th id="T_4ddb7_level0_col7" class="col_heading level0 col7" >^MXX</th>
      <th id="T_4ddb7_level0_col8" class="col_heading level0 col8" >ASURB.MX_lag</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_4ddb7_level0_row0" class="row_heading level0 row0" >2022-10-01</th>
      <td id="T_4ddb7_row0_col0" class="data row0 col0" >19.984500</td>
      <td id="T_4ddb7_row0_col1" class="data row0 col1" >8.930000</td>
      <td id="T_4ddb7_row0_col2" class="data row0 col2" >0.630000</td>
      <td id="T_4ddb7_row0_col3" class="data row0 col3" >113.121300</td>
      <td id="T_4ddb7_row0_col4" class="data row0 col4" >10.770000</td>
      <td id="T_4ddb7_row0_col5" class="data row0 col5" >100.900000</td>
      <td id="T_4ddb7_row0_col6" class="data row0 col6" >464.970001</td>
      <td id="T_4ddb7_row0_col7" class="data row0 col7" >49922.300781</td>
      <td id="T_4ddb7_row0_col8" class="data row0 col8" >478.500000</td>
    </tr>
    <tr>
      <th id="T_4ddb7_level0_row1" class="row_heading level0 row1" >2022-11-01</th>
      <td id="T_4ddb7_row1_col0" class="data row1 col0" >19.444900</td>
      <td id="T_4ddb7_row1_col1" class="data row1 col1" >9.420000</td>
      <td id="T_4ddb7_row1_col2" class="data row1 col2" >0.450000</td>
      <td id="T_4ddb7_row1_col3" class="data row1 col3" >116.340600</td>
      <td id="T_4ddb7_row1_col4" class="data row1 col4" >10.840000</td>
      <td id="T_4ddb7_row1_col5" class="data row1 col5" >103.440000</td>
      <td id="T_4ddb7_row1_col6" class="data row1 col6" >478.500000</td>
      <td id="T_4ddb7_row1_col7" class="data row1 col7" >51684.859375</td>
      <td id="T_4ddb7_row1_col8" class="data row1 col8" >454.010010</td>
    </tr>
    <tr>
      <th id="T_4ddb7_level0_row2" class="row_heading level0 row2" >2022-12-01</th>
      <td id="T_4ddb7_row2_col0" class="data row2 col0" >19.593000</td>
      <td id="T_4ddb7_row2_col1" class="data row2 col1" >9.960000</td>
      <td id="T_4ddb7_row2_col2" class="data row2 col2" >0.650000</td>
      <td id="T_4ddb7_row2_col3" class="data row2 col3" >115.577100</td>
      <td id="T_4ddb7_row2_col4" class="data row2 col4" >10.870000</td>
      <td id="T_4ddb7_row2_col5" class="data row2 col5" >107.190000</td>
      <td id="T_4ddb7_row2_col6" class="data row2 col6" >454.010010</td>
      <td id="T_4ddb7_row2_col7" class="data row2 col7" >48463.859375</td>
      <td id="T_4ddb7_row2_col8" class="data row2 col8" >510.230011</td>
    </tr>
    <tr>
      <th id="T_4ddb7_level0_row3" class="row_heading level0 row3" >2023-01-01</th>
      <td id="T_4ddb7_row3_col0" class="data row3 col0" >18.986300</td>
      <td id="T_4ddb7_row3_col1" class="data row3 col1" >10.610000</td>
      <td id="T_4ddb7_row3_col2" class="data row3 col2" >0.710000</td>
      <td id="T_4ddb7_row3_col3" class="data row3 col3" >112.093900</td>
      <td id="T_4ddb7_row3_col4" class="data row3 col4" >11.090000</td>
      <td id="T_4ddb7_row3_col5" class="data row3 col5" >106.030000</td>
      <td id="T_4ddb7_row3_col6" class="data row3 col6" >510.230011</td>
      <td id="T_4ddb7_row3_col7" class="data row3 col7" >54564.269531</td>
      <td id="T_4ddb7_row3_col8" class="data row3 col8" >523.859985</td>
    </tr>
    <tr>
      <th id="T_4ddb7_level0_row4" class="row_heading level0 row4" >2023-02-01</th>
      <td id="T_4ddb7_row4_col0" class="data row4 col0" >18.598600</td>
      <td id="T_4ddb7_row4_col1" class="data row4 col1" >10.920000</td>
      <td id="T_4ddb7_row4_col2" class="data row4 col2" >0.610000</td>
      <td id="T_4ddb7_row4_col3" class="data row4 col3" >109.613300</td>
      <td id="T_4ddb7_row4_col4" class="data row4 col4" >11.660000</td>
      <td id="T_4ddb7_row4_col5" class="data row4 col5" >105.420000</td>
      <td id="T_4ddb7_row4_col6" class="data row4 col6" >523.859985</td>
      <td id="T_4ddb7_row4_col7" class="data row4 col7" >52758.058594</td>
      <td id="T_4ddb7_row4_col8" class="data row4 col8" >nan</td>
    </tr>
  </tbody>
</table>

```

That missing value would´t allow us to perform some procedures below. Then we cut it, using the method shape, which shows the number of rows and columns of a data frame.





in this case the data set has 37 rows and 9 columns.   

```{}
dataset_lag=dataset.iloc[:dataset.shape[0]-1,]
dataset_lag.head()
```


```{=html}
<style type="text/css">
</style>
<table id="T_6faca">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_6faca_level0_col0" class="col_heading level0 col0" >TC</th>
      <th id="T_6faca_level0_col1" class="col_heading level0 col1" >Cetes_28</th>
      <th id="T_6faca_level0_col2" class="col_heading level0 col2" >infla</th>
      <th id="T_6faca_level0_col3" class="col_heading level0 col3" >igae</th>
      <th id="T_6faca_level0_col4" class="col_heading level0 col4" >Cetes_1a</th>
      <th id="T_6faca_level0_col5" class="col_heading level0 col5" >inv_fija</th>
      <th id="T_6faca_level0_col6" class="col_heading level0 col6" >ASURB.MX</th>
      <th id="T_6faca_level0_col7" class="col_heading level0 col7" >^MXX</th>
      <th id="T_6faca_level0_col8" class="col_heading level0 col8" >ASURB.MX_lag</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_6faca_level0_row0" class="row_heading level0 row0" >2020-02-01</th>
      <td id="T_6faca_row0_col0" class="data row0 col0" >18.844300</td>
      <td id="T_6faca_row0_col1" class="data row0 col1" >6.960000</td>
      <td id="T_6faca_row0_col2" class="data row0 col2" >0.360000</td>
      <td id="T_6faca_row0_col3" class="data row0 col3" >108.799200</td>
      <td id="T_6faca_row0_col4" class="data row0 col4" >6.680000</td>
      <td id="T_6faca_row0_col5" class="data row0 col5" >95.730000</td>
      <td id="T_6faca_row0_col6" class="data row0 col6" >303.918579</td>
      <td id="T_6faca_row0_col7" class="data row0 col7" >41324.308594</td>
      <td id="T_6faca_row0_col8" class="data row0 col8" >208.942261</td>
    </tr>
    <tr>
      <th id="T_6faca_level0_row1" class="row_heading level0 row1" >2020-03-01</th>
      <td id="T_6faca_row1_col0" class="data row1 col0" >22.378400</td>
      <td id="T_6faca_row1_col1" class="data row1 col1" >6.810000</td>
      <td id="T_6faca_row1_col2" class="data row1 col2" >0.290000</td>
      <td id="T_6faca_row1_col3" class="data row1 col3" >109.990100</td>
      <td id="T_6faca_row1_col4" class="data row1 col4" >6.610000</td>
      <td id="T_6faca_row1_col5" class="data row1 col5" >93.140000</td>
      <td id="T_6faca_row1_col6" class="data row1 col6" >208.942261</td>
      <td id="T_6faca_row1_col7" class="data row1 col7" >34554.531250</td>
      <td id="T_6faca_row1_col8" class="data row1 col8" >225.808441</td>
    </tr>
    <tr>
      <th id="T_6faca_level0_row2" class="row_heading level0 row2" >2020-04-01</th>
      <td id="T_6faca_row2_col0" class="data row2 col0" >24.265800</td>
      <td id="T_6faca_row2_col1" class="data row2 col1" >6.090000</td>
      <td id="T_6faca_row2_col2" class="data row2 col2" >0.360000</td>
      <td id="T_6faca_row2_col3" class="data row2 col3" >88.291500</td>
      <td id="T_6faca_row2_col4" class="data row2 col4" >5.800000</td>
      <td id="T_6faca_row2_col5" class="data row2 col5" >63.160000</td>
      <td id="T_6faca_row2_col6" class="data row2 col6" >225.808441</td>
      <td id="T_6faca_row2_col7" class="data row2 col7" >36470.109375</td>
      <td id="T_6faca_row2_col8" class="data row2 col8" >213.797256</td>
    </tr>
    <tr>
      <th id="T_6faca_level0_row3" class="row_heading level0 row3" >2020-05-01</th>
      <td id="T_6faca_row3_col0" class="data row3 col0" >23.423000</td>
      <td id="T_6faca_row3_col1" class="data row3 col1" >5.470000</td>
      <td id="T_6faca_row3_col2" class="data row3 col2" >0.300000</td>
      <td id="T_6faca_row3_col3" class="data row3 col3" >89.023800</td>
      <td id="T_6faca_row3_col4" class="data row3 col4" >5.030000</td>
      <td id="T_6faca_row3_col5" class="data row3 col5" >64.900000</td>
      <td id="T_6faca_row3_col6" class="data row3 col6" >213.797256</td>
      <td id="T_6faca_row3_col7" class="data row3 col7" >36122.730469</td>
      <td id="T_6faca_row3_col8" class="data row3 col8" >240.083420</td>
    </tr>
    <tr>
      <th id="T_6faca_level0_row4" class="row_heading level0 row4" >2020-06-01</th>
      <td id="T_6faca_row4_col0" class="data row4 col0" >22.299000</td>
      <td id="T_6faca_row4_col1" class="data row4 col1" >5.060000</td>
      <td id="T_6faca_row4_col2" class="data row4 col2" >0.370000</td>
      <td id="T_6faca_row4_col3" class="data row4 col3" >97.956700</td>
      <td id="T_6faca_row4_col4" class="data row4 col4" >4.910000</td>
      <td id="T_6faca_row4_col5" class="data row4 col5" >78.690000</td>
      <td id="T_6faca_row4_col6" class="data row4 col6" >240.083420</td>
      <td id="T_6faca_row4_col7" class="data row4 col7" >37716.429688</td>
      <td id="T_6faca_row4_col8" class="data row4 col8" >206.921692</td>
    </tr>
  </tbody>
</table>

```

Here we define the dependent variable (Y):

```python
Y=dataset_lag[var+"_lag"] # esto es el resultado, pero quiero automatizarlo
Y.head()
#> 2020-02-01    208.942261
#> 2020-03-01    225.808441
#> 2020-04-01    213.797256
#> 2020-05-01    240.083420
#> 2020-06-01    206.921692
#> Name: ASURB.MX_lag, dtype: float64
```

### Testing for stationary in the dependet variable (Y)

A stationary time series process is one whose probability distributions are stable over time in the following sense: If we take any collection of random variables in the sequence and then shift that sequence ahead h time periods, the joint probability distribution must remain unchanged.

On a practical level, if we want to understand the relationship between two or more variables using regression analysis, such as the model: 

$$ASURB.MX\_price_{t}=\beta_{0}+\beta_{1}\ ER_{t-1} + \beta_{2}\ inflation_{t-1} +.. + e_{t},$$

we need to assume some sort of stability over time. If we allow the relationship between two variables those variables in the equation to change arbitrarily in each time period, then we cannot hope to learn much about how a change in one variable affects the other variable if we only have access to a single time series realization [@wooldridge]. In the Appendix we extend this explanation.

Because this is an introductory machine learning book, we only would apply the Augmented Dickey-Fuller (ADF) (see [@wooldridge] for a further reading). 



```python
# This method shows the ADF statistic and P-value 
# Parameters:
# data: is a data frame of time Pandas Series with the time serie 
#---- Do not change anything from here ----

from statsmodels.tsa.stattools import adfuller
def adf(data):
    ADF = adfuller(data)  
    return print(f'ADF Statistic: {ADF[0]}',f'p-value: {ADF[1]}')
#----- To here ------------
adf(Y)
#> ADF Statistic: -1.9361997214419064 p-value: 0.31522784631900724
```

If the second term of the output, the P-value, is less than 10% (0.1), then we can perform the prediction with that time series. If it is greater than 10%(0.1), we have to adjust, such as the change of the variable.

in this case, the P-value is less than 10%(0.1), then the time series is not stationary, then we have to do an adjustment. The adjustment we will do is to do the change in the independent varible:

$$\Delta ASURB.MX\_price_{t}=\beta_{0}+\beta_{1}\ ER_{t-1} + \beta_{2}\ inflation_{t-1} +.. + e_{t},$$
The $\Delta ASURB.MX$ means the change in the variable, $ASURB.MX\_price_{t}-ASURB.MX\_price_{t-1}$. To do that we use the method diff. First we need to do a copy of the 
dataset_lag, other wise we could get a warning about the method. 
```{}
dataset_lag_dif=dataset_lag.copy() 
dataset_lag_dif[var+"_lag_dif"]=Y.diff()
dataset_lag_dif.head()
```


```{=html}
<style type="text/css">
</style>
<table id="T_590c9">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_590c9_level0_col0" class="col_heading level0 col0" >TC</th>
      <th id="T_590c9_level0_col1" class="col_heading level0 col1" >Cetes_28</th>
      <th id="T_590c9_level0_col2" class="col_heading level0 col2" >infla</th>
      <th id="T_590c9_level0_col3" class="col_heading level0 col3" >igae</th>
      <th id="T_590c9_level0_col4" class="col_heading level0 col4" >Cetes_1a</th>
      <th id="T_590c9_level0_col5" class="col_heading level0 col5" >inv_fija</th>
      <th id="T_590c9_level0_col6" class="col_heading level0 col6" >ASURB.MX</th>
      <th id="T_590c9_level0_col7" class="col_heading level0 col7" >^MXX</th>
      <th id="T_590c9_level0_col8" class="col_heading level0 col8" >ASURB.MX_lag</th>
      <th id="T_590c9_level0_col9" class="col_heading level0 col9" >ASURB.MX_lag_dif</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_590c9_level0_row0" class="row_heading level0 row0" >2020-02-01</th>
      <td id="T_590c9_row0_col0" class="data row0 col0" >18.844300</td>
      <td id="T_590c9_row0_col1" class="data row0 col1" >6.960000</td>
      <td id="T_590c9_row0_col2" class="data row0 col2" >0.360000</td>
      <td id="T_590c9_row0_col3" class="data row0 col3" >108.799200</td>
      <td id="T_590c9_row0_col4" class="data row0 col4" >6.680000</td>
      <td id="T_590c9_row0_col5" class="data row0 col5" >95.730000</td>
      <td id="T_590c9_row0_col6" class="data row0 col6" >303.918579</td>
      <td id="T_590c9_row0_col7" class="data row0 col7" >41324.308594</td>
      <td id="T_590c9_row0_col8" class="data row0 col8" >208.942261</td>
      <td id="T_590c9_row0_col9" class="data row0 col9" >nan</td>
    </tr>
    <tr>
      <th id="T_590c9_level0_row1" class="row_heading level0 row1" >2020-03-01</th>
      <td id="T_590c9_row1_col0" class="data row1 col0" >22.378400</td>
      <td id="T_590c9_row1_col1" class="data row1 col1" >6.810000</td>
      <td id="T_590c9_row1_col2" class="data row1 col2" >0.290000</td>
      <td id="T_590c9_row1_col3" class="data row1 col3" >109.990100</td>
      <td id="T_590c9_row1_col4" class="data row1 col4" >6.610000</td>
      <td id="T_590c9_row1_col5" class="data row1 col5" >93.140000</td>
      <td id="T_590c9_row1_col6" class="data row1 col6" >208.942261</td>
      <td id="T_590c9_row1_col7" class="data row1 col7" >34554.531250</td>
      <td id="T_590c9_row1_col8" class="data row1 col8" >225.808441</td>
      <td id="T_590c9_row1_col9" class="data row1 col9" >16.866180</td>
    </tr>
    <tr>
      <th id="T_590c9_level0_row2" class="row_heading level0 row2" >2020-04-01</th>
      <td id="T_590c9_row2_col0" class="data row2 col0" >24.265800</td>
      <td id="T_590c9_row2_col1" class="data row2 col1" >6.090000</td>
      <td id="T_590c9_row2_col2" class="data row2 col2" >0.360000</td>
      <td id="T_590c9_row2_col3" class="data row2 col3" >88.291500</td>
      <td id="T_590c9_row2_col4" class="data row2 col4" >5.800000</td>
      <td id="T_590c9_row2_col5" class="data row2 col5" >63.160000</td>
      <td id="T_590c9_row2_col6" class="data row2 col6" >225.808441</td>
      <td id="T_590c9_row2_col7" class="data row2 col7" >36470.109375</td>
      <td id="T_590c9_row2_col8" class="data row2 col8" >213.797256</td>
      <td id="T_590c9_row2_col9" class="data row2 col9" >-12.011185</td>
    </tr>
    <tr>
      <th id="T_590c9_level0_row3" class="row_heading level0 row3" >2020-05-01</th>
      <td id="T_590c9_row3_col0" class="data row3 col0" >23.423000</td>
      <td id="T_590c9_row3_col1" class="data row3 col1" >5.470000</td>
      <td id="T_590c9_row3_col2" class="data row3 col2" >0.300000</td>
      <td id="T_590c9_row3_col3" class="data row3 col3" >89.023800</td>
      <td id="T_590c9_row3_col4" class="data row3 col4" >5.030000</td>
      <td id="T_590c9_row3_col5" class="data row3 col5" >64.900000</td>
      <td id="T_590c9_row3_col6" class="data row3 col6" >213.797256</td>
      <td id="T_590c9_row3_col7" class="data row3 col7" >36122.730469</td>
      <td id="T_590c9_row3_col8" class="data row3 col8" >240.083420</td>
      <td id="T_590c9_row3_col9" class="data row3 col9" >26.286163</td>
    </tr>
    <tr>
      <th id="T_590c9_level0_row4" class="row_heading level0 row4" >2020-06-01</th>
      <td id="T_590c9_row4_col0" class="data row4 col0" >22.299000</td>
      <td id="T_590c9_row4_col1" class="data row4 col1" >5.060000</td>
      <td id="T_590c9_row4_col2" class="data row4 col2" >0.370000</td>
      <td id="T_590c9_row4_col3" class="data row4 col3" >97.956700</td>
      <td id="T_590c9_row4_col4" class="data row4 col4" >4.910000</td>
      <td id="T_590c9_row4_col5" class="data row4 col5" >78.690000</td>
      <td id="T_590c9_row4_col6" class="data row4 col6" >240.083420</td>
      <td id="T_590c9_row4_col7" class="data row4 col7" >37716.429688</td>
      <td id="T_590c9_row4_col8" class="data row4 col8" >206.921692</td>
      <td id="T_590c9_row4_col9" class="data row4 col9" >-33.161728</td>
    </tr>
  </tbody>
</table>

```

As we see the lost one observation at the begining, then we cut that row. 

```{}
dataset_lag_dif=dataset_lag_dif.iloc[1:,]
dataset_lag_dif.head()
```



```{=html}
<style type="text/css">
</style>
<table id="T_cb46e">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_cb46e_level0_col0" class="col_heading level0 col0" >TC</th>
      <th id="T_cb46e_level0_col1" class="col_heading level0 col1" >Cetes_28</th>
      <th id="T_cb46e_level0_col2" class="col_heading level0 col2" >infla</th>
      <th id="T_cb46e_level0_col3" class="col_heading level0 col3" >igae</th>
      <th id="T_cb46e_level0_col4" class="col_heading level0 col4" >Cetes_1a</th>
      <th id="T_cb46e_level0_col5" class="col_heading level0 col5" >inv_fija</th>
      <th id="T_cb46e_level0_col6" class="col_heading level0 col6" >ASURB.MX</th>
      <th id="T_cb46e_level0_col7" class="col_heading level0 col7" >^MXX</th>
      <th id="T_cb46e_level0_col8" class="col_heading level0 col8" >ASURB.MX_lag</th>
      <th id="T_cb46e_level0_col9" class="col_heading level0 col9" >ASURB.MX_lag_dif</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_cb46e_level0_row0" class="row_heading level0 row0" >2020-03-01</th>
      <td id="T_cb46e_row0_col0" class="data row0 col0" >22.378400</td>
      <td id="T_cb46e_row0_col1" class="data row0 col1" >6.810000</td>
      <td id="T_cb46e_row0_col2" class="data row0 col2" >0.290000</td>
      <td id="T_cb46e_row0_col3" class="data row0 col3" >109.990100</td>
      <td id="T_cb46e_row0_col4" class="data row0 col4" >6.610000</td>
      <td id="T_cb46e_row0_col5" class="data row0 col5" >93.140000</td>
      <td id="T_cb46e_row0_col6" class="data row0 col6" >208.942261</td>
      <td id="T_cb46e_row0_col7" class="data row0 col7" >34554.531250</td>
      <td id="T_cb46e_row0_col8" class="data row0 col8" >225.808441</td>
      <td id="T_cb46e_row0_col9" class="data row0 col9" >16.866180</td>
    </tr>
    <tr>
      <th id="T_cb46e_level0_row1" class="row_heading level0 row1" >2020-04-01</th>
      <td id="T_cb46e_row1_col0" class="data row1 col0" >24.265800</td>
      <td id="T_cb46e_row1_col1" class="data row1 col1" >6.090000</td>
      <td id="T_cb46e_row1_col2" class="data row1 col2" >0.360000</td>
      <td id="T_cb46e_row1_col3" class="data row1 col3" >88.291500</td>
      <td id="T_cb46e_row1_col4" class="data row1 col4" >5.800000</td>
      <td id="T_cb46e_row1_col5" class="data row1 col5" >63.160000</td>
      <td id="T_cb46e_row1_col6" class="data row1 col6" >225.808441</td>
      <td id="T_cb46e_row1_col7" class="data row1 col7" >36470.109375</td>
      <td id="T_cb46e_row1_col8" class="data row1 col8" >213.797256</td>
      <td id="T_cb46e_row1_col9" class="data row1 col9" >-12.011185</td>
    </tr>
    <tr>
      <th id="T_cb46e_level0_row2" class="row_heading level0 row2" >2020-05-01</th>
      <td id="T_cb46e_row2_col0" class="data row2 col0" >23.423000</td>
      <td id="T_cb46e_row2_col1" class="data row2 col1" >5.470000</td>
      <td id="T_cb46e_row2_col2" class="data row2 col2" >0.300000</td>
      <td id="T_cb46e_row2_col3" class="data row2 col3" >89.023800</td>
      <td id="T_cb46e_row2_col4" class="data row2 col4" >5.030000</td>
      <td id="T_cb46e_row2_col5" class="data row2 col5" >64.900000</td>
      <td id="T_cb46e_row2_col6" class="data row2 col6" >213.797256</td>
      <td id="T_cb46e_row2_col7" class="data row2 col7" >36122.730469</td>
      <td id="T_cb46e_row2_col8" class="data row2 col8" >240.083420</td>
      <td id="T_cb46e_row2_col9" class="data row2 col9" >26.286163</td>
    </tr>
    <tr>
      <th id="T_cb46e_level0_row3" class="row_heading level0 row3" >2020-06-01</th>
      <td id="T_cb46e_row3_col0" class="data row3 col0" >22.299000</td>
      <td id="T_cb46e_row3_col1" class="data row3 col1" >5.060000</td>
      <td id="T_cb46e_row3_col2" class="data row3 col2" >0.370000</td>
      <td id="T_cb46e_row3_col3" class="data row3 col3" >97.956700</td>
      <td id="T_cb46e_row3_col4" class="data row3 col4" >4.910000</td>
      <td id="T_cb46e_row3_col5" class="data row3 col5" >78.690000</td>
      <td id="T_cb46e_row3_col6" class="data row3 col6" >240.083420</td>
      <td id="T_cb46e_row3_col7" class="data row3 col7" >37716.429688</td>
      <td id="T_cb46e_row3_col8" class="data row3 col8" >206.921692</td>
      <td id="T_cb46e_row3_col9" class="data row3 col9" >-33.161728</td>
    </tr>
    <tr>
      <th id="T_cb46e_level0_row4" class="row_heading level0 row4" >2020-07-01</th>
      <td id="T_cb46e_row4_col0" class="data row4 col0" >22.403300</td>
      <td id="T_cb46e_row4_col1" class="data row4 col1" >4.820000</td>
      <td id="T_cb46e_row4_col2" class="data row4 col2" >0.400000</td>
      <td id="T_cb46e_row4_col3" class="data row4 col3" >102.413100</td>
      <td id="T_cb46e_row4_col4" class="data row4 col4" >4.610000</td>
      <td id="T_cb46e_row4_col5" class="data row4 col5" >80.340000</td>
      <td id="T_cb46e_row4_col6" class="data row4 col6" >206.921692</td>
      <td id="T_cb46e_row4_col7" class="data row4 col7" >37019.679688</td>
      <td id="T_cb46e_row4_col8" class="data row4 col8" >231.973053</td>
      <td id="T_cb46e_row4_col9" class="data row4 col9" >25.051361</td>
    </tr>
  </tbody>
</table>

```


Again we define the Y variable:


```python
Y=dataset_lag_dif[var+"_lag_dif"]
Y.head()
#> 2020-03-01    16.866180
#> 2020-04-01   -12.011185
#> 2020-05-01    26.286163
#> 2020-06-01   -33.161728
#> 2020-07-01    25.051361
#> Name: ASURB.MX_lag_dif, dtype: float64
```

Now we make a subset to get the independent variables (X´s): 

```python
X=dataset_lag_dif.drop(columns=[var,var+"_lag",var+"_lag_dif"],axis=1)
X.head()
#>                  TC  Cetes_28  infla  ...  Cetes_1a  inv_fija          ^MXX
#> 2020-03-01  22.3784      6.81   0.29  ...      6.61     93.14  34554.531250
#> 2020-04-01  24.2658      6.09   0.36  ...      5.80     63.16  36470.109375
#> 2020-05-01  23.4230      5.47   0.30  ...      5.03     64.90  36122.730469
#> 2020-06-01  22.2990      5.06   0.37  ...      4.91     78.69  37716.429688
#> 2020-07-01  22.4033      4.82   0.40  ...      4.61     80.34  37019.679688
#> 
#> [5 rows x 7 columns]
```

## Training and test set (Back testing)

In the machine learning literature is common to apply that testing procedure for several observations. We call this back-testing, which divides the data set into training and testing, often called in_sample and out_sample. 

To answer why we need a back-testing, think that if we want to validate the prediction performance, we have at least two alternatives:

Alternative 1: Estimate a ML model, make a prediction and wait in time, for example, 30 days, to verify if the prediction is good or not; if the forecast is not good (it is not close to the real value), then we have to train and test it again, let’s say other 30 days and so on.

Alternative 2 (The one we will apply): Take aside some observations, assuming those are observations we don’t know and store them in a data frame called “test.” Train and test the ML model. If we are not making a good prediction, we train and test the model again until we get a good prediction performance.


```python
validation_size = 0.2

#In case the data is not dependent on the time series, then train and test split randomly
# seed = 7
# X_train, X_test, Y_train, Y_test = train_test_split(X, Y, test_size=validation_size, random_state=seed)

#In case the data is not dependent on the time series, then train and test split should be done based on sequential sample
#This can be done by selecting an arbitrary split point in the ordered list of observations and creating two new datasets.
train_size = int(len(X) * (1-validation_size))
X_train, X_test = X[0:train_size], X[train_size:len(X)]
Y_train, Y_test = Y[0:train_size], Y[train_size:len(X)]
```

We start by running the Linear regression model, one of several machine learning models.


```python
from sklearn.linear_model import LinearRegression
model = LinearRegression()
model.fit(X_train, Y_train)
```

```{=html}
<style>#sk-container-id-1 {color: black;background-color: white;}#sk-container-id-1 pre{padding: 0;}#sk-container-id-1 div.sk-toggleable {background-color: white;}#sk-container-id-1 label.sk-toggleable__label {cursor: pointer;display: block;width: 100%;margin-bottom: 0;padding: 0.3em;box-sizing: border-box;text-align: center;}#sk-container-id-1 label.sk-toggleable__label-arrow:before {content: "▸";float: left;margin-right: 0.25em;color: #696969;}#sk-container-id-1 label.sk-toggleable__label-arrow:hover:before {color: black;}#sk-container-id-1 div.sk-estimator:hover label.sk-toggleable__label-arrow:before {color: black;}#sk-container-id-1 div.sk-toggleable__content {max-height: 0;max-width: 0;overflow: hidden;text-align: left;background-color: #f0f8ff;}#sk-container-id-1 div.sk-toggleable__content pre {margin: 0.2em;color: black;border-radius: 0.25em;background-color: #f0f8ff;}#sk-container-id-1 input.sk-toggleable__control:checked~div.sk-toggleable__content {max-height: 200px;max-width: 100%;overflow: auto;}#sk-container-id-1 input.sk-toggleable__control:checked~label.sk-toggleable__label-arrow:before {content: "▾";}#sk-container-id-1 div.sk-estimator input.sk-toggleable__control:checked~label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-1 div.sk-label input.sk-toggleable__control:checked~label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-1 input.sk-hidden--visually {border: 0;clip: rect(1px 1px 1px 1px);clip: rect(1px, 1px, 1px, 1px);height: 1px;margin: -1px;overflow: hidden;padding: 0;position: absolute;width: 1px;}#sk-container-id-1 div.sk-estimator {font-family: monospace;background-color: #f0f8ff;border: 1px dotted black;border-radius: 0.25em;box-sizing: border-box;margin-bottom: 0.5em;}#sk-container-id-1 div.sk-estimator:hover {background-color: #d4ebff;}#sk-container-id-1 div.sk-parallel-item::after {content: "";width: 100%;border-bottom: 1px solid gray;flex-grow: 1;}#sk-container-id-1 div.sk-label:hover label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-1 div.sk-serial::before {content: "";position: absolute;border-left: 1px solid gray;box-sizing: border-box;top: 0;bottom: 0;left: 50%;z-index: 0;}#sk-container-id-1 div.sk-serial {display: flex;flex-direction: column;align-items: center;background-color: white;padding-right: 0.2em;padding-left: 0.2em;position: relative;}#sk-container-id-1 div.sk-item {position: relative;z-index: 1;}#sk-container-id-1 div.sk-parallel {display: flex;align-items: stretch;justify-content: center;background-color: white;position: relative;}#sk-container-id-1 div.sk-item::before, #sk-container-id-1 div.sk-parallel-item::before {content: "";position: absolute;border-left: 1px solid gray;box-sizing: border-box;top: 0;bottom: 0;left: 50%;z-index: -1;}#sk-container-id-1 div.sk-parallel-item {display: flex;flex-direction: column;z-index: 1;position: relative;background-color: white;}#sk-container-id-1 div.sk-parallel-item:first-child::after {align-self: flex-end;width: 50%;}#sk-container-id-1 div.sk-parallel-item:last-child::after {align-self: flex-start;width: 50%;}#sk-container-id-1 div.sk-parallel-item:only-child::after {width: 0;}#sk-container-id-1 div.sk-dashed-wrapped {border: 1px dashed gray;margin: 0 0.4em 0.5em 0.4em;box-sizing: border-box;padding-bottom: 0.4em;background-color: white;}#sk-container-id-1 div.sk-label label {font-family: monospace;font-weight: bold;display: inline-block;line-height: 1.2em;}#sk-container-id-1 div.sk-label-container {text-align: center;}#sk-container-id-1 div.sk-container {/* jupyter's `normalize.less` sets `[hidden] { display: none; }` but bootstrap.min.css set `[hidden] { display: none !important; }` so we also need the `!important` here to be able to override the default hidden behavior on the sphinx rendered scikit-learn.org. See: https://github.com/scikit-learn/scikit-learn/issues/21755 */display: inline-block !important;position: relative;}#sk-container-id-1 div.sk-text-repr-fallback {display: none;}</style><div id="sk-container-id-1" class="sk-top-container"><div class="sk-text-repr-fallback"><pre>LinearRegression()</pre><b>In a Jupyter environment, please rerun this cell to show the HTML representation or trust the notebook. <br />On GitHub, the HTML representation is unable to render, please try loading this page with nbviewer.org.</b></div><div class="sk-container" hidden><div class="sk-item"><div class="sk-estimator sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-1" type="checkbox" checked><label for="sk-estimator-id-1" class="sk-toggleable__label sk-toggleable__label-arrow">LinearRegression</label><div class="sk-toggleable__content"><pre>LinearRegression()</pre></div></div></div></div></div>
```

Apparently, we did not get a result. But in the model object we store some information, such as the coefficients of the model:

$$ASURB.MX\_price_{t}=\beta_{0}+\beta_{1}\ ER_{t-1} + \beta_{2}\ inflation_{t-1} + \beta_{3}\ srate_{t-1}+ \beta_{4}\ growth_{t-1}+ \beta_{5}\ mrate_{t-1}+ \beta_{6}\ inv_{t-1} + \beta_{7}\ MR_{t-1} + e_{t}$$


```
#>    Betas     Values
#> 0  Beta1  -2.587712
#> 1  Beta2  -0.435412
#> 2  Beta3 -65.425396
#> 3  Beta4  -0.877509
#> 4  Beta5   0.507544
#> 5  Beta6   0.924602
#> 6  Beta7  -0.000094
```

We do the prediction of the ASURB.MX price, in the X_test data set:

```python
y_pred=model.predict(X_test)
y_pred
#> array([-2.1233116 , -5.14063355,  0.72607998,  2.69665977, 15.04951998,
#>         5.80142587,  4.68486414])
```

### Performance Measure Root Mean Square Error (RMSE).


In the previous section, we discussed whether the prediction was good (the prediction performance). This section formally defines the metrics for the prediction performance. 

The first metric is the Root Mean Square Error (RMSE). The mathematical formula to compute the RMSE is:

$$RMSE =\sqrt{\frac{1}{n}\ \sum_{i=1}^{n} (y_{i}-\hat{y_{i}})^{2}} $$


where $\hat{y_{i}}$ is the prediction for the ith observation, $y_{i}$ is the ith observation of the independent variable we store in the  test set, and n is the number of observations. 

$$\hat{y_{i}}=\hat{\beta_{0}}+\hat{\beta_{1}}x_{1}+,..,+\hat{\beta_{n}}x_{n}$$

We can estimate the RMSE using the training data set. But generally, we do not care how well the model works on the training set. Rather, we are interested in the model performance tested on unseen data; then, we try the RMSE on the test data set or the validation set for cross-validation. The lower the test RMSE, the better the prediction.


```python
from sklearn.metrics import mean_squared_error

mean_squared_error(Y_test, y_pred)
#> 1660.54739314444
```


## Appendix

### Stationary and Nonstationary Time Series

Historically, the notion of a stationary process has played an important role in the analysis of time series. A stationary time series process is one whose probability distributions are stable over time in the following sense: If we take any collection of random variables in the sequence and then shift that sequence ahead h time periods, the joint probability distribution must remain unchanged.

The stochastic process $x_{t}: t=1,2,..., $ is stationary if
for every collection of time indices $1 \leq t_{1} < t_{2} < … < t_{m}$, the joint distribution of $(x_{t1}  , x_{t2} , …, x_t{m})$ is the same as the joint distribution of $(x_{t1+h}=x_{t2+h},...,x_{tn+h})$ for all integers $h \geq 1$.

Covariance Stationary Process. A stochastic process ${x_{t}: t = 1, 2, …}$ with a finite
second moment $E(x_{t}^2) < \infty$ is covariance stationary if (i) $E(x_{t})$ is constant; (ii) $Var(x_{t})$ is constant; and (iii) for any $t, h \geq 1, Cov(x_{t}, x_{t+h})$ depends only on h and not on t.

On a practical level, if we want to understand the relationship between two or more variables using regression analysis, we need to assume some sort of stability over time.

If we allow the relationship between two variables (say, yt and xt) to change arbitrarily in each time period, then we cannot hope to learn much about how a change in one variable affects the other variable if we only have access to a single time series realization.

In stating a multiple regression model for time series data, we are assuming a certain
form of stationarity in that the $\beta_{j}$ do not change over time.
