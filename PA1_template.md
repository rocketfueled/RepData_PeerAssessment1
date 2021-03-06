# Reproducible Research: Peer Assessment 1

## Loading ibraries


```r
library(ggplot2)
library(scales)
library(Hmisc)
```

## Loading and preprocessing the data


```r
if(!file.exists('activity.csv')){
    unzip('activity.zip')
}
activityset <- read.csv('activity.csv')
```

## What is mean total number of steps taken per day?


```r
stepsdaily <- tapply(activityset$steps, activityset$date, sum, na.rm=TRUE)
```

#### Make a histogram of the total number of steps taken each day


```r
qplot(stepsdaily, xlab='Total steps per day', ylab='Frequency using binwith 500', binwidth=500)
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

#### Calculate and report the mean and media total number of steps taken per day


```r
stepsdailymean <- mean(stepsdaily)
stepsdailymedian <- median(stepsdaily)
```
##### stepsdailymean: 9354.23
##### stepsdailymedian: 10395

## What is the average daily activity pattern?


```r
averagepattern <- aggregate(x=list(stepmean=activityset$steps), by=list(interval=activityset$interval), FUN=mean, na.rm=TRUE)
```

#### Make a time series plot


```r
ggplot(data=averagepattern, aes(x=interval, y=stepmean)) +
    geom_line() +
    xlab("5-minute interval") +
    ylab("average number of steps taken") 
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

#### Which 5-minute interval, on average accross all the days in the dataset, contains the maximum number of steps?


```r
maxsteps <- which.max(averagepattern$stepmean)
maxstepstiming <-  gsub("([0-9]{1,2})([0-9]{2})", "\\1:\\2", averagepattern[maxsteps,'interval'])
```
##### maximum number of step interval time: 8:35

## Imputing missing values

##### Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
missingvalues <- length(which(is.na(activityset$steps)))
```

#### Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
activitysetimpute <- activityset
activitysetimpute$steps <- impute(activityset$steps, fun=mean)
```

#### Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
imputestepsdaily <- tapply(activitysetimpute$steps, activitysetimpute$date, sum)
qplot(imputestepsdaily, xlab='Total steps per day (Imputed)', ylab='Frequency using binwith 500', binwidth=500)
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

```r
imputestepsdailymean <- mean(imputestepsdaily)
imputestepsdailymedian <- median(imputestepsdaily)
```

## Are there differences in activity patterns between weekdays and weekends?

#### Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.


```r
activitysetimpute$dateType <-  ifelse(as.POSIXlt(activitysetimpute$date)$wday %in% c(0,6), 'weekend', 'weekday')
```

#### Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).


```r
activitysetimputeavg <- aggregate(steps ~ interval + dateType, data=activitysetimpute, mean)
ggplot(activitysetimputeavg, aes(interval, steps)) + 
    geom_line() + 
    facet_grid(dateType ~ .) +
    xlab("5-minute interval") + 
    ylab("avarage number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-13-1.png)<!-- -->
