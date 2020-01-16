---
title: Correlation and Rank-Based Inverse Normal Transformation
date: 2020-01-16
math: true
diagram: true
markup: mmark
---

This post is based on Bishara and Hittner's article (2012), introduced to me by Dr. Takashi Yamauchi.
Correlation is simple yet one of the most important tools in establishing the relationship between two variables. However, if Pearson's *r* is used and the data is non-normally distributed, the amount of Type I erros (false positives, i.e. seeing correlation where there is truly no correlation) might increase dramatically.
To offset this, the authors recommend a data normalization technique called Rank-Based Inverse Normal transformation (RIN).

This transformation is peformed according to the following formula:

$$ f(x) = \Phi^{-1} \Big(\frac{x_{r} - \frac{1}{2}} {n} \Big), $$

where "$x_{r}$ is ascending rank of $x$, such that $x_{r} = 1 $ for the lowest value of $x$" (p. 401), $\Phi^{-1}$ is the inverse normal cumulative distribution function and $n$ is the number of observations (sample size).
Let's now create a script to calculate it in Python. Note that ppf, percent point function, is an alternative name for the quantile function:


```python
from scipy.stats import norm
import pandas as pd

class RINtransform:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    def transform_col1(self):
        ds = self.x
        ds_rank = ds.rank()
        numerator = ds_rank - 0.5 
        par = numerator/ds_rank.size
        col1_transform = norm.ppf(par)
        return col1_transform
    def transform_col2(self):
        ds = self.y
        ds_rank = ds.rank()
        numerator = ds_rank - 0.5 
        par = numerator/ds_rank.size
        col2_transform = norm.ppf(par)
        return col2_transform
    def final_cols(self):
        col1 = self.transform_col1()
        col2 = self.transform_col2()
        data = {'x1_rin':col1,'x2_rin':col2}
        final = pd.DataFrame(data)
        return final
```

Let's test it:


```python
import pandas as pd

d = {'one' : pd.Series([10, 25, 3, 11, 24, 6]), 
      'two' : pd.Series([10, 20, 30, 40, 80, 70])}  
df = pd.DataFrame(d) 
results = RINtransform(x = df['one'], y = df['two'])
```


```python
results.final_cols()
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
      <th>x1_rin</th>
      <th>x2_rin</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-0.210428</td>
      <td>-1.382994</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.382994</td>
      <td>-0.674490</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-1.382994</td>
      <td>-0.210428</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.210428</td>
      <td>0.210428</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.674490</td>
      <td>1.382994</td>
    </tr>
    <tr>
      <th>5</th>
      <td>-0.674490</td>
      <td>0.674490</td>
    </tr>
  </tbody>
</table>
</div>



These two columns can be correlated with each other.


# References #

1. Bishara, A. J., & Hittner, J. B. (2012). Testing the significance of a correlation with nonnormal data: comparison of Pearson, Spearman, transformation, and resampling approaches. *Psychological methods, 17*(3), 399.
