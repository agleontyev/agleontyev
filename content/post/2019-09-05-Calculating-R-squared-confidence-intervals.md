---
title: Confidence intervals for R-squared
date: 2019-09-05
math: true
diagram: true
markup: mmark
---
$R^2$, or coefficient of determination, measures the proportion of dependent variable's variance explained (i.e., predictable) by an independent variable or variables. 
In this document, I define a few functions that compute the confidence interval for the coefficient of determination (also know as $R^2$). To calculate this, recall Cohen et al. (2003) formula of  the squared standard error of $R^2$:

$$SE_{R^{2}} = \sqrt{\frac{4R^2(1 - R^2)^2(n-k-1)^2)}{(n^2 - 1)(n + 3)}}$$

where $R^2$ is unadjusted $R^2$, k  is the number of independent variables and n is the number of cases (observations). Simplest is calculation of 67% confidence interval:
$$67\% CI  = R^{2} \pm SE_{R^{2}}$$

Transforming this formula into python:



```python
from math import sqrt
def rsquareCI (R2, n, k):
    SE = sqrt((4*R2*((1-R2)**2)*((n-k-1)**2))/((n**2-1)*(n + 3)))
    upper = R2 + SE
    lower = R2 - SE
    print("CI upper boundary:{}, CI lower boundary:{}".format(upper, lower))
```


```python
rsquareCI(R2 = 0.16, n = 53, k = 2)
```

    CI upper boundary:0.24473185457363233, CI lower boundary:0.07526814542636767
    

What if we need other $R^2$ confidence intervals, for example, 80, 95 or 99%? To calculate them, wee need multiple SE by appropriate factors:  

| CI      | Constant factor |
| ----------- | ----------- |
| 80%      | 1.3       |
| 95%   | 2        |
| 99%   | 2.6      |

Incorporating different factors into the code above:


```python
from math import sqrt
def rsquareCI (R2, n, k, CI):
    SE = sqrt((4*R2*((1-R2)**2)*((n-k-1)**2))/((n**2-1)*(n + 3)))
    if CI == 0.67:
        upper = R2 + SE
        lower = R2 - SE
    elif CI == 0.8:
        upper = R2 + 1.3*SE
        lower = R2 - 1.3*SE
    elif CI == 0.95:
        upper = R2 + 2*SE
        lower = R2 - 2*SE
    elif CI == 0.99:
        upper = R2 + 2.6*SE
        lower = R2 - 2.6*SE
    else:
        raise ValueError('Unknown value for CI. Please use 0.67, 0.8, 0.95 or 0.99')
    print("CI:{}\n CI lower boundary:{}\n CI upper boundary:{}".format(CI,lower, upper))
    return SE, lower, upper

```

Test with the same values as above:


```python
rsquareCI(R2 = 0.16, n = 53, k = 2, CI = 0.67)
```

    CI:0.67
     CI lower boundary:0.07526814542636767
     CI upper boundary:0.24473185457363233
    




    (0.08473185457363233, 0.07526814542636767, 0.24473185457363233)




```python
rsquareCI(R2 = 0.16, n = 53, k = 2, CI = 0.8)
```

    CI:0.8
     CI lower boundary:0.04984858905427797
     CI upper boundary:0.27015141094572204
    




    (0.08473185457363233, 0.04984858905427797, 0.27015141094572204)




```python
rsquareCI(R2 = 0.16, n = 53, k = 2, CI = 0.95)
```

    CI:0.95
     CI lower boundary:-0.00946370914726466
     CI upper boundary:0.32946370914726464
    




    (0.08473185457363233, -0.00946370914726466, 0.32946370914726464)




```python
rsquareCI(R2 = 0.16, n = 53, k = 2, CI = 0.99)
```

    CI:0.99
     CI lower boundary:-0.060302821891444064
     CI upper boundary:0.38030282189144404
    




    (0.08473185457363233, -0.060302821891444064, 0.38030282189144404)



Trying "unknown" value for desired confidence interval should throw an error:


