
---

title: "Cleaning Data from American Time Use Survey"
date: 2021-02-04
tags: [ATUS, American Time Use Survey, data analysis, data cleaning, data wrangling]
classes: wide
header:
  image:
excerpt: Cleaning, Merging Survey Data  
 
---

# ATUS Data Preprocessing
The following notebook describes the steps taken to clean and consolidate the three datasets used in this analysis.


```python
import pandas as pd
import numpy as np
```


```python
# Respondent File
atus_resp = pd.read_table('/Users/orenmerberg/Desktop/Data/ATUS/atusresp-0319/atusresp_0319.dat', sep=',')



# Activity Summary File
atus_sum = pd.read_table('/Users/orenmerberg/Desktop/Data/ATUS/atussum-0319/atussum_0319.dat', sep=',')

# Activity File (Note: Add converter to avoid truncation of leading zeros.)
atus_act = pd.read_table('/Users/orenmerberg/Desktop/Data/ATUS/atusact-0319/atusact_0319.dat', sep=',', 
                         converters={col: lambda x: str(x) for col in ['TRCODEP', 'TRTIER1P', 'TRTIER2P']})
```


```python
print('atus_resp :', "  {:3,} rows x {} columns".format(*atus_resp.shape))
print('atus_sum  :', "  {:3,} rows x {} columns".format(*atus_sum.shape))
print('atus_act  :', "{:3,} rows x  {} columns".format(*atus_act.shape))
```

    atus_resp :   210,586 rows x 132 columns
    atus_sum  :   210,586 rows x 455 columns
    atus_act  : 4,121,283 rows x  29 columns


 

Taking a closer look at each file 


```python
atus_resp.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>TUCASEID</th>
      <th>TULINENO</th>
      <th>TESPUHRS</th>
      <th>TRDTIND1</th>
      <th>TRDTOCC1</th>
      <th>TRERNHLY</th>
      <th>TRERNUPD</th>
      <th>TRHERNAL</th>
      <th>TRHHCHILD</th>
      <th>TRIMIND1</th>
      <th>...</th>
      <th>TRYHHCHILD</th>
      <th>TRWBMODR</th>
      <th>TRTALONE_WK</th>
      <th>TRTCCC_WK</th>
      <th>TRLVMODR</th>
      <th>TRTEC</th>
      <th>TUECYTD</th>
      <th>TUELDER</th>
      <th>TUELFREQ</th>
      <th>TUELNUM</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>2.105860e+05</td>
      <td>210586.0</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>...</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>2.010289e+13</td>
      <td>1.0</td>
      <td>13.442065</td>
      <td>19.042895</td>
      <td>6.922070</td>
      <td>497.994774</td>
      <td>-0.251133</td>
      <td>-0.659341</td>
      <td>1.551404</td>
      <td>6.930114</td>
      <td>...</td>
      <td>2.820254</td>
      <td>-0.659740</td>
      <td>191.454973</td>
      <td>59.382912</td>
      <td>-0.837235</td>
      <td>2.941036</td>
      <td>-0.774591</td>
      <td>0.277526</td>
      <td>-0.610368</td>
      <td>-0.801473</td>
    </tr>
    <tr>
      <th>std</th>
      <td>4.918548e+10</td>
      <td>0.0</td>
      <td>21.446450</td>
      <td>19.204672</td>
      <td>8.202737</td>
      <td>938.200737</td>
      <td>0.764833</td>
      <td>0.539654</td>
      <td>0.497352</td>
      <td>7.596953</td>
      <td>...</td>
      <td>5.529047</td>
      <td>0.743485</td>
      <td>276.935784</td>
      <td>168.934417</td>
      <td>0.543412</td>
      <td>43.421018</td>
      <td>0.764884</td>
      <td>1.444009</td>
      <td>1.343332</td>
      <td>0.700463</td>
    </tr>
    <tr>
      <th>min</th>
      <td>2.003010e+13</td>
      <td>1.0</td>
      <td>-4.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>1.000000</td>
      <td>-1.000000</td>
      <td>...</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-3.000000</td>
      <td>-3.000000</td>
      <td>-3.000000</td>
      <td>-2.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2.006050e+13</td>
      <td>1.0</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>1.000000</td>
      <td>-1.000000</td>
      <td>...</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>2.010061e+13</td>
      <td>1.0</td>
      <td>-1.000000</td>
      <td>21.000000</td>
      <td>3.000000</td>
      <td>-1.000000</td>
      <td>0.000000</td>
      <td>-1.000000</td>
      <td>2.000000</td>
      <td>6.000000</td>
      <td>...</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>2.014101e+13</td>
      <td>1.0</td>
      <td>40.000000</td>
      <td>40.000000</td>
      <td>16.000000</td>
      <td>875.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>2.000000</td>
      <td>15.000000</td>
      <td>...</td>
      <td>6.000000</td>
      <td>-1.000000</td>
      <td>325.000000</td>
      <td>0.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>2.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>2.019121e+13</td>
      <td>1.0</td>
      <td>99.000000</td>
      <td>51.000000</td>
      <td>22.000000</td>
      <td>9999.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>2.000000</td>
      <td>21.000000</td>
      <td>...</td>
      <td>17.000000</td>
      <td>1.000000</td>
      <td>1440.000000</td>
      <td>1440.000000</td>
      <td>1.000000</td>
      <td>1380.000000</td>
      <td>2.000000</td>
      <td>2.000000</td>
      <td>7.000000</td>
      <td>5.000000</td>
    </tr>
  </tbody>
</table>
<p>8 rows × 130 columns</p>
</div>




```python
atus_sum.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>TUCASEID</th>
      <th>GEMETSTA</th>
      <th>GTMETSTA</th>
      <th>PEEDUCA</th>
      <th>PEHSPNON</th>
      <th>PTDTRACE</th>
      <th>TEAGE</th>
      <th>TELFS</th>
      <th>TEMJOT</th>
      <th>TESCHENR</th>
      <th>...</th>
      <th>t181801</th>
      <th>t181899</th>
      <th>t189999</th>
      <th>t500101</th>
      <th>t500103</th>
      <th>t500104</th>
      <th>t500105</th>
      <th>t500106</th>
      <th>t500107</th>
      <th>t509989</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>2.105860e+05</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>...</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
      <td>210586.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>2.010289e+13</td>
      <td>-0.696357</td>
      <td>0.882048</td>
      <td>40.320007</td>
      <td>1.864084</td>
      <td>1.353385</td>
      <td>47.519674</td>
      <td>2.493613</td>
      <td>0.803482</td>
      <td>0.584317</td>
      <td>...</td>
      <td>0.073899</td>
      <td>0.018862</td>
      <td>1.902567</td>
      <td>6.474462</td>
      <td>0.670448</td>
      <td>0.442408</td>
      <td>0.268299</td>
      <td>4.062682</td>
      <td>0.114490</td>
      <td>0.032319</td>
    </tr>
    <tr>
      <th>std</th>
      <td>4.918548e+10</td>
      <td>0.774652</td>
      <td>0.841673</td>
      <td>2.876836</td>
      <td>0.342700</td>
      <td>1.057036</td>
      <td>17.826802</td>
      <td>1.876572</td>
      <td>1.428389</td>
      <td>1.434198</td>
      <td>...</td>
      <td>3.910262</td>
      <td>1.949870</td>
      <td>22.521028</td>
      <td>36.809435</td>
      <td>9.716821</td>
      <td>9.686125</td>
      <td>5.116452</td>
      <td>19.661665</td>
      <td>4.611797</td>
      <td>2.433426</td>
    </tr>
    <tr>
      <th>min</th>
      <td>2.003010e+13</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>31.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>15.000000</td>
      <td>1.000000</td>
      <td>-1.000000</td>
      <td>-3.000000</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2.006050e+13</td>
      <td>-1.000000</td>
      <td>1.000000</td>
      <td>39.000000</td>
      <td>2.000000</td>
      <td>1.000000</td>
      <td>34.000000</td>
      <td>1.000000</td>
      <td>-1.000000</td>
      <td>-1.000000</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>2.010061e+13</td>
      <td>-1.000000</td>
      <td>1.000000</td>
      <td>40.000000</td>
      <td>2.000000</td>
      <td>1.000000</td>
      <td>46.000000</td>
      <td>1.000000</td>
      <td>2.000000</td>
      <td>1.000000</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>2.014101e+13</td>
      <td>-1.000000</td>
      <td>1.000000</td>
      <td>43.000000</td>
      <td>2.000000</td>
      <td>1.000000</td>
      <td>61.000000</td>
      <td>5.000000</td>
      <td>2.000000</td>
      <td>2.000000</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>2.019121e+13</td>
      <td>3.000000</td>
      <td>3.000000</td>
      <td>46.000000</td>
      <td>2.000000</td>
      <td>25.000000</td>
      <td>85.000000</td>
      <td>5.000000</td>
      <td>2.000000</td>
      <td>2.000000</td>
      <td>...</td>
      <td>960.000000</td>
      <td>625.000000</td>
      <td>1420.000000</td>
      <td>1385.000000</td>
      <td>825.000000</td>
      <td>1155.000000</td>
      <td>179.000000</td>
      <td>179.000000</td>
      <td>565.000000</td>
      <td>500.000000</td>
    </tr>
  </tbody>
