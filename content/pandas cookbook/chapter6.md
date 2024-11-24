---
date: 2017-05-31
linktitle: Chapter 6
menu:
  main:
    parent: pandas
next: /pandas-cookbook/chapter7
prev: /pandas-cookbook/chapter5
title: Chapter 6 - String Operations
weight: 35
url: /pandas-cookbook/chapter6
description: String Operations in pandas. Using resampling and plotting temperature.
keywords:
  - pandas
  - string
  - subplots
---

```python
%matplotlib inline

import pandas as pd
import matplotlib.pyplot as plt
import numpy as np

plt.style.use('default')
plt.rcParams['figure.figsize'] = (15, 3)
plt.rcParams['font.family'] = 'sans-serif'
```

We saw earlier that pandas is really good at dealing with dates. It is also amazing with strings! We're going to go back to our weather data from Chapter 5, here.

```python
weather_2012 = pd.read_csv('weather_2012.csv', parse_dates=True, index_col='Date/Time')
weather_2012[:5]
```

Output:

<div class="output_html rendered_html output_subarea output_execute_result">
<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Temp (C)</th>
      <th>Dew Point Temp (C)</th>
      <th>Rel Hum (%)</th>
      <th>Wind Spd (km/h)</th>
      <th>Visibility (km)</th>
      <th>Stn Press (kPa)</th>
      <th>Weather</th>
    </tr>
    <tr>
      <th>Date/Time</th>
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
      <th>2012-01-01 00:00:00</th>
      <td>-1.8</td>
      <td>-3.9</td>
      <td> 86</td>
      <td> 4</td>
      <td> 8.0</td>
      <td> 101.24</td>
      <td>                  Fog</td>
    </tr>
    <tr>
      <th>2012-01-01 01:00:00</th>
      <td>-1.8</td>
      <td>-3.7</td>
      <td> 87</td>
      <td> 4</td>
      <td> 8.0</td>
      <td> 101.24</td>
      <td>                  Fog</td>
    </tr>
    <tr>
      <th>2012-01-01 02:00:00</th>
      <td>-1.8</td>
      <td>-3.4</td>
      <td> 89</td>
      <td> 7</td>
      <td> 4.0</td>
      <td> 101.26</td>
      <td> Freezing Drizzle,Fog</td>
    </tr>
    <tr>
      <th>2012-01-01 03:00:00</th>
      <td>-1.5</td>
      <td>-3.2</td>
      <td> 88</td>
      <td> 6</td>
      <td> 4.0</td>
      <td> 101.27</td>
      <td> Freezing Drizzle,Fog</td>
    </tr>
    <tr>
      <th>2012-01-01 04:00:00</th>
      <td>-1.5</td>
      <td>-3.3</td>
      <td> 88</td>
      <td> 7</td>
      <td> 4.8</td>
      <td> 101.23</td>
      <td>                  Fog</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 7 columns</p>
</div>
</div>

## 6.1 String operations

You'll see that the 'Weather' column has a text description of the weather that was going on each hour. We'll assume it's snowing if the text description contains "Snow".