```python
rsquareCI(R2 = 0.16, n = 53, k = 2, CI = 0.7)
```


    ---------------------------------------------------------------------------

    ValueError                                Traceback (most recent call last)

    <ipython-input-14-6a099e67e18b> in <module>
    ----> 1 rsquareCI(R2 = 0.16, n = 53, k = 2, CI = 0.7)
    

    <ipython-input-3-ecd3a8024d75> in rsquareCI(R2, n, k, CI)
         15         lower = R2 - 2.6*SE
         16     else:
    ---> 17         raise ValueError('Unknown value for CI. Please use 0.67, 0.8, 0.95 or 0.99')
         18     print("CI:{}\n CI lower boundary:{}\n CI upper boundary:{}".format(CI,lower, upper))
         19     return SE, lower, upper
    

    ValueError: Unknown value for CI. Please use 0.67, 0.8, 0.95 or 0.99


Everything works as intended.

## Comparison between $R^2$s between two models ##

Comparison between two models can be done as well. First (simpler) is not pooled difference.


1. Calculate the square root difference between $SE^2$ between two studies:
$$SE_{diff} = \sqrt{SE_{1}^{2} + SE_{2}^{2}}$$

2. Calculate the difference in $R^2$ between two studies:
$$R^{2}_{diff} = R^{2}_{1} - R^{2}_{2} $$

3. Calculate the z-score:

$$Z = \frac{R^{2}_{diff}}{SE_{diff}}$$

4. Calculate the two-tailed p-value from z value

Translating the above steps into Python code:



```python
from scipy.stats import norm
def R2difference(R1, R2, SE1, SE2, pooled):
    if pooled == False:
        SEdiff = sqrt(SE1**2 + SE2**2)
        Rdiff = R1 - R2
        z = Rdiff/SEdiff
        p = 2 * (1 - norm.cdf(z))
        print ("P-value is {}".format(p))
```


```python
R2difference(R1 = 0.08, R2 = 0.16, SE1 = 0.07, SE2 = 0.08, pooled = False)
```

    P-value is 1.548295673433068
    

The non-pooled version is a little more complex. Specifically, the formula for calculating $SE_{diff}$ is more sophisticated:

$$SE_{diff} = \sqrt{\frac{SE_{1}^{2}(n_{1}-1) +SE_{2}^{2}(n_{2}-1)}{n_{1}+n_{2}-2}}$$

where $SE_1$ and $SE_2$ are SEs calculated for first and second regression models, and $n_1$ and $n_2$ are number of observations.
Otherwise, the process is the same.
Adding the non-pooled version to the Python code above:


```python
from scipy.stats import norm
from math import sqrt

def R2difference(R1, R2, SE1, SE2, n1, n2, pooled):
    if pooled == False:
        SEdiff = sqrt(SE1**2 + SE2**2)
    elif pooled == True:
        SEdiff = sqrt(((SE1**2)*(n1-1) + (SE2**2)*(n2 - 1))/(n1+n2-2))
    Rdiff = R1 - R2
    z = Rdiff/SEdiff
    p = 2 * (1 - norm.cdf(z))
    print ("P-value is {}".format(p))
    return(p, z)
```


```python
R2difference(R1 = 0.08, R2 = 0.16, SE1=0.068640903, SE2 = 0.084731855, n1 = 50, n2 = 53, pooled = True)
```

    P-value is 1.6990192335482046
    




    (1.6990192335482046, -1.0343324611382851)




```python
R2difference(R1 = 0.08, R2 = 0.16, SE1=0.068640903, SE2 = 0.084731855, n1 = 50, n2 = 53, pooled = False)
```

    P-value is 1.5368284103310432
    




    (1.5368284103310432, -0.733634399498801)



These results are similar to those obtained manually (in Excel).

# References #

1. Fritz, C. O., Morris, P. E., & Richler, J. J. (2012). Effect size estimates: current use, calculations, and interpretation. *Journal of experimental psychology: General*, 141(1), 2-18
2. Cohen, J., Cohen, P., West, S. G., & Aiken, L. S. (2003). Applied multiple regression/correlation analysis for the behavioral sciences (3rd ed.). Mahwah: NJ: Erlbaum.

