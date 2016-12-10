---
title: "PA1_template.RMD"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## 1. Code for reading in the dataset and/or processing the data

```{r}
    setwd("~/Desktop/Course 5_Reproducible Research")
    library(dplyr)

    data <- read.csv("activity.csv")
    data2 <- data %>% group_by (date) %>% summarize(total_steps = sum(steps, na.rm = TRUE))
    data2$dayofweek <- weekdays(as.Date(data2$date))
```

## 2. Histogram of the total number of steps taken each day

```{r}
    hist(data2$total_steps, main = "Histogram of the Total Number of Steps Taken each Day")
```

## 3. Mean and median number of steps taken each day
  
```{r}  
    mean(data2$total_steps, na.rm = TRUE)
    median(data2$total_steps, na.rm = TRUE)
```

## 4. Time series plot of the average number of steps taken

```{r}
    data3 <- data %>% group_by (interval) %>% summarize(average_steps = mean(steps, na.rm = TRUE))
    plot(x = data3$interval, y = data3$ average_steps, type = "l", main = "Average (Mean) Number of Steps taken at each 5-minute Interval")
```

## 5. The 5-minute interval that, on average contains the maximum number of steps

```{r}
    filter (data3, average_steps == max(average_steps))

    # The output shows that the 5-minute interval, 835, contains the maximum number of steps
```

## 6. Code to describe and show a strategy for imputing missing data

```{r}
    #using the median value for imputatation at 5-minute intervals
    
    median_imputation_values <- data %>% group_by (interval) %>% summarize(average_steps = median(steps, na.rm = TRUE))
    count_missing_values <- dim(data[!complete.cases(data),])
    count_missing_values[1] # The number of missing values
    
    #divide data into two subsets: those with missing values and those without missing values
    
    data_with_missingvalues <- data[!complete.cases(data),]
    data_without_missingvalues <- data[complete.cases(data),]

    data_imputed <- inner_join(data_with_missingvalues,median_imputation_values, by = "interval")
    data_imputed$steps <- data_imputed$average_steps # replacde NAs with median value at specific time interval
    data_imputed$average_steps <- NULL # remove extra column in order to rbind below

    #combine tables back together
    
    final_data <- rbind(data_imputed,data_without_missingvalues)
```

## 7. Histogram of the total number of steps taken each day after missing values are imputed
 
```{r}   
    final_data2 <- final_data %>% group_by (date) %>% summarize(total_steps = sum(steps, na.rm = TRUE))
    final_data2$dayofweek <- weekdays(as.Date(final_data2$date))
    hist(final_data2$total_steps, main = "Histogram of the Total Number of Steps Taken each Day (imputation)")
 
    mean(final_data2$total_steps, na.rm = TRUE)
    median(final_data2$total_steps, na.rm = TRUE)
```

## 8. Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends

```{r}  
    df <- final_data 
    df$dayofweek <- weekdays(as.Date(df$date))
    
    #create two datasets: one for weekend and one for weekday
    
    df_weekend <- df %>% filter(dayofweek %in% c("Saturday", "Sunday"))
    df_weekday <- df %>% filter(dayofweek %in% c("Monday", "Tuesday","Wednesday","Thursday","Friday"))
    
    df_weekend$dayofweek <- NULL
    df_weekday$dayofweek <- NULL
    
    
    df_weekend2 <- df_weekend %>% group_by (interval) %>% summarize(average_steps = mean(steps, na.rm = TRUE))
    df_weekday2 <- df_weekday %>% group_by (interval) %>% summarize(average_steps = mean(steps, na.rm = TRUE))
    
    df2 <- df_weekend2
    df2$factor <- "weekend"
    
    df3 <- df_weekday2
    df3$factor <- "weekday"
    
    #combine two datasets back together
    final_df <- rbind(df2,df3)
    
    #generate panel plot
    library(lattice)
    xyplot(average_steps ~ interval | factor, data = final_df, layout = c(1,2), type = "l")
```

## 9. All of the R code needed to reproduce the results (numbers, plots, etc.) in the report

```{r}  
# contained in the knitr report generated
# Github: https://github.com/skydiver-1/RepData_PeerAssessment1
# SHA: f1558b7b715e62c74f6332e3e31641f9d9bf13c2
```