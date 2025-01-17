---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

## Loading and preprocessing the data
In order to have a look at the data, we first have to unzip it and load it:

```r
unzip("activity.zip", exdir = ".")
activity <- read.csv("activity.csv",header = TRUE)
```

In order to be able to process the data further, we first need to transform the data into a suitable format:

```r
# Create an empty vector with length = amount of days present in dataset
daysum <- vector(mode="numeric",length=length(unique(activity$date)))

# For each date, calculate the sum of all steps taken that day, ignoring NAs
for(i in 1:length(unique(activity$date))){
    daysum[i] <- sum(activity$steps[activity$date==unique(activity$date)[i]], na.rm=TRUE)
}
```


## What is the mean total number of steps taken per day?

Here we'll create a histogram of the total number of steps taken each day:


```r
hist(daysum, main="Histogram of total number of steps taken per day",
     xlab="Number of steps", ylab="Frequency",
     col="darkmagenta")
```

![](PA1_template_files/figure-html/histogram-1.png)<!-- -->


We will now calculate the **mean** and **median** total number of steps taken per day:

```r
meansteps <- mean(daysum)
mediansteps <- median(daysum)
```

**The mean total amount of steps taken per day is 9354.2295082, the median is 1.0395\times 10^{4}.**

## What is the average daily activity pattern?

Now we're going to make a time series plot of the 5-minute interval (x) vs. the number of steps taken (y), averaged across all days.

This means that we now have to average, not over the days, but over the intervals. 

```r
timeinterval <- unique(activity$interval)

# Create an empty vector with length = amount of intervals present in a day
meansteps <- vector(mode="numeric",length=length(unique(activity$interval)))

# For each interval, calculate the mean of all steps taken in that interval over all days, ignoring NAs
for(i in 1:length(unique(activity$interval))){
    meansteps[i] <- mean(activity$steps[activity$interval==unique(activity$interval)[i]], na.rm=TRUE)
}

# Create the timeseries plot
plot(timeinterval,meansteps,type="l",main= "Average steps over one day", xlab="Time interval", ylab="Average number of steps taken", col="darkblue")
```

![](PA1_template_files/figure-html/timeseries-1.png)<!-- -->

Here we calculate the 5-minute interval in which, on average across all days, the participant made the most steps:

```r
# Determine maximum amount of steps
maxsteps <- max(meansteps)

# Determine index of that maximum (where in the vector is the maximum?)
index <- match(maxsteps,meansteps)

# Get the interval belonging to that maximum
maxinterval <- timeinterval[index]
```

**The maximum average amount of steps made is 206.1698113. This is in time interval 835.**

## Imputing missing values

First, let's see how many missing values are in the dataset:

```r
# Calculate the total numner of rows with NAs
nmissing <- sum(is.na(activity$steps))

# Calculate % of missing data
nrows <- length(activity$steps)
percentmissing <- round(nmissing/nrows*100,2)
```

**There are 2304 rows with missing data in this dataset.** That is 13.11 % of the all rows in the dataset.

To impute missing values, we're going to replace NAs with the average amount of steps taken during that 5-minute interval over all days. This could be a better indication than the mean for the entire day, since it is a more granulated way of filling in datapoints.


```r
# Create a copy of the activity dataset
activity2 <- activity

# Replace the NAs with the mean of that specific time interval over all days
for(j in 1:length(activity2$steps)){ # For all rows
    # Check whether the value is NA
    if(is.na(activity2$steps[j])){
        # Check in which time interval in the original dataset the NA occurred
        originterval <- activity2$interval[j] 
        
        # Retrieve the index of the interval in the timeinterval variable
        timeindex <- match(originterval,timeinterval)
        
        # Replace the NA with the mean value belonging to the corresponding time interval  
        activity2$steps[j] <- meansteps[timeindex]     
    }
}
```


Now we are going to make another histogram with the total number of steps taken each day:

```r
# Create an empty vector with length = amount of days present in dataset
daysum2 <- vector(mode="numeric",length=length(unique(activity2$date)))

# For each date, calculate the sum of all steps taken that day
for(k in 1:length(unique(activity2$date))){
    daysum2[k] <- sum(activity2$steps[activity2$date==unique(activity2$date)[k]], na.rm=TRUE)
}

hist(daysum2, main="Histogram of total - incl. imputed - number of steps taken per day",
     xlab="Number of steps", ylab="Frequency", col="darkgreen")
```

![](PA1_template_files/figure-html/imputedhistogram-1.png)<!-- -->

Now, the mean and median total numbers of steps taken each day are:

```r
meansteps2 <- mean(daysum2)
mediansteps2 <- median(daysum2)
```

**1.0766189\times 10^{4} (mean) and 1.0766189\times 10^{4} (median), respectively.** The mean value is now larger than in the original dataset.  


## Are there differences in activity patterns between weekdays and weekends?

We will now create a new variable in the dataset called `weekdays` indicating whether a given date is a weekday or weekend day:

```r
# Convert the date column to an actual Date format
activity2$date2 <- as.Date(activity2$date)

# Add a factor variable with levels weekday and weekend
activity2$weekdays <- weekdays(activity2$date2) # new column with day of the week

# Determine whether the day is a week day or a weekend day
for(l in 1:length(activity2$weekdays)){ # for all rows
    # If the day is a Saturday or Sunday, replace it with "weekend"
    if(activity2$weekdays[l]=="zaterdag"|activity2$weekdays[l]=="zondag"){
        activity2$weekdays[l] <- "weekend"
    }
    # Otherwise replace it with "weekday"
    else{ 
        activity2$weekdays[l] <- "weekday"
    }
}
```

Finally, this code creates a panel plot containing a time series plot of the 5-minute interval (x) and the average number of steps taken (y) for both weekend days (plot 1) and week days (plot 2):


```r
# Load packages
library(plyr)
library("lattice")

# Calculate mean per type of day and interval
activity3 <- ddply(activity2, .(interval, weekdays), summarize, mean=mean(steps))

# Plot weekend and weekdays panel plot
xyplot(mean ~ interval | factor(weekdays), data=activity3, type = "l", xlab="5-minute time interval", ylab="Average amount of steps taken")
```

![](PA1_template_files/figure-html/panelplot-1.png)<!-- -->
