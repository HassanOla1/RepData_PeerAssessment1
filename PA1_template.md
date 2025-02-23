---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true

---

## Loading and preprocessing the data


```r
unzip("activity.zip")
ac <- read.csv("activity.csv")
head(ac)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```
## What is mean total number of steps taken per day?

```r
library(ggplot2)
total_steps_per_day <- tapply(ac$steps,ac$date,FUN = sum, na.rm = TRUE)
#plot
qplot(total_steps_per_day,binwidth=1000, xlab = "total number of steps taken each day" )
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

```r
#mean of the total number of steps taken per day
mean(total_steps_per_day, na.rm=TRUE)
```

```
## [1] 9354.23
```

```r
#median of the total number of steps taken per day
median(total_steps_per_day, na.rm=TRUE)
```

```
## [1] 10395
```
## What is the average daily activity pattern?

```r
aver_daily_activity <- aggregate(x = list(steps = ac$steps), by=list(interval =ac$interval),
                                 FUN = mean,na.rm=TRUE)
#Make a time series plot
ggplot(data = ac, aes(x = interval,y = steps)) +geom_line() + xlab("average number of steps taken") + ylab("averaged across all days")
```

```
## Warning: Removed 2 row(s) containing missing values (geom_path).
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->
 on average across all the days in the dataset, contains the maximum number of steps?

```r
aver_daily_activity[which.max(aver_daily_activity$steps),]
```

```
##     interval    steps
## 104      835 206.1698
```
## Imputing missing values

```r
#total number of missing values in the dataset
missing_value <- sum(is.na(ac$steps))
missing_value
```

```
## [1] 2304
```

```r
#Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated.
fillin_value <- function(steps,interval){
  filled <- NA
  if(!is.na(steps))
    filled <- steps
  else
    filled <- (aver_daily_activity[aver_daily_activity$interval == interval,"steps"])
  return(filled)
}
#Create a new dataset that is equal to the original dataset but with the missing data filled in
new_data <- ac
new_data$steps <- mapply(fillin_value, new_data$steps, new_data$interval)
```
 Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
total_number <- tapply(new_data$steps,new_data$date,FUN = sum, na.rm = TRUE)
#plot
hist(total_number,xlab = "total number of steps taken each day" )
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

```r
#mean of the total number of steps taken per day
mean(total_number)
```

```
## [1] 10766.19
```

```r
#median of the total number of steps taken per day
median(total_number)
```

```
## [1] 10766.19
```
Mean and median values are higher after imputing missing data. In the original dataset the total number of `steps´ are set to 0s by default.  However, after replacing missing `steps` values with the mean `steps`
of associated `interval` value, these 0 values are removed from the histogram
of total number of steps taken each day.

## Are there differences in activity patterns between weekdays and weekends?

```r
# Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day. We use the dataset with the filled-in values.
new_data$date <- as.Date(new_data$date)
weekdays1 <- c("Monday","Tuesday","Wednessday","Thursday","Friday")
new_data$wday <- factor((weekdays(new_data$date) %in% weekdays1), levels=c(FALSE, TRUE), labels=c('weekend', 'weekday'))
```
Make a panel plot containing a time series

```r
aver_daily_activity <- aggregate(steps~interval+wday, data =new_data,FUN = mean,na.rm=TRUE)

ggplot(data = aver_daily_activity, aes(x = interval,y = steps)) +geom_line() + facet_grid(wday~.) + labs(title = "Avg. Daily Steps by Weektype", x = "Interval", y = "No. of Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

```
