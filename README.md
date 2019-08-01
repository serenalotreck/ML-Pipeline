# ML-Pipeline
Shiu Lab Machine Learning Pipeline


**Please see our [workshop material](https://github.com/azodichr/ML-Pipeline/blob/master/Workshop/ML_workshop.ipynb) for an introduction to the ML_Pipeline workflow and an example of how this pipeline can be used to ask questions.**


<img src="Tutorial/pipeline_summary.png" alt="ML-Pipeline Overview" width="300"/>

## Environment Requirements
* biopython                 1.68
* matplotlib                1.5.1
* numpy                     1.11.3
* pandas                    0.18.1
* python                    3.4.4
* scikit-learn              0.18.1
* scipy                     0.18.1

Example: 

    wget http://repo.continuum.io/miniconda/Miniconda3-3.7.0-Linux-x86_64.sh -O ~/miniconda.sh
    bash ~/miniconda.sh -b -p $HOME/miniconda
    export PATH="$HOME/miniconda/bin:$PATH"
    conda install biopython
    conda install matplotlib
    conda install pandas
    conda install scikit-learn
    

## Basic ML Pipeline

Code provided to:

1. Clean your data (ML_preprocess.py)
2. Define a testing set (test_set.py)
3. Select the best subset of features to use as predictors (Feature_Selection.py)
4. Train and apply a classification (ML_classification.py) or regression (ML_regression.py) machine learning model
5. Assess the results of your model (output from the ML_classification/ML_regression scripts with additional options in scripts_PostAnalysis


## Example workflow (see Workshop or Tutorial ipython notebook for more details!)

### 0. Format data

Your data should look something like this:

| ID   | Class    | Feature_1   | Feature_2   | Feature_n   |
|---    |---    |---    |---    |---    |
| instanceA    |  1     | sunny     | 98     | 1     |
| instanceB    |  0    |  overcast     | 87     | 0     |
| instanceC   |  0     |  rain    | n/a     | 1     |
| instanceD   | 1     |  sunny    | 73     | 1     |
| instanceE   | unknown     |  overcast    | 75     | 0     |

Where the first column contains the instance IDs, one column contains the value you want to predict with name (default = 'Class', specify your own column name using -y_name), and the remaining columns contain the predictive features (i.e. independent variables).

If you want to classify instances by class (in the above example 1 vs. 0), use ML_classification.py. You can specify what classes you want to include in your model using -cl_train. This is useful if your dataset also contains instances with other values in the Class column, such as "unknown", that you want to apply your trained model to. If you want to predict the value in the y_name column, use ML_regression.py. 


### 1. Clean your data

```
python ML_preprocess.py -df data.txt -na_method median -onehot t -
```

### 2. Define a testing set (test_set.py)

```
python test_set.py -df data_mod.txt -use 1,0 -type c -p 0.1 -save test_instances.txt
```

### 3. Select the best subset of features to use as predictors (Feature_Selection.py)

```
python Feature_Selection.py -df data_mod.txt -cl_train 1,0 -type c -alg lasso -p 0.01 -save top_feat_lasso.txt
```

### 4. Train and apply a classification (ML_classification.py) or regression (ML_regression.py) machine learning model

Example using the data shown above:
```
python ML_classification.py -df data_mod.txt -test test_instances.txt -cl_train 1,0 -alg SVM -apply unknown
```

Example of a multiclass prediction where classes are A, B, and C:
```
python ML_classification.py -df data_mod.txt -test test_instances.txt -cl_train A,B,C -alg SVM -apply unknown
```

Example of a regression prediction (e.g. predicting plant height in meters):
```
python ML_regression.py -df data_mod.txt -test test_instances.txt -y_name height -alg SVM -apply unknown
```

**For more options, run either ML_classification.py or ML_regression.py with no parameters or with -h**

### 5. Assess the results of your model (output from the ML_classification/ML_regression scripts with additional options in scripts_PostAnalysis

**See scripts_PostAnalysis/README.md for more information on additional post-modeling analysis and figure making.**

The following files are generated by default:

- **data.txt_results:** A summary of the model generated (e.g. what algorithm/parameters were used, number of instances/features, etc.) and the results from applying that model during validation and on the test set. These results are similar to what is printed on the commond line, but more performance metrics are provided, including performance metrics specific to your type of model (i.e. binary classification, multiclass, regression). For example, for binary and multiclass classification models you will see two additional sections: the Mean Balanced Confusion Matrix (CM) and the Final Full CM. The mean balanced CM is generated by taking the average number of true positives (TP), true negatives (TN), false positives (FP), and false negatives (FN) across all replicate (which have been downsampled randomly to be balanced). The Final Full CM represents the final TP, TN, FP, and FN results from the final pos/neg classifications (descirbed in data.txt_scores) for all instances in your input dataset. 

- **data.txt_scores:** This file includes the true value/class for each instance and the predicted value/class & predicted probability (pp) for each instance for each replicate of the model (-n). The pp score represents how confident the model was in its classification, where a pp=1 means it is certain the instance is positive and pp=0 means it is certain the instance is negative. For multiclass models, the class with the greatest pp is selected as the predicted class. For binary models, for each replicate, an instance is classified as pos if pp > threshold, which is defined as value between 0.001-0.999 that maximises the F-measure. While the performance metrics generated by the pipeline are calcuated for each replicate independently, we want to be able to make a final statement about which instances were called as positive and which were called as negative. You'll find those results in this file. To make this final call we calculated the mean threshold and the mean pp for each instance and called the instance pos if the mean pp > mean threshold. 

- **data.txt_imp:** the importance of each feature in your model. For RF and GTB this score represents the [Gini Index](https://medium.com/the-artificial-impostor/feature-importance-measures-for-tree-models-part-i-47f187c1a2c3), while for LogReg and SVM it is the [coefficient](https://medium.com/@aneesha/visualising-top-features-in-linear-svm-with-scikit-learn-and-matplotlib-3454ab18a14d). SVM with non-linear kernels (i.e. poly, rbf) does not report importance scores.

- **data.txt_GridSearch:** the average model performance across the whole parameter space tested via the grid search (i.e. every possible combination of parameters). 

- **data.txt_BalancedID:** Not generated for ML_regression.py models. Each row lists the instances that were included in each replicate (-n) after downsampling. 

Additional Notes for Multiclass models:

- *An important note: For binary classification using balanced datasets, you would expect a ML model that was just randomly guessing the class to be correct ~50% of the time, because of this the random expectation for performance metrics like AUC-ROC and the F-measure are 0.50. This is not the case for multi-class predictions. Using our model above as an example, a ML model that was randomly guessing top, middle, or bottom, would only be correct ~33% of the time. That means models performing with >33% accuracy are performing better than random expectation.*

- *There are two types of performance metrics for multi-class models, commonly referred to as macro and micro metrics. Micro performance metrics are generated for each class in your ML problem. For example, from our model we will get three micro F-measures (F1-top, F1-middle, F1-bottom). These micro scores are available in the *_results* output file. Macro performance metrics are generated by taking the average of all the micro performance metrics. These scores are what are printed in the command line.*




## TO DO LIST
- Add additional classification models: Naive Bayes, basic neural network (1-2 layers)
- Look into using MCC as a performance metric - would be useful for selecting the threshold since it doesn't depend on the ratio of +/- instances (https://en.wikipedia.org/wiki/Matthews_correlation_coefficient)
- Incorporate PCA summary features into pre-processing script

