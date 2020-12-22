---
title: "Practical Machine Learning Project"
author: "Amanda Hughes"
date: "12/21/2020"
output: 
  html_document: 
    keep_md: yes
---

## Background

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).



```r
## loading R packages and data

library(caret)
```

```
## Loading required package: lattice
```

```
## Loading required package: ggplot2
```

```r
URLTrain <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"

URLTest <-  "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"

training <- read.csv(url(URLTrain))
testing  <- read.csv(url(URLTest))

## Making a new data frame with mostly NAs columns taken out
training[training == ""] <- NA
newtraining <- training[, colSums(is.na(training)) < nrow(training) * 0.95]

## Deselect the ID columns
newtraining <- newtraining[,-c(1:5)]

## Partition new training set into a training and testing sets for cross validation
set.seed(4356)
inTrain  <- createDataPartition(newtraining$classe, p=0.7, list=FALSE)
Train <- newtraining[inTrain, ]
Test  <- newtraining[-inTrain, ]
```

Data frame clean up: If columns had more that 95% NAs, they were taken out of the data frame.  The first five columns were also taken out since they were columns used to ID the participant of the study and not needed for our model.  

The training set was partitioned into a training and testing set so we could cross valid after a model has been completed.  

## Model Fit

A model was build using random forest method.  


```r
# model fit
set.seed(3453)
control <- trainControl(method="cv")
modFitRF <- train(classe ~ ., data=Train, method="rf",
                  trControl=control)
print(modFitRF)
```

```
## Random Forest 
## 
## 13737 samples
##    54 predictor
##     5 classes: 'A', 'B', 'C', 'D', 'E' 
## 
## No pre-processing
## Resampling: Cross-Validated (10 fold) 
## Summary of sample sizes: 12364, 12363, 12364, 12364, 12363, 12362, ... 
## Resampling results across tuning parameters:
## 
##   mtry  Accuracy   Kappa    
##    2    0.9952684  0.9940144
##   28    0.9969428  0.9961330
##   54    0.9945397  0.9930932
## 
## Accuracy was used to select the optimal model using the largest value.
## The final value used for the model was mtry = 28.
```

```r
modFitRF$finalModel
```

```
## 
## Call:
##  randomForest(x = x, y = y, mtry = param$mtry) 
##                Type of random forest: classification
##                      Number of trees: 500
## No. of variables tried at each split: 28
## 
##         OOB estimate of  error rate: 0.27%
## Confusion matrix:
##      A    B    C    D    E  class.error
## A 3903    2    0    0    1 0.0007680492
## B    5 2647    6    0    0 0.0041384500
## C    0    5 2391    0    0 0.0020868114
## D    0    0   11 2240    1 0.0053285968
## E    0    0    0    6 2519 0.0023762376
```

Accuracy and Kappa were both ~ 99%.  The final model shows an estimate of error rate of 0.25%, and very low class.errors for each class.  This seems to be a good model fit for the training data set.  

## Predicting using the Test partition set

The model was used to predict the test set.


```r
predmodFit <- predict(modFitRF,Test)
table(Test$classe,predmodFit)
```

```
##    predmodFit
##        A    B    C    D    E
##   A 1674    0    0    0    0
##   B    3 1133    3    0    0
##   C    0    0 1026    0    0
##   D    0    0    6  958    0
##   E    0    0    0    3 1079
```

The table shows the actual vs predicted classification for the test set using the random forest model.

## Using the model to predict the testing set.


```r
predTest <- predict(modFitRF,testing)
print(predTest)
```

```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```
