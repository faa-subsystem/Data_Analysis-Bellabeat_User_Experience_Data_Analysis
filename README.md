---
title: "Bellabeat Business Analysis"
author: "Faury A. Abreu"
date: "2023-07-18"
output: 
  html_document:
    toc: true
    collapse: false
    theme: "journal"
    #number_sections: true
    toc_float: true
    
    
---



# <span style="color:#9e4513">Business task</span>

## <span style="color:#f89875">Summary</span>
Bellabeat, a high-tech manufacturer of health-focused products for women. Co-founder and Chief Creative Officer of Bellabeat, believes that
analyzing smart device fitness data could help unlock new growth opportunities for the company. The goal for this analysis is to analyze smart device data to gain insight into how consumers are using their smart devices. The insights you discover will then help guide marketing strategies for the company.
Products: 

- Bellabeat App

- Leaf (wellness tracker can be worn as a bracelet, necklace, or clip)

- Time (watch) 

- Spring (smart water bottle) 

- Bellabeat membership (subscription-based membership program for users)

#### <span style="color:#9e4513"> Objectives: </span>####

**a)  Analyze smart device usage data in order to gain insight into how consumers use non-Bellabeat smart devices.**

**b) What are some trends in smart device usage?** 

**c) How could these trends apply to Bellabeat customers?**

**d) How could these trends help influence Bellabeat marketing strategy?** 



## <span style="color:#f89875"> Loading Packages</span>

Packages:

- tidyverse

- lubridate

- ggplot2

- gctorture(on = FALSE) to avoid memory overload

```{r load packages, results='hide'}
colour_scl <- c("#ffd1bc", "#f89875", "#9e4513")
library(tidyverse)
library(lubridate)
library(ggplot2)
gctorture(on = FALSE)
```

## <span style="color:#f89875"> Import Data Sets</span>

