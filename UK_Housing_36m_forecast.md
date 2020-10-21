ARIMA UK Housing index forecasting
================

<style>

  .col2 {
    columns: 2 200px;         /* number of columns and width in pixels*/
    -webkit-columns: 2 200px; /* chrome, safari */
    -moz-columns: 2 200px;    /* firefox */
  }
  .col3 {
    columns: 3 100px;
    -webkit-columns: 3 100px;
    -moz-columns: 3 100px;
  }
</style>

## Libraries

``` r
#install.packages(c("kableExtra","tidyverse","patchwork","bestNormalize","caret","forecast","tseries"))
library(kableExtra)
library(tidyverse)
library(patchwork)
library(bestNormalize)
library(caret)
library(knitr)
library(ggpubr)
library(grid)
library(gridExtra)
require(forecast)
require(tseries)
```

## Data

We used open source data on the UK Housing Price Index (HPI); 1970 to
August 2020. (Note: the House type cannot be differentiated until 2005)

The UK House Price Index (HPI) uses house sales data from HM Land
Registry, Registers of Scotland, and Land and Property Services Northern
Ireland and is calculated by the Office for National Statistics. The
index applies a statistical method, called a hedonic regression model,
to the various sources of data on property price and attributes to
produce estimates of the change in house prices each period.

As of July 2020 the average house price in the UK is `£237,963`, and the
index stands at `124.81`. Property prices have risen by `0.5%` compared
to the previous month, and risen by `2.3%` compared to the previous
year.

You can verify the data here
<https://landregistry.data.gov.uk/app/ukhpi>

<div class="col2">

``` r
head(df[,1:2]) %>%
  kbl() %>%
  kable_styling()
```

<table class="table" style="margin-left: auto; margin-right: auto;">

<thead>

<tr>

<th style="text-align:left;">

</th>

<th style="text-align:left;">

Period

</th>

<th style="text-align:right;">

Average.price.All.property.types

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

22

</td>

<td style="text-align:left;">

1970-01

</td>

<td style="text-align:right;">

3920

</td>

</tr>

<tr>

<td style="text-align:left;">

23

</td>

<td style="text-align:left;">

1970-02

</td>

<td style="text-align:right;">

3920

</td>

</tr>

<tr>

<td style="text-align:left;">

24

</td>

<td style="text-align:left;">

1970-03

</td>

<td style="text-align:right;">

3920

</td>

</tr>

<tr>

<td style="text-align:left;">

25

</td>

<td style="text-align:left;">

1970-04

</td>

<td style="text-align:right;">

3980

</td>

</tr>

<tr>

<td style="text-align:left;">

26

</td>

<td style="text-align:left;">

1970-05

</td>

<td style="text-align:right;">

3980

</td>

</tr>

<tr>

<td style="text-align:left;">

27

</td>

<td style="text-align:left;">

1970-06

</td>

<td style="text-align:right;">

3980

</td>

</tr>

</tbody>

</table>

``` r
tail(df[,1:2]) %>%
  kbl() %>%
  kable_styling()
```

<table class="table" style="margin-left: auto; margin-right: auto;">

<thead>

<tr>

<th style="text-align:left;">

</th>

<th style="text-align:left;">

Period

</th>

<th style="text-align:right;">

Average.price.All.property.types

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

623

</td>

<td style="text-align:left;">

2020-02

</td>

<td style="text-align:right;">

231162

</td>

</tr>

<tr>

<td style="text-align:left;">

624

</td>

<td style="text-align:left;">

2020-03

</td>

<td style="text-align:right;">

234464

</td>

</tr>

<tr>

<td style="text-align:left;">

625

</td>

<td style="text-align:left;">

2020-04

</td>

<td style="text-align:right;">

230533

</td>

</tr>

<tr>

<td style="text-align:left;">

626

</td>

<td style="text-align:left;">

2020-05

</td>

<td style="text-align:right;">

231778

</td>

</tr>

<tr>

<td style="text-align:left;">

627

</td>

<td style="text-align:left;">

2020-06

</td>

<td style="text-align:right;">

236798

</td>

</tr>

<tr>

<td style="text-align:left;">

628

</td>

<td style="text-align:left;">

2020-07

</td>

<td style="text-align:right;">

237963

</td>

</tr>

</tbody>

</table>

</div>

``` r
summary(df[,2:5]) %>% 
  kbl() %>% 
  kable_styling()
```

<table class="table" style="margin-left: auto; margin-right: auto;">

<thead>

<tr>

