# ASN5.3. Paper Replication: Make Table 1
2024 FIRE198 Sustainability Analytics

In this assignment you will use the functions that you have learned to
replicate the Pollution emissions and Air quality section of Table 1 in
*Defensive Investments and the Demand for Air Quality: Evidence from the
NOx Budget Program* by Deschênes et al. (2017).

<img src="T1.PNG" data-fig-align="center" width="400" />

#### Step 1 (1 point)

Declare that you will use the **tidyverse** package.

``` r
library("tidyverse")
```

    ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ✔ dplyr     1.1.1     ✔ readr     2.1.4
    ✔ forcats   1.0.0     ✔ stringr   1.5.0
    ✔ ggplot2   3.5.0     ✔ tibble    3.2.1
    ✔ lubridate 1.9.3     ✔ tidyr     1.3.0
    ✔ purrr     1.0.1     
    ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ✖ dplyr::filter() masks stats::filter()
    ✖ dplyr::lag()    masks stats::lag()
    ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

#### Step 2 (2 points)

Upload pollution emissions data from the year 2001 to 2007 into the
environment and combine them together into one dataframe called **df.**

``` r
df2001<-read.csv("EPA AMPD/emission_2001.csv")
df2002<-read.csv("EPA AMPD/emission_2002.csv")
df2003<-read.csv("EPA AMPD/emission_2003.csv")
df2004<-read.csv("EPA AMPD/emission_2004.csv")
df2005<-read.csv("EPA AMPD/emission_2005.csv")
df2006<-read.csv("EPA AMPD/emission_2006.csv")
df2007<-read.csv("EPA AMPD/emission_2007.csv")

df<-rbind(df2001, df2002, df2003, df2004, df2005, df2006, df2007)
```

#### Question 1: What is the unit of NOx, SO2, and CO2 pollution emissions in Table 1? Fill in the blank. **(3 points)**

#### Answer: \_Thousand\_ tons per \_county_during the \_summer_months of each year.

#### Step 3 (4 points)

Create a new dataframe named **df2** which has the total **summertime**
NOx, SO2, and CO2 emissions (in **thousand tons**) for each **County**,
**State**, and **Year**. Note that the summer months include May through
September.

``` r
df2<-df %>%
  filter(Month>=5 & Month<=9) %>%
  mutate(NOx=ifelse(!is.na(NOx..tons.), NOx..tons., 0)) %>%
  mutate(SO2=ifelse(!is.na(SO2..tons.), SO2..tons., 0)) %>%
  mutate(CO2=ifelse(!is.na(CO2..short.tons.), CO2..short.tons., 0)) %>%
  group_by(County, State, Year) %>%
  summarize(NOx=sum(NOx)/1000, SO2=sum(SO2)/1000, CO2=sum(CO2)/1000)
```

    `summarise()` has grouped output by 'County', 'State'. You can override using
    the `.groups` argument.

#### Step 4 (1 point)

Upload the **counties_t2a.csv** file into the environment. Name the
dataframe **counties**. This dataframe is the list of all counties that
satisfies the selection criteria in the paper *Defensive Investments and
the Demand for Air Quality: Evidence from the NOx Budget Program* as
described below.

<img src="emit_data.JPG" data-fig-align="center" width="400" />

``` r
counties<-read.csv("counties_t2a.csv")
```

#### Step 5

To make an accurate summary statistics table, we need to make a balanced
panel with all the counties and year that we are interested in. The
table will summarize data from all counties and all years.

This can be done by first creating a vector of all the years we are
interested in. Afterward, we will use the **crossing** function to
create a combination of all the counties and year we are interested in.
Since we have 7 years and 2253 counties, the balanced panel will have 7
x 2,253 = 15,771 observations.

The code is shown below. You just have to run it.

``` r
Year<-seq(2001,2007,by=1)

cy<-crossing(counties, Year)
```

Next, we will add the emissions data from **df2** into **cy**.

#### Question 2: What three variables do df2 and cy have in common? (3 points)

#### Answer: State, County, Year

#### Step 6 (4 points)

Add the emissions data from **df2** into **cy** by using the **merge**
function by matching observations that has the same **State**,
**County**, and **Year**.

Name the merged dataframe **emit.**

Make sure that the dataframe **emit** has all the observations in
**cy**, but not the observations in **df2** that does not match with
**cy**.

``` r
emit<-merge(cy, df2, by=c("State", "County", "Year"), all.x=TRUE)
```

#### Step 7 (2 points)

Create a new dataframe named **emit_mean** from **emit.**

Replace all the NA values that the columns that represent NOx, SO2, and
CO2 emissions with 0.

Afterward, use the **summarize** and **mean** function to find the
average NOx, SO2, and CO2 emissions across all counties and years. Name
the variables **mean_NOx**, **mean_SO2**, and **mean_CO2**.

``` r
emit_mean<-emit %>%
  mutate(NOx=ifelse(!is.na(NOx), NOx, 0)) %>%
  mutate(SO2=ifelse(!is.na(SO2), SO2, 0)) %>%
  mutate(CO2=ifelse(!is.na(CO2), CO2, 0)) %>%
  summarize(mean_SO2=mean(SO2), mean_CO2=mean(CO2), mean_NOx=mean(NOx))
```

#### Step 8 (2 points)

Create a new dataframe named **emit_sd** from **emit.**

Replace all the NA values that the columns that represent NOx, SO2, and
CO2 emissions with 0.

Afterward, use the **summarize** and **sd** function to find the
standard deviation of NOx, SO2, and CO2 emissions across all counties
and years. Name the variables **sd_NOx**, **sd_SO2**, and **sd_CO2**.

``` r
emit_sd<-emit %>%
  mutate(NOx=ifelse(!is.na(NOx), NOx, 0)) %>%
  mutate(SO2=ifelse(!is.na(SO2), SO2, 0)) %>%
  mutate(CO2=ifelse(!is.na(CO2), CO2, 0)) %>%
  summarize(sd_SO2=sd(SO2), sd_CO2=sd(CO2), sd_NOx=sd(NOx))
```

#### Step 9

We will use the **pivot_longer** function to format our results in table
format of the original paper. The code is shown below. You just have to
run it.

``` r
mean_long<-pivot_longer(emit_mean,
                        cols=1:3,
                        names_to = "Pollution emissions (000's of tons/summer)",
                        names_prefix="mean_", 
                        values_to = "Mean") 

sd_long<-pivot_longer(emit_sd,
                        cols=1:3,
                        names_to = "Pollution emissions (000's of tons/summer)",
                        names_prefix="sd_", 
                        values_to = "SD") 

T1a<-merge(mean_long, sd_long, 
           by="Pollution emissions (000's of tons/summer)") %>%
  mutate(n=nrow(counties)) 

print(T1a)
```

      Pollution emissions (000's of tons/summer)        Mean          SD    n
    1                                        CO2 378.5097853 1296.843658 2253
    2                                        NOx   0.5380792    2.015730 2253
    3                                        SO2   1.5561285    6.692972 2253

#### Question 3: Compare the mean of NOx, standard deviation of NOx, and counties with data to the numbers presented in the original paper. Which ones are greater? (3 points)

#### Answer: Our mean is larger, our sd is larger, but the number of counties we have is less.

#### Step 10: Click Render to generate an html file of this document. **(1 point)**

You have reached the end of the assignment. Save the Quarto document and
push the completed assignment back into the GitHub repository.
