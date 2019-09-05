---
published: true
category: python
tags: python
---
Psychopy is (rightfully) one of the most popular tools to create behavioral experiments. However, since it outputs so much data, it might be very bothersome to analyze it.
Assuming default options (data for each participant is saved in a separate file), I usually do the following:

1) import necessary libraries:
```python
import pandas as pd
import csv
import glob
import os
```

1) read and combine every participant's data file into one mega dataframe:
```python
path = r'F:\Final results'
all_files = glob.glob(os.path.join(path, "*.csv"))
all_files_data = (pd.read_csv(f) for f in all_files)
df_all  = pd.concat(all_files_data, ignore_index=True)
```

3) in case I am combining questionnaires and experiments themselves into one experiment, I separate experimental data. In this example, I take all rows for which variable "vol" was used, in other words, all experimental trials.
```python
all_exp = df_all[df_all.vol.notnull()]
experimental_columns = ['subID', 'Age','Gender','soa','vol','coh', 'direct', 'response', 'correct', 'RT_exp','mouse.x', 'mouse.y', 'mouse.time']
df_exp = all_exp[experimental_columns]
df_exp_real = df_exp[df_exp.UIN.notnull()]
exp_countpersubj = df_exp_real.groupby(['UIN'])['UIN'].count()
```