</table>
<p>8 rows × 455 columns</p>
</div>




```python
atus_act.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>TUCASEID</th>
      <th>TUACTIVITY_N</th>
      <th>TUACTDUR24</th>
      <th>TUCC5</th>
      <th>TUCC5B</th>
      <th>TRTCCTOT_LN</th>
      <th>TRTCC_LN</th>
      <th>TRTCOC_LN</th>
      <th>TUCC8</th>
      <th>TUCUMDUR</th>
      <th>...</th>
      <th>TRTONHH_LN</th>
      <th>TRTOHH_LN</th>
      <th>TRTHH_LN</th>
      <th>TRTNOHH_LN</th>
      <th>TEWHERE</th>
      <th>TUCC7</th>
      <th>TRWBELIG</th>
      <th>TRTEC_LN</th>
      <th>TUEC24</th>
      <th>TUDURSTOP</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>4.121283e+06</td>
      <td>4.121283e+06</td>
      <td>4.121283e+06</td>
      <td>4.121283e+06</td>
      <td>4.121283e+06</td>
      <td>4.121283e+06</td>
      <td>4.121283e+06</td>
      <td>4.121283e+06</td>
      <td>4.121283e+06</td>
      <td>4.121283e+06</td>
      <td>...</td>
      <td>4.121283e+06</td>
      <td>4.121283e+06</td>
      <td>4.121283e+06</td>
      <td>4.121283e+06</td>
      <td>4.121283e+06</td>
      <td>4.121283e+06</td>
      <td>4.121283e+06</td>
      <td>4.121283e+06</td>
      <td>4.121283e+06</td>
      <td>4.121283e+06</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>2.010223e+13</td>
      <td>1.197230e+01</td>
      <td>7.357996e+01</td>
      <td>6.689982e-01</td>
      <td>1.142829e-01</td>
      <td>6.040197e+00</td>
      <td>5.662692e+00</td>
      <td>7.575733e-01</td>
      <td>4.888736e+00</td>
      <td>6.957297e+02</td>
      <td>...</td>
      <td>-9.596150e-01</td>
      <td>4.541912e+00</td>
      <td>4.962332e+00</td>
      <td>-5.199332e-01</td>
      <td>4.098267e+00</td>
      <td>-9.312998e-02</td>
      <td>-6.971647e-01</td>
      <td>-7.751642e-01</td>
      <td>-9.025199e-01</td>
      <td>-2.165961e-01</td>
    </tr>
    <tr>
      <th>std</th>
      <td>4.932327e+10</td>
      <td>8.497220e+00</td>
      <td>1.001475e+02</td>
      <td>7.498292e+00</td>
      <td>4.445179e+00</td>
      <td>2.609507e+01</td>
      <td>2.611321e+01</td>
      <td>1.154668e+01</td>
      <td>2.118854e+01</td>
      <td>3.739806e+02</td>
      <td>...</td>
      <td>2.190437e+00</td>
      <td>2.372223e+01</td>
      <td>2.468423e+01</td>
      <td>7.644256e+00</td>
      <td>7.007204e+00</td>
      <td>8.254573e-01</td>
      <td>6.894248e-01</td>
      <td>5.418079e+00</td>
      <td>2.987491e+00</td>
      <td>1.101805e+00</td>
    </tr>
    <tr>
      <th>min</th>
      <td>2.003010e+13</td>
      <td>1.000000e+00</td>
      <td>1.000000e+00</td>
      <td>-3.000000e+00</td>
      <td>-3.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>-3.000000e+00</td>
      <td>1.000000e+00</td>
      <td>...</td>
      <td>-1.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>-3.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>-3.000000e+00</td>
      <td>-1.000000e+00</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2.006040e+13</td>
      <td>5.000000e+00</td>
      <td>1.500000e+01</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>3.950000e+02</td>
      <td>...</td>
      <td>-1.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>1.000000e+00</td>
      <td>0.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>-1.000000e+00</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>2.010040e+13</td>
      <td>1.000000e+01</td>
      <td>3.000000e+01</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>6.900000e+02</td>
      <td>...</td>
      <td>-1.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>1.000000e+00</td>
      <td>0.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>-1.000000e+00</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>2.014101e+13</td>
      <td>1.700000e+01</td>
      <td>9.000000e+01</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>9.360000e+02</td>
      <td>...</td>
      <td>-1.000000e+00</td>
      <td>0.000000e+00</td>
      <td>0.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>8.000000e+00</td>
      <td>0.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>-1.000000e+00</td>
      <td>1.000000e+00</td>
    </tr>
    <tr>
      <th>max</th>
      <td>2.019121e+13</td>
      <td>9.100000e+01</td>
      <td>1.350000e+03</td>
      <td>9.700000e+01</td>
      <td>9.700000e+01</td>
      <td>1.195000e+03</td>
      <td>1.195000e+03</td>
      <td>1.195000e+03</td>
      <td>9.700000e+01</td>
      <td>2.700000e+03</td>
      <td>...</td>
      <td>7.200000e+02</td>
      <td>1.195000e+03</td>
      <td>1.195000e+03</td>
      <td>8.550000e+02</td>
      <td>9.900000e+01</td>
      <td>9.700000e+01</td>
      <td>1.000000e+00</td>
      <td>1.097000e+03</td>
      <td>9.700000e+01</td>
      <td>2.000000e+00</td>
    </tr>
  </tbody>
</table>
<p>8 rows × 24 columns</p>
</div>



 

There are a lot of variables (especially `atus_sum`) and the variable names are not straightforward. Luckily, ATUS provides a 
[table](https://www.bls.gov/tus/freqvariables.pdf) of frequently used variables. I decide to only take their suggested variables for now (I can always come back to grab more).  


```python
freq_vars = pd.read_excel('/Users/orenmerberg/Desktop/Data/ATUS/freqvariables.xlsx')
```

$\left(\text{Activity} \cup \text{Summary} \cup \text{Respondent}\right ) \cap \text{Frequent}$


```python
# Union of columns in all three ATUS datasets.
all_cols = set(atus_act.columns) | set(atus_sum.columns) | set(atus_resp.columns)

