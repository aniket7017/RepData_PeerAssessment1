---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data
library(dplyr)
library(ggplot2)
activity <- read.csv("activity.csv")
str(activity)
summary(activity)
head(activity)
act.complete <- na.omit(activity)



## What is mean total number of steps taken per day?
act.day <- group_by(act.complete, date)
act.day <- summarize(act.day, steps=sum(steps))
summary(act.day)
qplot(steps, data=act.day)
mean(act.day$steps)
median(act.day$steps)



## What is the average daily activity pattern?
act.int <- group_by(act.complete, interval)
act.int <- summarize(act.int, steps=mean(steps))
ggplot(act.int, aes(interval, steps)) + geom_line()
act.int[act.int$steps==max(act.int$steps),]



## Imputing missing values
nrow(activity)-nrow(act.complete)
names(act.int)[2] <- "mean.steps"
act.impute <- merge(activity, act.int)
act.impute$steps[is.na(act.impute$steps)] <- act.impute$mean.steps[is.na(act.impute$steps)]
act.day.imp <- group_by(act.impute, date)
act.day.imp <- summarize(act.day.imp, steps=sum(steps))
qplot(steps, data=act.day.imp)
mean(act.day.imp$steps)
median(act.day.imp$steps)




## Are there differences in activity patterns between weekdays and weekends?
act.impute$dayofweek <- weekdays(as.Date(act.impute$date))
act.impute$weekend <-as.factor(act.impute$dayofweek=="Saturday"|act.impute$dayofweek=="Sunday")
levels(act.impute$weekend) <- c("Weekday", "Weekend")
act.weekday <- act.impute[act.impute$weekend=="Weekday",]
act.weekend <- act.impute[act.impute$weekend=="Weekend",]
act.int.weekday <- group_by(act.weekday, interval)
act.int.weekday <- summarize(act.int.weekday, steps=mean(steps))
act.int.weekday$weekend <- "Weekday"
act.int.weekend <- group_by(act.weekend, interval)
act.int.weekend <- summarize(act.int.weekend, steps=mean(steps))
act.int.weekend$weekend <- "Weekend"
act.int <- rbind(act.int.weekday, act.int.weekend)
act.int$weekend <- as.factor(act.int$weekend)
ggplot(act.int, aes(interval, steps)) + geom_line() + facet_grid(weekend ~ .)


