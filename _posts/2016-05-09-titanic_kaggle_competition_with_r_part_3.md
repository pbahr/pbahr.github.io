---
layout: post
title: "Titanic Kaggle Machine Learning Competition With R - Part 3: Selecting and Tuning The Model"
author: "Payam Bahreyni"
date: 2016-05-09
categories: [tutorials]
tags : [caret, Classification, Cross Validation, Decision Trees, Kaggle, Machine Learning, Model Tuning, Model Selection, R, rpart]
output: 
  html_document: 
    toc: yes
---






## Tutorial Table of Contents

* **Part 1**: [Knowing and Preparing the Data]({% post_url 2016-04-25-titanic_kaggle_competition_with_r_part_1 %})
* **Part 2**: [Learning From Data]({% post_url 2016-05-02-titanic_kaggle_competition_with_r_part_2 %})
* **Part 3**: Selecting and Tuning the Model
<hr />

In [part 2]({% post_url 2016-05-02-titanic_kaggle_competition_with_r_part_2%}), we tried different models to predict passenger survival on Titanic. We know that an increase in training accurracy, does not always result in increase in the Kaggle score. Herein, we discuss more details.

## Overfitting & Cross-validation

In the machine learning practice, we are intersted in predictive models capable of predicting something in the real world. We'd like to *predict* something given all we know about the instance. So, our models should be capable of being generalized to cover real-world scenarios.

When we check the accuracy of our predictions against the training data and maximize our accuracy in this setting, there is a risk of **overfitting** the training data, meaning we have captured so much of the outliers, exceptions, and particularities of the training data that the resulting model cannot be generalized anymore. This is what has happened with our `sex.class.age.tree` model.

What can be done to avoid overfitting? We can create different versions of the training data, keeping part of the training data as test set, evaluate our model, and pick the model that performs best, on average. This is called **cross-validation**. Let's see a practical example of cross-validation.

### K-fold Cross Validation

One of the popular methods of cross validation is K-fold cross validation. In this method, the training data is split into `k` partitions. Each time, we leave out one of the partitions and train the model on the rest, then test and measure the performance on the partition not used for training. To determine the model performance, average performance of all the `k` runs will be used.


{% highlight r linenos %}
cvCtrl <- trainControl(method= "cv", number = 10) # use 10-fold cross validation
sex.class.age.tree <- train(Survived ~ Sex.factor + Pclass.factor + Age, 
                            data= train.data.impute, method= "rpart",
                            trControl= cvCtrl)
{% endhighlight %}

`trainControl` function is used to create a 10-fold cross validation control object. This object is then passed to the `train` function. Decision trees are controlled by `cp` (complexity parameter), which tells the algorithm to stop when the measure (accuracy here) does not improve by this factor. So, we are using 10-fold cross validation to find an appropriate `cp`.

`plot.train` and `print.train` show the details of training, i.e. how accuracy changed by changing `cp`.


{% highlight r linenos %}
plot.train(sex.class.age.tree)
{% endhighlight %}

![plot of chunk unnamed-chunk-3](/figure/source/2016-05-09-titanic_kaggle_competition_with_r_part_3/unnamed-chunk-3-1.png) 

{% highlight r linenos %}
print.train(sex.class.age.tree)
{% endhighlight %}



{% highlight text %}
## CART 
## 
## 891 samples
##  14 predictor
##   2 classes: '0', '1' 
## 
## No pre-processing
## Resampling: Cross-Validated (10 fold) 
## Summary of sample sizes: 802, 802, 802, 802, 801, 803, ... 
## Resampling results across tuning parameters:
## 
##   cp          Accuracy   Kappa      Accuracy SD  Kappa SD  
##   0.01754386  0.7787189  0.5249494  0.04416875   0.09320387
##   0.01949318  0.7764717  0.5199655  0.04168930   0.08853944
##   0.44444444  0.6994311  0.2851155  0.07529678   0.24749698
## 
## Accuracy was used to select the optimal model using  the
##  largest value.
## The final value used for the model was cp = 0.01754386.
{% endhighlight %}



{% highlight r linenos %}
plot.decision.tree(sex.class.age.tree$finalModel)
{% endhighlight %}