# Intersection of all_cols and freq_vars. 
freq_cols = all_cols & set(freq_vars.Variables)

def filter_cols(df):
    """ Outputs new dataframe keeping only columns that are in freq_cols, sets index to TUCASEID. """
    df_new = df.filter(freq_cols)
    df_new['TUCASEID'] = df_new.TUCASEID.astype(str)
    return df_new.set_index('TUCASEID')
```


```python
df_act = filter_cols(atus_act)
df_sum = filter_cols(atus_sum)
df_res = filter_cols(atus_resp)

print('df_res :', "  {:3,} rows x {} columns".format(*df_res.shape))
print('df_sum :', "  {:3,} rows x {} columns".format(*df_sum.shape))
print('df_act :', "{:3,} rows x  {} columns".format(*df_act.shape))
```

    df_res :   210,586 rows x 43 columns
    df_sum :   210,586 rows x 23 columns
    df_act : 4,121,283 rows x  9 columns


This is easier to work with.  

Next, I merge `df_sum` and `df_res` on `TUCASEID` because it's one-to-one, but leave `df_act` alone since each `TUCASEID` is repeated for each activity done by a respondent.


```python
# Drop columns from df_sum that are in df_res and merge on TUCASEID.
cols_to_use = df_sum.columns.difference(df_res.columns)
df_res_sum = pd.merge(df_res, df_sum[cols_to_use], how='inner', on='TUCASEID')
```


```python
print('df_res_sum :', "{:3,} rows x {} columns\n".format(*df_res_sum.shape))
display(df_res_sum.head())
```

    df_res_sum : 210,586 rows x 50 columns
    



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>TROHHCHILD</th>
      <th>TRHOLIDAY</th>
      <th>TRTHH</th>
      <th>TRERNWA</th>
      <th>TRWBMODR</th>
      <th>TRERNHLY</th>
      <th>TRHHCHILD</th>
      <th>TUDIARYDATE</th>
      <th>TEMJOT</th>
      <th>TUELDER</th>
      <th>...</th>
      <th>TUYEAR</th>
      <th>TRSPFTPT</th>
      <th>TUDIARYDAY</th>
      <th>GEMETSTA</th>
      <th>GTMETSTA</th>
      <th>PEEDUCA</th>
      <th>PEHSPNON</th>
      <th>PTDTRACE</th>
      <th>TEAGE</th>
      <th>TESEX</th>
    </tr>
    <tr>
      <th>TUCASEID</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>20030100013280</th>
      <td>2</td>
      <td>0</td>
      <td>-1</td>
      <td>66000</td>
      <td>-1</td>
      <td>2200.0</td>
      <td>2</td>
      <td>20030103</td>
      <td>2</td>
      <td>-1</td>
      <td>...</td>
      <td>2003</td>
      <td>-1</td>
      <td>6</td>
      <td>1</td>
      <td>-1</td>
      <td>44</td>
      <td>2</td>
      <td>2</td>
      <td>60</td>
      <td>1</td>
    </tr>
    <tr>
      <th>20030100013344</th>
      <td>1</td>
      <td>0</td>
      <td>-1</td>
      <td>20000</td>
      <td>-1</td>
      <td>-1.0</td>
      <td>1</td>
      <td>20030104</td>
      <td>2</td>
      <td>-1</td>
      <td>...</td>
      <td>2003</td>
      <td>1</td>
      <td>7</td>
      <td>2</td>
      <td>-1</td>
      <td>40</td>
      <td>2</td>
      <td>1</td>
      <td>41</td>
      <td>2</td>
    </tr>
    <tr>
      <th>20030100013352</th>
      <td>2</td>
      <td>0</td>
      <td>-1</td>
      <td>20000</td>
      <td>-1</td>
      <td>1250.0</td>
      <td>2</td>
      <td>20030104</td>
      <td>2</td>
      <td>-1</td>
      <td>...</td>
      <td>2003</td>
      <td>-1</td>
      <td>7</td>
      <td>1</td>
      <td>-1</td>
      <td>41</td>
      <td>2</td>
      <td>1</td>
      <td>26</td>
      <td>2</td>
    </tr>
    <tr>
      <th>20030100013848</th>
      <td>1</td>
      <td>0</td>
      <td>-1</td>
      <td>-1</td>
      <td>-1</td>
      <td>-1.0</td>
      <td>1</td>
      <td>20030102</td>
      <td>-1</td>
      <td>-1</td>
      <td>...</td>
      <td>2003</td>
      <td>1</td>
      <td>5</td>
      <td>2</td>
      <td>-1</td>
      <td>39</td>
      <td>2</td>
      <td>2</td>
      <td>36</td>
      <td>2</td>
    </tr>
    <tr>
      <th>20030100014165</th>
      <td>1</td>
      <td>0</td>
      <td>-1</td>
      <td>-1</td>
      <td>-1</td>
      <td>-1.0</td>
      <td>1</td>
      <td>20030109</td>
      <td>2</td>
      <td>-1</td>
      <td>...</td>
      <td>2003</td>
      <td>-1</td>
      <td>5</td>
      <td>2</td>
      <td>-1</td>
      <td>45</td>
      <td>2</td>
      <td>1</td>
      <td>51</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 50 columns</p>
</div>


 

I would now like to make the data easier to read. The variable names are not very descriptive and each variable takes on numerical values (as opposed to categorical). Luckily, ATUS provides a data dictionary PDF on their website (pictured below).

![Screen Shot 2021-02-01 at 11.41.29 PM.png](attachment:d7c06688-d631-49f4-a8de-b859678ab9c8.png)  
[Source](https://www.bls.gov/tus/atusintcodebk0319.pdf)

PDF's are generally difficult to parse, but I think it's worth the effort. Python has some modules to deal with PDF's, but I decide to convert it to .csv using Adobe Acrobat and use standard Python modules to clean it up.


```python
atus_codes = pd.read_excel('/Users/orenmerberg/Desktop/Data/ATUS/atusintcodebk19.xlsx', header=None)
atus_codes.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
      <th>8</th>
      <th>9</th>
      <th>...</th>
      <th>29</th>
      <th>30</th>
      <th>31</th>
      <th>32</th>
      <th>33</th>
      <th>34</th>
      <th>35</th>
      <th>36</th>
      <th>37</th>
      <th>38</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Name</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>File</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>TEABSRSN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Respondent File</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 39 columns</p>
