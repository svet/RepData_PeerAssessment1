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

The mean steps per day is 1.0766 &times; 10<sup>4</sup> (shown in blue) and the median is 10765 (shown in green)


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



## Are there differences in activity patterns between weekdays and weekends?
