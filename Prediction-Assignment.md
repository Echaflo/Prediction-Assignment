---
html_document:
  keep_md: yes
author: "echf"
date: "8/9/2021"
title: "Prediction Assignment"
---

```{r setup, include=TRUE}
knitr::opts_chunk$set(echo = TRUE)
###knitr::opts_chunk$set(out.width="400px", dpi=120)
options(width=80)
```

### Synopsis

##### Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).

### Data

##### The training data for this project are available here:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv

##### The test data are available here:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv

##### The data for this project come from this source: http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har. 


### LIBRARY'S

```{r}
options(warn=-1)
library(caret)
library(randomForest)
library(Hmisc)
library(foreach)
library(doParallel)
library(rattle)
library(rpart)
library(rpart.plot)
library(RColorBrewer)
library(gbm)
library(plyr)
set.seed(21243)
```


### DATA LOAD

```{r}

train_url <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
test_url  <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"


training_data <- read.csv(url(train_url), na.strings = c("NA", "#DIV/0!", ""))
testing_data <- read.csv(url(test_url), na.strings = c("NA", "#DIV/0!", ""))

dim(training_data)
dim(testing_data)
```


### Data Cleaning train_url
```{r}
# Removing Variables which are having nearly zero variance

cData <- training_data
for(i in c(8:ncol(cData)-1)) {cData[,i] = as.numeric(as.character(cData[,i]))}

# First look at the data for each column and remove variables unrelated
# to exercise (column number and time stamps)
featuresnames <- colnames(cData[colSums(is.na(cData)) == 0])[-(1:7)]
features <- cData[featuresnames]


```

### Data Partitioning 

```{r}
#### in this course it is recommended to make a partition of 60%-40% 
# but in other documents it is 70%-30%. so what was seen in this
#course will be used. 

inTrain <- createDataPartition(features$classe, p=0.60, list=FALSE)
training <- features[inTrain,]
testing <- features[-inTrain,]

dim(training)
dim(testing)

```


### Decision Tree Model and Prediction
```{r fig.align="center" , include=TRUE}
DT_model<- train(classe ~. , data=training, method= "rpart")
fancyRpartPlot(DT_model$finalModel)

DT_prediction<- predict(DT_model, newdata=testing)
identical(DT_prediction , testing$classe)
confusionMatrix(table(DT_prediction, testing$classe)) ##


```

### GBM model and Prediction
```{r fig.align="center" , include=TRUE}
gbm_model<- train(classe ~. , data=training, method= "gbm", 
                  trControl= trainControl(method="repeatedcv",
                                          number = 5,repeats = 1), 
                  verbose=FALSE)


gbm_prediction<- predict(gbm_model, newdata=testing)
identical(gbm_prediction , testing$classe)
confusionMatrix(table(gbm_prediction, testing$classe)) ##

plot(gbm_model)
```


###  Random Forest Model and Prediction

```{r}
RForest <- train(classe~., data=training, method="rf", 
                 trControl=trainControl(method="cv", classProbs=TRUE,
                                         savePredictions=TRUE,
                                         allowParallel=TRUE, number = 10))

predRF<-predict(RForest, testing)
CMatrixRF <- confusionMatrix(table(predRF, testing$classe))
CMatrixRF

```


### Data Cleaning testing
```{r}
### Removing Variables which are having nearly zero variance

cData_t <- testing_data
for(i in c(8:ncol(cData_t)-1)) {cData_t[,i] = 
                               as.numeric(as.character(cData_t[,i]))}

# First look at the data for each column and remove variables 
# unrelated to exercise (column number and time stamps)
featuresnames <- colnames(cData_t[colSums(is.na(cData_t)) == 0])[-(1:7)]
features_t <- cData_t[featuresnames]


```
### Prediction on testing dataset

```{r}
##prediction on Test dataset
predict_Test <- predict(RForest, newdata=features_t)
predict_Test
print(as.data.frame(predict_Test))
```

### Conclusion
#### As we can see from the three methods used, each one is improving in its precision but it also has to take into account the time it takes in each of its internal calculations. therefore we stick with rainforest for 99% accuracy. 



#####  --------