<th style="text-align:left;">

</th>

<th style="text-align:left;">

Average.price.All.property.types

</th>

<th style="text-align:left;">

Average.price.Detached.houses

</th>

<th style="text-align:left;">

Average.price.Semi.detached.houses

</th>

<th style="text-align:left;">

Average.price.Terraced.houses

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

</td>

<td style="text-align:left;">

Min. : 3920

</td>

<td style="text-align:left;">

Min. :234509

</td>

<td style="text-align:left;">

Min. :145913

</td>

<td style="text-align:left;">

Min. :119434

</td>

</tr>

<tr>

<td style="text-align:left;">

</td>

<td style="text-align:left;">

1st Qu.: 22360

</td>

<td style="text-align:left;">

1st Qu.:258520

</td>

<td style="text-align:left;">

1st Qu.:159793

</td>

<td style="text-align:left;">

1st Qu.:136399

</td>

</tr>

<tr>

<td style="text-align:left;">

</td>

<td style="text-align:left;">

Median : 58250

</td>

<td style="text-align:left;">

Median :270720

</td>

<td style="text-align:left;">

Median :169788

</td>

<td style="text-align:left;">

Median :145778

</td>

</tr>

<tr>

<td style="text-align:left;">

</td>

<td style="text-align:left;">

Mean : 89710

</td>

<td style="text-align:left;">

Mean :286678

</td>

<td style="text-align:left;">

Mean :178871

</td>

<td style="text-align:left;">

Mean :153030

</td>

</tr>

<tr>

<td style="text-align:left;">

</td>

<td style="text-align:left;">

3rd Qu.:167017

</td>

<td style="text-align:left;">

3rd Qu.:322865

</td>

<td style="text-align:left;">

3rd Qu.:201184

</td>

<td style="text-align:left;">

3rd Qu.:173246

</td>

</tr>

<tr>

<td style="text-align:left;">

</td>

<td style="text-align:left;">

Max. :237963

</td>

<td style="text-align:left;">

Max. :361986

</td>

<td style="text-align:left;">

Max. :227832

</td>

<td style="text-align:left;">

Max. :193619

</td>

</tr>

<tr>

<td style="text-align:left;">

</td>

<td style="text-align:left;">

NA

</td>

<td style="text-align:left;">

NA’s :420

</td>

<td style="text-align:left;">

NA’s :420

</td>

<td style="text-align:left;">

NA’s :420

</td>

</tr>

</tbody>

</table>

## Building a time series for all House types

<div class="col1">

``` r
names(df)[2] <- "Units"
dfUnits <- df$Units
tdf <- ts(dfUnits, start = c(1970, 1), frequency = 12)
head(tdf,12)
```

    ##       Jan  Feb  Mar  Apr  May  Jun  Jul  Aug  Sep  Oct  Nov  Dec
    ## 1970 3920 3920 3920 3980 3980 3980 4163 4163 4163 4163 4163 4163

</div>

## Exploratory ARIMA

``` r
fit <- auto.arima(tdf)
fit
```

    ## Series: tdf 
    ## ARIMA(3,2,2)(0,0,2)[12] 
    ## 
    ## Coefficients:
    ##           ar1      ar2      ar3     ma1     ma2    sma1    sma2
    ##       -1.4200  -1.1652  -0.4699  0.7083  0.1996  0.3932  0.2108
    ## s.e.   0.1238   0.1091   0.0651  0.1287  0.0932  0.0431  0.0360
    ## 
    ## sigma^2 estimated as 992508:  log likelihood=-5033.59
    ## AIC=10083.18   AICc=10083.42   BIC=10118.42

``` r
accfit <-accuracy(fit) 
accfit
```

    ##                    ME     RMSE      MAE       MPE      MAPE      MASE
    ## Training set 5.960677 988.8338 611.1948 0.0156202 0.8465972 0.1014046
    ##                      ACF1
    ## Training set -0.005331425

<div class="col2">

``` r
pred_values <- forecast(fit, 12)

plot(pred_values,
     
     xlab = "Date", 
     ylab = "Price/£",
     
     
     main = "ARIMA 12m Forecast for House price Index"
     
     
     )
```