</div>



Yikes...


```python
# Handle missing values.
atus_codes = atus_codes.dropna(axis=1, how='all').dropna(axis=0, how='all')
atus_codes[0] = atus_codes[0].fillna(method='ffill')
atus_codes = atus_codes.set_index(0)
atus_codes.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>10</th>
      <th>16</th>
      <th>22</th>
      <th>33</th>
    </tr>
    <tr>
      <th>0</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Name</th>
      <td>Description</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>File</td>
    </tr>
    <tr>
      <th>TEABSRSN</th>
      <td>Edited: what was the main reason you were abse...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Respondent File</td>
    </tr>
    <tr>
      <th>TEABSRSN</th>
      <td>Edited Universe:</td>
      <td>TELFS = 2</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>TEABSRSN</th>
      <td>Valid Entries:</td>
      <td>1</td>
      <td>On layoff (temporary or indefinite)</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>TEABSRSN</th>
      <td>NaN</td>
      <td>2</td>
      <td>Slack work/business conditions</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



This is workable. Now I just need to parse the data to extract a map for the categorical variables. This took some trial and error, so the following function `get_val_map` is quite specifically designed for this exact problem and might not make much sense. 


```python
def get_val_map(df):
    val_map = {}   
    for idx in df.index.unique()[1:]:        
        if len(df.loc[idx].iloc[2:][22]) > 2:        
            if df.loc[idx].iloc[-1][10] == '*Note':
                val_map[idx] = dict(df.loc[idx].iloc[2:][[16,22]].values[:-1])
            else:
                val_map[idx] = dict(df.loc[idx].iloc[2:][[16,22]].values)           
    return val_map

