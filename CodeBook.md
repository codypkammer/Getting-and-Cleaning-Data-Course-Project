---
title: "CodeBook"
output: html_document
---

### Getting and Cleaning Data Course Project

#### Load packages 
tidyverse not required, just dplyr
```{r}  
  library(tidyverse)
```

#### Download File and extract
Assigned file url to object "file" named downloaded file "data.zip"
```{r}  
  file <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
  download.file(file, "data.zip")
  unzip("data.zip")
```

#### Assigning data frames to objects
Used abbreviations for efficiency  

```{r}
  act <- read.table("UCI HAR Dataset/activity_labels.txt", col.names = c("code", "activity"))
  feat <- read.table("UCI HAR Dataset/features.txt", col.names = c("n", "functions"))
  sub_test <- read.table("UCI HAR Dataset/test/subject_test.txt", col.names = "subject")
  x_test <- read.table("UCI HAR Dataset/test/X_test.txt", col.names = feat$functions)
  y_test <- read.table("UCI HAR Dataset/test/y_test.txt", col.names = "code")
  sub_train <- read.table("UCI HAR Dataset/train/subject_train.txt", col.names = "subject")
  x_train <- read.table("UCI HAR Dataset/train/X_train.txt", col.names = feat$functions)
  y_train <- read.table("UCI HAR Dataset/train/y_train.txt", col.names = "code")
```
#### Merges the training and the test sets to create one dataset
Used rbind() function to combin test and train datasets, then cbind to create one dataset.
```{r}
  x <- rbind(x_train, x_test)
  y <- rbind(y_train, y_test)
  sub <- rbind(sub_train, sub_test)
  merged <- cbind(sub, y, x)
```
#### Extracts only the measurements on the mean and standard deviation for each measurement. 
Subset only subject and code columns then calculated the mean and standard deviation
```{r}
  tidy <- merged %>% select(subject, code, contains("mean"), contains("std"))
```
#### Uses descriptive activity names to name the activities in the data set
Converted numeric value to the corresponding activity 
```{r}  
  tidy$code <- act[tidy$code, 2]
```
#### Appropriately labels the data set with descriptive variable names
```{r}
  names(tidy)[2] = "activity"
  names(tidy)<-gsub("Acc", "Accelerometer", names(tidy))
  names(tidy)<-gsub("Gyro", "Gyroscope", names(tidy))
  names(tidy)<-gsub("BodyBody", "Body", names(tidy))
  names(tidy)<-gsub("Mag", "Magnitude", names(tidy))
  names(tidy)<-gsub("^t", "Time", names(tidy))
  names(tidy)<-gsub("^f", "Frequency", names(tidy))
  names(tidy)<-gsub("tBody", "TimeBody", names(tidy))
  names(tidy)<-gsub("-mean()", "Mean", names(tidy), ignore.case = TRUE)
  names(tidy)<-gsub("-std()", "STD", names(tidy), ignore.case = TRUE)
  names(tidy)<-gsub("-freq()", "Frequency", names(tidy), ignore.case = TRUE)
  names(tidy)<-gsub("angle", "Angle", names(tidy))
  names(tidy)<-gsub("gravity", "Gravity", names(tidy))
```  
#### From the data set in step 4, creates a second, independent tidy data set with
#### the average of each variable for each activity and each subject.
Grouped by subject, and activity,then summerized by mean.
```{r}  
  final <- tidy %>%
    group_by(subject, activity) %>%
    summarise_all(funs(mean))
  write.table(final, "final.txt", row.name=FALSE)
```  
#### Check
```{r}
str(final)
```