![](UK_Housing_36m_forecast_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

``` r
plot(pred_values$residuals, 
     xlab = "Date", 
     ylab = "",
     
     
     main = "ARIMA Forecast Residuals"
     
     
     )
```

![](UK_Housing_36m_forecast_files/figure-gfm/unnamed-chunk-8-2.png)<!-- -->

``` r
pred_values  %>% kbl() %>%  kable_styling()
```

<table class="table" style="margin-left: auto; margin-right: auto;">

<thead>

<tr>

<th style="text-align:left;">

</th>

<th style="text-align:right;">

Point Forecast

</th>

<th style="text-align:right;">

Lo 80

</th>

<th style="text-align:right;">

Hi 80

</th>

<th style="text-align:right;">

Lo 95

</th>

<th style="text-align:right;">

Hi 95

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

Aug 2020

</td>

<td style="text-align:right;">

238800.4

</td>

<td style="text-align:right;">

237523.6

</td>

<td style="text-align:right;">

240077.1

</td>

<td style="text-align:right;">

236847.7

</td>

<td style="text-align:right;">

240753.0

</td>

</tr>

<tr>

<td style="text-align:left;">

Sep 2020

</td>

<td style="text-align:right;">

242541.7

</td>

<td style="text-align:right;">

240459.5

</td>

<td style="text-align:right;">

244623.9

</td>

<td style="text-align:right;">

239357.3

</td>

<td style="text-align:right;">

245726.2

</td>

</tr>

<tr>

<td style="text-align:left;">

Oct 2020

</td>

<td style="text-align:right;">

243261.9

</td>

<td style="text-align:right;">

240325.6

</td>

<td style="text-align:right;">

246198.2

</td>

<td style="text-align:right;">

238771.2

</td>

<td style="text-align:right;">

247752.6

</td>

</tr>

<tr>

<td style="text-align:left;">

Nov 2020

</td>

<td style="text-align:right;">

243965.6

</td>

<td style="text-align:right;">

239857.4

</td>

<td style="text-align:right;">

248073.7

</td>

<td style="text-align:right;">

237682.7

</td>

<td style="text-align:right;">

250248.4

</td>

</tr>

<tr>

<td style="text-align:left;">

Dec 2020

</td>

<td style="text-align:right;">

246361.5

</td>

<td style="text-align:right;">

240964.4

</td>

<td style="text-align:right;">

251758.6

</td>

<td style="text-align:right;">

238107.3

</td>

<td style="text-align:right;">

254615.7

</td>

</tr>

<tr>

<td style="text-align:left;">

Jan 2021

</td>

<td style="text-align:right;">

248284.1

</td>

<td style="text-align:right;">

241619.3

</td>

<td style="text-align:right;">

254948.8

</td>

<td style="text-align:right;">

238091.2

</td>

<td style="text-align:right;">

258476.9

</td>

</tr>

<tr>

<td style="text-align:left;">

Feb 2021

</td>

<td style="text-align:right;">

248775.2

</td>

<td style="text-align:right;">

240645.2

</td>

<td style="text-align:right;">

256905.2

</td>

<td style="text-align:right;">

236341.4

</td>

<td style="text-align:right;">

261209.0

</td>

</tr>

<tr>

<td style="text-align:left;">

Mar 2021

</td>

<td style="text-align:right;">

252361.2

</td>

<td style="text-align:right;">

242677.7

</td>

<td style="text-align:right;">

262044.7

</td>

<td style="text-align:right;">

237551.6

</td>

<td style="text-align:right;">

267170.9

</td>

</tr>

<tr>

<td style="text-align:left;">

Apr 2021

</td>

<td style="text-align:right;">

252469.5

</td>

<td style="text-align:right;">

241193.9

</td>

<td style="text-align:right;">

263745.0

</td>

<td style="text-align:right;">

235225.0

</td>

<td style="text-align:right;">

269713.9

</td>

</tr>

<tr>

<td style="text-align:left;">

May 2021

</td>

<td style="text-align:right;">

254329.9

</td>

<td style="text-align:right;">

241352.3

</td>

<td style="text-align:right;">

267307.4

</td>

<td style="text-align:right;">

234482.4

</td>

<td style="text-align:right;">

274177.3

</td>

</tr>

<tr>

<td style="text-align:left;">

Jun 2021

</td>

<td style="text-align:right;">

258179.8

</td>

<td style="text-align:right;">

243408.7

</td>

<td style="text-align:right;">

272950.9

</td>

<td style="text-align:right;">

235589.3

</td>

<td style="text-align:right;">

280770.3

</td>

</tr>

<tr>

<td style="text-align:left;">

Jul 2021

</td>

<td style="text-align:right;">

260315.1

</td>

<td style="text-align:right;">

243704.3

</td>

<td style="text-align:right;">

276925.9

</td>

<td style="text-align:right;">

234911.0

</td>

<td style="text-align:right;">

285719.1

</td>

</tr>

</tbody>

</table>

</div>

``` r
UK_Housing_Index <- window(tdf, start = tail(time(tdf), 67)[1])
ggseasonplot(UK_Housing_Index, year.labels = TRUE, col = rainbow(20))
```

![](UK_Housing_36m_forecast_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

<div class="col2">

``` r
qqnorm(fit$residuals)
```

![](UK_Housing_36m_forecast_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

``` r
Box.test(fit$residuals, type = "Ljung-Box")
```

    ## 
    ##  Box-Ljung test
    ## 
    ## data:  fit$residuals
    ## X-squared = 0.017339, df = 1, p-value = 0.8952

``` r
fit$loglik
```

    ## [1] -5033.588

</div>

The data has a high p-value, so the autocorrelations not significantly
different than 0. We apply a log transformation to force normality.

``` r
ltdf <- log(tdf)
head(ltdf,12)
```

    ##           Jan      Feb      Mar      Apr      May      Jun      Jul      Aug
    ## 1970 8.273847 8.273847 8.273847 8.289037 8.289037 8.289037 8.333991 8.333991
    ##           Sep      Oct      Nov      Dec
    ## 1970 8.333991 8.333991 8.333991 8.333991

Refitting log transformation with seasonal decomposition

``` r
fit2 <- stl(ltdf, s.window = "period")
plot(fit2, main = "Seasonal Decomposition of log(Units)")
```

![](UK_Housing_36m_forecast_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

## ARIMA model

``` r
fit3 <- auto.arima(ltdf)
fitAccuracy <- data.frame(accuracy(fit))
fitAccuracy2 <- data.frame(accuracy(fit3))
fitAccuracyFinal <- rbind(fitAccuracy, fitAccuracy2)
fitAccuracyFinal %>% 
  kbl() %>% 
  kable_styling()
```

<table class="table" style="margin-left: auto; margin-right: auto;">

<thead>

<tr>

<th style="text-align:left;">

</th>

<th style="text-align:right;">

ME

</th>

<th style="text-align:right;">

RMSE

</th>

<th style="text-align:right;">

MAE

</th>

<th style="text-align:right;">

MPE

</th>

<th style="text-align:right;">

MAPE

</th>

<th style="text-align:right;">

MASE

</th>

<th style="text-align:right;">

ACF1

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

Training set

</td>

<td style="text-align:right;">

5.9606767

</td>

<td style="text-align:right;">

988.8338010

</td>

<td style="text-align:right;">

611.194813

</td>

<td style="text-align:right;">

0.0156202

</td>

<td style="text-align:right;">

0.8465972

</td>

<td style="text-align:right;">

0.1014046

</td>

<td style="text-align:right;">

\-0.0053314

</td>

</tr>

<tr>

<td style="text-align:left;">

Training set1

</td>

<td style="text-align:right;">

0.0000203

</td>

<td style="text-align:right;">

0.0118251

</td>

<td style="text-align:right;">

0.007403

</td>

<td style="text-align:right;">

0.0002833

</td>

<td style="text-align:right;">

0.0693741

</td>

<td style="text-align:right;">

0.0800607

</td>

<td style="text-align:right;">

\-0.0197401

</td>

</tr>

</tbody>

</table>

<div class="col2">

``` r
qqnorm(fit3$residuals)
```

![](UK_Housing_36m_forecast_files/figure-gfm/unnamed-chunk-14-1.png)<!-- -->

``` r
Box.test(fit3$residuals, type = "Ljung-Box")
```

    ## 
    ##  Box-Ljung test
    ## 
    ## data:  fit3$residuals
    ## X-squared = 0.2377, df = 1, p-value = 0.6259

``` r
fit3$loglik
```

    ## [1] 1823.513

</div>

``` r
plot(forecast(fit3, 12), xlab = "Date", ylab = "Units", main = "ARIMA Forecast for House price Index")
```

![](UK_Housing_36m_forecast_files/figure-gfm/unnamed-chunk-15-1.png)<!-- -->

``` r
pred_values <- data.frame(forecast(fit, 12))
pred_values2 <- data.frame(forecast(fit3, 12))
pred_values2[,1:5] <- exp(pred_values2[,1:5])
mergedDF <- data.frame(Date = rownames(pred_values), Original_Data_Forecast = pred_values$Point.Forecast, Log_Transformed_Data_Forecast = pred_values2$Point.Forecast, Difference = round(pred_values$Point.Forecast - pred_values2$Point.Forecast, 2))
mergedDF
```

    ##        Date Original_Data_Forecast Log_Transformed_Data_Forecast Difference
    ## 1  Aug 2020               238800.4                      239599.2    -798.86
    ## 2  Sep 2020               242541.7                      243659.1   -1117.33
    ## 3  Oct 2020               243261.9                      244240.2    -978.31
    ## 4  Nov 2020               243965.6                      245553.1   -1587.55
    ## 5  Dec 2020               246361.5                      248680.4   -2318.91
    ## 6  Jan 2021               248284.1                      250335.2   -2051.12
    ## 7  Feb 2021               248775.2                      251413.8   -2638.55
    ## 8  Mar 2021               252361.2                      255828.3   -3467.11
    ## 9  Apr 2021               252469.5                      255850.1   -3380.66
    ## 10 May 2021               254329.9                      258257.9   -3927.98
    ## 11 Jun 2021               258179.8                      263032.1   -4852.29
    ## 12 Jul 2021               260315.1                      265320.2   -5005.15

``` r
write.csv(mergedDF,"Out.csv")
```

``` r
p1<-autoplot(ltdf)
p2<-autoplot(stl(ltdf, s.window="periodic", robust=TRUE))
p3<-autoplot(fit3)
p4<-ggtsdisplay(ltdf)
```

``` r
ggarrange(p1,p2,p3,p4)
```

![](UK_Housing_36m_forecast_files/figure-gfm/unnamed-chunk-18-1.png)<!-- -->

``` r
UK_Housing_Index <- window(ltdf, start = tail(time(ltdf), 67)[1])
tfitAccuracy2 <- t(fitAccuracy2)
a<-autoplot(UK_Housing_Index) + geom_forecast(h=36)+
  
  ggtitle("ARIMA Average UK House Price 36 month Forecast")+xlab("Year") +ylab("log(Price/£)")+
  
  annotation_custom(tableGrob(tfitAccuracy2), xmin=2020, xmax=2024, ymin=12.2, ymax=12.3)
a
```

![](UK_Housing_36m_forecast_files/figure-gfm/unnamed-chunk-19-1.png)<!-- -->

Bringing in the property type split.

``` r
# Detached
dfd<-df
names(dfd)[3] <- "Units2"
dfdUnits2 <- dfd$Units2
tdfd <- ts(dfdUnits2, start = c(1970, 1), frequency = 12)
ltdfd <- log(tdfd)
fit3d <- auto.arima(ltdfd)
subtsd<-(window(ltdfd, start = tail(time(tdfd), 67)[1]))
# Semi-
dfsd<-df
names(dfsd)[4] <- "Units3"
dfsdUnits3 <- dfsd$Units3
tdfsd <- ts(dfsdUnits3, start = c(1970, 1), frequency = 12)
ltdfsd <- log(tdfsd)
fit3sd <- auto.arima(ltdfsd)
subtsd<-(window(ltdfsd, start = tail(time(tdfsd), 67)[1]))
# Terraced
dft<-df
names(dft)[5] <- "Units4"
dftUnits4 <- dft$Units4
tdft <- ts(dftUnits4, start = c(1970, 1), frequency = 12)
ltdft <- log(tdft)
fit3t <- auto.arima(ltdft)
subtsd<-(window(ltdft, start = tail(time(tdft), 67)[1]))

# Charts

a2<-autoplot(UK_Housing_Index) + geom_forecast(h=36)+
  
  ggtitle("Average UK House Price ")+xlab("Year") +ylab("log(Price/£)")

b<-autoplot(subtsd) + geom_forecast(h=36)+
  
  ggtitle("Average UK Detached House Price")+xlab("Year") +ylab("log(Price/£)")

c<-autoplot(subtsd) + geom_forecast(h=36)+
  
  ggtitle("Average UK Semi-Detached House Price")+xlab("Year") +ylab("log(Price/£)")

d<-autoplot(subtsd) + geom_forecast(h=36)+
  
  ggtitle("Average UK Terraced House Price")+xlab("Year") +ylab("log(Price/£)")
```

``` r
figure<-ggarrange(a2, b, c,d, ncol = 2, nrow = 2)
annotate_figure(figure,
                top = text_grob("Average UK House Price ARIMA 36-month Forecast\n    ", face = "bold", size = 15))
```

![](UK_Housing_36m_forecast_files/figure-gfm/unnamed-chunk-21-1.png)<!-- -->