val_map = get_val_map(atus_codes)
```


```python
# Print example.
val_map['TEIO1COW']
```




    {1: 'Government, federal',
     2: 'Government, state',
     3: 'Government, local',
     4: 'Private, for profit',
     5: 'Private, nonprofit',
     6: 'Self-employed, incorporated',
     7: 'Self-employed, unincorporated',
     8: 'Without pay'}



 

Applying `val_map` to the data...


```python
def map_columns(df):
    """ Decode variables with previously defined val_map. """
    for col in df.columns:
        if col in val_map.keys():
            df[col] = df[col].map(val_map[col])
    # Additional definitions provided by ATUS.
    df = df.replace({-1: np.nan, -2: "Unsure", -3: "Refused"})
    return df

df_res_sum = map_columns(df_res_sum)
df_act = map_columns(df_act)
```


```python
df_res_sum.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>TROHHCHILD</th>
      <th>TRHOLIDAY</th>
      <th>TRTHH</th>
      <th>TRERNWA</th>
      <th>TRWBMODR</th>
      <th>TRERNHLY</th>
      <th>TRHHCHILD</th>
      <th>TUDIARYDATE</th>
      <th>TEMJOT</th>
      <th>TUELDER</th>
      <th>...</th>
      <th>TUYEAR</th>
      <th>TRSPFTPT</th>
      <th>TUDIARYDAY</th>
      <th>GEMETSTA</th>
      <th>GTMETSTA</th>
      <th>PEEDUCA</th>
      <th>PEHSPNON</th>
      <th>PTDTRACE</th>
      <th>TEAGE</th>
      <th>TESEX</th>
    </tr>
    <tr>
      <th>TUCASEID</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>20030100013280</th>
      <td>2.0</td>
      <td>Diary day was not a holiday</td>
      <td>NaN</td>
      <td>66000.0</td>
      <td>NaN</td>
      <td>2200.0</td>
      <td>2.0</td>
      <td>20030103.0</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>...</td>
      <td>2003.0</td>
      <td>NaN</td>
      <td>Friday</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>44.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>60.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>20030100013344</th>
      <td>1.0</td>
      <td>Diary day was not a holiday</td>
      <td>NaN</td>
      <td>20000.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>20030104.0</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>...</td>
      <td>2003.0</td>
      <td>Full time</td>
      <td>Saturday</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>40.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>41.0</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>20030100013352</th>
      <td>2.0</td>
      <td>Diary day was not a holiday</td>
      <td>NaN</td>
      <td>20000.0</td>
      <td>NaN</td>
      <td>1250.0</td>
      <td>2.0</td>
      <td>20030104.0</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>...</td>
      <td>2003.0</td>
      <td>NaN</td>
      <td>Saturday</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>41.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>26.0</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>20030100013848</th>
      <td>1.0</td>
      <td>Diary day was not a holiday</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>20030102.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>2003.0</td>
      <td>Full time</td>
      <td>Thursday</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>39.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>36.0</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>20030100014165</th>
      <td>1.0</td>
      <td>Diary day was not a holiday</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>20030109.0</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>...</td>
      <td>2003.0</td>
      <td>NaN</td>
      <td>Thursday</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>45.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>51.0</td>
      <td>1.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 50 columns</p>
</div>




