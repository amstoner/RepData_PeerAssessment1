### Read the activity data and change second column to Date format

First I read in the data.

    activity <- read.csv('/Users/Anne/Documents/*Work/Coursera/Course 5 - Reproducible Research/Course project 1/activity.csv')
    activity$date <- as.Date(activity$date,"%Y-%m-%d",tz="UTC")

    ## Warning in strptime(x, format, tz = "GMT"): unknown timezone 'default/
    ## America/Chicago'

Set the global options to round the numbers since.

    options(scipen = 1, digits = 0)

### Calculate the mean total number of steps taken per day

Ignoring the missing values I calculate the daily total steps, using the
tapply function. Then I plot a histogram, using the hist function,
before calculating the daily total mean and median.

    daily_total <- tapply(activity$steps,as.factor(activity$date),sum)
    hist(daily_total, main="Histogram of daily total number of steps",xlab="Steps",breaks=10)

![](PA1_template_files/figure-markdown_strict/calculate%20mean-1.png)

    daily_total_mean <- round(mean(daily_total,na.rm=T))
    daily_total_median <- round(median(daily_total,na.rm=T))

The mean of the daily total number of steps is 10766 and the median is
10765.

### Calculate the daily average activity pattern

Now I calculate the daily average pattern, again using the tapply
function, which is then plotted.

    daily_ave_pattern <- tapply(activity$steps,as.factor(activity$interval),mean,na.rm=T)
    plot(names(daily_ave_pattern),daily_ave_pattern,type='l',main='Daily Average Activity Pattern',xlab='5-minute interval',ylab='Number of steps')

![](PA1_template_files/figure-markdown_strict/average%20pattern-1.png)

To find the 5-minute interval that, on average, has the highest number
of steps per day, we can use the function which.max on the daily average
pattern.

    ave_max <- names(daily_ave_pattern)[which.max(daily_ave_pattern)]

The most active 5-minute interval, on average, is the 835 interval.

### Imputing missing values

First I calculate the total number of missing values.

    total_missing_values <- length(which(is.na(activity$steps)))

The total number of missing values is: 2304.

Next I fill in the missing values by replacing them with the mean for
that 5-minute interval.

    activity2 <- activity
    for (i in 1:nrow(activity)){
      if (is.na(activity[i,1])) activity2[i,1] <- round(mean(activity[which(activity[,3]==activity[i,3]),1],na.rm=T))
    }

I now recalculate the daily total number of steps with the new dataset
without missing values and plot the histogram.

    daily_total2 <- tapply(activity2$steps,as.factor(activity2$date),sum)
    hist(daily_total2, main="Histogram of daily total number of steps\nwith missing values imputed",xlab="Steps",breaks=10)

![](PA1_template_files/figure-markdown_strict/calculate%20mean%20again-1.png)

    daily_total_mean2 <- round(mean(daily_total2))
    daily_total_median2 <- round(median(daily_total2))

The new mean of the daily total is now 10766 and the new median is
10762. To compare with the previous mean and median, let's make a table:

    summary <- matrix(round(c(daily_total_mean,daily_total_median,daily_total_mean2,daily_total_median2)),nrow=2,dimnames=list(c("With NAs","Without NAs"),c("Mean","Median")))
    library(xtable)
    xt <- xtable(summary)
    print(xt,type="html")

<!-- html table generated in R 3.4.1 by xtable 1.8-2 package -->
<!-- Sun Dec 10 08:30:15 2017 -->
<table border="1">
<tr>
<th>
</th>
<th>
Mean
</th>
<th>
Median
</th>
</tr>
<tr>
<td align="right">
With NAs
</td>
<td align="right">
10766.00
</td>
<td align="right">
10766.00
</td>
</tr>
<tr>
<td align="right">
Without NAs
</td>
<td align="right">
10765.00
</td>
<td align="right">
10762.00
</td>
</tr>
</table>
The table shows that, because we used a method to impute the NAs that
took the average of the other days at the same 5-minute interval, there
isn't much difference between the two sets of means and medians. The new
median is lower by 3 steps, but that is very insignificant, hence the
impact from replacing the NAs is very small in this case.

### Finding differences in activity patterns between weekdays and weekends

To separate the weekdays and weekends we can use the function weekdays
on the date.

    wk <- weekdays(activity2$date)
    wk[which(wk=="Saturday" | wk=="Sunday")] <- "weekend"
    wk[which(! wk=="weekend")] <- "weekday"
    weekday <- as.factor(wk)
    activity2 <- cbind(activity2,weekday)

Now we can plot the time series separately for weekdays and weekends to
see if there is a difference in activity patterns.

    weekday_weekend_pattern <- round(tapply(activity2$steps,list(as.factor(activity2$interval),activity2$weekday),mean,na.rm=T))
    library(reshape2)
    weekday_weekend_pattern2 <- melt(weekday_weekend_pattern)
    colnames(weekday_weekend_pattern2) <- c("Interval","Weekday","Steps")
    library(lattice)
    xyplot(Steps ~ Interval|Weekday, data=weekday_weekend_pattern2, main="Activity Pattern by Weekday or Weekend",type='l',layout=c(1,2))

![](PA1_template_files/figure-markdown_strict/panel%20plot-1.png)