Project manager suggested to use public data with similar information about smart device user's daily habits. This Kaggle data set contains personal fitness tracker from thirty fitbit users.
Data set imported from [*kaggle: MÖBIUS: FitBit Fitness Tracker Data*](https://www.kaggle.com/datasets/arashnic/fitbit)

Data sets:

- daily_activity: Users activity merged by day

- calories: Amount of calories merged by day

- weight_loginfo: Record of user's weight merged by day

- sleep_day: Daily sleep information

- intensity: Daily activity's Intensity measures

- heart: User's Heart rate record by second

- hour_intensity: Hourly activity's Intensity measures

```{r data import}
daily_activity <- read.csv("Fitabase Data 4.12.16-5.12.16/dailyActivity_merged.csv")
calories <- read.csv("Fitabase Data 4.12.16-5.12.16/dailyCalories_merged.csv")
weight_loginfo <- read.csv("Fitabase Data 4.12.16-5.12.16/weightLogInfo_merged.csv")
sleep_day <- read.csv("Fitabase Data 4.12.16-5.12.16/sleepDay_merged.csv")
intensity <- read.csv("Fitabase Data 4.12.16-5.12.16/dailyIntensities_merged.csv")
hour_intensity <- read.csv("Fitabase Data 4.12.16-5.12.16/hourlyIntensities_merged.csv")

```

# <span style="color:#9e4513">Prepare</span>

The data sets are organized chronologically in long format. The entries are time based measurements and characteristics of user's activity.

## <span style="color:#f89875"> Observe Data</span>

To observe the data, first apply glimpse(), in this way we will be able to see the names of all the columns in the data set, including seeing the first entries of each column horizontally. Also, to see the preliminary appearance of the tables I will use head(), this function allows to see the column's name, the type of data in the column and the first rows of the data set.

```{r check data for errors, results='hide'}
glimpse(daily_activity)
head(daily_activity)

glimpse(calories)
head(calories)

glimpse(intensity)
head(intensity)

glimpse(weight_loginfo) 
head(weight_loginfo)

glimpse(sleep_day)
head(sleep_day)

```
After observing the data sets it can be confirmed that there is no bad spell in the variables, except in "ActivityDate" a character format that should be in date format, other than that all formats are well defined and formatted as they should be. However, there is no effective way to confirm veracity of the data since there is no referent data to further compare. Additionally, there is no geographical reference so there is no way to measure the sample size compared to a whole population, and the time limitation (2 months) could be part of data bias.

## <span style="color:#f89875">  Convert date (character variable) into Date format </span>

As observed before when applied head() function, the data sets have the date variable in character format, in order to apply calculations effectively it is necessary to change it to Date format. To do this, I will use as.Date and as.POSIXct which is part of the Date-time conversion functions. This conversion functions allows to specify the format in which we want to rearrange the date.

```{r COnvert to date, results='hide'}
daily_activity$ActivityDate <- as.Date(daily_activity$ActivityDate, format = "%m/%d/%Y" )
intensity$ActivityDay <- as.Date(intensity$ActivityDay, format = "%m/%d/%Y" )
calories$ActivityDay <- as.Date(calories$ActivityDay, format = "%m/%d/%Y" )
sleep_day$SleepDay <- as.POSIXct(sleep_day$SleepDay, tz = "", tryFormats = "%m/%d/%Y %H:%M:%OS")
weight_loginfo$Date <- as.POSIXct(weight_loginfo$Date, tz = "", tryFormats = "%m/%d/%Y %H:%M:%OS")
hour_intensity$TimeDate <- as.POSIXct(hour_intensity$ActivityHour, tz = "", tryFormats = "%m/%d/%Y %H:%M:%S %p")

```

## <span style="color:#f89875"> Observe Data to confirm changes</span>

```{r reobservation, results='hide'}
glimpse(daily_activity)
glimpse(intensity)
glimpse(calories)
glimpse(sleep_day)
glimpse(weight_loginfo)
glimpse(hour_intensity)
```

## <span style="color:#f89875"> Look for any repeated data </span>

Looking for repeated data is an important step ti ensure precision and veracity in the analysis.

```{r Cleaning repeated}
sum(duplicated(hour_intensity))
sum(duplicated(weight_loginfo))
sum(duplicated(sleep_day))
sum(duplicated(calories))
sum(duplicated(intensity))
sum(duplicated(daily_activity))

```
Here we observe that sleep_day data set has 3 repeated rows, it needs to be eliminated.

## <span style="color:#f89875"> Eliminating repeated data </span>

```{r Eliminating repeated data}
sleep_day <- sleep_day %>% 
  distinct() %>%
  drop_na()

sum(duplicated(sleep_day)) #confirm there is no repeated data

```


## <span style="color:#f89875"> Separate date into year, month, day </span>

Now let's add new columns to data frames. These new columns will contain weekday, day, month, year.

```{r datetime split}
daily_activity <- daily_activity %>%
  mutate(weekday = wday(ActivityDate, label = TRUE)) %>% 
  mutate(day = day(ActivityDate)) %>% 
  mutate(month = month(ActivityDate, label = TRUE)) %>% 
  mutate(year = year(ActivityDate))

intensity <- intensity %>% 
  mutate(weekday = wday(ActivityDay, label = TRUE)) %>% 
  mutate(day = day(ActivityDay)) %>% 
  mutate(month = month(ActivityDay, label = TRUE)) %>% 
  mutate(year = year(ActivityDay))

calories <- calories %>% 
  mutate(weekday = wday(ActivityDay, label = TRUE)) %>%
  mutate(day = day(ActivityDay)) %>% 
  mutate(month = month(ActivityDay, label = TRUE)) %>% 
  mutate(year = year(ActivityDay))


sleep_day <- sleep_day %>% 
  mutate(weekday = wday(SleepDay, label = TRUE)) %>%
  mutate(month = month(SleepDay, label = TRUE)) %>% 
  mutate(day = day(SleepDay)) %>% 
  mutate(year = year(SleepDay)) %>% 
  mutate(hour = hour(SleepDay)) %>% 
  mutate(minute = minute(SleepDay)) %>% 
  mutate(second = second(SleepDay))

weight_loginfo <- weight_loginfo %>% 
  mutate(weekday = wday(Date, label = TRUE)) %>%
  mutate(day = day(Date)) %>% 
  mutate(month = month(Date, label = TRUE)) %>% 
  mutate(year = year(Date)) %>% 
  mutate(hour = hour(Date)) %>% 
  mutate(minute = minute(Date)) %>% 
  mutate(second = second(Date))

AMPMFLAG2 <- +grepl("PM", hour_intensity$ActivityHour) 
AMPMFLAG2 <- ifelse(AMPMFLAG2 == "0", "AM", "PM")
hour_intensity <- hour_intensity %>% 
  mutate(weekday = wday(TimeDate, label = TRUE)) %>%
  mutate(day = day(TimeDate)) %>% 
  mutate(month = month(TimeDate, label = TRUE)) %>% 
  mutate(year = year(TimeDate)) %>% 
  mutate(hour = hour(TimeDate)) %>% 
  mutate(minute = minute(TimeDate)) %>% 
  mutate(second = second(TimeDate)) %>%
  mutate(AMPMFLAG2) 
```

Now the data sets have the date column separated into various columns containing the time information. This makes it clear, to apply filters and groups. Consequently, it will be simple to understand when apply calculations.

## <span style="color:#f89875"> How many participants in each data set </span>

Count the amount of users evaluated in each data set.

```{r count participants}
n_distinct(daily_activity$Id)
n_distinct(intensity$Id)
n_distinct(calories$Id)
n_distinct(weight_loginfo$Id)
n_distinct(sleep_day$Id)
n_distinct(hour_intensity$Id)

```
#### <span style="color:#9e4513">Observations: </span> #### 

- The heart rate table has only data from  14 participants, weight table has 8 and sleep has only 24, in order to be able to analyze the sleep changes depending on activity factors it will be necessary to merge the data by users id's.

- The data set with fewest users is the weight table, this can be a signal that users are less interested in weight loss activities or they track their weight in another way instead of using the device to track their weight changes.


## <span style="color:#f89875"> View summary statistics </span>

Apply statistics calculations to know the data.

```{r statistical calculations}
#activity 
daily_activity %>% 
  select(TotalSteps,TotalDistance,Calories,day,month) %>% 
  summary()

#intensity
intensity %>% 
  mutate(TotalActiveMinutes = VeryActiveMinutes + LightlyActiveMinutes + FairlyActiveMinutes) %>% 
  select(TotalActiveMinutes,SedentaryMinutes,day,month) %>% 
  summary()

#calories
calories %>% 
  select(Calories,day,month) %>% 
  summary()

#weight_loginfo
weight_loginfo %>%
  select(WeightPounds,day,month) %>% 
  summary()

#sleep_day
sleep_day %>% 
  select(TotalMinutesAsleep,TotalTimeInBed) %>% 
  summary()

```
#### <span style="color:#9e4513">Observations</span> ####
- Tables activity,intensity, and calories are consistent with the number of records in each one, over all, users reported more lightly active time than very active time. Also, it can be noticed the diverse weight ranges in the weight_Loginfo table, from 116 to 294.3 pounds. 

- The mean sedentary minutes is 991.2 that is almost 17 hours, it is recommendable to encourage users to have more activity.

- The median minutes in bed is 463, while the median minutes asleep is 433, this means the users lasted a median of 30 minutes in the bed with no sleep. According to [*The study – published in volume 2018:10 edition of the Nature of Science and Sleep journal*]( https://doi.org/10.2147/NSS.S168841), the recommended time to fall asleep is between 10 to 20 minutes. If it goes over 30, the quality of sleep begins to decrease drastically; therefore, there should be a feature implemented that encourages users to adopt a healthy sleep routine.

# <span style="color:#9e4513">Process</span>

For this analysis I'm choosing R programming, since it allows me to create descriptive, aesthetic and objective visualizations with the package ggplot2. During data processing its necessary to filter, calculate, group, etc, which I can do using the package dplyr.

## <span style="color:#f89875">Filtering to rectify consistency</span>
To confirm consistency between intensity and daily_activity a filter was applied following the common observations and to discard or fix any inconsistent entries.

```{r rectificate / compare, results='hide'}
View(daily_activity %>% filter(SedentaryMinutes == "1440"))
View(intensity %>% filter(SedentaryMinutes == "1440"))

#now compare with date range:

View(daily_activity %>% filter(ActivityDate  >= "2016-05-10" & ActivityDate <= "2016-05-12"))
View(intensity %>% filter(ActivityDay  >= "2016-05-10" & ActivityDay <= "2016-05-12"))
```
After observing the data once those filters are applied, it can be confirmed that both tables are consistent.


## <span style="color:#f89875">Merge data </span>

Merging sleep data with the daily activity will add the sleep information to the overall activity. This will allow to analyze the influence in user's sleep as they use the device during different intensity of activity. New table will have the 24 users recorded on sleep data and their activity data in one table.

```{r merge, results='hide'}
daily_activity_sleep <- merge(sleep_day, daily_activity, by = c("Id","day", "weekday","month","year"))

#remove redundant data:

daily_activity_sleep %>% select(-hour) %>% select(-second) %>% select(-minute) %>% select(-SleepDay)
View(daily_activity_sleep)

#Add a new column with the total active minutes per day
daily_activity_sleep <- daily_activity_sleep %>% 
  mutate(TotalActiveMinutes = VeryActiveMinutes + LightlyActiveMinutes+FairlyActiveMinutes ) %>% 
  mutate(MinutesNoSleep = TotalTimeInBed - TotalMinutesAsleep)
#statistical info
daily_activity_sleep %>%
  select(SleepDay,TotalMinutesAsleep,TotalTimeInBed, TotalSteps,TotalDistance) %>% 
  summary()
head(daily_activity_sleep)  

```
#### <span style="color:#9e4513">Observations</span> ####
- Users tend to spend more time in bed than time sleeping, this data could be used to measure the sleep quality of the users or even the "insomnia" time, the more time in bed and less time sleep = less sleep quality.

# <span style="color:#9e4513">Analysis</span>

## <span style="color:#f89875">Which day of the week does users burn the most calories </span>

This graph divides the data per day of week, presenting the mean proportion of calories burned by weekday.

```{r day burn calories}

AvrCalories_weekday <- calories %>% aggregate(Calories ~ weekday, FUN = mean)


AvrCalories_weekday %>% ggplot(aes(x = weekday, y = Calories)) + geom_col(position = "dodge", fill = "#f89875")+ coord_cartesian(ylim = c(2100,2400))
```

The days with most high intense activity where Saturday and Tuesdays.

## <span style="color:#f89875"> Activity intensity by weekdays </span>

```{r Activity intensity by weekday, error=FALSE, warning=FALSE}
intensity %>% 
  summarise(VeryActiveMinutes,FairlyActiveMinutes,LightlyActiveMinutes, weekday) %>% 
  group_by(weekday) %>% 
  summarise(n=n(),
            VeryActiveMinutes=mean(VeryActiveMinutes),
            FairlyActiveMinutes = mean(FairlyActiveMinutes),
            LightlyActiveMinutes =mean(LightlyActiveMinutes)
            ) %>% 
  gather("Intensity", "Minutes", -c(weekday,n)) %>% 
  ggplot(aes(x = weekday, y = Minutes, group = Intensity, fill = Intensity)) + geom_col()+
  scale_fill_manual(values = colour_scl)


```

The graph shows that user's spend more light activity time during the day.

## <span style="color:#f89875">Correlation between Sedentary minutes and minutes asleep </span>

```{r Sedentary minutes and minutes asleep}
#head(daily_activity_sleep)
ggplot(data = daily_activity_sleep)+
  geom_point(mapping = aes(x= TotalMinutesAsleep, y=SedentaryMinutes, alpha=ActivityDate))+
  geom_smooth(mapping = aes(x= TotalMinutesAsleep, y=SedentaryMinutes), color="#9e4513", fill = "#f89875")+
  labs(title = "Sedentary Minutes vs. Sleep minutes", subtitle = "Correlation", caption = "Negative correlation")

```

The negative correlation between sedentary time and sleep time suggests that the more time a person spends sitting or inactive during the day, the less sleep they are likely to get at night. Several studies and papers have investigated this relationship between sedentary time and sleep.A [*2022 study published in the Journal of Sleep Research*](https://jassb.biomedcentral.com/articles/10.1186/s44167-022-00005-1) found that greater sedentary time was associated with poorer sleep quality, independent of physical activity and other factors. The study also suggested that reducing sedentary time may be a potential intervention for improving sleep quality.

## <span style="color:#f89875">How does the number of calories influence the sleep time  </span>

Scatter plot presenting the correlation between calories and time of sleep per day.

```{r correlation between sleep minutes and calories, warning=FALSE }
ggplot(data = daily_activity_sleep)+
  geom_point(mapping = aes(x= Calories, y=TotalMinutesAsleep, alpha=ActivityDate))+
  geom_smooth(mapping = aes(x= Calories, y=TotalMinutesAsleep), color="#9e4513", fill = "#f89875")+
  labs(title = "Calories burned vs. Sleep minutes", subtitle = "Correlation", caption = "Partially Possitive correlation")

```

As it can be observed there is there is a partially positive correlation between the quantity of calories burned in a day and the minutes of sleep. The graph also presents a consistency of values between ~400 minutes of sleep and ~2000 calories, this means that these values represent the normative.

## <span style="color:#f89875"> Correlation between activity and sleep time  </span>

Scatter plot presenting the correlation between total activity and time of sleep per day.

```{r activity vs. sleep time, warning=FALSE}
ggplot(data = daily_activity_sleep)+
  geom_point(mapping = aes(x= TotalActiveMinutes, y=TotalMinutesAsleep, alpha=ActivityDate)) +
  geom_smooth(mapping = aes(x= TotalActiveMinutes, y=TotalMinutesAsleep), color="#9e4513", fill = "#f89875")+
  labs(title = " Total Active Minutes vs. Sleep minutes", subtitle = "Correlation", caption = "Not important correlation")
```

Even if there isn't any important correlation, it can be observe that the total minutes asleep are higher between ~50 and ~250 minutes of activity, while lowest sleep time is recorded between ~200 and ~300 minutes of activity per day.

## <span style="color:#f89875">Correlation between Total distane walked and sleep time </span>

Scatter plot presenting the correlation between distance walked and time of sleep per day.

```{r  distance vs sleep time, warning=FALSE}
ggplot(data = daily_activity_sleep)+
  geom_point(mapping = aes(x= MinutesNoSleep, y=TotalActiveMinutes, alpha=ActivityDate)) +
  geom_smooth(mapping = aes(x= MinutesNoSleep, y=TotalActiveMinutes), color="#9e4513", fill = "#f89875")+
  labs(title = " Total distance vs. Sleep minutes", subtitle = "Correlation", caption = "Not important correlation")+coord_cartesian(xlim = c(0,100))
```


## <span style="color:#f89875">How does Calories quantity chaged alond use time </span>

A bar char presenting the progression in change on the amount of calories per day.

```{r change in calories burned}
mean_calories_perday<-daily_activity %>% group_by(ActivityDate) %>%summarise(AvgCalories = mean(Calories))


mean_calories_perday %>% ggplot(aes(x = ActivityDate, y = AvgCalories, fill = AvgCalories)) + geom_col(position = "dodge")+coord_cartesian(ylim = c(1000,2500))+labs(title = "Calories per day", subtitle = "Change along the days", caption = "Descendent change")+scale_fill_gradient(low = "green", high = "red")+geom_smooth(color="#9e4513", fill = "#f89875")
```

The amount of calories contributed by users decreased over time. This may be a sign of low consistency on the part of the users.

##  <span style="color:#f89875">The hour with most average activity </span>

A bar char presenting the hour of the day with most average intensity.

```{r hour with most average activity, warning=FALSE}
l <- c('12 AM', '1 AM', '2 AM', '3 AM','4 AM','5 AM', '6 AM', '7 AM', '8 AM', '9 AM', '10 AM', '11 AM', '12 PM', '1 PM', '2 PM', '3 PM','4 PM','5 PM', '6 PM', '7 PM', '8 PM', '9 PM', '10 PM', '11 PM' )
hour_intensity$Hourap <- paste(hour_intensity$hour, hour_intensity$AMPMFLAG2, sep=" ")
#glimpse(hour_intensity)
int_new<-hour_intensity %>% 
  group_by(Hourap) %>%
  summarise(Active_minutes = mean(TotalIntensity)) 
  
int_new %>% 
  ggplot(aes(x=factor(Hourap, level= l), y=Active_minutes, fill = Active_minutes)) + geom_col(position = "dodge") +theme(axis.text.x = element_text(angle = 90))+labs(x="Hour",y="Active Time",title="Most Active Hours of the Day")+ scale_fill_gradient2(high="#6e3523", mid="#f89875",low = "#ffd1bc")

```

##  <span style="color:#f89875">Proportion by user's weight groups </span>

The data set weigth_loginfo only contains weight data from 8 users (24.24% of users). To get a clear idea about user's weight relative proportion it will be precise to divide the weight data into groups. This groups will contain weight ranges; 100-150, 150-200, and 200+ pounds. Once the weight data is grouped, the groups will be calculated in proportion to the total using percentage calculation. 

```{r weight groups, warning=FALSE, error=FALSE}

# calculate user's average weight then assign it to a range (in a new data frame) 
wr<-weight_loginfo %>% 
  aggregate(WeightPounds ~ Id, FUN = mean) %>% 
  mutate(weight_range = case_when(
    WeightPounds > 100 & WeightPounds < 150 ~ "100-150 Pounds",
    WeightPounds > 150 & WeightPounds < 200 ~ "150-200 Pounds", 
    WeightPounds > 200  ~ "200+ Pounds", ))

# Calculate the proportion of each group then convert each proportion to percentage format
wr_percent <-wr %>%
  group_by(weight_range) %>%
  summarise(total = n()) %>%
  mutate(total_weights = sum(total)) %>%
  summarise(total_percent = total / total_weights) %>%
  mutate(labels = scales::percent(total_percent)) %>% 
  mutate(weight_range = c("100-150 Pounds","150-200 Pounds","200+ Pounds"))


```

Now, lets observe the proportions by weight range in a pie chart

```{r graph weight groups, warning=FALSE, error=FALSE}
pie(wr_percent$total_percent, clockwise = TRUE, labels = wr_percent$labels, col = colour_scl)
legend("topright",legend =wr_percent$weight_range, fill = colour_scl,  )

```

As it can be observed, the data collected presents a greater number of users between 150 and 200. Unfortunately the data set only contains fat percentage data re is no data describing use's age, height or genre which can be determining data to make feasible conclusions about each user's weight. To determine if the user has a healthy BMI it is necessary to record their height, nevertheless it is a very good idea to implement this function in the app.

# <span style="color:#9e4513">Conclusions </span>

After analyzing the smart device usage data for this project, I did find very interesting insights in user's behavior, including some correlations that may look intuitive but it turns out they are not. In the activity data I find some predictable behavior, such as the most active hour on the day, which is to be expected to be in after regular work hours. Another observation to consider is the day of the week with more activity

- The mean sedentary minutes is 991.2 that is almost 17 hours, it is recommendable to encourage users to have more activity.

- The days of the week with the greatest high-intensity activity are Monday and Tuesday with ~23 minutes. The day of the week with most overall activity in Saturday with a mean of 307 minutes (~6 hours).

- The correlation between sedentary minutes and sleep time is negative, meaning that as more sedentary time the less sleep time, i.e. less rest quality. The more time a person spends sitting or inactive during the day, the less sleep they are likely to get at night. To ensure a good quality of sleep it is advisable to do more physical activity, as stated in the [*2022 study published in the Journal of Sleep Research*](https://jassb.biomedcentral.com/articles/10.1186/s44167-022-00005-1) 

- Users lasted a mean of 30 minutes at bed before they get to fall asleep. This is more than recommendable according to [*The study – published in volume 2018:10 edition of the Nature of Science and Sleep journal*]( https://doi.org/10.2147/NSS.S168841), lasting more that 30 minutes reduces the sleep quality.

- According to the weight data, 50% of users have a weight between 150 and 200, 38% between 100 and 200, and 12% more than 200 pounds. Comparatively, the weight table has data from the 24.24% of total users. Also, in order to calculate the BMI precisely, it is necessary to record the user's height, this would be a very good implementation for the app.

- The hours with the most activity on the part of the users are between 5pm and 7pm, although the users present considerable activity at 12pm. This is useful for publishing purposes, given that during those hours users are more active and are more susceptible to interacting with advertising.

# <span style="color:#9e4513">Recommendations </span>

### <span style="color:#9e4513"> Implementation the Bellabeat app </span>

- Implement a function that allows the user to calculate their BMI accurately, also allowing them to add information about their height when creating an account. This will attract more of the user's attention making them feel more committed to maintain or obtain a recommended BMI in order to achieve a healthier lifestyle.

- Implement a function to track user's sleep time and provide recommendations and information about healthy sleep habits.This will motivate the user to improve their quality of life and their experience using the application. 

- Allow users to create a daily schedule to do physical activity, then send users notifications in the form of reminders to maintain motivation and avoid prolonging sedentary time.

### <span style="color:#9e4513"> Marketing </span>

- Take advantage of the busiest hours (12pm and 5pm to 7pm) and the days with the most activity of the week (Monday, Tuesday, and Saturday) to increase the advertising rate to promote the products. In this way, an increase in the interaction between the user and the products offers by the brand is ensured.