```python
df_act.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>TRCODEP</th>
      <th>TEWHERE</th>
      <th>TRTIER2P</th>
      <th>TRTHH_LN</th>
      <th>TUACTDUR24</th>
      <th>TRTIER1P</th>
      <th>TUSTOPTIME</th>
      <th>TRTEC_LN</th>
      <th>TUSTARTTIM</th>
    </tr>
    <tr>
      <th>TUCASEID</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>20030100013280</th>
      <td>130124</td>
      <td>Outdoors away from home</td>
      <td>1301</td>
      <td>NaN</td>
      <td>60.0</td>
      <td>13</td>
      <td>05:00:00</td>
      <td>NaN</td>
      <td>04:00:00</td>
    </tr>
    <tr>
      <th>20030100013280</th>
      <td>010201</td>
      <td>NaN</td>
      <td>0102</td>
      <td>NaN</td>
      <td>30.0</td>
      <td>01</td>
      <td>05:30:00</td>
      <td>NaN</td>
      <td>05:00:00</td>
    </tr>
    <tr>
      <th>20030100013280</th>
      <td>010101</td>
      <td>NaN</td>
      <td>0101</td>
      <td>NaN</td>
      <td>600.0</td>
      <td>01</td>
      <td>15:30:00</td>
      <td>NaN</td>
      <td>05:30:00</td>
    </tr>
    <tr>
      <th>20030100013280</th>
      <td>120303</td>
      <td>Respondent's home or yard</td>
      <td>1203</td>
      <td>NaN</td>
      <td>150.0</td>
      <td>12</td>
      <td>18:00:00</td>
      <td>NaN</td>
      <td>15:30:00</td>
    </tr>
    <tr>
      <th>20030100013280</th>
      <td>110101</td>
      <td>Respondent's home or yard</td>
      <td>1101</td>
      <td>NaN</td>
      <td>5.0</td>
      <td>11</td>
      <td>18:05:00</td>
      <td>NaN</td>
      <td>18:00:00</td>
    </tr>
  </tbody>
</table>
</div>



Some variables were missed. `TRCODEP`, `TRTIER2P`, and `TRTIER2P` in the Activity File have a separate coding lexicon provided by ATUS. I followed a similar process as before to map these variables. For any additional categorical variables that are missed, I will manually search them on ATUS's site. 


```python
lex = pd.read_excel('/Users/orenmerberg/Desktop/Data/ATUS/lexiconnoex0319.xlsx', converters={'Code': lambda x: str(x)})
lex.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Category</th>
      <th>Sub Category</th>
      <th>Code</th>
      <th>Activity</th>
      <th>Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>01 Personal Care Activities</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NaN</td>
      <td>0101  Sleeping</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>010101</td>
      <td>Sleeping</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>010102</td>
      <td>Sleeplessness</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>010199</td>
      <td>Sleeping, n.e.c.*</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Fill missing values
lex[['Category', 'Sub Category']] = lex[['Category', 'Sub Category']].fillna(method='ffill')
lex = lex.dropna(subset=['Code'])

# Extract (code, description) pairs for the three activity variables.
tier1 = lex['Category'].drop_duplicates()
tier1_map = dict(zip(tier1.str[:2], tier1.str[2:].str.strip()))

tier2 = lex['Sub Category'].drop_duplicates()
tier2_map = dict(zip(tier2.str[:4], tier2.str[4:].str.strip()))

tier3_map = dict(zip(lex['Code'], lex['Activity']))
```


```python
# Map values in activity dataset.
df_act['TRTIER1P'] = df_act['TRTIER1P'].map(tier1_map)
df_act['TRTIER2P'] = df_act['TRTIER2P'].map(tier2_map)
df_act['TRCODEP'] = df_act['TRCODEP'].map(tier3_map)
```


```python
df_act.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>TRCODEP</th>
      <th>TEWHERE</th>
      <th>TRTIER2P</th>
      <th>TRTHH_LN</th>
      <th>TUACTDUR24</th>
      <th>TRTIER1P</th>
      <th>TUSTOPTIME</th>
      <th>TRTEC_LN</th>
      <th>TUSTARTTIM</th>
    </tr>
    <tr>
      <th>TUCASEID</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>20030100013280</th>
      <td>Running</td>
      <td>Outdoors away from home</td>
      <td>Participating in Sports, Exercise, and Recreation</td>
      <td>NaN</td>
      <td>60.0</td>
      <td>Sports, Exercise, &amp; Recreation</td>
      <td>05:00:00</td>
      <td>NaN</td>
      <td>04:00:00</td>
    </tr>
    <tr>
      <th>20030100013280</th>
      <td>Washing, dressing and grooming oneself</td>
      <td>NaN</td>
      <td>Grooming</td>
      <td>NaN</td>
      <td>30.0</td>
      <td>Personal Care Activities</td>
      <td>05:30:00</td>
      <td>NaN</td>
      <td>05:00:00</td>
    </tr>
    <tr>
      <th>20030100013280</th>
      <td>Sleeping</td>
      <td>NaN</td>
      <td>Sleeping</td>
      <td>NaN</td>
      <td>600.0</td>
      <td>Personal Care Activities</td>
      <td>15:30:00</td>
      <td>NaN</td>
      <td>05:30:00</td>
    </tr>
    <tr>
      <th>20030100013280</th>
      <td>Television and movies (not religious)</td>
      <td>Respondent's home or yard</td>
      <td>Relaxing and Leisure</td>
      <td>NaN</td>
      <td>150.0</td>
      <td>Socializing, Relaxing, and Leisure</td>
      <td>18:00:00</td>
      <td>NaN</td>
      <td>15:30:00</td>
    </tr>
    <tr>
      <th>20030100013280</th>
      <td>Eating and drinking</td>
      <td>Respondent's home or yard</td>
      <td>Eating and Drinking</td>
      <td>NaN</td>
      <td>5.0</td>
      <td>Eating and Drinking</td>
      <td>18:05:00</td>
      <td>NaN</td>
      <td>18:00:00</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_res_sum.loc[:, ['TROHHCHILD', 'TEMJOT', 'TESCHENR', 'TRNHHCHILD']].replace({1:'Yes', 2:'No'}, inplace=True)

df_res_sum['TESPEMPNOT'].replace({1:'Employed', 2:'Unemployed'}, inplace=True)

df_res_sum['TRDPFTPT'].replace({1:'Full Time', 2:'Part Time'}, inplace=True)

