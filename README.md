Analysis files - steps to generate tidy\_dataset.txt
================

### Using R version x64 4.0.2

## Load libraries

``` r
library(data.table)
library(stringr)
library(dplyr)
```

## Download data, save it to the working directory as dataset.zip and unzip the file

``` r
if (!file.exists("dataset.zip")) {
    fileUrl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
    download.file(fileUrl, destfile = "dataset.zip", method = "curl")
    unzip(zipfile = "dataset.zip", overwrite = TRUE)
}
```

## Read data con fread()

### Read variable names

``` r
col_names <- fread("./UCI HAR Dataset/features.txt", select = 2)
col_names <- col_names[[1]]
```

### Read train data

``` r
dt_X_train <- fread("./UCI HAR Dataset/train/X_train.txt", col.names = col_names)
dt_activity_train <- fread("./UCI HAR Dataset/train/Y_train.txt", col.names = "activity")
dt_subject_train <- fread("./UCI HAR Dataset/train/subject_train.txt", col.names="subject")
```

### Read test data

``` r
dt_X_test <- fread("./UCI HAR Dataset/test/X_test.txt", col.names = col_names)
dt_activity_test <- fread("./UCI HAR Dataset/test/Y_test.txt", col.names = "activity")
dt_subject_test <- fread("./UCI HAR Dataset/test/subject_test.txt", col.names="subject")
```

## Merge data

### Merge the columns to create two complete datasets: dt\_train and dt\_test

``` r
dt_train <- cbind(dt_activity_train, dt_subject_train, dt_X_train)
dt_test <- cbind(dt_activity_test, dt_subject_test, dt_X_test)
```

### Merge both datasets to create just one: dt\_initial

``` r
dt_initial <- rbind(dt_train, dt_test)
```

## Extract only the measurements on the mean and standard deviation for each measurement and create one new dataset containing only those measurements: dt\_tidy

``` r
valid_columns <- names(dt_initial)
valid_columns <- grep("activity|subject|(mean|std)\\(\\)", valid_columns, value = FALSE)
dt_tidy <- dt_initial[,..valid_columns]
```

## Uses descriptive activity names to name the activities in the data set

``` r
activities <- fread("./UCI HAR Dataset/activity_labels.txt", col.names = c("number", "description"))
activities$description <- gsub("_", " ", activities$description)
activities$description <- str_to_lower(activities$description)
dt_tidy$activity <- activities$description[dt_tidy$activity]
dt_tidy$activity <- as.factor(dt_tidy$activity)
```

## Appropriately labels the data set with descriptive variable names

``` r
col_names <- names(dt_tidy)
col_names <- sub("(mean\\(\\))","Mean",col_names)
col_names <- sub("(std\\(\\))","Std",col_names)
col_names <- gsub("-","",col_names)
col_names <- sub("^t","time",col_names)
col_names <- sub("f","frequency",col_names)
col_names <- sub("Acc","Acceleration",col_names)
col_names <- sub("Gyro","Gyroscope",col_names)
col_names <- sub("BodyBody","Body",col_names)
```

## Create a new dataset called dt\_average, from the previous dataset dt\_tidy with the average of each variable for each activity and each subject.

``` r
names(dt_tidy) <- col_names
dt_average <- dt_tidy %>%
    group_by(activity, subject) %>%
    summarize_all(mean)
```

## Save dt\_average to tidy\_dataset.txt

``` r
if (!file.exists("tidy_dataset.txt")) {
write.table(dt_average, file = "tidy_dataset.txt", row.names = FALSE)
}
```

## Help to generate code book

``` r
library(dataMaid)
makeCodebook(dt_average)
```