![plot of chunk unnamed-chunk-3](/figure/source/2016-05-09-titanic_kaggle_competition_with_r_part_3/unnamed-chunk-3-2.png) 




{% highlight r linenos %}
conMat$table
{% endhighlight %}



{% highlight text %}
##           Reference
## Prediction   0   1
##          0 511 125
##          1  38 217
{% endhighlight %}



{% highlight r linenos %}
percent(conMat$overall["Accuracy"])
{% endhighlight %}



{% highlight text %}
## Accuracy 
##    81.71
{% endhighlight %}

The default value for `cp` is `0.01` and that's why our tree didn't change compared to what we had at the end of part 2. 

Another parameter to control the training behavior is `tuneLength`, which tells how many instances to use for training. The default value for `tuneLength` is `3`, meaning 3 different values will be used per control parameter. Since we only have one control parameter (`cp`), it will try three different trees with different values for `cp`. Let's change this default to see what happens.


{% highlight r linenos %}
cvCtrl <- trainControl(method= "cv", number = 10)
sex.class.age.tree <- train(Survived ~ Sex.factor + Pclass.factor + Age, 
                            data= train.data.impute, method= "rpart",
                            trControl= cvCtrl,
                            tuneLength= 5)
{% endhighlight %}


{% highlight r linenos %}
plot.train(sex.class.age.tree)
{% endhighlight %}

![plot of chunk unnamed-chunk-5](/figure/source/2016-05-09-titanic_kaggle_competition_with_r_part_3/unnamed-chunk-5-1.png) 

{% highlight r linenos %}
print.train(sex.class.age.tree)
{% endhighlight %}



{% highlight text %}
## CART 
## 
## 891 samples
##  14 predictor
##   2 classes: '0', '1' 
## 
## No pre-processing
## Resampling: Cross-Validated (10 fold) 
## Summary of sample sizes: 802, 802, 801, 801, 803, 802, ... 
## Resampling results across tuning parameters:
## 
##   cp           Accuracy   Kappa      Accuracy SD  Kappa SD  
##   0.003654971  0.8104824  0.5906400  0.05059508   0.10916542
##   0.005847953  0.8138659  0.5925945  0.05209365   0.11362365
##   0.017543860  0.7812927  0.5287542  0.05152307   0.10568062
##   0.019493177  0.7767983  0.5185902  0.04676792   0.09669188
##   0.444444444  0.7048635  0.2951663  0.07967502   0.26103823
## 
## Accuracy was used to select the optimal model using  the
##  largest value.
## The final value used for the model was cp = 0.005847953.
{% endhighlight %}

As we see in the output, `5` different trees with different `cp`s have been tried and the `cp` with highest accuracy has been picked. Because the resulting `cp` is lower that the default value, we get a different tree this time.


{% highlight r linenos %}
plot.decision.tree(sex.class.age.tree$finalModel)
{% endhighlight %}

![plot of chunk unnamed-chunk-6](/figure/source/2016-05-09-titanic_kaggle_competition_with_r_part_3/unnamed-chunk-6-1.png) 


{% highlight r linenos %}
sex.class.age.survival <- predict(sex.class.age.tree, train.data.impute)
conMat <- confusionMatrix(sex.class.age.survival, train.data.impute$Survived)
{% endhighlight %}


{% highlight r linenos %}
conMat$table
{% endhighlight %}



{% highlight text %}
##           Reference
## Prediction   0   1
##          0 511 113
##          1  38 229
{% endhighlight %}



{% highlight r linenos %}
percent(conMat$overall["Accuracy"])
{% endhighlight %}



{% highlight text %}
## Accuracy 
##    83.05
{% endhighlight %}


{% highlight r linenos %}
sex.class.age.tl.solution <- get.solution(sex.class.age.tree, test.data.impute)

if(!file.exists("output/new versions/sex_class_age_tl_cv.csv"))
    write.csv(sex.class.age.tl.solution, file= "output/new versions/sex_class_age_tl_cv.csv" , row.names= FALSE)
{% endhighlight %}

Compared to our last `sex.class.age.tree` we have **both** higher training accuracy (84.18%) *and* better Kaggle score (0.75598). What if we continue increasing `tuneLength`?


