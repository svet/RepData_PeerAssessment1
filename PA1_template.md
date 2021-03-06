---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data
Load the source CSV file setting the locale to English (my default locale is Spanish)


```r
activity = read.csv("activity.csv", header=T, colClasses=c("integer", "Date", "integer"))
Sys.setlocale("LC_TIME", "en_US")
```

```
## [1] "en_US"
```

## What is mean total number of steps taken per day?

For data summarization we will use the dplyr library

```r
library(dplyr)
```


```r
activity_by_day = group_by(activity, date)
totals_by_day = summarise(activity_by_day, tot=sum(steps))
hist(totals_by_day$tot, main = 'Total Steps per Day Histogram', 
                        xlab = 'Total number of steps per day')
mean_steps = mean(totals_by_day$tot, na.rm = TRUE)
median_steps = median(totals_by_day$tot, na.rm = TRUE)
abline(v=mean_steps, col="blue")
abline(v=median_steps, col="green")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 

The mean steps per day is 1.0766 &times; 10<sup>4</sup> (shown in blue) and 
the median is 10765 (shown in green)


## What is the average daily activity pattern?


```r
activity_by_interval = group_by(activity, interval)
ave_by_interval = summarise(activity_by_interval, ave=mean(steps, na.rm=TRUE))
plot(ave ~ interval, data = ave_by_interval, type = 'l', 
                     main = 'Average Steps per Interval',
                     ylab = 'Steps')
max_activity_interval = ave_by_interval[which.max(ave_by_interval$ave),]$interval
abline(v=max_activity_interval, col="red")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 

On average the interval with highest number of steps is 835 (shown in red)


## Imputing missing values

Number of missing values in the datasrt:

```r
sum(is.na(activity$steps))
```

```
## [1] 2304
```

To fill the missing values we will use the average of the same interval from another days. 
Actually we already have these avarages already calculated above. Now we just define a 
function which returns this average for given interval


```r
ave_steps_for_interval = function (interval){
  ave_by_interval[which(ave_by_interval$interval == interval),]$ave
}
```

Now create a new extended activity data frame

```r
activity_extended = activity
```

Now update the missing value with the average number of steps for the same interval

```r
activity_extended[which(is.na(activity_extended)),]$steps = sapply(
         activity_extended[which(is.na(activity_extended)),]$interval, 
         FUN=ave_steps_for_interval
      )
```

Now let calculate againg the histogram of steps per day, mean and median with the new extended dataframe: 

```r
activity_by_day_ex = group_by(activity_extended, date)
totals_by_day_ex = summarise(activity_by_day_ex, tot=sum(steps))
hist(totals_by_day_ex$tot, main = 'Total Steps per Day Histogram (extended data)', 
                           xlab = 'Total number of steps per day')
mean_steps_ex = mean(totals_by_day_ex$tot, na.rm = TRUE)
median_steps_ex = median(totals_by_day_ex$tot, na.rm = TRUE)
abline(v=mean_steps_ex, col="blue")
abline(v=median_steps_ex, col="green")
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9.png) 

The mean steps per day is 1.0766 &times; 10<sup>4</sup> (shown in blue) and 
the median is 1.0766 &times; 10<sup>4</sup> (shown in green).

There is no significant change in the values for the extended data set 
(actually this is due to the method used to fill na NAs).

## Are there differences in activity patterns between weekdays and weekends?

Add new factor variable with levels "weekend" and "weekday"

```r
activity_extended = mutate(activity_extended, 
                            daytype = ifelse( strftime(date, "%u") >= 6, "weekend", "weekday"))
activity_extended$daytype <- as.factor(activity_extended$daytype)
```

Difference between weekend and weekdays:

```r
library(lattice)
ave_int <- aggregate(steps ~ interval + daytype,
                     data = activity_extended, FUN = 'mean')

xyplot(steps ~ interval | factor(daytype), 
         data = ave_int, layout=c(1,2), type = 'l')
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11.png) 



