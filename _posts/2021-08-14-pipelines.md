---
layout: post
title: Machine Learning Workflow with Pipelines
subtitle: Pipeline Creation
cover-img: /assets/img/Pipelines/pipeline_stock_photo.jpeg
thumbnail-img: /assets/img/Pipelines/pipeline_thumbnail.png
share-img: /assets/img/Pipelines/pipeline_thumbnail.png
tags: [Pipeline, Transform, ColumnTransformer, FeatureUnion]
readtime: True
---

# What is a Machine Learning Pipeline?
A Machine Learning Pipeline is a way to automate the workflow it takes to produce a machine learning model. It consists a sequential steps chained together that to perform various tasks such as data ingestion, data cleaning, feature engineering, model training and deployment.

# Why Use A Pipeline?
A Machine Learning workflow consists of a series of steps.
- Data ingestion
- Data cleaning
- Data preprocessing / feature engineering
- Modeling
- Deployment


A Pipeline helps to break up each of the process by splitting up each section of the project into individual components that can be executed by the user. Therefore having a pipeline ensures order during the transformation process, makes it compact, easy to understand and reproducible.

# Pipeline Creation

Creating a Pipeline is done using the Pipeline object `sklearn.pipeline.Pipeline`

# Feature Transformations
In most datasets, there will be numerical and categorical data in the same DataFrame. Machine Learning Algorithms accept numerical values instead of strings and we would need to convert these categorical or string values before passing them into the Algorithm. In order to do this, we use transform these features using preprocessing methods such as `LabelEncoder()` and `OneHotEncoder()` or to create new features in the dataset.


## Transforming DataFrames with Different Transformations
Depending on the data, we may not want to implement the same transformation on all of the columns.

For example, for a dataset with the following columns of *Income* and *Height*. Income data tends to be skewed, where the median is chosen to fill missing values, while height is normally distributed and mean is chosen to fill in missing values. Another set of data could contain numerical and categorical data and they would need to be processed separately.

The typical preprocessing steps apply to the entire DataFrame instead of a specific column. (Example below). Another downside is that we are not able to build in feature engineering steps in the pipeline shown below.

```python
pipeline = Pipeline(
  steps = [
    ('imp', SimpleImputer()),
    ('ohe', OneHotEncoder())
    ])
transformed_data = pipeline.fit_transform(df)
```

In order to perform different transformations on individual columns, there are a few ways to do this.

### <u>Method 1</u>
1. Select first set of columns in a DataFrame to perform first transformation
2. Select second set of columns in a DataFrame to perform second transformation
3. Combine them laterally using `sklearn.pipeline.FeatureUnion`

<u>Example</u>

1) Create a class to select the features of interest in the DataFrame
```python
from sklearn.base import BaseEstimator, TransformerMixin

# Create a class to select the features of the DataFrame
class FeatureSelector(BaseEstimator, TransformerMixin):
  def __init__(self, feature_names):
    self._feature_names = feature_names

  def fit(self, X, y = None):
    return self

  def transform(self, X, y = None):
    return X[self._feature_names]

```
2) Create individual transformers to process the selected dataset
```python
# Create class to perform numerical transformations
class NumericalTransformer(BaseEstimator, TransformerMixin):
  def __init__(self):
    pass

  def fit(self, X, y = None):
    return self

  def transform(self, X, y = None):
    # Perform your transformations or feature engineering

    return X  
```

```python
class CategoricalTransformer(BaseEstimator, TransformerMixin):
  def __init__(self):
    pass

  def fit(self, X, y = None):
    return self

  def transform(self, X, y = None):
    # Perform transformations or feature engineering

    return X

```
3) Putting it together into a pipeline, if we desire, we can chain the existing transformers such as `SimpleImputer()` and `OneHotEncoder()` into the pipeline
```python
# Create column of numerical features in the DataFrame
numerical_cols = ['income', 'height']
categorical_cols = ['gender']

# Create a pipeline to process numerical features
# We can chain built in transformations such as SimpleImputer in the pipeline.
numerical_pipeline = Pipeline(
  steps = [
    ('num_selector', FeatureSelector(numerical_cols)),
    ('num_transformer', NumericalTransformer()),
    ('imputer', SimpleImputer()),
    ('std_scaler', StandardScaler())
    ])

# Create a pipeline to process categorical features
categorical_pipeline = Pipeline(
  steps = [
    ('cat_selector', FeatureSelector(categorical_cols)),
    ('cat_transformer', CategoricalTransformer())
    ('ohe', OneHotEncoder())
    ])

# Combine the pipelines
full_pipeline = FeatureUnion(
  transformer_list = [
    ('categorical_pipeline', categorical_pipeline),
    ('numerical_pipeline', numerical_pipeline)
    ])

# Call .fit_transform on the dataframe
transformed_data = full_pipeline.fit_transform(df)
```

### <u>Method 2</u>
1. Declare a pipeline using `sklearn.compose.ColumnTransformer`
2. Add in transformations in the pipeline and select the columns for each transformation

<u>Example</u>

1) Create a function that returns a DataFrame. As the returned value will be the input of the next transformation step, we will return the DataFrame to be processed by the next transformer.
```python
def extract_year(dataframe):
  # Write function to extract the get_year

  return dataframe

```

2) Create FunctionTransformer object
```python
get_year = FunctionTransformer(get_year, validate = False)
```

3) Create ColumnTransformer with the new FunctionTransformer in the Pipeline

```python
from sklearn.compose import ColumnTransformer

ct = ColumnTransformer(
  [
  ('get_year', get_year, ['year']),
  ('imp', SimpleImputer(), ['income', 'height']),
  ('ohe', OneHotEncoder(), ['gender'])
  ])

```

*Note: Do take note that if all of the object (string) variables are not encoded during the transformation, it will return an array of strings which will give an error when you pass it to your Machine Learning Algorithm*


# Conclusion

There are different ways of creating pipelines and two ways that can be done are shown here. Pipeline creation is an important step in making sure that the transformation step is ordered, compact, easy to understand and reproducible which is crucial.
