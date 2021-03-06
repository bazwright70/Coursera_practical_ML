# Practical Machine Learning
Brian Wright  
5 June 2017  



## Question  
  The question we are trying to answer is, can we predict the type of exercise a person is carrying out based on data from positional sensors.
  
    
## Data  
The data we will use comes from sensors placed on a subjects body and exercise equipment. 
 We will download 2 datasets, a training and test set. The training set will be further split into training and a second test set for building our model. The downloaded test set will be used for validation of our model.
 After examining the dataframe summary from the  raw csv file we can see missing values denoted by character strings "NA","" and "#DIV/0!" which we need to set as NA values when importing the file. We will import the file with stringsAsFactors = FALSE as the predictors are mainly numerical represented as character strings. We will then convert the character outome variable into a factor.
 
 
## Exploration
 The variable of interest is a categorical variable with 5 possible levels and the predictor vaiables are positive and negative numeric values.  
 If we look at the split of outcomes accross the 5 classes we can see they are fairly balanced.  
 

```
## classe
##    A    B    C    D    E 
## 5580 3797 3422 3216 3607
```


 When we look at the dataframe summary it shows a large number of NA values in many of the features. One particular set of features which contain all NA values are logicals so we will remove these features.  It is important that we understand what the remaining NA values represent in our data. As the numerical values in the dataframe represent numerical sensor output, it is possible that an NA value represents a lack of output or output below a noise threshold. We will look at the number of NA values across the 5 classes to to see if the are biased towards any particular classe. Although tree models can be built by internally handling NA values, we will need to impute values in place of NA's to allow us to look for possible colinearity and linear combinations.  
 

classe      NA's    obs   NA per obs
-------  -------  -----  -----------
A         514614   5580     92.22473
B         349662   3797     92.08902
C         315265   3422     92.12887
D         295984   3216     92.03483
E         331845   3607     92.00028


 The table shows all classes have an even proportion of NA values in the observations. We can begin building our models using the theory that NA's relate to lack of sensor output and therefore a directional output of zero we won't bias any of the classes unfairly.  
 
 
## Dataset Transformation 
 We will first transform out dataset to reduce those features that don't add information to our model. We will first remove features such as indexes, names, times. This will reduce the features by 7. Once we have imputed our NA values with zeros we be able to build a correlation matrix  colinearity, near zero variance and linear combinations. These add to the variance in the model but add no information. If we preprocess with Caret's findCorrelation function on our correlation matrix from the dataframe predictors we find 26 features with a correlation of 0.90 or more. The nzv function will find features with zero variance or near to zero variance which can then be removed. 
 The predictors are numerical with positive and negative values and are on different scales. We could use preprocessing functions to scale, center and transform the predictors but we will use Tree based models which are robust against outliers and predictor scale, unlike regression based models.   
 

    
   
## Model Building  
 We will attempt find the most accurate model which which can reliably predict on the test set with least complexity. We will do this using the Caret package. The 3 models we will use will be: 
   
   
 * Single decision tree  - ctree
 * Bagged decision tree  - treebag
 * Random Forest         - rf 
   
We will use 10 fold cross validation and caret's default tuning values for all 3 models. We can then attempt to improve our models using different tuning parameters if nescessary.  
 To reduce model processing time we will make use of Caret's
parallel processing abilites by using the doParallel package. This will allow model processing using all 4 processor cores (Intel core i5) which reduces model building time.  
 Although we have a separate testing set, we will use this later for model validation. We will split our training data frame into a train and test set.  
  

 


Single tree with 'ctree'.  



```
## Aggregating results
## Selecting tuning parameters
## Fitting mincriterion = 0.5 on full training set
```

```
##  Accuracy 
## 0.8900595
```
  
Bagged Decision Tree with 'treebag'.
  

```
## Aggregating results
## Fitting final model on full training set
```

```
##  Accuracy 
## 0.9899745
```
Random Forest with 'randomforest'.  

  

```
## Aggregating results
## Selecting tuning parameters
## Fitting mtry = 23 on full training set
```

```
##  Accuracy 
## 0.9950722
```
## Cross-Validaion   
We will look at the Cross Validation accuracy for the random forest model for. The plot shows the accuracy for various values of the tuning parameter 'mtry' which determines how many random predictors each of the individual trees will use to build the model.  

![](index_files/figure-html/plot first rf model-1.png)<!-- -->

The default settings for caret used three different values for mtry, 2,23 and 46. A recommended value for mtry for a classification model is the rounded squareroot of the number of predictors. This would be an mtry of 6. We will run the Random Forest algoritm again with 7 mtry values ranging from 2 to 23 and including our recommended value of 6. We will again plot the cross validation errors to see how they compare to the default Caret values.  

  
![](index_files/figure-html/tune rf model-1.png)<!-- -->
Using cross validation, the model was chosen using which resulted in the best accuracy using the different values of mtry from the tuning grid. An mtry value of 12 predictors gives the smallest cross validation error.

```
## Accuracy 
## 0.995582
```

  
## Test set Predictions  
Now we have a model which looks to have a good cross validation accuracy we can can test it againt the pml-testing dataset which we left out for model validation.  


```
##    Case Prediction
## 1     1          B
## 2     2          A
## 3     3          B
## 4     4          A
## 5     5          A
## 6     6          E
## 7     7          D
## 8     8          B
## 9     9          A
## 10   10          A
## 11   11          B
## 12   12          C
## 13   13          B
## 14   14          A
## 15   15          E
## 16   16          E
## 17   17          A
## 18   18          B
## 19   19          B
## 20   20          B
```



 

