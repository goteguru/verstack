##################
verstack 0.5.1 Documentation
##################
Machine learning tools to make a Data Scientist's work efficient

veratack package contains the following tools:

- **NaNImputer** impute all missing values in a pandas dataframe using advanced machine learning with 1 line of code
- **Multicore** execute any function in concurrency using all the available cpu cores
- **ThreshTuner** tune threshold for binary classification predictions
- **stratified_continuous_split** create train/test splits stratified on the continuous variable
- **timer** convenient timer decorator to quickly measure and display time of any function execution


.. note:: 

  Getting verstack

  $ ``pip install verstack``

  $ ``pip install --upgrade verstack``


******************
NaNImputer
******************

Impute all missing values in a pandas dataframe by xgboost models in multiprocessing mode using a single line of code.

Logic
================================================================

With NaNImputer you can fill missing values in numeric, binary and categoric columns in your pandas dataframe using advanced XGBRegressor/XGBClassifier models with just 1 line of code. Regardless of the data types in your dataframe (string/bool/numeric): 

 - all of the columns will be checked for missing values
 - transformed into numeric formats
 - split into subsets with and without missing values
 - applicalbe models will be selected and configured for each of the columns with NaNs
 - models will be trained in multiprocessing mode utilizing all the available cores and threads of your cpu (this saves a lot of time)
 - NaNs will be predicted and placed into corresponding indixes
 - columns with all NaNs will be droped
 - columns containing NaNs and known values as a single constant
 - data will be reverse-transformed into original format

The module is highly configurable with default argumets set for the highest performance and verbosity

The only limitation is:

- NaNs in pure text columns are not imputed. By default they are filled with 'Missing_data' value. Configurable. If disabled - will return these columns with missing values untouched

Initialize NaNImputer
===========================
.. code-block:: python

  from verstack import NaNImputer
  
  # initialize with default parameters
  imputer = NaNImputer()
  
  # initialize with selected parameters
  imputer = NaNImputer(conservative = False, 
                       n_feats = 10, 
                       nan_cols = None, 
                       fix_string_nans = True, 
                       multiprocessing_load = 3, 
                       verbose = True, 
                       fill_nans_in_pure_text = True, 
                       drop_empty_cols = True, 
                       drop_nan_cols_with_constant = True)

Parameters
===========================
* ``conservative`` [default=False]

  - Model complexity level used to impute missing values. If ``True``: model will be set to less complex and much faster.

* ``n_feats`` [default=10]

  - Number of corellated independent features to be used forcorresponding column (with NaN) model training and imputation.

* ``nan_cols`` [default=None]

  - List of columns to impute missing values in. If None: all the columns with missing values will be used.


* ``fix_string_nans`` [default=True]

  - Find possible missing values in numeric columns that had been (mistakenly) encoded as strings, E.g. 'Missing'/'NaN'/'No data' and replace them with np.nan for further imputation.

* ``multiprocessing_load`` [default=3]

  - Levels of parallel multiprocessing compute
    - 1 = single core
    - 2 = half of all available cores
    - 3 = all available cores

* ``verbose`` [default=True]

  - Print the imputation progress.

* ``fill_nans_in_pure_text`` [default=True]

  - Fill the missing values in text fields by string 'Missing_data'.Applicable for text fields (not categoric).

* ``drop_empty_cols`` [default=True]

  - Drop columns with all NaNs.

* ``drop_nan_cols_with_constant`` [default=True]

  - Drop columns containing NaNs and known values as a single constant.

* ``feature_selection`` [default="correlation"]
  - Define algorithm to select most important feats for each column imputation. Quick option: "correlation" is based on selecting n_feats with the highest binary correlation with each column for NaNs imputation. Less quick but more precise: "feature_importance" is based on extracting feature_importances from an xgboost model.

Methods
===========================
* ``impute(data)``

  - Execute NaNs imputation columnwise in a pd.DataFrame

    Parameters

    - ``data`` pd.DataFrame

      dataframe with missing values in a single/multiple columns

Examples
================================================================

