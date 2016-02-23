# Cousera_Module5_RR_CP1
---
title: "Reproducible Research - Assignment 1"
author: "Made by AJMP"
date: "23 Feb 2016"
output: html_document
---

The following document is the resolution of the assignment 1 from the Reproducible Research Module. 
The data used for completing this assignment is located at the url:  

#### Loading and preprocessing the data
```{r}
file.url <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
destfile <- "activity.zip"
download.file(url = file.url, destfile)
filename <- "activity.csv"

unzip(destfile, filename)

raw_data <- read.csv(file = filename)
AMD <- raw_data[complete.cases(raw_data),]

#Just checking table structure
head(AMD)
colnames(AMD)
nrow(AMD)
```

#### What is mean total number of steps taken per day?
1. Calculate the total number of steps taken per day
```{r}
steps_day <- aggregate(AMD$steps, list(AMD$date), sum)
names(steps_day)[2] <- "Steps"

#Just checking table structure
head(steps_day,5)
max(steps_day$Steps) 
min(steps_day$Steps)
```

2. Make a histogram of the total number of steps taken each day
```{r}
library(ggplot2)
qplot(steps_day$Steps,
      geom = "histogram",
      main = "Distribution of Total Steps per Day",
      xlab = "Steps",  
      col=I("white"), 
      binwidth = 1000 )
```

3. Calculate and report the mean and median of the total number of steps taken per day
```{r}
mean <- mean(steps_day$Steps)
median <- median(steps_day$Steps)
mean
median
```

The **Mean** is equal to 10766.19  
The **Median** is equal to 10765  
  
#### What is the average daily activity pattern?
1. Make a time series plot (i.e. 𝚝𝚢𝚙𝚎 = "𝚕") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
```{r}
#First create a dataframe with the mean value of steps per 5-minute interval
av_steps <- aggregate(AMD$steps, list(interval = AMD$interval), mean)
names(av_steps)[1] <- "Interval"
names(av_steps)[2] <- "Steps" 

#Secondly the plot can be computed
plot(av_steps$Interval,
      av_steps$Steps,
      type = "l",
      xlab = "5 minutes interval",
      ylab = "Steps Average",
      main = "Average number of steps taken")
```

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
```{r}

MaxSteps <- max(av_steps$Steps)
MaxSteps
# 206 steps is the maximum number of steps walked during a 5 minute interval
```


#### Imputing missing values
1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with
```{r}
Locate_NAs <- is.na(raw_data$steps)
Num_NAs <- sum(Locate_NAs)
Num_NAs
# The number of missing values is equal to 2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc. 

```{r}
# Create a data.frame with a median replacement for the NA values values 
av_interval <- tapply(raw_data$steps, raw_data$interval, median, na.rm=TRUE, simplify=TRUE)

# Checking is there are still NAs
sum(is.na(av_interval))

```

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.
```{r}
#Fill in the raw_data(original data with NA) data.frame with the data frame that contains only the replacements for the NA values
raw_data$steps[Locate_NAs] <- av_interval[as.character(raw_data$interval[Locate_NAs])]

# Checking is there are still NAs
sum(is.na(raw_data))
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?
```{r}
#First create a NEW data.frame with the mean value of steps per 5-minute interval
new_steps_day <- aggregate(raw_data$steps, list(raw_data$date), sum)
names(new_steps_day)[1] <- "Day"
names(new_steps_day)[2] <- "Steps"

# Checking is there are still NAs
sum(is.na(new_steps_day))

#Secondly create the NEW plot 
qplot(new_steps_day$Steps,
      geom = "histogram",
      main = "New Distribution of Total Steps per Day",
      xlab = "Steps", 
      col=I("Blue"), 
      binwidth = 1000 )

#Finally compare the diferences between means and medians after adding values to the NAs
new_mean <- mean(new_steps_day$Steps)
new_median <- median(new_steps_day$Steps)

#The mean and median differences are:  
new_mean - mean
new_median - median
```


#### Are there differences in activity patterns between weekdays and weekends?
1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.
```{r}
#Create a new collumn with values Weekday or Weekend by using mutate funcion (adds or replaces existing collumns - this part was tricky, as I had to look up for informations about the function)
library(dplyr)
raw_data <- mutate(raw_data, DayType = ifelse(weekdays(as.Date(raw_data$date)) == c("Saturday","Sunday"), "weekend", "weekday"))
head(raw_data)
```

2. Make a panel plot containing a time series plot (i.e. 𝚝𝚢𝚙𝚎 = "𝚕") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.
```{r}
interval_full <- raw_data %>%
  group_by(interval, DayType) %>%
  summarise(steps = mean(steps))

FinalPlot <- ggplot(interval_full, aes(x=interval, y=steps, color = DayType)) +
  geom_line() +
  facet_wrap(~DayType, ncol = 1, nrow=2)
print(FinalPlot)
```
