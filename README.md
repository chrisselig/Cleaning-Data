# Cleaning-Data
Getting and Cleaning Data Project

---
title: "README"
author: "Chris Selig"
date: "November 12, 2016"
output: html_document
---
##README file for cleaning and tidying data project

##Packages Used in this file
library(dplyr)

##Setting Up the Workspace
###Create a directory for the data to be stored if one does not exist already
if(!file.exists("./data")){dir.create("./data")}
setwd("./data")

##Downloading Data
###Download dataset and unzip
fileurl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(fileurl, "./dataset.zip")
unzip("./dataset.zip")

###Load Variable Names
activitynames <- read.table("./UCI HAR Dataset/activity_labels.txt")
featuresnames <- read.table("./UCI HAR Dataset/features.txt")

###List text files in test directory and in train directory
test_files_list <- list.files("./UCI HAR Dataset/test", full.names = TRUE, pattern = "\\.txt$")
train_files_list <- list.files("./UCI HAR Dataset/train", full.names = TRUE, pattern = "\\.txt$")

###Load test and train data
testdata <- do.call(cbind,lapply(test_files_list,read.table))
traindata <- do.call(cbind, lapply(train_files_list, read.table))

##Requirement 1 - Merge the dataset
###Merge both test and train data together and remove old datasets
mergeddata <- rbind(testdata, traindata)
rm(testdata, traindata)

###Add names to the variables.  Names will be used to filter out everything but mean, std, subject and activity
names(mergeddata) <- c("subjectid", as.character(featuresnames$V2),"activity")
rm(featuresnames)

##Requirement 2 - Remove columns not mean, standard deviation, subject or activity
###Create search parameters then use them to subset data
searchparameters <- c("[Mm]ean|subjectid|std|activity")
columnlist <- grep(searchparameters, names(mergeddata))

###Create a new subsetted dataset
subsetteddata <- mergeddata[,columnlist]

##Requirement 3 - Use descriptive activity names to name the activities in the dataset
###Find the intersect of the activity column, and the activity names, and return the activity names instead of number
intersect <- function(x,y) {y[match(x,y,nomatch = NA)]}
subsetteddata$activity <- as.character(activitynames[intersect(subsetteddata$activity,activitynames$V1),2])

##Requirement 4 - Appropriately label the data set with descriptive variable names
###Process the variable names (set to lower case letters, remove all "-"'s, and remove all "()" )
tidyvariablenames <- tolower(names(subsetteddata))
tidyvariablenames <- gsub("-","",tidyvariablenames)
tidyvariablenames <- gsub("\\(\\)","", tidyvariablenames)
tidyvariablenames <- gsub("\\(","", tidyvariablenames)
tidyvariablenames <- gsub("\\)","", tidyvariablenames)
tidyvariablenames <- gsub(",","",tidyvariablenames)

###Set tidyvariablenames as variable names in subsetteddata
names(subsetteddata) <- tidyvariablenames

##Requirement 5 - Create a second tidy data set with the average of each variable for each activity and each subject
groupedbydata <- subsetteddata %>% group_by(activity, subjectid) %>% summarize_each(funs(mean))

###Export Table
write.table(groupedbydata, file = "finaltidydata.txt", row.names = FALSE)