Using NaNImputer with all default parameters

.. code-block:: python

  imputer = NaNImputer()
  df_imputed = imputer.impute(df)

Say you would like to impute missing values in a list of specific columns, use 20 most important features for each of these columns imputation and deploy a half of the available cpu cores

.. code-block:: python

  imputer = NaNImputer(nan_cols = ['col1', 'col2'], n_feats = 20, multiprocessing_load = 2)
  df_imputed = imputer.impute(df)

******************
Multicore
******************

Execute any function in concurrency using all the available cpu cores.

Logic
================================================================

  Multicore module is built on top of concurrent.futures package. Passed iterables are divided into chunks according to the number of workers and passed into separate processes.

  Results are extracted from finished processes and combined into a single/multiple output as per the defined function output requirements.

  Multiple outputs are returned as a nested list.

Initialize Multicore
===========================
.. code-block:: python

  from verstack import Multicore
  
  # initialize with default parameters
  multicore = Multicore()
  
  # initialize with selected parameters
  multicore = Multicore(workers = 6,
                        multiple_iterables = True)

Parameters
===========================
* ``workers`` int or bool [default=False]

  - Number of workers if passed by user. If ``False``: all available cpu cores will be used.

* ``multiple_iterables`` bool [default=False]

  - If function needs to iterate over multiple iterables, set to ``True``.
  Multiple iterables must be passed as a list (see examples below).

Methods
===========================
* ``execute(func, iterable)``

  - Execute passed function and iterable(s) in concurrency.

    Parameters

    - ``func`` function

      function to execute in parallel


    - ``iterable`` list/pd.Series/pd.DataFrame/dictionary

      data to iterate over


Examples
================================================================

Use Multicore with all default parameters

.. code-block:: python

  multicore = Multicore()
  result = multicore.execute(function, iterable_list)

If you want to use a limited number of cpu cores and need to iterate over two objects:

.. code-block:: python

  multicore = Multicore(workers = 2, multiple_iterables = True)
  result = multicore.execute(function, [iterable_dataframe, iterable_list])

******************
ThreshTuner
******************

Find the best threshold to split your predictions in a binary classification task. Most applicable for imbalance target cases. 
In addition to thresholds & loss_func scores, the predicted_ratio (predicted fraction of 1) will be calculated and saved for every threshold. This will help the identify the appropriate threshold not only based on the score, but also based on the resulting distribution of 0 and 1 in the predictions.

Logic
================================================================

  Default behavior (only pass the labels and predictions): 
   - Calculate the labels balance (fraction_of_1 in labels)
   - Define the min_threshold as fraction_of_1 * 0.8
   - Define the max_threshold as fraction_of_1 * 1.2 but not greater than 1
   - Define the n_thresholds = 200
   - Create 200 threshold options uniformly distributed between min_threshold & max_threshold
   - Deploy the balanced_accuracy_score as loss_func
   - Peform loss function calculation and save results in class instance placeholders

  Customization options
   - Change the n_thresholds to the desired value
   - Change the min_threshold & max_threshold to the desired values
   - Pass the loss_func of choice, e.g. sklearn.metrics.f1_score
  This will result in user defined granulation of thresholds to test

Initialize ThreshTuner
===========================
.. code-block:: python

  from verstack import ThreshTuner
  
  # initialize with default parameters
  thresh = ThreshTuner()
  
  # initialize with selected parameters
  thresh = ThreshTuner(n_thresholds = 500,
                       min_threshold = 0.3,
                       max_threshold = 0.7)

Parameters
===========================
* ``n_thresholds`` int [default=200]

  - Number of thresholds to test. If not set by user: 200 thresholds will be tested.

* ``min_threshold`` float or int [default=None]

  - Minimum threshold value. If not set by user: will be infered from labels balance based on fraction_of_1

* ``max_threshold`` float or int [default=None]

  - Maximum threshold value. If not set by user: will be infered from labels balance based on fraction_of_1