pandas provides vectorized string functions, to make it easy to operate on columns containing text. There are some great [examples](https://pandas.pydata.org/pandas-docs/stable/basics.html#vectorized-string-methods) in the documentation.

```python
weather_description = weather_2012['Weather']
is_snowing = weather_description.str.contains('Snow')
```

This gives us a binary vector, which is a bit hard to look at, so we'll plot it.

```python
# Not super useful
is_snowing[:5]
```

Output:

```bash
Date/Time
2012-01-01 00:00:00    False
2012-01-01 01:00:00    False
2012-01-01 02:00:00    False
2012-01-01 03:00:00    False
2012-01-01 04:00:00    False
Name: Weather, dtype: bool
```

```python
# More useful!
is_snowing.plot()
```

Output:

<div>
<img src="/img/snow_plot.png" alt="Plot which borough has the most noise complaints" />
</div>

## 6.2 Use resampling to find the snowiest month

If we wanted the median temperature each month, we could use the [resample()](https://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.resample.html) method like this:

```python
weather_2012['Temp (°C)'].resample('ME').apply('median').plot(kind='bar')
```

Output:

<div>
<img src="/img/resample_plot.png" alt="Plot which borough has the most noise complaints" />
</div>

Unsurprisingly, July and August are the warmest.

So we can think of snowiness as being a bunch of 1s and 0s instead of Trues and Falses:

```python
is_snowing.astype(float)[:10]
```

Output:

```bash
Date/Time
2012-01-01 00:00:00    0
2012-01-01 01:00:00    0
2012-01-01 02:00:00    0
2012-01-01 03:00:00    0
2012-01-01 04:00:00    0
2012-01-01 05:00:00    0
2012-01-01 06:00:00    0
2012-01-01 07:00:00    0
2012-01-01 08:00:00    0
2012-01-01 09:00:00    0
Name: Weather, dtype: float64
```

and then use resample to find the percentage of time it was snowing each month

```python
is_snowing.astype(float).resample('ME').apply('mean')
```

Output:

```bash
Date/Time
2012-01-31    0.240591
2012-02-29    0.162356
2012-03-31    0.087366
2012-04-30    0.015278
2012-05-31    0.000000
2012-06-30    0.000000
2012-07-31    0.000000
2012-08-31    0.000000
2012-09-30    0.000000
2012-10-31    0.000000
2012-11-30    0.038889
2012-12-31    0.251344
Freq: M, Name: Weather, dtype: float64
```

```python
is_snowing.astype(float).resample('ME').apply('mean').plot(kind='bar')
```

Output:

<div>
<img src="/img/astype_resample_plot.png" alt="Plot which borough has the most noise complaints" />
</div>


So now we know! In 2012, December was the snowiest month. Also, this graph suggests something that I feel -- it starts snowing pretty abruptly in November, and then tapers off slowly and takes a long time to stop, with the last snow usually being in April or May.

## 6.3 Plotting temperature and snowiness stats together

We can also combine these two statistics (temperature, and snowiness) into one dataframe and plot them together:

```python
temperature = weather_2012['Temp (°C)'].resample('ME').apply('median')
is_snowing = weather_2012['Weather'].str.contains('Snow')
snowiness = is_snowing.astype(float).resample('ME').apply('mean')

# Name the columns
temperature.name = "Temperature"
snowiness.name = "Snowiness"
```

We'll use concat again to combine the two statistics into a single dataframe.

```python
stats = pd.concat([temperature, snowiness], axis=1)
stats
```

Output:

<div class="output_html rendered_html output_subarea output_execute_result">
<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Temperature</th>
      <th>Snowiness</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2012-01-31</th>
      <td> -7.05</td>
      <td> 0.240591</td>
    </tr>
    <tr>
      <th>2012-02-29</th>
      <td> -4.10</td>
      <td> 0.162356</td>
    </tr>
    <tr>
      <th>2012-03-31</th>
      <td>  2.60</td>
      <td> 0.087366</td>
    </tr>
    <tr>
      <th>2012-04-30</th>
      <td>  6.30</td>
      <td> 0.015278</td>
    </tr>
    <tr>
      <th>2012-05-31</th>
      <td> 16.05</td>
      <td> 0.000000</td>
    </tr>
    <tr>
      <th>2012-06-30</th>
      <td> 19.60</td>
      <td> 0.000000</td>
    </tr>
    <tr>
      <th>2012-07-31</th>
      <td> 22.90</td>
      <td> 0.000000</td>
    </tr>
    <tr>
      <th>2012-08-31</th>
      <td> 22.20</td>
      <td> 0.000000</td>
    </tr>
    <tr>
      <th>2012-09-30</th>
      <td> 16.10</td>
      <td> 0.000000</td>
    </tr>
    <tr>
      <th>2012-10-31</th>
      <td> 11.30</td>
      <td> 0.000000</td>
    </tr>
    <tr>
      <th>2012-11-30</th>
      <td>  1.05</td>
      <td> 0.038889</td>
    </tr>
    <tr>
      <th>2012-12-31</th>
      <td> -2.85</td>
      <td> 0.251344</td>
    </tr>
  </tbody>
</table>
<p>12 rows × 2 columns</p>
</div>
</div>

```python
stats.plot(kind='bar')
```

Output:

<div>
<img src="/img/temp_snow.png" alt="Plot which borough has the most noise complaints" />
</div>

Uh, that didn't work so well because the scale was wrong. We can do better by plotting them on two separate graphs:

```python
stats.plot(kind='bar', subplots=True, figsize=(15, 10))
```

Output:

<div>
<img src="/img/temp_snow_subplots.png" alt="Plot which borough has the most noise complaints" />
</div>
