<style>
.reveal section code {
  font-size: 1em;
}
</style>

dplyr (simplified)
========================================================
author: Etienne Low-Décarie
transition: fade

January 5, 2017


Split-Apply-Combine
===

- Equivalent
  - SQL GROUP BY
  - Pivot Tables (Excel, SPSS, …)
  
![split](./dplyr-figure/split_apply_combine.png)

***
  
- Split
  - Define a subset of your data
- Apply
  - Do anything to this subset 
    - calculation, modeling, simulations, plotting
- Combine

- Repeat this for all subsets
  - collect the results
  


dplyr    
===

Cheatsheet
[https://www.rstudio.com/wp-content/uploads/2015/02/data-wrangling-cheatsheet.pdf](https://www.rstudio.com/wp-content/uploads/2015/02/data-wrangling-cheatsheet.pdf)

install
===


```r
if(!require(dplyr)){install.packages("dplyr")}
require(dplyr)
```

Grouping
===

**Split** -Apply-Combine


```r
group_by(data,
         grouping_variable_1,
         grouping_variable_2)
```

Group data
===


```r
CO2_grouped <- group_by(CO2,
                           Plant,
                           Type,
                           Treatment)
```


group data
===

`group_by` implies `tbl_df`
`tbl_df` wraps a `data.frame`

```r
print(CO2_grouped)
```

```
Source: local data frame [84 x 5]
Groups: Plant, Type, Treatment [12]

   Plant   Type  Treatment  conc uptake
*  <ord> <fctr>     <fctr> <dbl>  <dbl>
1    Qn1 Quebec nonchilled    95   16.0
2    Qn1 Quebec nonchilled   175   30.4
3    Qn1 Quebec nonchilled   250   34.8
4    Qn1 Quebec nonchilled   350   37.2
5    Qn1 Quebec nonchilled   500   35.3
6    Qn1 Quebec nonchilled   675   39.2
7    Qn1 Quebec nonchilled  1000   39.7
8    Qn2 Quebec nonchilled    95   13.6
9    Qn2 Quebec nonchilled   175   27.3
10   Qn2 Quebec nonchilled   250   37.1
# ... with 74 more rows
```

```r
class(CO2_grouped)
```

```
[1] "grouped_df" "tbl_df"     "tbl"        "data.frame"
```

group data
===

**Caution when working with functions that expect a `data.frame`**


```r
class(CO2_grouped)
```

```
[1] "grouped_df" "tbl_df"     "tbl"        "data.frame"
```

```r
head(as.data.frame(CO2_grouped))
```

```
  Plant   Type  Treatment conc uptake
1   Qn1 Quebec nonchilled   95   16.0
2   Qn1 Quebec nonchilled  175   30.4
3   Qn1 Quebec nonchilled  250   34.8
4   Qn1 Quebec nonchilled  350   37.2
5   Qn1 Quebec nonchilled  500   35.3
6   Qn1 Quebec nonchilled  675   39.2
```



Apply verbes
===
class: small-code

Split-**Apply-Combine**

- `mutate`


![plot of chunk unnamed-chunk-6](dplyr-figure/unnamed-chunk-6-1.png)

- `summarise`


![plot of chunk unnamed-chunk-7](dplyr-figure/unnamed-chunk-7-1.png)


Summarize grouped data
===
class: small-code

Summarize by extracting maximum uptake per plant


```r
CO2_max_per_plant <- dplyr:::summarise(CO2_grouped,
                               max_uptake=max(uptake))
print(CO2_max_per_plant)
```

```
Source: local data frame [12 x 4]
Groups: Plant, Type [?]

   Plant        Type  Treatment max_uptake
   <ord>      <fctr>     <fctr>      <dbl>
1    Qn1      Quebec nonchilled       39.7
2    Qn2      Quebec nonchilled       44.3
3    Qn3      Quebec nonchilled       45.5
4    Qc1      Quebec    chilled       38.7
5    Qc3      Quebec    chilled       41.4
6    Qc2      Quebec    chilled       42.4
7    Mn3 Mississippi nonchilled       28.5
8    Mn2 Mississippi nonchilled       32.4
9    Mn1 Mississippi nonchilled       35.5
10   Mc2 Mississippi    chilled       14.4
11   Mc3 Mississippi    chilled       19.9
12   Mc1 Mississippi    chilled       22.2
```


Plot summarized data
===
left: 75%


 

```r
dodge <- position_dodge(1)

p <- ggplot(data=CO2_max_per_plant,
           aes(x=Type,
           y=max_uptake,
           fill=Treatment))+
           geom_bar(stat="summary",
           fun.y=mean,
           position=dodge)+
  stat_summary(fun.data="mean_cl_normal", 
               geom="errorbar",
               position=dodge)
```

***


![plot of chunk unnamed-chunk-11](dplyr-figure/unnamed-chunk-11-1.png)


Create a beautiful table of summarized data
===


```r
if(!require(gridExtra)){install.packages("gridExtra")}
require(gridExtra)
grid.newpage()
grid.table(head(data.frame(CO2_max_per_plant)))
```

![plot of chunk unnamed-chunk-12](dplyr-figure/unnamed-chunk-12-1.png)

Exercise 1
===

- Calculate the mean for each species value for each dimension of sepals and petals in the `iris` data set
- Produce an elegant table


<div class="centered">

<script src="countdown.js" type="text/javascript"></script>
<script type="application/javascript">
var myCountdown1 = new Countdown({
    							time: 300, 
									width:150, 
									height:80, 
									rangeHi:"minute"	// <- no comma on last item!
									});

</script>

</div>


dplyr mutate data
===

Calculate deviation from the mean uptake for each plant


```r
CO2_with_deviation <- mutate(CO2_grouped,
                               deviation_from_mean=uptake-mean(uptake))
```

Plot mutated data
===


```r
CO2.plot <- qplot(data = CO2_with_deviation,
                  x = conc,
                  y = deviation_from_mean,
                  colour = Treatment) + 
  facet_grid(.~Type)+
  geom_line(aes(group = Plant))
print(CO2.plot)
```

![plot of chunk unnamed-chunk-14](dplyr-figure/unnamed-chunk-14-1.png)



Writing your own functions
===

Calculate the slope and intercept for each plant


```r
intercept_slope <- function(conc, uptake){
  fit <- lm(uptake~conc)
  coefficients <- coef(fit)
  return(data.frame(intercept=coefficients[1],
                    slope=coefficients[2]))
}
```

note: there is now a better way to do this using require(broom)

Using your new function with dplyr
===


```r
CO2_fit <- summarise(CO2_grouped,
                     intercept=intercept_slope(conc, uptake)$intercept,
                     slope=intercept_slope(conc, uptake)$slope)
```

Plot the results of your new function
===
class: small-code


```r
dodge <- position_dodge(1)
p <- ggplot(data=CO2_fit,aes(x=Type,
           y=slope,
           fill=Treatment))+
           geom_bar(stat="summary",
           fun.y=mean,
           position=dodge)+
  stat_summary(fun.data="mean_cl_normal", 
               geom="errorbar",
               position=dodge)
```

***

![plot of chunk unnamed-chunk-18](dplyr-figure/unnamed-chunk-18-1.png)



Exercise 2
===

- Use the exercise_data_temp_par.csv file
- Calculate the deviations from the annual means  
- Calculate the annual absolute integrated anomaly


$$
\sum_{i=January}^{i=December}\left\lvert x_i- \bar x _{i2002-2014} \right\rvert
$$


Chaining operations
===


`%>%`  allows you to apply multiple functions sequentially  
(equivalent to "|" in bash)


```r
CO2_grouped <- group_by(CO2,
                                        Plant,
                                        Type,
                                        Treatment)
CO2_max_per_plant <- summarise(CO2_grouped,
                               max_uptake=max(uptake))
```

becomes


```r
CO2_max_per_plant <-CO2 %>%
                      group_by(Plant,
                                Type,
                                Treatment)  %>%
                      summarise(max_uptake=max(uptake)) 
```




Exercise 3
===

- convert code from previous exercises (1 & 2) using `%>%` chaining
  -convert the `summarise` of the iris data set
  -convert the `mutate` of the monthly temperature


<div class="centered">

<script src="countdown.js" type="text/javascript"></script>
<script type="application/javascript">
var myCountdown1 = new Countdown({
    							time: 300, 
									width:150, 
									height:80, 
									rangeHi:"minute"	// <- no comma on last item!
									});

</script>

</div>


dplyr database
===

dplyr can do most calculations in the database

note that `dplyr` will only execute database calls when needed  
(when manipulated is being called eg. by `print()`)




Exercise 4
===

- load all files in `temperature_timeseries` using `ldply()`
- Calculate the annual absolute integrated anomaly for each site (`mutate`)
- Plot the annual absolute integrated anomaly for each site (`qplot`)

<div class="centered">

<script src="countdown.js" type="text/javascript"></script>
<script type="application/javascript">
var myCountdown1 = new Countdown({
    							time: 300, 
									width:150, 
									height:80, 
									rangeHi:"minute"	// <- no comma on last item!
									});

</script>

</div>





Feedback
===

[Feedback form](https://goo.gl/forms/1eCOYxMYKuzDfTu62)
http://goo.gl/forms/3mH1UC0fH3
