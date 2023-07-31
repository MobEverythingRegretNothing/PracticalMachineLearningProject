# Project Writeup


### Data Exploration

```r
training <- read.csv("pml-training.csv") # Read in the training data
testing <- read.csv("pml-testing.csv") # Read in the testing data

# Create validation set for x-validation
inTrain <- createDataPartition(y=training$classe, p=0.7, list=FALSE)
training <- training[inTrain,]
validation <- training[-inTrain,]

dim(training)

[1] 19622  160  # Training data has ~ 20,000 entries, 160 factors

# NOTE 'classe' is the outcome variable in this data set (represents grade), convert to factor
training$classe <- as.factor(training$classe)

# NOTE: Get rid of predictors who have significant portion NA
traininSubset <- subset(training, select=colMeans(is.na(x)) == 0)
```

Doing some analysis on the data looks like a lot of it is movement data related to the x, y, z dimensions, and much of the 
difference in grading comes being within a particular region and the further away from that region you go the worse the grade. Looking at 
the various factors it seems like we can reduce the number of factors based on how similar they are. There are also a fair amount of predictors missing data, 
meaning we will have to be careful which predictors to choose, and make sure to exclude from the model any such predictors

_Predictor Analysis_:
* Gyros_Forearm measurements are clearly defined in clusters and could be used as a good predictor. Also no missing data.
* Acceleration data seem to have no determination on classe
* Pitch/Yaw/Roll data tends to be missing and does not seem to by itself have a ton of correlation on classe

### Plan

Due to some data analysis, I decided to train my model after selecting out predictors that do not have a lot of missing data as well as predictors that I felt give a good
differential in the data. After selecting out predictors with a significant portion of NA answers using the VIM package, I could then run models on the dataset.

I decided to to use two models I was well-versed in with cross validation and then 

### Fitting Models

```r
# Fit a Random Forest Model with Cross Validation
modFitRf <- train(classe ~., method="rf", data=trainingSubset, trControl=trainControl(method="cv"), number=3)

# Fit a Boosting model
modFitGbm <- train(classe ~., method="gbm", data=trainingSubset, verbose=FALSE)

# Combine the models together on validation set (to help with bias/variance)
predRf <- predict(modFitRf, validation); predGbm <- predict(modFitGbm, validation);

# Run on Testing set
predDF <- data.frame(pred1, pred2, classe=validation$classe)

comboModFit <- train(classe~., method="gam", data=predDF)
comboPred <- (comboModFit, predDF)
```

## Results & Observations

Using a Random Forest/GBM by themselves, even with x-validation was a pretty bad fit, with only ~60% accuracy on the testing set for Random Forest and only slightly better for boosting. Combining them significantly helped my accuracy, although the computation time was heavy. Given more time & more familiarity with the data set I would have liked to do some more automated analysis on predictors in order to trim down the predictor space because removing the NAs only got me so far and I believe some of the predictors didn't contribute much to the overall accuracy and may have biased the accuracy to the testing set heavily.

