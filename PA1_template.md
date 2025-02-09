---
output: 
  html_document: 
    keep_md: yes
---

Reproducible Research - Course Project 1
==========================================


#### **1. Code for reading in the dataset and/or processing the data**

In the first step I loaded the needed packages, unzipped the file and stored the data in a dataframe called "data".


```r
library(tidyverse)
library(lubridate)
unzip("repdata_data_activity.zip")
data <- read.csv("activity.csv")
```

Since the date variable is not formatted in a datetime format, I used the lubridate packge to transform it into a datetime object.


```r
data$date <- ymd(data$date)
```

The data is now prepared to answer the first questions of the assigment, as a quick lock at the structure of the data shows


```r
str(data)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Date, format: "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```



#### **2. Histogram of the total number of steps taken each day**

Using dplyr and ggplot, I grouped the data by date, summarized the steps for each date and finally plotted the data in a histogram, using 5 bins. The histogram shows that most of the days steps totalled ~10.000 steps.


```r
data %>% group_by(date) %>% summarize(steps = sum(steps, na.rm = T)) %>% ggplot(aes(x = steps)) + geom_histogram(bins = 5, color = "white")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->


### **3. Mean and median number of steps taken each day**

Similar to what was done in the histogram, the data is first grouped by date and then summarized by the sum of steps, which yields the total steps for each day.

Since the goal is to have just two single numbers (mean steps per day, median steps per day) an additional summary is needed, which computes the mean and meadian based on the summary done in the step before.



```r
data %>% group_by(date) %>% summarize(total_steps = sum(steps, na.rm = T)) %>% summarize(mean_steps = mean(total_steps), median_steps = median(total_steps))
```

```
## # A tibble: 1 x 2
##   mean_steps median_steps
##        <dbl>        <int>
## 1      9354.        10395
```



### **4. Time series plot of the average number of steps taken**

In my understanding the goal is to show the steps taken "On an average day". A useful approach, in my opinion, is to show the mean steps taken for each interval.

To do so, I grouped the data on the variable "interval" and summarized the steps in the next step, which shows the average step for each interval in the 2 month period.

The result shows that there is nearly no activity between 0 and 5 o´clock (which one would expect), and the highest activity is around ~9 o´cock.


```r
data %>% group_by(interval) %>% summarize(steps = mean(steps, na.rm = T)) %>% ggplot(aes(x = interval, y = steps)) + geom_line()
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->



### **4. The 5-minute interval that, on average, contains the maximum number of steps**

Similar to the approach used in question 4, I grouped the data by interval and computed the mean steps taken for each interval.

Finally I rearranged the data in descending order to show the highest value of mean_steps first and limited the output to just the first entry,


```r
data %>% group_by(interval) %>% summarize(mean_steps = mean(steps, na.rm = T)) %>% arrange(desc(mean_steps)) %>% head(1)
```

```
## # A tibble: 1 x 2
##   interval mean_steps
##      <int>      <dbl>
## 1      835       206.
```



### **5. Code to describe and show a strategy for imputing missing data**

First, I couned the total number of missing values in the steps column to get a quick overview of the rate of missing values.

Creating a logical vector of missing values in the steps column and summing those up shows around 2.304 missing entries compared to 15.264 non-missing values.


```r
sum(is.na(data$steps))
```

```
## [1] 2304
```

```r
sum(!is.na(data$steps))
```

```
## [1] 15264
```

In my opinion, a good approach to fill the missing values in each interval is to use average values for this specific interval.

To do so, I frist created a new dataframe that I will use as a reference table. This dataframe called "reference" includes the average steps taken for each interval in the 61 days period covered in the dataset.


```r
reference <- data %>% group_by(interval) %>% summarize(mean_steps_interval = mean(steps, na.rm = T))
```

Using a left_join (on the column "interval"), I added the reference column "Mean_steps_interval" from the reference dataframe to the original dataframe ("data").

Every column has now 2 Steps-columns: The original one "Steps", which shows the measured steps (whether its missing or not) and the newly added "mean_steps_interval", which shows the average steps taken for this interval across all days.


```r
data <- left_join(data, reference, ON = interval)
```

```
## Joining, by = "interval"
```

```r
head(data)
```

```
##   steps       date interval mean_steps_interval
## 1    NA 2012-10-01        0           1.7169811
## 2    NA 2012-10-01        5           0.3396226
## 3    NA 2012-10-01       10           0.1320755
## 4    NA 2012-10-01       15           0.1509434
## 5    NA 2012-10-01       20           0.0754717
## 6    NA 2012-10-01       25           2.0943396
```

In the next step I created a logical vector that identifies those rows in the data dataset where the steps value is NA.

Using this logical vector, I filtered the full data dataset for only those rows where the steps value equals NA and filled those values with the ones from the mean_steps_interval column.

Finally, I deleted the mean_steps_interval column as it is no longer needed.


```r
data_logical <- is.na(data$steps)
data[data_logical,] <- data %>% mutate(steps = mean_steps_interval)
data <- data %>% select(-(mean_steps_interval))
head(data)
```

```
##       steps       date interval
## 1 1.7169811 2012-10-01        0
## 2 0.3396226 2012-10-01        5
## 3 0.1320755 2012-10-01       10
## 4 0.1509434 2012-10-01       15
## 5 0.0754717 2012-10-01       20
## 6 2.0943396 2012-10-01       25
```

```r
sum(is.na(data$steps))
```

```
## [1] 0
```



### **6. Histogram of the total number of steps taken each day after missing values are imputed**

The code is actually the same as used in question 2, except the "na.rm = T" argument in the sum operation, which is no longer needed.

The histogram looks slightly different from the one in question 2, with a distribution nearly normal (which was not the case in the histogram at quastion 2).


```r
data %>% group_by(date) %>% summarize(steps = sum(steps)) %>% ggplot(aes(x = steps)) + geom_histogram(bins = 5, color = "white")
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png)<!-- -->



### **7. Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends**

To make the subsetting of the data easier when creating the plot (weekday vs. weekend), I decided to add an additional column to the dataset containing the information whether the specific day is a weekday or weekend day.

In the second step I took this dataset and created a time series plot similar to what was done at question 2, but added a facet_grid to distinguish between the weekdays and weekends.

When comparing the weekdays view with the weekend view, it turns out that the person who created the data tend to sleep longer at the weekends, but was more active during the day.


```r
data <- data %>% mutate(Day_Week = ifelse(weekdays(date) == "Saturday" | weekdays(date) == "Sunday", "weekend", "weekday"))

data %>% group_by(interval, Day_Week) %>% summarize(steps = mean(steps)) %>% ggplot(aes(x = interval, y = steps, color = Day_Week)) + geom_line() + facet_grid(cols = vars(Day_Week))
```

![](PA1_template_files/figure-html/unnamed-chunk-13-1.png)<!-- -->
