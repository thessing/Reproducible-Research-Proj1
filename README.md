---
title: 'Coursera: Reproducible Research Project #1'
author: "Tim Hessing, PhD MBA"
date: "June 27, 2018"
---


## Introduction
It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the “quantified self” movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This project makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

## Activity Data
The data for this assignment was downloaded from the course web site:

**Dataset:**  [Activity Monitoring Data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) [52k]

The variables included in this dataset are:

   * **steps**: Number of steps taking in a 5-minute interval (missing values are coded as NA)  
   * **date**: The date on which the measurement was taken in YYYY-MM-DD format  
   * **interval**: Identifier for the 5-minute interval in which measurement was taken  

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

## Loading & Processing the Data

### Loading


```{r}
fileName <- "repdata_data_activity.zip"
dataFile <- unz(fileName, filename="activity.csv")
activity <- read.csv(dataFile, header=TRUE)
```

### Processing the Data

```{r}
activity$day      <- weekdays(as.Date(activity$date))
```

```{r}
stepsPerDay          <- aggregate(activity$steps ~ activity$date, FUN=sum)
colnames(stepsPerDay)<- c("Date", "Steps")

meanTotalSteps   <- mean(stepsPerDay$Steps)
medianTotalSteps <- median(stepsPerDay$Steps)

meanStepsPerInterval <- aggregate(activity$steps ~ activity$interval, FUN=mean)
colnames(meanStepsPerInterval)<- c("Interval", "Steps")

```

## Results

### Question 1 Number of Steps Each Day
To address the **Question 1** for this project, the mean total number of steps taken each day is `r meanTotalSteps` and the median total number of steps taken each day is `r medianTotalSteps`. The following histogram shows the frequency of the total number of steps taken per day, where the bin width is 1500 steps. 

`r
bins <- as.integer( (max(stepsPerDay$Steps) - min(stepsPerDay$Steps))/1500 )
hist(stepsPerDay$Steps, breaks=bins, xlab="Steps", main = "Total Steps per Day")
`

### Question 2 Daily Activity Pattern
The following plot shows the average daily activity pattern. During the `r meanStepsPerInterval$Interval[ meanStepsPerInterval$Steps == max(meanStepsPerInterval$Steps) ]` interval there is a peak in the average number of daily steps, corresponging to `r max(meanStepsPerInterval$Steps)`.

`r
plot(meanStepsPerInterval$Interval, meanStepsPerInterval$Steps, type="l", xlab="Daily Interval", ylab="Average Steps per Interval")
`

### Question 3 Imputing Missing Values
This data set contains `r sum(is.na(activity$steps))` missing values. This missing values could be re-assigned with the mean number of steps for that time interval, in order to eliminate these missing values, as follows:

```{r}
newActivity <- activity

newActivity$steps[is.na(newActivity$steps)] <- meanStepsPerInterval$Steps[ meanStepsPerInterval$Interval == newActivity$interval[is.na(newActivity$steps)]]

```
The Total Number of Steps per Day can be recalculated using this new data set, as follows:

```{r}
nstepsPerDay          <- aggregate(newActivity$steps ~ newActivity$date, FUN=sum)
colnames(nstepsPerDay)<- c("Date", "Steps")

nmeanTotalSteps   <- mean(nstepsPerDay$Steps)
nmedianTotalSteps <- median(nstepsPerDay$Steps)

meanStepsPerInterval <- aggregate(activity$steps ~ activity$interval, FUN=mean)
colnames(meanStepsPerInterval)<- c("Interval", "Steps")
```

The new mean total number of steps taken each day is `r nmeanTotalSteps` and the median total number of steps taken each day is `r nmedianTotalSteps`. The following histogram shows the frequency of the total number of steps taken per day, where the bin width is 1500 steps. 

`r
bins <- as.integer( (max(nstepsPerDay$Steps) - min(nstepsPerDay$Steps))/1500 )
hist(nstepsPerDay$Steps, breaks=bins, xlab="Steps", main = "Total Steps per Day")
`

### Question 4 Difference in Activity Patterns
In order to seperate week-ends from week-days, the following code snippet was used to identify week-days.
```{r}
wends       <- c("Saturday", "Sunday")
activity$wd <- !(activity$day %in% wends)
```
The factor activity$wd can then be used to look at activity patterns on week-days and week-ends.

```{r}
wdmeanStepsPerInterval <- aggregate(activity$steps[activity$wd] ~ activity$interval[activity$wd], FUN=mean)
colnames(wdmeanStepsPerInterval)<- c("Interval", "Steps")

wemeanStepsPerInterval <- aggregate(activity$steps[!activity$wd] ~ activity$interval[!activity$wd], FUN=mean)
colnames(wemeanStepsPerInterval)<- c("Interval", "Steps")
```

This can be used to cgenerate a time-series plot for the week-days and another for the week-ends, as shown below.

`r
plot(wdmeanStepsPerInterval$Interval, wdmeanStepsPerInterval$Steps, type="l", xlab="Daily Interval", ylab="Average Steps per Interval", main="Week-Day")
`

`r
plot(wemeanStepsPerInterval$Interval, wemeanStepsPerInterval$Steps, type="l", xlab="Daily Interval", ylab="Average Steps per Interval", main="Week-End")
`
