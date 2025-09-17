# Google Data Analytics Capstone Bellabeat
## How can Bellabeat Play It Smart?

Creator: Sheryl Kleppe

Date: September 17, 2025

Table of Contents:

1. [Purpose](https://github.com/SKleppe/Google-Data-Analytics-Capstone-Bellabeat/blob/main/README.md#Purpose)
2. [Data Credibility](https://github.com/SKleppe/Google-Data-Analytics-Capstone-Bellabeat/blob/main/README.md#Data-Credibility)
3. [Data Source](https://github.com/SKleppe/Google-Data-Analytics-Capstone-Bellabeat/blob/main/README.md#Data-Source)
4. [Data Dictionaries](https://github.com/SKleppe/Google-Data-Analytics-Capstone-Bellabeat/blob/main/README.md#Data-Dictionaries)
5. [Data Preparation](https://github.com/SKleppe/Google-Data-Analytics-Capstone-Bellabeat/blob/main/README.md#Data-Preparation)
6. [Data Analysis](https://github.com/SKleppe/Google-Data-Analytics-Capstone-Bellabeat/blob/main/README.md#Data-Analysis)
7. [Key Takeaways](https://github.com/SKleppe/Google-Data-Analytics-Capstone-Bellabeat/blob/main/README.md#Key-Takeaways)
8. [Next Steps](https://github.com/SKleppe/Google-Data-Analytics-Capstone-Bellabeat/blob/main/README.md#Next-Steps)

### Purpose 
[Back to Top](https://github.com/SKleppe/Google-Data-Analytics-Capstone-Bellabeat/blob/main/README.md#Google-Data-Analytics-Capstone-Bellabeat)

Analyze non-Bellabeat smart device usage to determine consumer trends and how they can be applied to Bellabeat customers and help influence Bellabeat marketing for the Leaf product.

Requested by: Urška Sršen, Bellabeat’s co-founder and Chief Creative Officer 
Presenting to: the Bellabeat Executive Team

### Data Credibility
[Back to Top](https://github.com/SKleppe/Google-Data-Analytics-Capstone-Bellabeat/blob/main/README.md#Google-Data-Analytics-Capstone-Bellabeat)

#### Reliability of Fitbit Data
Potential sources of error/biases can cause overestimated/underestimated data. 
  1. Type and speed of movement 
  2. Placement of the Fitbit
  3. Skin tone
  4. Motion artifacts

Refer to https://support.researchallofus.org/hc/en-us/articles/9651723386388-Considerations-while-using-Fitbit-Data-in-the-All-of-Us-Research-Program

#### Small and Inconsistent Sample Size
Data Description states: Thirty eligible Fitbit users
Problems:
  1. Actual dailyActivity_merged.csv has 35 and 33 unique ids, more than the data description declares
  2. Risk of misleading results
  3. Increased influence of outliers
  4. Unreliable and imprecise estimates

#### No Gender Field
Data could be male or female; The Bellabeat business model is uniquely focused on women's health and wellness. 

#### Age of Data
The data was collected in 2016, 9 years ago.
  1. Decreased relevance and accuracy
  2. Risk of misleading insights

### Data Source
[Back to Top](https://github.com/SKleppe/Google-Data-Analytics-Capstone-Bellabeat/blob/main/README.md#Google-Data-Analytics-Capstone-Bellabeat)

Crowd-sourced Fitbit datasets 03.12.2016-05.12.2016

Published May 31, 2016, Version v1

https://zenodo.org/records/53894#.X9oeh3Uzaao

### Data Dictionaries
[Back to Top](https://github.com/SKleppe/Google-Data-Analytics-Capstone-Bellabeat/blob/main/README.md#Google-Data-Analytics-Capstone-Bellabeat)

Fitabase Fitbit Dictionary

Website with PDF link: https://www.fitabase.com/resources/knowledge-base/exporting-data/data-dictionaries/

PDF version: https://www.fitabase.com/media/2126/fitabase-fitbit-data-dictionary-as-of-05162025.pdf

### Data Preparation
[Back to Top](https://github.com/SKleppe/Google-Data-Analytics-Capstone-Bellabeat/blob/main/README.md#Google-Data-Analytics-Capstone-Bellabeat)

#### Programming Language: R in Posit Cloud
  1. [dailyActivity_merged](https://github.com/SKleppe/Google-Data-Analytics-Capstone-Bellabeat/blob/main/README.md#dailyactivity_merged)
  2. [minuteSleep_merged](https://github.com/SKleppe/Google-Data-Analytics-Capstone-Bellabeat/blob/main/README.md#minuteSleep_merged)
  3. [weightLogInfo_merged](https://github.com/SKleppe/Google-Data-Analytics-Capstone-Bellabeat/blob/main/README.md#weightLogInfo_merged)

###### Install and load libraries
```R
install.packages("tidyverse")
library(tidyverse)
install.packages("janitor")
library(janitor)
```
#### dailyActivity_merged
###### Import the .csv file for dailyActivity_merged for March 12 - April 11, 2016 and April 12 - May 12, 2016
```R
daily_activity_march_april <- read_csv("mturkfitbit_export_3.12.16-4.11.16/Fitabase Data 3.12.16-4.11.16/dailyActivity_merged.csv")

daily_activity_april_may <- read_csv("mturkfitbit_export_4.12.16-5.12.16/Fitabase Data 4.12.16-5.12.16/dailyActivity_merged.csv")
```

###### Clean column names
```R
daily_activity_march_april <- clean_names(daily_activity_march_april)

daily_activity_april_may <- clean_names(daily_activity_april_may)
```

###### Change activity_date from chr to date
```R
daily_activity_march_april <- daily_activity_march_april %>% 
  mutate(activity_date = mdy(daily_activity_march_april$activity_date))

daily_activity_april_may <- daily_activity_april_may %>% 
  mutate(activity_date = mdy(daily_activity_april_may$activity_date))
```

###### Date Range needs to be within 3-12-16 to 4-11-16; Min = 3-12-16, Max = 4-12-16; 4-12-16 dates are removed
```R
daily_activity_march_april <- daily_activity_march_april %>%
  filter(activity_date!="2016-04-12")
```

###### Add dataset column
```R
daily_activity_march_april <- daily_activity_march_april %>% 
  mutate(dataset = "1")

daily_activity_april_may <- daily_activity_april_may %>% 
  mutate(dataset = "2")
```

###### Merge dailyActivity_merged 3.12.16-4.11.16 with 4.12.16-5.12.16
```R
daily_activity <- rbind(daily_activity_march_april, daily_activity_april_may) 
```

###### Duplicate Check; zero duplicates
```R
sum(duplicated(daily_activity))
```

###### Calculate very active hours, fairly active hours, lightly active hours, and sedentary sours
```R
daily_activity <- daily_activity %>% 
  mutate(
    vah = (very_active_minutes / 60),
    fah = (fairly_active_minutes / 60),
    lah = (lightly_active_minutes / 60),
    sh = (sedentary_minutes / 60)
    )
```

###### Select necessary columns for final analysis
```R
daily_activity_filtered <- daily_activity %>% 
  select(id, activity_date, total_steps, total_distance, calories, vah, fah, lah, sh, dataset)
```

###### Write to a .csv file
```R
path_out = '/cloud/project/'
file_name = paste(path_out, 'daily_activity_clean_v3.csv', sep = '')
write.csv(daily_activity_filtered, file_name, row.names=FALSE)
```
#### minuteSleep_merged

###### Import the .csv file for minuteSleep_merged for March 12 - April 11, 2016 and April 12 - May 12, 2016
```R
sleep_march_april <- read_csv("mturkfitbit_export_3.12.16-4.11.16/Fitabase Data 3.12.16-4.11.16/minuteSleep_merged.csv")

sleep_april_may <- read_csv("mturkfitbit_export_4.12.16-5.12.16/Fitabase Data 4.12.16-5.12.16/minuteSleep_merged.csv")
```

###### Clean column names
```R
sleep_march_april <- clean_names(sleep_march_april)

sleep_april_may <- clean_names(sleep_april_may)
```

###### Change activity_date from chr to date
```R
sleep_march_april <- sleep_march_april %>% 
  mutate(date = mdy_hms(sleep_march_april$date))

sleep_april_may <- sleep_april_may %>% 
  mutate(date = mdy_hms(sleep_april_may$date))
```

###### Date Range needs to be within 03-12-16 to 04-11-16; Min = 03-11-2016, Max = 04-12-2016; 03-11-2016 & 04-12-16 dates removed
```R
sleep_march_april <- sleep_march_april %>%
  filter(date(date) != "2016-04-12") %>% 
  filter(date(date) != "2016-03-11")
```

###### Date Range needs to be within 04-12-16 to 05-12-16; Min = 04-11-2016, Max = 05-12-2016; 04-11-2016 dates removed
```R
sleep_april_may <- sleep_april_may %>%
  filter(date(date) != "2016-04-11")
```

###### Add dataset column
```R
sleep_march_april <- sleep_march_april %>% 
  mutate(dataset = "1")

sleep_april_may <- sleep_april_may %>% 
  mutate(dataset = "2")
```

###### Merge minuteSleep_merged 3.12.16-4.11.16 with 4.12.16-5.12.16 
```R
sleep_log <- rbind(sleep_march_april, sleep_april_may)
```

###### Duplicate Check; 382,682 - 1,068 duplicates = 381,614 entries
```R
sum(duplicated(sleep_log))

sleep_log <- sleep_log %>%
  distinct()
```

###### Write to a .csv file
```R
path_out = '/cloud/project/'
file_name = paste(path_out, 'sleep_log_clean_v2.csv', sep = '')
write.csv(sleep_log, file_name, row.names=FALSE)
```

#### weightLogInfo_merged

###### Import the .csv file for weightLogInfo_merged for March 12 - April 11, 2016 and April 12 - May 12, 2016
```R
weight_log_march_april <- read_csv("mturkfitbit_export_3.12.16-4.11.16/Fitabase Data 3.12.16-4.11.16/weightLogInfo_merged.csv")

weight_log_april_may <- read_csv("/cloud/project/mturkfitbit_export_4.12.16-5.12.16/Fitabase Data 4.12.16-5.12.16/weightLogInfo_merged.csv")
```

###### Clean column names
```R
weight_log_march_april <- clean_names(weight_log_march_april)

weight_log_april_may <- clean_names(weight_log_april_may)
```

###### Change activity_date from chr to date
```R
weight_log_march_april <- weight_log_march_april %>% 
  mutate(date = mdy_hms(weight_log_march_april$date))

weight_log_april_may <- weight_log_april_may %>% 
  mutate(date = mdy_hms(weight_log_april_may$date))
```

###### Date Range needs to be within 03-12-16 to 04-11-16; Min = 03-30-16, Max = 04-12-16; 04-12-16 dates removed
```R
weight_log_march_april <- weight_log_march_april %>%
  filter(date(date) != "2016-04-12")
```

###### Add dataset column
```R
weight_log_march_april <- weight_log_march_april %>% 
  mutate(dataset = "1")

weight_log_april_may <- weight_log_april_may %>% 
  mutate(dataset = "2")
```

###### Merge weightLogInfo_merged 3.12.16-4.11.16 with 4.12.16-5.12.16 
```R
weight_log <- rbind(weight_log_march_april, weight_log_april_may) 
```

###### Duplicate Check; zero duplicates
```R
sum(duplicated(weight_log))
```

###### Write to a .csv file
```R
path_out = '/cloud/project/'
file_name = paste(path_out, 'weight_log_clean_v2.csv', sep = '')
write.csv(weight_log, file_name, row.names=FALSE)
```

### Data Analysis
[Back to Top](https://github.com/SKleppe/Google-Data-Analytics-Capstone-Bellabeat/blob/main/README.md#Google-Data-Analytics-Capstone-Bellabeat)

  1. [dailyActivity_merged](https://github.com/SKleppe/Google-Data-Analytics-Capstone-Bellabeat/blob/main/README.md#dailyactivity_merged-1)
  2. [minuteSleep_merged](https://github.com/SKleppe/Google-Data-Analytics-Capstone-Bellabeat/blob/main/README.md#minuteSleep_merged-1)
  3. [weightLogInfo_merged](https://github.com/SKleppe/Google-Data-Analytics-Capstone-Bellabeat/blob/main/README.md#weightLogInfo_merged-1)

#### dailyActivity_merged
###### Number of unique id's in dataset 1 (03-12-2016 to 04-11-2016) = 35
```R
dataset1_unique_ids <- daily_activity_clean_v3 %>% 
  filter(dataset == 1)

length(unique(dataset1_unique_ids$id))
```

###### Number of unique id's in dataset 2 (04-12-2016 to 05-12-2016) = 33
```R
dataset2_unique_ids <- daily_activity_clean_v3 %>% 
  filter(dataset == 2)

length(unique(dataset2_unique_ids$id))
```

###### 35 unique id's
```R
length(unique(daily_activity_clean_v3$id))
```

###### Change data from wide to long format
```R
daily_activity_clean_v3_long <- daily_activity_clean_v3 %>%
  select(id, vah, fah, lah, sh) %>% 
  pivot_longer(!id, names_to = "activity_level", values_to = "hours_per_day")
```

###### Average daily hours by activity level
```R
ggplot(daily_activity_clean_v3_long, aes(reorder(x = activity_level, -hours_per_day), y = hours_per_day, label = hours_per_day))  + 
  geom_bar(stat = "summary", fun = "mean", fill = "#A4BED5FF", color = "#453947FF") +
  geom_text(stat = "summary", fun = "mean", aes(label = round(after_stat(y), 2)), nudge_y = 1) +
  labs(title = "Average Daily Hours by Activity Level", x = "Activity Level", y = "Hours") +
  scale_x_discrete(labels = c("Sedentary", "Lightly Active", "Very Active", "Fairly Active"))
```
<img width="774" height="346" alt="Average Daily Hours by Activity Level" src="https://github.com/user-attachments/assets/e1574f4c-45d9-4e41-aa20-734e554c0afb" />

###### Distribution of user daily total steps
```R
ggplot(daily_activity_clean_v3, aes(x = total_steps)) + 
  geom_histogram(fill = "#A4BED5FF", color = "#453947FF") +
  labs(title = "Distribution of User Daily Total Steps", x = "Total Steps", y = "Count of Total Steps")
```
<img width="774" height="546" alt="Distribution of User Daily Total Steps" src="https://github.com/user-attachments/assets/0ec80f95-4c8a-49c1-8df5-d732f09d1396" />

###### Distribution of user daily total distance
```R
ggplot(daily_activity_clean_v3, aes(x = total_distance)) + 
  geom_histogram(fill = "#A4BED5FF", color = "#453947FF") +
  labs(title = "Distribution of User Daily Total Distance", x = "Total Distance", y = "Count of Total Distance")
```
<img width="774" height="546" alt="Distribution of User Daily Total Distance" src="https://github.com/user-attachments/assets/41f5d80d-6f0a-4e3a-80ca-28a5f29618d3" />

###### Total steps vs total calories
```R
ggplot(daily_activity_clean_v3, aes(x = total_steps, y = calories)) +
  geom_point(color = "#A4BED5FF")+
  geom_smooth(method = "lm", color = "#453947FF") +
  scale_x_continuous(labels = function(x) format(x, scientific = FALSE)) + 
  labs(title = "Total Steps vs Calories", x = "Total Steps", y = "Calories")
```
<img width="774" height="546" alt="Total Steps vs Calories" src="https://github.com/user-attachments/assets/7b670c6b-0988-4f37-875b-e9ab798b12ba" />

#### minuteSleep_merged
###### Number of unique id's in dataset 1 (03-12-2016 to 04-11-2016) = 23
```R
dataset1_unique_ids <- sleep_log_clean_v2 %>% 
  filter(dataset == 1)

length(unique(dataset1_unique_ids$id))
```

###### Number of unique id's in dataset 2 (04-12-2016 to 05-12-2016) = 24
```R
dataset2_unique_ids <- sleep_log_clean_v2 %>% 
  filter(dataset == 2)

length(unique(dataset2_unique_ids$id))
```

###### 25 unique id's
```R
length(unique(sleep_log_clean_v2$id))
```

###### Sleep log frequency
```R
user_entry_frequency <- sleep_log_clean_v2 %>% 
  group_by(id) %>% 
  summarise(log_entry_count = n_distinct(log_id)) %>% 
  mutate(category = ifelse(log_entry_count <= 31 , "<= 31 Logs", "> 31 Logs")) %>%
  group_by(category) %>%
  summarise(count = n()) %>%
  mutate(proportion = count / sum(count)) %>%
  arrange(desc(category)) 

ggplot(user_entry_frequency, aes(x = "", y = proportion, fill = category)) +
  geom_bar(width = 1, stat = "identity", color = "white") +
  coord_polar("y", start = 0) +
  geom_text(aes(label = paste0(round(proportion * 100), "%")),
            position = position_stack(vjust = 0.5), color = "black") +
  labs(title = "Sleep Log Frequency", subtitle = "Percent of Users",
       fill = "Category") +
  theme_void() + 
  theme(plot.title = element_text(hjust = 0.5), plot.subtitle = element_text(hjust = 0.5))
```
<img width="774" height="546" alt="Sleep Log Frequency" src="https://github.com/user-attachments/assets/e93fa666-244d-4df8-9a7f-353240359706" />

###### Average hours of sleep per user based on log id
```R
hours_of_sleep <- sleep_log_clean_v2 %>% 
  group_by(id, log_id) %>% 
  summarise(hours_of_sleep = n_distinct(date) / 60)

average_hours_of_sleep <- hours_of_sleep %>% 
  group_by(id) %>% 
  summarise(count = n(),
            average_hours_of_sleep = sum(hours_of_sleep) / count) %>% 
  mutate(category = case_when(
    average_hours_of_sleep < 7 ~ "Below Recommended Amount",
    average_hours_of_sleep >= 7 & average_hours_of_sleep <= 9 ~ "Meeting Recommended Amount",
    average_hours_of_sleep >9 ~ "Above Recommended Amount")) %>% 
    group_by(category) %>%
  summarise(count = n()) %>%
  mutate(proportion = count / sum(count)) %>%
  arrange(desc(category)) 
    
ggplot(average_hours_of_sleep, aes(x = "", y = proportion, fill = category)) +
  geom_bar(width = 1, stat = "identity", color = "white") +
  coord_polar("y", start = 0) +
  geom_text(aes(label = paste0(round(proportion * 100), "%")),
            position = position_stack(vjust = 0.5), color = "black") +
  labs(title = "Average Hours of Sleep", subtitle = "Per User Based on Log ID",
       fill = "Category") +
  theme_void() + 
  theme(plot.title = element_text(hjust = 0.5), plot.subtitle = element_text(hjust = 0.5))
```
<img width="774" height="546" alt="Average Hours of Sleep" src="https://github.com/user-attachments/assets/e9a18f49-0785-41b1-b242-43624054abb3" />

#### weightLogInfo_merged
###### Number of unique id's in dataset 1 (03-12-2016 to 04-11-2016) = 11
```R
dataset1_unique_ids <- weight_log_clean_v2 %>% 
  filter(dataset == 1)

length(unique(dataset1_unique_ids$id))
```

###### Number of unique id's in dataset 2 (04-12-2016 to 05-12-2016) = 8
```R
dataset2_unique_ids <- weight_log_clean_v2 %>% 
  filter(dataset == 2)

length(unique(dataset2_unique_ids$id))
```

###### 13 unique id's
```R
length(unique(weight_log_clean_v2$id))
```

###### Manual vs Data Sync Logs
```R
manual_datasync_count <- weight_log_clean_v2 %>% 
  group_by(is_manual_report) %>%
  summarise(count = n()) %>%
  mutate(proportion = 100 * (count / sum(count))) %>%
  arrange(desc(is_manual_report)) 

View(manual_datasync_count)

ggplot(manual_datasync_count, aes(reorder(x = is_manual_report, -proportion), y = proportion, label = proportion))  + 
  geom_bar(stat = "identity", fill = "#A4BED5FF", color = "#453947FF") +
  geom_text(stat = "identity", nudge_y = 3, aes(label = paste0(round(proportion, 2), "%"))) +
  labs(title = "Manual vs Data Sync Logs", x = "Log Method", y = "Percentage of Users") + 
    scale_x_discrete(labels = c("Manual", "Data Sync")) + 
  theme(plot.title = element_text(hjust = 0.5))
```
<img width="774" height="546" alt="Manual vs Data Sync Logs" src="https://github.com/user-attachments/assets/dc87e072-49c4-4254-be20-dd75a682e761" />

### Key Takeaways
[Back to Top](https://github.com/SKleppe/Google-Data-Analytics-Capstone-Bellabeat/blob/main/README.md#Google-Data-Analytics-Capstone-Bellabeat)

#### Target Market
* Sedentary lifestyle

* Light to moderate daily activity levels

#### Marketing Strategies
##### Daily Activity

* Guide users in gradually increasing their daily step count to enhance calorie expenditure

##### Sleep
* Focus on optimizing user engagement during sleep cycles

* Provide methods to enhance user’s sleep duration

##### Weight
* Increase frequency of weight logs

### Next Steps
[Back to Top](https://github.com/SKleppe/Google-Data-Analytics-Capstone-Bellabeat/blob/main/README.md#Google-Data-Analytics-Capstone-Bellabeat)

#### Increase Daily Step Count
* Encourage users to moderately increase activity through in-app notifications and newsletter/in-app tips and tricks such as parking farther from a store or office building.

#### Optimize User Sleep Engagement
* Discover why sleep log frequency is low through the creation of a user survey.
  
* Use results to develop a strategy to increase sleep log frequency.

#### Enhance User Sleep Duration
* Provide sleep strategies through a newsletter or in-app to build better sleep habits. 

* Create an in-app tool to develop  personalized optimal sleep routines for users.

#### Increase Weight Log Frequency
* Discover why weight log frequency is low through the development of user surveys.

* Areas to Explore:
  * Data sync scale accessibility
  * Connectability between scales and device applications
  * Methods to increase manual data logs