df_res_sum['TEERNHRY'].replace({1:'Hourly', 2:'Not Hourly'}, inplace=True)

df_res_sum['GEMETSTA'].replace({1:'Metropolitan', 2:'Non-metropolitan', 3:'Not Identified'}, inplace=True)

df_res_sum['PEHSPNON'].replace({1:'Hispanic', 2:'Not Hispanic'}, inplace=True)

df_res_sum['TESEX'].replace({1:'Male', 2:'Female'}, inplace=True)

df_res_sum['PEEDUCA'].replace({ 31: 'Less than 1st grade',
                                32: '1st, 2nd, 3rd, or 4th grade',
                                33: '5th or 6th grade',
                                34: '7th or 8th grade',
                                35: '9th grade',
                                36: '10th grade',
                                37: '11th grade',
                                38: '12th grade - no diploma',
                                39: 'High school graduate - diploma or equivalent (GED)',
                                40: 'Some college but no degree',
                                41: 'Associate degree - occupational/vocational',
                                42: 'Associate degree - academic program',
                                43: "Bachelor's degree (BA, AB, BS, etc.)",
                                44: "Master's degree (MA, MS, MEng, MEd, MSW, etc.)",
                                45: 'Professional school degree (MD, DDS, DVM, etc.)',
                                46: 'Doctoral degree (PhD, EdD, etc.)'},
                              inplace=True)

df_res_sum['PTDTRACE'].replace({1: 'Asian Indian',
                                2: 'Chinese',
                                3: 'Filipino',
                                4: 'Japanese',
                                5: 'Korean',
                                6: 'Vietnamese',
                                7: 'Other'},
                               inplace=True)
```

Lastly, I rename the columns to make things easier.


```python
new_cols = { 'TEERNHRY': 'job_paid_hourly',
             'TEHRUSL1': 'weekly_work_hrs_main',
             'TEHRUSL2': 'weekly_work_hrs_other',
             'TEIO1COW': 'worker_class',
             'TEIO1ICD': 'employment_industry1',
             'TEIO1OCD': 'occupation1',
             'TESCHFT': 'school_enrollment_type',
             'TESPUHRS': 'spouse_work_hrs',
             'TRDTIND1': 'employment_industry2',
             'TRDTOCC1': 'occupation2',
             'TREMODR': 'eat_health_module',
             'TRERNHLY': 'hourly_earnings',
             'TRHHCHILD': 'child_lt18_present',
             'TRIMIND1': 'employment_industry3',
             'TRLVMODR': 'leave_module',
             'TRMJIND1': 'employment_industry4',
             'TRMJOCC1': 'occupation3',
             'TRMJOCGR': 'occupation4',
             'TRNHHCHILD': 'nonhousehold_child_lt18',
             'TROHHCHILD': 'household_child_lt18',
             'TRTEC': 'diary_time_eldercare',
             'TRTHH': 'diary_time_childcare2',
             'TRWBMODR': 'wellbeing_module',
             'TUDIARYDATE': 'date',
             'TUECYTD': 'eldercare_yesterday',
             'TUELDER': 'eldercare_prev_months',
             'TUMONTH': 'month',
             'TUFNWGTP': 'weight',
             'TRSPPRES': 'spouse_present',
             'TRSPFTPT': 'spouse_employment_type',
             'TESCHENR': 'school_enrollment',
             'TESCHLVL': 'school_level',
             'TUSTARTTIM': 'activity_start',
             'TUSTOPTIME': 'activity_stop',
             'TUACTDUR24': 'activity_duration',
             'TRTHH_LN': 'activity_time_childcare',
             'TEWHERE': 'activity_location',
             'TRTEC_LN': 'activity_time_eldercare',
             'TRTIER2P': 'activity_category_ter',
             'TRCODEP': 'activity_category_prim',
             'TRTIER1P': 'activity_category_sec',
             'TEHRUSLT': 'weekly_work_hours',
             'PTDTRACE': 'race',
             'TRDPFTPT': 'employment_type',
             'TUYEAR': 'year',
             'TRCHILDNUM': 'num_child_lt18',
             'TEAGE': 'age',
             'PEHSPNON': 'hispanic_origin',
             'PEEDUCA': 'education',
             'TRHOLIDAY': 'holiday',
             'TUDIARYDAY': 'day',
             'TRERNWA': 'weekly_earnings',
             'TEMJOT': 'multiple_job_status',
             'TESEX': 'gender',
             'TESPEMPNOT': 'spouse_employment_status',
             'TELFS': 'labor_status',
             'TRYHHCHILD': 'age_youngest_child',
             'GTMETSTA': 'metropolitan1',
             'GEMETSTA': 'metropolitan2' 
           }
