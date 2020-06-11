---
title: Calculating indifference points using consistent choice detection algoritm in the delay discounting task
date: 2020-06-04
math: true
diagram: true
markup: mmark
---
Delay discounting task is one of the most powerful instruments of
behavioral analysis of impulsivity. This paradigm presents an individual
with two options: smaller and immeditely avaliable one (sooner/smaller)
and a larger option avaliable after a variable delay. Based on a
participant’s choices, we can estimate the rate at which a participant
devalues delayed rewards (k, or discounting rate). To do so, we need to
calculate at which point does an individual habve equal preferejce
towards delayed and immediate rewards. In other words, we need to find
the immediate amount that is worth the same to an individual that is a
greater but delayed one. There is a lot of ways to calculate this
indifference point. In this post, I want to focus on a specific one,
used by Amlung and MacKillop (2013). In their study, indifference point
is defined as the arithmetic average of values offered when a
partidcipant swithces their prefernce and before. Consider this example:

<table>
<colgroup>
<col style="width: 2%" />
<col style="width: 19%" />
<col style="width: 17%" />
<col style="width: 12%" />
<col style="width: 48%" />
</colgroup>
<thead>
<tr class="header">
<th></th>
<th>Smaller/immediate amount</th>
<th>larger/delayed amount</th>
<th>Delay (in days)</th>
<th>Choice (0 = larger option chosen, 1 = smaller option chosen)</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1</td>
<td>$10</td>
<td>$100</td>
<td>10</td>
<td>0</td>
</tr>
<tr class="even">
<td>2</td>
<td>$20</td>
<td>$100</td>
<td>10</td>
<td>0</td>
</tr>
<tr class="odd">
<td>3</td>
<td>$30</td>
<td>$100</td>
<td>10</td>
<td>0</td>
</tr>
<tr class="even">
<td>4</td>
<td>$40</td>
<td>$100</td>
<td>10</td>
<td>0</td>
</tr>
<tr class="odd">
<td>5</td>
<td>$50</td>
<td>$100</td>
<td>10</td>
<td>0</td>
</tr>
<tr class="even">
<td>6</td>
<td>$60</td>
<td>$100</td>
<td>10</td>
<td>1</td>
</tr>
<tr class="odd">
<td>7</td>
<td>$70</td>
<td>$100</td>
<td>10</td>
<td>1</td>
</tr>
<tr class="even">
<td>8</td>
<td>$80</td>
<td>$100</td>
<td>10</td>
<td>1</td>
</tr>
<tr class="odd">
<td>9</td>
<td>$90</td>
<td>$100</td>
<td>10</td>
<td>1</td>
</tr>
</tbody>
</table>

In this case, the switch occurs when an individual is offered \$60.
Previous offer is \$50. Thus, the indifference point is equal to (60 +
50)/2 = \$55. That is, \$100 with a 10 days waiting period is worth the
same as \$55 avaliable immediately. Notice that the participant in this
example is consistent in his choice (i.e. doesn’t keep switching back
when offered a larger amount). But some people are indeed inconsistent.
How can we estimate indifference points with different levels of
“tolerance” for consistency? Let me present a (slighly changed)
brilliant solution provided by Mr. Ethan Armstrong.

First, let’s create some sample data

    library(tidyverse)

    ## -- Attaching packages --------------------------------------------------------------------------------------- tidyverse 1.3.0 --

    ## v ggplot2 3.3.1     v purrr   0.3.4
    ## v tibble  3.0.1     v dplyr   0.8.5
    ## v tidyr   1.0.3     v stringr 1.4.0
    ## v readr   1.3.1     v forcats 0.5.0

    ## -- Conflicts ------------------------------------------------------------------------------------------ tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

    df<- tibble(value = seq(0,100,10),
                        choice = c(0,0,0,0,0,1,0,0,1,1,1))
    print(df)

    ## # A tibble: 11 x 2
    ##    value choice
    ##    <dbl>  <dbl>
    ##  1     0      0
    ##  2    10      0
    ##  3    20      0
    ##  4    30      0
    ##  5    40      0
    ##  6    50      1
    ##  7    60      0
    ##  8    70      0
    ##  9    80      1
    ## 10    90      1
    ## 11   100      1

Then, define Mr. Armstrong’s function:

    choice_detector <- function (df,consistency_period = 3) {
    df_lagged <- df %>% mutate(previous_choice = lag(choice),
                               previous_choice = replace_na(previous_choice, 0),
                               no_switch = as.character((choice == previous_choice)))
      for(i in 1:nrow(df_lagged)) {

        if (i + consistency_period > nrow(df_lagged)) {

          cat('Insufficient observations to determine consistent switch.  Exiting...                                 \r')
          break

        }

        df_row <- df_lagged[i,]

        if (df_row$no_switch == F) {

          switch_average    <- (df_row$value + df_lagged[i-1,]$value)/2
          df_look_ahead     <- df_lagged[i:(i+consistency_period),] # How far do we need to look ahead to determine if it is consistent?
          check_consistency <- df_look_ahead %>% count(no_switch) %>% filter(no_switch == F) %>% pull(n)

          if (check_consistency == 1) {

            cat('Consistent switch found with period of', consistency_period, 'rows.  Returning average of initial instance of this change...                                 \r')
            return(list(average = switch_average, row_where_consistent_switch = i))

          } else {

            cat('Switch is not consistent for given consistency period, moving to next row.                                     \r')
            next

          }

        }

      }

    }

See how it works:

    z <- choice_detector(df = df, 2) # will return 75

    ## Switch is not consistent for given consistency period, moving to next row.                                     Switch is not consistent for given consistency period, moving to next row.                                     Consistent switch found with period of 2 rows.  Returning average of initial instance of this change...                                 

    print(z)

    ## $average
    ## [1] 75
    ## 
    ## $row_where_consistent_switch
    ## [1] 9


# References
1. Amlung, M., & MacKillop, J. (2011). Delayed reward discounting and alcohol misuse: the roles of response consistency and reward magnitude. Journal of experimental psychopathology, 2(3), 418-431.
