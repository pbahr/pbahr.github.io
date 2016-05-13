---
layout: post
title: "Titanic Kaggle Machine Learning Competition With R - Part 1: Knowing and Preparing The Data"
author: "Payam Bahreyni"
date: 2016-04-25
categories: [tutorials]
tags : [Classification, EDA, Kaggle, Machine Learning, R]
output: 
  html_document: 
    toc: yes
---

## Context

There is a famous "Getting Started" machine learning competition on Kaggle, called [Titanic: Machine Learning from Disaster](https://www.kaggle.com/c/titanic). It is just there for us to experiment with the data and the different algorithms and to measure our progress against benchmarks. We are given the data about passengers of Titanic. Our goal is to predict which passengers survived the tragedy.

There are multiple useful tutorials out there discussing the same problem and dataset [^1] [^2]. This tutorial is geared towards people who are already familiar with R willing to learn some machine learning concepts, without dealing with too much technical details.

In part 1, we will know the data a little bit and prepare it for further analysis.

## Loading Data & Initial Analysis

Data is given as two separate files for training and test. Our goal is to predict `Survived` variable for the **test** dataset. We will use the **training** set to learn from the data.





I have moved my user-defined functions to `library.R` file to keep the code clean here. You can check it out at the GitHub repository for this project.


{% highlight r linenos %}
source("library.R")
{% endhighlight %}

I'm using the `printr` package for a better-looking print output. You can [download](https://github.com/yihui/printr) it if you liked the output.

{% highlight r linenos %}
library(printr)
{% endhighlight %}


{% highlight r linenos %}
# Load data
train.data <- read.csv("input/train.csv", stringsAsFactors = F)
test.data <- read.csv("input/test.csv", stringsAsFactors = F)

# Keep the outcome in a separate variable
train.outcome <- train.data$Survived
train.outcome <- factor(train.outcome)

# Remove "Survived" column
train.data <- train.data[,-2]

train.len <- nrow(train.data)
test.len <- nrow(test.data)

# Combine training & testing data
full.data <- rbind(train.data, test.data)

# Create factor version of categorical vars
full.data$Pclass.factor <- factor(full.data$Pclass)
levels(full.data$Pclass.factor) <- paste0("Class.", levels(full.data$Pclass.factor))

full.data$Sex.factor <- factor(full.data$Sex)
full.data$Embarked.factor <- factor(full.data$Embarked)
{% endhighlight %}


{% highlight r linenos %}
str(full.data)
{% endhighlight %}



{% highlight text %}
## 'data.frame':	1309 obs. of  14 variables:
##  $ PassengerId    : int  1 2 3 4 5 6 7 8 9 10 ...
##  $ Pclass         : int  3 1 3 1 3 3 1 3 3 2 ...
##  $ Name           : chr  "Braund, Mr. Owen Harris" "Cumings, Mrs. John Bradley (Florence Briggs Thayer)" "Heikkinen, Miss. Laina" "Futrelle, Mrs. Jacques Heath (Lily May Peel)" ...
##  $ Sex            : chr  "male" "female" "female" "female" ...
##  $ Age            : num  22 38 26 35 35 NA 54 2 27 14 ...
##  $ SibSp          : int  1 1 0 1 0 0 0 3 0 1 ...
##  $ Parch          : int  0 0 0 0 0 0 0 1 2 0 ...
##  $ Ticket         : chr  "A/5 21171" "PC 17599" "STON/O2. 3101282" "113803" ...
##  $ Fare           : num  7.25 71.28 7.92 53.1 8.05 ...
##  $ Cabin          : chr  "" "C85" "" "C123" ...
##  $ Embarked       : chr  "S" "C" "S" "S" ...
##  $ Pclass.factor  : Factor w/ 3 levels "Class.1","Class.2",..: 3 1 3 1 3 3 1 3 3 2 ...
##  $ Sex.factor     : Factor w/ 2 levels "female","male": 2 1 1 1 2 2 2 2 1 1 ...
##  $ Embarked.factor: Factor w/ 4 levels "","C","Q","S": 4 2 4 4 4 3 4 4 4 2 ...
{% endhighlight %}


{% highlight r linenos %}
# Split data back into training and testing
train.data <- full.data[1:train.len, ]
test.data <- full.data[(train.len+1):nrow(full.data), ]

# Check the summary of quantitative and categorical variables
sapply(train.data[, -c(1, 3, 8, 10)], summary)
{% endhighlight %}



{% highlight text %}
## $Pclass
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##   1.000   2.000   3.000   2.309   3.000   3.000 
## 
## $Sex
##    Length     Class      Mode 
##       891 character character 
## 
## $Age
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
##    0.42   20.12   28.00   29.70   38.00   80.00     177 
## 
## $SibSp
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##   0.000   0.000   0.000   0.523   1.000   8.000 
## 
## $Parch
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##  0.0000  0.0000  0.0000  0.3816  0.0000  6.0000 
## 
## $Fare
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##    0.00    7.91   14.45   32.20   31.00  512.30 
## 
## $Embarked
##    Length     Class      Mode 
##       891 character character 
## 
## $Pclass.factor
## Class.1 Class.2 Class.3 
##     216     184     491 
## 
## $Sex.factor
## female   male 
##    314    577 
## 
## $Embarked.factor
##       C   Q   S 
##   2 168  77 644
{% endhighlight %}



{% highlight r linenos %}
# Add the outcome column
train.data$Survived <- train.outcome
{% endhighlight %}

So far, we have put together the **training** and **test** data, converted the categorical variables into `factor`, and then separated the data sets. It is easier to apply the transformations this way, rather than doing the same thing twice on different data sets. Finally, we summarized the quantitative and categorical data to get some sense of the **training** data.

### Variable Definitions

The following definitions are given at the competition website:

**Variable Definitions**

* survival:  Survival (0 = No; 1 = Yes)  
* Pclass:    Passenger Class (1 = 1st; 2 = 2nd; 3 = 3rd)  
* name:      Name  
* sex:       Sex  
* age:       Age  
* sibsp:     Number of Siblings/Spouses Aboard  
* parch:     Number of Parents/Children Aboard  
* ticket:    Ticket Number  
* fare:      Passenger Fare  
* cabin:     Cabin  
* embarked:  Port of Embarkation (C = Cherbourg; Q = Queenstown; S = Southampton)

**Special Notes**

* Pclass is a proxy for socio-economic status (SES): 1st ~ Upper; 2nd ~ Middle; 3rd ~ Lower

* Age is in Years; Fractional if Age less than One (1). If the Age is Estimated, it is in the form xx.5  

* With respect to the family relation variables (i.e. sibsp and parch) some relations were ignored. The following are the definitions used for sibsp and parch.

* Sibling:  Brother, Sister, Stepbrother, or Stepsister of Passenger Aboard Titanic  
* Spouse:   Husband or Wife of Passenger Aboard Titanic (Mistresses and Fiances Ignored)  
* Parent:   Mother or Father of Passenger Aboard Titanic  
* Child:    Son, Daughter, Stepson, or Stepdaughter of Passenger Aboard Titanic  

* Other family relatives excluded from this study include cousins, nephews/nieces, aunts/uncles, and in-laws.  
* Some children travelled only with a nanny, therefore parch=0 for them.  
* As well, some travelled with very close friends or neighbors in a village, however, the definitions do not support such relations.

### Findings 

* Most people travelled in Class 3.
* Most passengers were male.
* For the passengers with Age specified, 50% are older than 28. We have a lot of missing values in Age variable.
* Most passengers did not travel with their spouse or siblings on board. 
* Most passengers do not have their parents or children on board.
* 75% of the passengers paid less than $31 for fare, and the maximum fare paid was $512.
* Most passengers came on board on the port specified by "S" (Southampton).

## Exploratory Data Analysis (EDA)

The next logical step is to do some exploratory analysis to get more familiar with the data at hand. Let's take a look at different groups and their survival rate.


{% highlight r linenos %}
prop.sex <- prop.table(table(train.data$Sex.factor, useNA = "ifany"))
prop.class <-  prop.table(table(train.data$Pclass.factor, useNA = "ifany"))

percent(prop.sex, digits = 2)
{% endhighlight %}



| female|  male|
|------:|-----:|
|  35.24| 64.76|

{% highlight r linenos %}
percent(prop.class, digits = 2)
{% endhighlight %}



| Class.1| Class.2| Class.3|
|-------:|-------:|-------:|
|   24.24|   20.65|   55.11|

Majority of passengers are men (65%) and passengers have 3 different classes, 1 (24%), 2 (21%), or 3 (55%).

### Survival Rate


{% highlight r linenos %}
table(train.data$Survived, useNA = "ifany")
{% endhighlight %}



|   0|   1|
|---:|---:|
| 549| 342|

{% highlight r linenos %}
prop.survived <- prop.table(table(train.data$Survived, useNA = "ifany"))
percent(prop.survived, digits = 2)
{% endhighlight %}



|     0|     1|
|-----:|-----:|
| 61.62| 38.38|

{% highlight r linenos %}
prop.sex.survived <- prop.table(table(Sex= train.data$Sex.factor, Survived= train.data$Survived), margin = 1)
prop.class.survived <- prop.table(table(Pclass= train.data$Pclass.factor, Survived= train.data$Survived), margin = 1)

percent(prop.sex.survived, digits = 2)
{% endhighlight %}



|Sex/Survived |     0|     1|
|:------------|-----:|-----:|
|female       | 25.80| 74.20|
|male         | 81.11| 18.89|

{% highlight r linenos %}
percent(prop.class.survived, digits = 2)
{% endhighlight %}



|Pclass/Survived |     0|     1|
|:---------------|-----:|-----:|
|Class.1         | 37.04| 62.96|
|Class.2         | 52.72| 47.28|
|Class.3         | 75.76| 24.24|

Most people didn't survive (62%). But the survival rate is not the same across different groups. Females had higher chance of survival, 74% as compared to 19% for men. Class 1 passengers had 63% chance of survival, compared to 47% and 24% for class 2 and 3, respectively. 

We see that survival rate is different across different classes, but we're not sure yet if this is the result of different proportion of females across different passenger classes. Let's check if this is the case.


{% highlight r linenos %}
prop.table(table(Pclass= train.data$Pclass, Sex= train.data$Sex), margin = 1)
{% endhighlight %}



|Pclass/Sex |    female|      male|
|:----------|---------:|---------:|
|1          | 0.4351852| 0.5648148|
|2          | 0.4130435| 0.5869565|
|3          | 0.2932790| 0.7067210|

The classes with a better survival rate have higher proportion of females.


{% highlight r linenos %}
prop.class.sex.survived <- prop.table(table(Pclass= train.data$Pclass, Sex= train.data$Sex.factor, Survived= train.data$Survived), margin = 1:2)
percent(prop.class.sex.survived, digits = 2)
{% endhighlight %}



|Pclass |Sex    |Survived |  Freq|
|:------|:------|:--------|-----:|
|1      |female |0        |  3.19|
|       |       |1        | 96.81|
|       |male   |0        | 63.11|
|       |       |1        | 36.89|
|2      |female |0        |  7.89|
|       |       |1        | 92.11|
|       |male   |0        | 84.26|
|       |       |1        | 15.74|
|3      |female |0        | 50.00|
|       |       |1        | 50.00|
|       |male   |0        | 86.46|
|       |       |1        | 13.54|

The **Freq** [^3] column shows survival proportion of people with the same gender and passenger class. The survival rate in different classes may have some relationship to percentage of women in those classes. Obviously, the male passengers have a disadvantage across the board.

### Data Partitioning


{% highlight r linenos %}
percent(train.len/(train.len + test.len), digits = 2)
{% endhighlight %}



{% highlight text %}
## [1] 68.07
{% endhighlight %}



{% highlight r linenos %}
percent(test.len/(train.len + test.len), digits = 2)
{% endhighlight %}



{% highlight text %}
## [1] 31.93
{% endhighlight %}

The data is partitioned into 68% for training dataset and 32% for test data set.

## Graphical Analysis

{% highlight r linenos %}
library(ggplot2)
{% endhighlight %}


{% highlight r linenos %}
ggplot(train.data, aes(Sex.factor, fill= Survived)) +
    geom_bar(stat = "bin", position = "stack") +
    ggtitle("Women have higher chance of survival")
{% endhighlight %}

![plot of chunk ga-2](/figure/source/2016-04-25-titanic_kaggle_competition_with_r_part_1/ga-2-1.png) 

{% highlight r linenos %}
ggplot(train.data, aes(x=Pclass.factor, fill= Survived)) +
    geom_bar(stat = "bin", position = "stack") +
    ggtitle("In class 1 most people survived and in class 3 most did not")
{% endhighlight %}

![plot of chunk ga-2](/figure/source/2016-04-25-titanic_kaggle_competition_with_r_part_1/ga-2-2.png) 

{% highlight r linenos %}
ggplot(train.data, aes(x=Pclass.factor, fill= Survived))+
    geom_bar(stat = "bin", position = "stack") +
    facet_wrap(~Sex.factor) +
    ggtitle("Most women survived across passenger classes")
{% endhighlight %}

![plot of chunk ga-2](/figure/source/2016-04-25-titanic_kaggle_competition_with_r_part_1/ga-2-3.png) 

{% highlight r linenos %}
ggplot(train.data, aes(x= Pclass.factor, y= Age, color= Survived)) +
    geom_violin() +
    geom_jitter(alpha= .5) +
    ggtitle("Age Matters!")
{% endhighlight %}



{% highlight text %}
## Warning: Removed 177 rows containing non-finite values
## (stat_ydensity).
{% endhighlight %}



{% highlight text %}
## Warning: Removed 177 rows containing missing values (geom_point).
{% endhighlight %}

![plot of chunk ga-2](/figure/source/2016-04-25-titanic_kaggle_competition_with_r_part_1/ga-2-4.png) 

Although we have more that 10% missing values for `Age` variable in the training data, we can still see some patterns regarding the effect of passenger's age on survival. Specifically, kids and teenagers had a better chance of survival while elderly were at a disadvantage.


{% highlight r linenos %}
ggplot(train.data, aes(x= Sex.factor, y= Age, color= Survived)) +
    geom_jitter() +
    geom_violin() +
    facet_wrap(~Pclass.factor)
{% endhighlight %}



{% highlight text %}
## Warning: Removed 30 rows containing non-finite values (stat_ydensity).
{% endhighlight %}



{% highlight text %}
## Warning: Removed 11 rows containing non-finite values (stat_ydensity).
{% endhighlight %}



{% highlight text %}
## Warning: Removed 136 rows containing non-finite values
## (stat_ydensity).
{% endhighlight %}



{% highlight text %}
## Warning: Removed 30 rows containing missing values (geom_point).
{% endhighlight %}



{% highlight text %}
## Warning: Removed 11 rows containing missing values (geom_point).
{% endhighlight %}



{% highlight text %}
## Warning: Removed 136 rows containing missing values (geom_point).
{% endhighlight %}

![plot of chunk ga-3](/figure/source/2016-04-25-titanic_kaggle_competition_with_r_part_1/ga-3-1.png) 

{% highlight r linenos %}
ggplot(train.data, aes(y=Fare, x= Survived)) +
    geom_boxplot() +
    facet_wrap(~Pclass.factor, ncol = 3, scales = "free") +
    ggtitle("People who survived paid higher fare on average")
{% endhighlight %}

![plot of chunk ga-3](/figure/source/2016-04-25-titanic_kaggle_competition_with_r_part_1/ga-3-2.png) 

## Statistical Analysis

We saw that fare may have some effect on the survival rate. Let's see if the effect is real.


{% highlight r linenos %}
aggregate(Fare ~ Survived + Pclass.factor, data=train.data, mean)
{% endhighlight %}



|Survived |Pclass.factor |     Fare|
|:--------|:-------------|--------:|
|0        |Class.1       | 64.68401|
|1        |Class.1       | 95.60803|
|0        |Class.2       | 19.41233|
|1        |Class.2       | 22.05570|
|0        |Class.3       | 13.66936|
|1        |Class.3       | 13.69489|

{% highlight r linenos %}
t.test(Fare ~ Survived, data= train.data, subset = train.data$Pclass.factor == "Class.1")$conf.int
{% endhighlight %}



{% highlight text %}
## [1] -50.58826 -11.25978
## attr(,"conf.level")
## [1] 0.95
{% endhighlight %}



{% highlight r linenos %}
t.test(Fare ~ Survived, data= train.data, subset = train.data$Pclass.factor == "Class.2")$conf.int
{% endhighlight %}



{% highlight text %}
## [1] -6.475511  1.188767
## attr(,"conf.level")
## [1] 0.95
{% endhighlight %}



{% highlight r linenos %}
t.test(Fare ~ Survived, data= train.data, subset = train.data$Pclass.factor == "Class.3")$conf.int
{% endhighlight %}



{% highlight text %}
## [1] -2.319980  2.268933
## attr(,"conf.level")
## [1] 0.95
{% endhighlight %}

So, we see that in class 1, the difference in fare is statistiscally significant between passengers who survived and who didn't . Maybe the rich found a way to buy lifeboats :).

In [part 2]({% post_url 2016-05-02-titanic_kaggle_competition_with_r_part_2 %}), we will start doing machine learning and submit our first prediction to Kaggle!

## References and Footnotes

[^1]: http://trevorstephens.com/post/72916401642/titanic-getting-started-with-r
[^2]: https://campus.datacamp.com/courses/kaggle-r-tutorial-on-machine-learning/chapter-1-raising-anchor?ex=1
[^3]: I know it's not a proper name for the column. It is the default name of the column used by `printr` package when the table is represented as a data frame.