{% highlight r linenos %}
cvCtrl <- trainControl(method= "cv", number = 10)
sex.class.age.tree <- train(Survived ~ Sex.factor + Pclass.factor + Age, 
                            data= train.data.impute, method= "rpart",
                            trControl= cvCtrl,
                            tuneLength= 10)
{% endhighlight %}


{% highlight r linenos %}
plot.train(sex.class.age.tree)
{% endhighlight %}

![plot of chunk unnamed-chunk-8](/figure/source/2016-05-09-titanic_kaggle_competition_with_r_part_3/unnamed-chunk-8-1.png) 

{% highlight r linenos %}
print.train(sex.class.age.tree)
{% endhighlight %}



{% highlight text %}
## CART 
## 
## 891 samples
##  14 predictor
##   2 classes: '0', '1' 
## 
## No pre-processing
## Resampling: Cross-Validated (10 fold) 
## Summary of sample sizes: 802, 802, 802, 803, 802, 801, ... 
## Resampling results across tuning parameters:
## 
##   cp          Accuracy   Kappa      Accuracy SD  Kappa SD  
##   0.00000000  0.8238412  0.6222064  0.05171043   0.10660279
##   0.04938272  0.7868239  0.5417912  0.03245147   0.07181552
##   0.09876543  0.7868239  0.5417912  0.03245147   0.07181552
##   0.14814815  0.7868239  0.5417912  0.03245147   0.07181552
##   0.19753086  0.7868239  0.5417912  0.03245147   0.07181552
##   0.24691358  0.7868239  0.5417912  0.03245147   0.07181552
##   0.29629630  0.7868239  0.5417912  0.03245147   0.07181552
##   0.34567901  0.7868239  0.5417912  0.03245147   0.07181552
##   0.39506173  0.7868239  0.5417912  0.03245147   0.07181552
##   0.44444444  0.6901901  0.2446715  0.08101274   0.26126230
## 
## Accuracy was used to select the optimal model using  the
##  largest value.
## The final value used for the model was cp = 0.
{% endhighlight %}



{% highlight r linenos %}
plot.decision.tree(sex.class.age.tree$finalModel)
{% endhighlight %}

![plot of chunk unnamed-chunk-8](/figure/source/2016-05-09-titanic_kaggle_competition_with_r_part_3/unnamed-chunk-8-2.png) 

We may get too many similar trees that won't help. The result is a complete tree with branches not pruned and some nodes too sepcific. So, it is not always a good idea to try a lot of different trees.

## Putting it together

Let's bring `Fare` back with all the tuning in place.


{% highlight r linenos %}
cvCtrl <- trainControl(method= "cv", number = 10)
sex.class.fare.age.tree <- train(Survived ~ Sex + Pclass + Fare + Age, 
                                 data= train.data.impute, method= "rpart",
                                 trControl= cvCtrl,
                                 tuneLength= 5)

sex.class.fare.age.survival <- predict(sex.class.fare.age.tree, train.data.impute)
conMat <- confusionMatrix(sex.class.fare.age.survival, train.data.impute$Survived)
{% endhighlight %}


{% highlight r linenos %}
conMat$table
{% endhighlight %}



{% highlight text %}
##           Reference
## Prediction   0   1
##          0 497  91
##          1  52 251
{% endhighlight %}



{% highlight r linenos %}
percent(conMat$overall["Accuracy"])
{% endhighlight %}



{% highlight text %}
## Accuracy 
##    83.95
{% endhighlight %}



{% highlight r linenos %}
plot.decision.tree(sex.class.fare.age.tree$finalModel)
{% endhighlight %}

![plot of chunk unnamed-chunk-9](/figure/source/2016-05-09-titanic_kaggle_competition_with_r_part_3/unnamed-chunk-9-1.png) 


{% highlight r linenos %}
sex.class.fare.age.solution <- get.solution(sex.class.fare.age.tree, test.data.impute)
write.csv(sex.class.fare.age.solution, file= "output/new versions/sex_class_fare_age_cv_tl.csv" , row.names= FALSE)
{% endhighlight %}


{% highlight r linenos %}
plot.train(sex.class.fare.age.tree)
{% endhighlight %}

![plot of chunk unnamed-chunk-10](/figure/source/2016-05-09-titanic_kaggle_competition_with_r_part_3/unnamed-chunk-10-1.png) 

