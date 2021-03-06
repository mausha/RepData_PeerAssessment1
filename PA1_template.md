---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

## Loading and preprocessing the data

```r
library(dplyr)
library(lubridate)
library(ggplot2)
```

```r
if (!file.exists("activity.csv")) unzip("activity.zip")
activity <- read.csv("activity.csv", colClasses=c("integer", "character", "integer"))
activity <- mutate(activity, date=ymd(date))
```

## What is mean total number of steps taken per day?
<!-- For this part of the assignment, you can ignore the missing values in the dataset.
     Calculate the total number of steps taken per day
     Make a histogram of the total number of steps taken each day
     Calculate and report the mean and median of the total number of steps taken per day -->


```r
activityByDate <- select(activity, -interval) %>%
                  group_by(date) %>%
                  summarize(steps=sum(steps,na.rm=TRUE))
qplot(activityByDate$steps, geom="histogram", binwidth=500, 
      xlab="Total Daily Steps", main="Step Activity Histogram")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

The mean of the total number of steps taken per day is: 

```r
averageSteps <- mean(activityByDate$steps)
print(averageSteps)
```

```
## [1] 9354.23
```

The median of the total number of steps taken per day is: 

```r
medianSteps <- median(activityByDate$steps)
print(medianSteps)
```

```
## [1] 10395
```

## What is the average daily activity pattern?
<!-- Make a time series plot (i.e. type="l") of the 5-minute interval (x-axis)
     and the average number of steps taken, averaged across all days (y-axis)
     Which 5-minute interval, on average across all the days in the dataset,
     contains the maximum number of steps? -->


```r
activityByInterval <- select(activity, -date) %>%
                      group_by(interval) %>%
                      summarize(average_steps=mean(steps,na.rm=TRUE))

ggplot(activityByInterval, aes(x=interval, y=average_steps)) + geom_line() + 
       labs(x="Daily 5-Minute Interval", y="Average Steps", 
            title="Average Daily Activity By Interval")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

The 5-minute interval, on average across all the days in the dataset, 
with the maximum number of steps is: 

```r
maxSteps <- filter(activityByInterval, average_steps==max(average_steps))
print(maxSteps$interval)
```

```
## [1] 835
```

The maximum number of average steps per 5-minute interval is:  

```r
print(maxSteps$average_steps)
```

```
## [1] 206.1698
```

## Imputing missing values
<!-- Note that there are a number of days/intervals where there are missing
values (coded as NA). The presence of missing days may introduce bias into some
calculations or summaries of the data. Calculate and report the total number of
missing values in the dataset (i.e. the total number of rows with NAs) Devise a
strategy for filling in all of the missing values in the dataset. The strategy
does not need to be sophisticated. For example, you could use the mean/median
for that day, or the mean for that 5-minute interval, etc. Create a new dataset
that is equal to the original dataset but with the missing data filled in. Make
a histogram of the total number of steps taken each day and Calculate and report
the mean and median total number of steps taken per day. Do these values differ
from the estimates from the first part of the assignment? What is the impact of
imputing missing data on the estimates of the total daily number of steps? -->

The total number of missing values in the dataset is: 

```r
totalMissingVales <- sum(is.na(activity$steps))
print(totalMissingVales)
```

```
## [1] 2304
```
Missing step values will be imputed, using the average steps for that interval.

```r
imputedMissingSteps <- merge(activity, activityByInterval) %>%
                       mutate(steps=if_else(is.na(steps), 
                                            as.integer(average_steps), steps)) %>%
                       select(-average_steps) %>% arrange(date, interval)

newActivityByDate <- select(imputedMissingSteps, -interval) %>%
                  group_by(date) %>%
                  summarize(steps=sum(steps))
qplot(newActivityByDate$steps, geom="histogram", binwidth=500, 
      xlab="Total Daily Steps", main="Step Activity Histogram (Imputed Missing Values)")
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

The mean of the total number of steps taken per day (with imputed missing values) is: 

```r
newAverageSteps <- mean(newActivityByDate$steps)
print(newAverageSteps)
```

```
## [1] 10749.77
```

The difference of the mean with and without imputed values:  

```r
print(newAverageSteps - averageSteps)
```

```
## [1] 1395.541
```

The median of the total number of steps taken per day (with imputed missing values) is: 

```r
newMedianSteps <- median(newActivityByDate$steps)
print(newMedianSteps)
```

```
## [1] 10641
```

The difference of the median with and without imputed values:  

```r
print(newMedianSteps - medianSteps)
```

```
## [1] 246
```

Missing step values were imputed using the average steps for that time interval
and the result of using imputed values was to decrease the number of days with 
zero steps and increase the overall mean and median steps per day.

## Are there differences in activity patterns between weekdays and weekends?
<!-- Use the dataset with the filled-in missing values for this part. Create a
new factor variable in the dataset with two levels – “weekday” and “weekend”
indicating whether a given date is a weekday or weekend day. Make a panel plot
containing a time series plot (i.e. type="l") of the 5-minute interval (x-axis)
and the average number of steps taken, averaged across all weekday days or
weekend days (y-axis). See the README file in the GitHub repository to see an
example of what this plot should look like using simulated data. -->

```r
activityWithWeekdays <- mutate(imputedMissingSteps, 
                               day=as.factor(if_else(wday(date) == 1 | 
                                                     wday(date) == 7, 
                                                     "weekend", "weekday")))
activityByDayAndInterval <- select(activityWithWeekdays, -date) %>%
                      group_by(day, interval) %>%
                      summarize(average_steps=mean(steps))

ggplot(activityByDayAndInterval, aes(x=interval, y=average_steps)) + geom_line() + 
       labs(x="Daily 5-Minute Interval", y="Average Steps", 
            title="Average Daily Activity By Interval (Weekdays vs Weekends)") +
       facet_grid(rows = vars(day))
```

![](PA1_template_files/figure-html/unnamed-chunk-15-1.png)<!-- -->

The average activity patterns differ between weekdays and Weekends:

* The most active period on weekdays is 5AM-10AM

* On weekends activity starts later and is spread more evenly across the days

* On weekends the most activity occurs in the 7:30AM-7:00PM interval

