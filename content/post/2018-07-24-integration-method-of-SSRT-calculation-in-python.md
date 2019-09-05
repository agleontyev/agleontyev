---
title: SSRT calculation with integration method
category: python
published: true
tags: python
---
Stop-signal reaction time (SSRT) is one of the most popular metrics of behavioral disinhibition. It can be estimated with a number of different methods. The exact method which you should use depends on the exeperimental design. If one uses a number of pre-determined stop-signal delays (SSD), then the method of choice is the integration method.

To use this method, we need to first calculate accuracy for each predetermined stop-signal delay, and select one for which accuracy in stop trials equals 0.5 (50 %).
In this picture, no specific SSD (on y-axis, expressed in ms) correpsonds to 0.5 accuracy.

![figure1](https://raw.githubusercontent.com/agleontyev/agleontyev.github.io/master/pics/x1.png "Accuracy vs SSD")

To estimate SSD corresponding to 50% accuracy, we need to interpolate it.

First, let's import the necessary packages: 
```python
import pandas as pd
from scipy.interpolate import UnivariateSpline
from scipy import interpolate
from scipy.interpolate import interp1d
from scipy.interpolate import InterpolatedUnivariateSpline
```
Second, assuming we calculated accuracy for each individual (expressed as subjID, which is set as index), we need to transform the dataframe first.

```python
df_ssrt = stop_error_rate_overall.reset_index()
df = df_ssrt.melt(id_vars=['subjID'])
df['variable'] = df['variable'].map({'100_stop_acc': 100, '200_stop_acc': 200, '300_stop_acc':300, '400_stop_acc':400,'500_stop_acc':500,'600_stop_acc':600})
```

Finally, define the smoothing function:
```python
def SmootheSplines(x):
    x1 = x['variable'].values
    y1 = x['value'].values
    s = InterpolatedUnivariateSpline(x1, y1)
    return s
splinedf = df.groupby('UIN').apply(SmootheSplines)
```
Done! Now we only have to subtract median "go" time from the value in splinedf.



```python
def evaluate_spline_distance(row):
    spline_obj, splice_val = row
    return spline_obj(splice_val)

df['results'] = df[[0, 'splice']].apply(evaluate_spline_distance, axis=1)
```