{% highlight r linenos %}
print.train(sex.class.fare.age.tree)
{% endhighlight %}



{% highlight text %}
## CART 
## 
## 891 samples
##  14 predictor
##   2 classes: '0', '1' 
## 
## No pre-processing
## Resampling: Cross-Validated (10 fold) 
## Summary of sample sizes: 801, 802, 802, 802, 802, 802, ... 
## Resampling results across tuning parameters:
## 
##   cp          Accuracy   Kappa      Accuracy SD  Kappa SD  
##   0.00877193  0.8102321  0.5877584  0.02782632   0.05804027
##   0.01461988  0.8045758  0.5727552  0.03785584   0.08719859
##   0.02046784  0.8011664  0.5710380  0.04142650   0.08729472
##   0.03070175  0.7866221  0.5409911  0.03946421   0.08365380
##   0.44444444  0.7039255  0.2974281  0.08055357   0.26199359
## 
## Accuracy was used to select the optimal model using  the
##  largest value.
## The final value used for the model was cp = 0.00877193.
{% endhighlight %}



.75598


{% highlight r linenos %}
sex.class.fare.age.tree <- train(Survived ~ Sex + Pclass + Fare + Age, 
                                 data= train.data.impute, method= "rpart",
                                 trControl= cvCtrl,
                                 tuneLength= 5)

sex.class.fare.age.survival <- predict(sex.class.fare.age.tree, train.data.impute)
conMat <- confusionMatrix(sex.class.fare.age.survival, train.data.impute$Survived)
conMat$table
{% endhighlight %}



{% highlight text %}
##           Reference
## Prediction   0   1
##          0 497  91
##          1  52 251
{% endhighlight %}



{% highlight r linenos %}
percent(conMat$overall["Accuracy"])
{% endhighlight %}



{% highlight text %}
## Accuracy 
##    83.95
{% endhighlight %}



{% highlight r linenos %}
plot.decision.tree(sex.class.fare.age.tree$finalModel)
{% endhighlight %}

![plot of chunk unnamed-chunk-12](/figure/source/2016-05-09-titanic_kaggle_competition_with_r_part_3/unnamed-chunk-12-1.png) 

{% highlight r linenos %}
plot.train(sex.class.fare.age.tree)
{% endhighlight %}

![plot of chunk unnamed-chunk-12](/figure/source/2016-05-09-titanic_kaggle_competition_with_r_part_3/unnamed-chunk-12-2.png) 

{% highlight r linenos %}
print.train(sex.class.fare.age.tree)
{% endhighlight %}



{% highlight text %}
## CART 
## 
## 891 samples
##  14 predictor
##   2 classes: '0', '1' 
## 
## No pre-processing
## Resampling: Cross-Validated (10 fold) 
## Summary of sample sizes: 802, 802, 802, 802, 801, 801, ... 
## Resampling results across tuning parameters:
## 
##   cp          Accuracy   Kappa      Accuracy SD  Kappa SD  
##   0.00877193  0.8249648  0.6182833  0.03437089   0.07644665
##   0.01461988  0.8170996  0.6058558  0.03711911   0.08132032
##   0.02046784  0.8070247  0.5817373  0.04340037   0.09911763
##   0.03070175  0.7913063  0.5495555  0.05164865   0.11283711
##   0.44444444  0.6812388  0.2272669  0.07289580   0.24558983
## 
## Accuracy was used to select the optimal model using  the
##  largest value.
## The final value used for the model was cp = 0.00877193.
{% endhighlight %}



{% highlight r linenos %}
sex.class.fare.age.solution <- get.solution(sex.class.fare.age.tree, test.data.impute)
write.csv(sex.class.fare.age.solution, file= "output/new versions/sex_class_fare_age_cv_5.csv" , row.names= FALSE)
{% endhighlight %}

This time we get slightly lower training accuracy than the model without `Fare`. Our Kaggle score has improved to 0.7799 even with a smaller tree of depth 5 as compared to a depth of 8 we had before.

It should be noted that the best score we have had upto this point is for the model using `Sex`, `Pclass`, and `Fare`. My guess is that it is because of inherent errors in imputing missing values for `Age`.

I look forward to writing the next parts of this series.