```


```python
df_res_sum = df_res_sum.rename(columns=new_cols)
df_act = df_act.rename(columns=new_cols)
```


```python
df_res_sum.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>household_child_lt18</th>
      <th>holiday</th>
      <th>diary_time_childcare2</th>
      <th>weekly_earnings</th>
      <th>wellbeing_module</th>
      <th>hourly_earnings</th>
      <th>child_lt18_present</th>
      <th>date</th>
      <th>multiple_job_status</th>
      <th>eldercare_prev_months</th>
      <th>...</th>
      <th>year</th>
      <th>spouse_employment_type</th>
      <th>day</th>
      <th>metropolitan2</th>
      <th>metropolitan1</th>
      <th>education</th>
      <th>hispanic_origin</th>
      <th>race</th>
      <th>age</th>
      <th>gender</th>
    </tr>
    <tr>
      <th>TUCASEID</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>20030100013280</th>
      <td>No</td>
      <td>Diary day was not a holiday</td>
      <td>NaN</td>
      <td>66000.0</td>
      <td>NaN</td>
      <td>2200.0</td>
      <td>No</td>
      <td>20030103.0</td>
      <td>No</td>
      <td>NaN</td>
      <td>...</td>
      <td>2003.0</td>
      <td>NaN</td>
      <td>Friday</td>
      <td>Metropolitan</td>
      <td>NaN</td>
      <td>Master's degree (MA, MS, MEng, MEd, MSW, etc.)</td>
      <td>Not Hispanic</td>
      <td>Chinese</td>
      <td>60.0</td>
      <td>Male</td>
    </tr>
    <tr>
      <th>20030100013344</th>
      <td>Yes</td>
      <td>Diary day was not a holiday</td>
      <td>NaN</td>
      <td>20000.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Yes</td>
      <td>20030104.0</td>
      <td>No</td>
      <td>NaN</td>
      <td>...</td>
      <td>2003.0</td>
      <td>Full time</td>
      <td>Saturday</td>
      <td>Non-metropolitan</td>
      <td>NaN</td>
      <td>Some college but no degree</td>
      <td>Not Hispanic</td>
      <td>Asian Indian</td>
      <td>41.0</td>
      <td>Female</td>
    </tr>
    <tr>
      <th>20030100013352</th>
      <td>No</td>
      <td>Diary day was not a holiday</td>
      <td>NaN</td>
      <td>20000.0</td>
      <td>NaN</td>
      <td>1250.0</td>
      <td>No</td>
      <td>20030104.0</td>
      <td>No</td>
      <td>NaN</td>
      <td>...</td>
      <td>2003.0</td>
      <td>NaN</td>
      <td>Saturday</td>
      <td>Metropolitan</td>
      <td>NaN</td>
      <td>Associate degree - occupational/vocational</td>
      <td>Not Hispanic</td>
      <td>Asian Indian</td>
      <td>26.0</td>
      <td>Female</td>
    </tr>
    <tr>
      <th>20030100013848</th>
      <td>Yes</td>
      <td>Diary day was not a holiday</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Yes</td>
      <td>20030102.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>2003.0</td>
      <td>Full time</td>
      <td>Thursday</td>
      <td>Non-metropolitan</td>
      <td>NaN</td>
      <td>High school graduate - diploma or equivalent (...</td>
      <td>Not Hispanic</td>
      <td>Chinese</td>
      <td>36.0</td>
      <td>Female</td>
    </tr>
    <tr>
      <th>20030100014165</th>
      <td>Yes</td>
      <td>Diary day was not a holiday</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Yes</td>
      <td>20030109.0</td>
      <td>No</td>
      <td>NaN</td>
      <td>...</td>
      <td>2003.0</td>
      <td>NaN</td>
      <td>Thursday</td>
      <td>Non-metropolitan</td>
      <td>NaN</td>
      <td>Professional school degree (MD, DDS, DVM, etc.)</td>
      <td>Not Hispanic</td>
      <td>Asian Indian</td>
      <td>51.0</td>
      <td>Male</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 50 columns</p>
</div>




```python
df_act.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>activity_category_prim</th>
      <th>activity_location</th>
      <th>activity_category_ter</th>
      <th>activity_time_childcare</th>
      <th>activity_duration</th>
      <th>activity_category_sec</th>
      <th>activity_stop</th>
      <th>activity_time_eldercare</th>
      <th>activity_start</th>
    </tr>
    <tr>
      <th>TUCASEID</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>20030100013280</th>
      <td>Running</td>
      <td>Outdoors away from home</td>
      <td>Participating in Sports, Exercise, and Recreation</td>
      <td>NaN</td>
      <td>60.0</td>
      <td>Sports, Exercise, &amp; Recreation</td>
      <td>05:00:00</td>
      <td>NaN</td>
      <td>04:00:00</td>
    </tr>
    <tr>
      <th>20030100013280</th>
      <td>Washing, dressing and grooming oneself</td>
      <td>NaN</td>
      <td>Grooming</td>
      <td>NaN</td>
      <td>30.0</td>
      <td>Personal Care Activities</td>
      <td>05:30:00</td>
      <td>NaN</td>
      <td>05:00:00</td>
    </tr>
    <tr>
      <th>20030100013280</th>
      <td>Sleeping</td>
      <td>NaN</td>
      <td>Sleeping</td>
      <td>NaN</td>
      <td>600.0</td>
      <td>Personal Care Activities</td>
      <td>15:30:00</td>
      <td>NaN</td>
      <td>05:30:00</td>
    </tr>
    <tr>
      <th>20030100013280</th>
      <td>Television and movies (not religious)</td>
      <td>Respondent's home or yard</td>
      <td>Relaxing and Leisure</td>
      <td>NaN</td>
      <td>150.0</td>
      <td>Socializing, Relaxing, and Leisure</td>
      <td>18:00:00</td>
      <td>NaN</td>
      <td>15:30:00</td>
    </tr>
    <tr>
      <th>20030100013280</th>
      <td>Eating and drinking</td>
      <td>Respondent's home or yard</td>
      <td>Eating and Drinking</td>
      <td>NaN</td>
      <td>5.0</td>
      <td>Eating and Drinking</td>
      <td>18:05:00</td>
      <td>NaN</td>
      <td>18:00:00</td>
    </tr>
  </tbody>
</table>
</div>


