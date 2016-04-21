---
title: "Cleaning and Aggregating UCI HAR Sensor Data"
layout: post
author: "Payam Bahreyni"
date: "2015-08-23"
categories: [projects]
tags: [Data Cleaning, dplyr, HAR, R, Sensor Data, Tidy Datasets, UCI]
description: In this project, Human Activity Recignition data from UCI machine learning repository is aggregated from multiple sources and cleaned to have a dataset ready for analysis.
---

### Purpose
Integrating and consolidating data from multiple sources.

### Context
In this project, data for Human Activity Recognition[^1] (HAR) [project](https://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones) from UCI Machine Learning Repository is downloaded, cleaned, and aggregated as a tidy data set. I have done this project as the course project for "Getting and Cleaning Data" in [Data Science Specialization](https://www.coursera.org/specializations/jhu-data-science) by John Hopkins University offered at Coursera.

[^1]: Davide Anguita, Alessandro Ghio, Luca Oneto, Xavier Parra and Jorge L. Reyes-Ortiz. A Public Domain Dataset for Human Activity Recognition Using Smartphones. 21th European Symposium on Artificial Neural Networks, Computational Intelligence and Machine Learning, ESANN 2013. Bruges, Belgium 24-26 April 2013.

### Challenge

The data for this project is dispersed across multiple files:

 * Training Data
    + **X_train.txt**: Predictor feature values (normalized -1 to 1)
    + **y_train.txt**: Class labels (1 to 6)
    + **subject_train.text**: Subject identifier (1 to 30)

 * Test Data
    + **X_test.txt**: Predictor feature values (normalized -1 to 1)
    + **y_test.txt**: Class labels (1 to 6)
    + **subject_test.text**: Subject identifier (1 to 30)
 
 * Meta-data
    + **features.txt**: Feature names
    + **activity_labels.txt**: Activity labels
 
These files need to be merged and consolidated into a tidy dataframe before any meaningful analysis can be performed. Moreover, most data analysis, visualization, and machine learning libraries require data to be in a tidy dataframe.

`dplyr` library made it easy to `select()` the features needed, `merge()` (join) datasets on feature values, and calculating group summaries using `group_by()` and `summarize_each()`.

The sensor data is finally averaged across subjects and activities and the data for mean and standard deviation for each activity are reported per subject.

### Sensor Data

There are two types on sensors used in this experiement, accelerometer and gyroscope. Each produce 3-dimensional raw signals, tAcc-XYZ and tGyro-XYZ. Body and gravity acceleration signals are extracted from the raw signals. Then body linear acceleration and angular velocity are derived in time. Also the magnitude of these three-dimensional signals are calculated. 

Finally a Fast Fourier Transform (FFT) was applied to some of these signals. The feature names and the transformations involved are detailed in the [features_info](https://github.com/pbahr/aggregating_UCI_HAR_sensor_data/blob/master/UCI%20HAR%20Dataset/features_info.txt) file.

###Links
 * Project Repository: [Cleaning and Aggregating UCI HAR Sensor Data](https://github.com/pbahr/aggregating_UCI_HAR_sensor_data)  
 * Code and Comments: [Detailed Documentation](http://htmlpreview.github.io/?http://github.com/pbahr/aggregating_UCI_HAR_sensor_data/blob/master/reports/analysis.html)

### References
