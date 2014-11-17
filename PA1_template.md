# Reproducible Research: Peer Assessment 1
Marek Schmidt  
Sunday, November 16, 2014  


## Loading and preprocessing the data

First load data from default directory path, file 'activity.csv' (downloaded & unzipped before):


```r
md <- read.csv("activity.csv")
observations <- nrow(md)
```

There are 17568 observations. Clean data by removing NAs (not measured steps during interval):


```r
mdrm <- md[!is.na(md$steps),]
observations_clean <- nrow(mdrm)
```

Now we have 15264 observations.  

## What is mean total number of steps taken per day?

Preparing data into md_hist data.frame: sum of steps grouped by day:


```r
library(reshape2)
md_prep <- melt(mdrm, id="date", measure.vars="steps")
md_hist <- dcast(md_prep, date ~ variable, sum)
```

And print histogram from only non-empty days (NAs were removed):


```r
library(ggplot2)
qplot(md_hist$date, md_hist$steps, geom="histogram", stat = "identity", xlab="Days", ylab="Sum of Steps")
```

![plot of chunk unnamed-chunk-4](./PA1_template_files/figure-html/unnamed-chunk-4.png) 

Now calculate mean & median for 'Sum of steps per day' (without NAs):


```r
mdrm_mean <- mean(md_hist$steps)
mdrm_median <- as.numeric(median(md_hist$steps))
```

Mean value is 1.0766 &times; 10<sup>4</sup> and median value is 1.0765 &times; 10<sup>4</sup>.

## What is the average daily activity pattern?

Now we prepare time series plot of the 5-minute interval vs. average # of steps


```r
md_prep2 <- melt(mdrm, id="interval", measure.vars="steps")
md_plot <- dcast(md_prep2, interval ~ variable, mean)
plot(md_plot, type="l", ylab="AVG # of steps")
```

![plot of chunk unnamed-chunk-6](./PA1_template_files/figure-html/unnamed-chunk-6.png) 

Find maximum number of steps and associated interval:


```r
max_steps <- max(md_plot$steps)
max_interval <- md_plot[md_plot$steps==max_steps,"interval"]
```

Maximum average of steps was 206.1698 and it was in interval: 835.

## Imputing missing values

1. Now calculate total missing values (NAs):


```r
missing <- sum(is.na(md$steps))
```

There is 2304 values where steps were not measured.

2. Our strategy for missing values will be to calculate average value from available data per interval and round it to integer, so we utilize md_plot and fill values rounded, eg. for particular interval it will be: missing_steps = round(md_plot$steps).

3. New dataset will be (we replace NAs with rounded averages from md_plot using inline function):


```r
md_fill <- md[is.na(md$steps),]
md_fill$steps <- as.integer(lapply(md_fill$interval, function(x) {round(md_plot[md_plot$interval == x,"steps"])} ) )
md_imputs <- rbind(mdrm, md_fill)
mdfc <- nrow(md_imputs)
```


Total count of new dataset called md_imputs  is 17568. Now once again we print histogram from new dataset:

Preparing data into md_hist data.frame: sum of steps grouped by day:


```r
md_prep <- melt(md_imputs, id="date", measure.vars="steps")
md_hist2 <- dcast(md_prep, date ~ variable, sum)
```

And print histogram from only non-empty days (NAs were removed):


```r
library(ggplot2)
qplot(md_hist2$date, md_hist2$steps, geom="histogram", stat = "identity", xlab="Days", ylab="Sum of Steps")
```

![plot of chunk unnamed-chunk-11](./PA1_template_files/figure-html/unnamed-chunk-11.png) 

Now calculate mean & median for 'Sum of steps per day' (without NAs):


```r
mdrm_mean2 <- mean(md_hist2$steps)
mdrm_median2 <- as.numeric(median(md_hist2$steps))
```

Mean value is 1.0766 &times; 10<sup>4</sup> and median value is 1.0762 &times; 10<sup>4</sup>. Impact of imputed data is: new median is smaller and mean is as before.

## Are there differences in activity patterns between weekdays and weekends?



