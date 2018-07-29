---
layout: post
title: "This is a test post for my R blog"
date: 2016-12-01
categories: rblogging
tags: test ggplot2
---

Setup
-----

Re-creation and expansion of [this blog](https://blog.codecentric.de/en/2017/09/data-science-fraud-detection/)

Get data from [kaggle-data](https://www.kaggle.com/ntnu-testimon/paysim1). Save in to the `\data` folder in the root of the repo and unzip. Re-name to `fd_data.csv`.

Get functions

``` r
# LINE FOR INSTALLING FROM GITHUB HERE
library(JP.fraudDetection)

# remove once dependencies setup
library(data.table)
library(ggplot2)
library(ggridges)
```

Data
----

``` r
# load data
location = file.path(getwd(), "/data")
file = "fd_data.csv"
dt <- fread(file.path(location, file),
            colClasses = list(character=c("type","nameOrig","nameDest"),
                              numeric=c("amount","oldbalanceOrg","newbalanceOrig",
                                        "oldbalanceDest","newbalanceDest"),
                              integer=c("step","isFlaggedFraud"),
                              factor=c("isFraud"))
)
head(dt)
```

    ##    step     type   amount    nameOrig oldbalanceOrg newbalanceOrig
    ## 1:    1  PAYMENT  9839.64 C1231006815        170136      160296.36
    ## 2:    1  PAYMENT  1864.28 C1666544295         21249       19384.72
    ## 3:    1 TRANSFER   181.00 C1305486145           181           0.00
    ## 4:    1 CASH_OUT   181.00  C840083671           181           0.00
    ## 5:    1  PAYMENT 11668.14 C2048537720         41554       29885.86
    ## 6:    1  PAYMENT  7817.71   C90045638         53860       46042.29
    ##       nameDest oldbalanceDest newbalanceDest isFraud isFlaggedFraud
    ## 1: M1979787155              0              0       0              0
    ## 2: M2044282225              0              0       0              0
    ## 3:  C553264065              0              0       1              0
    ## 4:   C38997010          21182              0       1              0
    ## 5: M1230701703              0              0       0              0
    ## 6:  M573487274              0              0       0              0

``` r
dim(dt)
```

    ## [1] 6362620      11

``` r
# imbalance in classes: fraud is uncommon
dt[, .(.N, prop = .N/nrow(dt)), by = isFraud]
```

    ##    isFraud       N       prop
    ## 1:       0 6354407 0.99870918
    ## 2:       1    8213 0.00129082

Plots
-----

sample the data for plotting, to save memory and speed

``` r
p = 0.2
set.seed(1)
sample_dt <- rbindlist(list(dt[isFraud == 1,],
                            dt[isFraud == 0,][sample(1:.N, size = floor(.N*p)),])) 
sample_dt[, logAmount := log(amount + 1)]
```

Can see the amount is higher usually in Fraudulent cases

``` r
pl <- ggplot(sample_dt, aes(x = logAmount, y = isFraud)) + 
  geom_density_ridges(fill = "goldenrod1") + theme_minimal()
pl
```

    ## Picking joint bandwidth of 0.186

![](C:/Users/jonathan.phelps/OneDrive/Documents/Github/jphelps13.github.io/_posts/fraud-detection_files/figure-markdown_github/unnamed-chunk-4-1.png)

Only affects Transfer and Cash\_out types of purchases

``` r
pl <- ggplot(sample_dt, aes(x = logAmount, y = type, fill = isFraud)) + 
  geom_density_ridges(alpha = .6) + theme_minimal()
pl
```

    ## Picking joint bandwidth of 0.153

![](C:/Users/jonathan.phelps/OneDrive/Documents/Github/jphelps13.github.io/_posts/fraud-detection_files/figure-markdown_github/unnamed-chunk-5-1.png)

``` r
prop_types <- dt[type %in% c("TRANSFER", "CASH_OUT"), .N/nrow(dt)]
```

These make up 43.5% of the data
