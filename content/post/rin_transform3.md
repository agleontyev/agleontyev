---
title: Correlation and Rank-Based Inverse Normal Transformation
date: 2020-02-12
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

def rinfunc(ds):
    ds_rank = ds.rank()
    numerator = ds_rank - 0.5 
    par = numerator/len(ds)
    result = norm.ppf(par)
    return result
```

Let's test it. 


```python
import pandas as pd

d = {'one' : pd.Series([10, 25, 3, 11, 24, 6]), 
      'two' : pd.Series([10, 20, 30, 40, 80, 70]),
    'index': ['p','r','o','g','r','a']}  
df = pd.DataFrame(d) 
df.set_index('index', inplace = True)
```


```python
df['one_transformed'] = rinfunc(df['one'])
df['two_transformed'] = rinfunc(df['two'])

```


```python
df
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
      <th>one</th>
      <th>two</th>
      <th>one_transformed</th>
      <th>two_transformed</th>
    </tr>
    <tr>
      <th>index</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>p</th>
      <td>10</td>
      <td>10</td>
      <td>-0.210428</td>
      <td>-1.382994</td>
    </tr>
    <tr>
      <th>r</th>
      <td>25</td>
      <td>20</td>
      <td>1.382994</td>
      <td>-0.674490</td>
    </tr>
    <tr>
      <th>o</th>
      <td>3</td>
      <td>30</td>
      <td>-1.382994</td>
      <td>-0.210428</td>
    </tr>
    <tr>
      <th>g</th>
      <td>11</td>
      <td>40</td>
      <td>0.210428</td>
      <td>0.210428</td>
    </tr>
    <tr>
      <th>r</th>
      <td>24</td>
      <td>80</td>
      <td>0.674490</td>
      <td>1.382994</td>
    </tr>
    <tr>
      <th>a</th>
      <td>6</td>
      <td>70</td>
      <td>-0.674490</td>
      <td>0.674490</td>
    </tr>
  </tbody>
</table>
</div>



# References

1. Bishara, A. J., & Hittner, J. B. (2012). Testing the significance of a correlation with nonnormal data: comparison of Pearson, Spearman, transformation, and resampling approaches. *Psychological methods, 17*(3), 399.