Methods
===========================
* ``fit(labels, pred, loss_func)``

  - Calculate loss_func results for labels & preds for the defined/default thresholds. Print the threshold(s) with the best loss_func scores

    Parameters

    - ``labels`` array/list/series [default=balanced_accuracy_score]

      y_true labels represented as 0 or 1


    - ``pred`` array/list/series

      predicted probabilities of 1


    - ``loss_func`` function

      loss function for scoring the predictions, e.g. sklearn.metrics.f1_score



* ``result()``

  - Display a dataframe with thresholds/loss_func_scores/fraction_of_1 for for all the the defined/default thresholds

* ``best_score()``

  - Display a dataframe with thresholds/loss_func_scores/fraction_of_1 for the best loss_func_score

* ``best_predict_ratio()``

  - Display a dataframe with thresholds/loss_func_scores/fraction_of_1 for the (predicted) fraction_of_1 which is closest to the (actual) labels_fraction_of_1 

Examples
================================================================

Use ThreshTuner with all default parameters

.. code-block:: python

  thresh = ThreshTuner()
  thres.fit(labels, pred)

Customized ThreshTuner application

.. code-block:: python

  from sklearn.metrics import f1_score
  
  thresh = ThreshTuner(n_thresholds = 500, min_threshold = 0.2, max_threshold = 0.6)
  thresh.fit(labels, pred, f1_score)

Access the results after .fit()

.. code-block:: python

  thresh = ThreshTuner()
  thres.fit(labels, pred)
  
  # return pd.DataFrame with all the results
  thresh.result
  # return pd.DataFrame with the best loss_func score
  thresh.best_score()
  thresh.best_score()['threshold']
  # return pd.DataFrame with the best predicted fraction_of_1
  thresh.best_predict_ratio()
  # return the actual labels fraction_of_1
  thresh.labels_fractio_of_1

******************
stratified_continuous_split
******************

Create stratified splits based on either continuous or categoric target variable.
  - For continuous target variable verstack uses binning and categoric split based on bins
  - For categoric target enhanced sklearn.model_selection.train_test_split is used: in case there are not enough categories for the split, the minority classes will be combined with nearest neighbors.

Can accept only pandas.DataFrame/pandas.Series as data input.

.. code-block:: python 

  verstack.stratified_continuous_split.scsplit(*args, 
                                               stratify, 
                                               test_size = 0.3, 
                                               train_size = 0.7, 
                                               continuous = True, 
                                               random_state = None)

Parameters
===========================
* ``X,y,data`` 

  - data input for the split in pandas.DataFrame/pandas.Series format.

* ``stratify`` 

  - target variable for the split in pandas/eries format.

* ``test_size`` [default=0.3]

  - test split ratio.

* ``train_size`` [default=0.7]

  - train split ratio.

* ``continuous`` [default=True]

  - stratification target definition. If True, verstack will perform the stratification on the continuous target variable, if False, sklearn.model_selection.train_test_split will be performed with verstack enhancements.

* ``random_state`` [default=5]

  - random state value.


Examples
================================================================

.. code-block:: python

  from verstack.stratified_continuous_split import scsplit
  
  train, test = scsplit(data, stratify = data['continuous_column_name'])
  X_train, X_val, y_train, y_val = scsplit(X, y, stratify = y, 
                                           test_size = 0.3, random_state = 5)

******************
timer
******************

Timer decorator to measure any function execution time and create elapsed time output: hours/minues/seconds will be calculated and returned conveniently.

.. code-block:: python 

  verstack.tools.timer


Examples
================================================================

timer is a decorator function: it must placed above the function (that needs to be timed) definition

.. code-block:: python

  from verstack.tools import timer

  @timer
  def func(a,b):
      print(f'Result is: {a + b}')

  func(2,3)

  >>>Result is: 5
  >>>Time elapsed for func execution: 0.0002 seconds



******************
Links
******************
`Git <https://github.com/DanilZherebtsov/verstack>`_

`pypi <https://pypi.org/project/verstack/>`_

`author <https://www.linkedin.com/in/danil-zherebtsov/>`_