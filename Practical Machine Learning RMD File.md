Practical Machine Learning Project Markdown
========================================================

This is a R Markdown document explaining the modalities involved with my solution for the mandatory project assignment of "Practical Machine Learning" course on coursera.org.

To start with, I studied the training dataset "pml-training" and test dataset "pml-testing" provided as part of the assignment. 

The first observation in training set is that there were quiet a number of cells with blank values therefore while reading the file for processing I ensured to read all blanks to be replaced with NAs so as to ease the processing.


```r
training<-read.csv("pml-training.csv",na.strings=c("", "NA"),stringsAsFactors=FALSE)
```

An attempt was made to read all factors as character vectors so the training of the model was smooth. When I tried with direct factor variables the training was never ending so this explicit decision.

Manual verification reveled that there are a number of predictors with full of NAs therefore I felt a need to remove these columns for the training data set.


```r
training<-training[,!apply(is.na(training), 2, any)]
```
At each stage the reduction in the number of predictors was observed by printing it on the R Studio console using :

```r
print(dim(training))
```

```
## [1] 19622    56
```
Near zero varience was calculated to eliminate predictors of no use to model building and it was done by :

```r
library(MASS)
```

```
## Warning: package 'MASS' was built under R version 3.0.3
```

```r
library(caret)
```

```
## Warning: package 'caret' was built under R version 3.0.3
```

```
## Loading required package: lattice
```

```
## Warning: package 'lattice' was built under R version 3.0.3
```

```
## Loading required package: ggplot2
```

```
## Warning: package 'ggplot2' was built under R version 3.0.3
```

```r
nzv <- nearZeroVar(training)
training <- training[, -nzv]
```
Inorder to perform cross validation, I choose to split the training dataset into training and cross validation datasets in the ratio of 75 - 25% of the training dataset and the task was accomplished using the commands below :

```r
smp_size <- floor(0.75 * nrow(training))

trainindex <- sample(seq_len(nrow(training)), size = smp_size)
trainset <- training[trainindex, ]
testset <- training[-trainindex, ]
```
Principal component analysis was run on the training set and the test data sets so as to reduce the transform the number of predictors. Varience thresholds of 90%,95% and 98% were tried on PCA.Sample code showing 95% varience threshold is below :

```r
preproc <- preProcess(trainset[,-55]+1,method="pca",thresh=0.95)
trainPC <- predict(preproc,trainset[,-55]+1)
testPC <- predict(preproc,testset[,-55]+1)
```
classe variable is converted as a factor to avoid errors with randomForest model that I intend to build. It is accomplished as below :

```r
trainset$classe <- as.factor(trainset$classe)
```
model was built using PCA applied training dataset as below :

```r
library(randomForest)
```

```
## Warning: package 'randomForest' was built under R version 3.0.3
```

```
## randomForest 4.6-10
## Type rfNews() to see new features/changes/bug fixes.
```

```r
model<-randomForest(formula = trainset$classe ~ ., data = trainPC)
```
predictions were made and confusion matrix was printed throgh the code :

```r
print(confusionMatrix(testset$classe,predict(model,testPC)))
```

```
## Warning: package 'e1071' was built under R version 3.0.3
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1388    0    1    0    3
##          B    6  992    6    1    0
##          C    0    3  857    0    0
##          D    0    0   23  767    1
##          E    0    0    1    0  857
## 
## Overall Statistics
##                                         
##                Accuracy : 0.991         
##                  95% CI : (0.988, 0.993)
##     No Information Rate : 0.284         
##     P-Value [Acc > NIR] : <2e-16        
##                                         
##                   Kappa : 0.988         
##  Mcnemar's Test P-Value : NA            
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity             0.996    0.997    0.965    0.999    0.995
## Specificity             0.999    0.997    0.999    0.994    1.000
## Pos Pred Value          0.997    0.987    0.997    0.970    0.999
## Neg Pred Value          0.998    0.999    0.992    1.000    0.999
## Prevalence              0.284    0.203    0.181    0.157    0.175
## Detection Rate          0.283    0.202    0.175    0.156    0.175
## Detection Prevalence    0.284    0.205    0.175    0.161    0.175
## Balanced Accuracy       0.997    0.997    0.982    0.996    0.998
```
As discussed above PCA was tried with various varience thresholds. From confusion matrices,99% accuracy was achieved on cross validation dataset with 90% and 95% PCA varience and a 100% accuracy is achieved with 98% varience threshold on PCA.

To avoid overfitting of trainingdata , I decided to use a 90% varience threshold on PCA.

Similar approach as above was followed for predictions on test data given in the assignment however I used the entire training data to train the model.The code is give below -


```r
training<-read.csv("pml-training.csv",na.strings=c("", "NA"),stringsAsFactors=FALSE)
training<-training[,!apply(is.na(training), 2, any)]
testing<-read.csv("pml-testing.csv",na.strings=c("", "NA"),stringsAsFactors=FALSE)
testing<-testing[,!apply(is.na(testing), 2, any)]
library(MASS)
library(caret)
library(randomForest)
nzv <- nearZeroVar(training)
training <- training[, -nzv]
nzv <- nearZeroVar(testing)
testing <- testing[, -nzv]
preproc <- preProcess(training[,-55]+1,method="pca",thresh=0.90)
trainPC <- predict(preproc,training[,-55]+1)
training$classe <- as.factor(training$classe)
model<-randomForest(formula = training$classe ~ ., data = trainPC)
testPC <- predict(preproc,testing)
print(predict(model,testPC))
```

```
##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
##  E  A  A  A  E  E  E  B  A  A  B  B  D  A  B  A  B  D  B  B 
## Levels: A B C D E
```
Unfortunately , the code did not predict 100% right predictions on test dataset. The accuracy appears to be less than 80%. This is despite the fact that I got a 99+% accuracy on cross validation dataset! The results indicate more work to be done on preprocessing side of this problem! 

