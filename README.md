## Missing value imputer
This component is for missing value imputation. It will give the evaluation result of different imputation method. Now the functionality is limited to:

* one label problem and label target is the second column in label file

### Dependencies
[check here](environment.yml)

if you have conda, simply do the following:

```sh
conda-env create .
source activate mvi
python test_example.py
```

### Usage:
see [test_example.py](test_example.py):

```python
from sklearn import svm
from sklearn import metrics
from sklearn.metrics import f1_score, make_scorer

from dsbox.datapreprocessing.cleaner import Imputation
from dsbox.datapreprocessing.cleaner import helper_func

# STEP 1: get data
data_path = "../dsbox-data/o_38/data/"
data_name = data_path + "trainData.csv"
label_name = data_path + "trainTargets.csv" # make sure your label target is in the second column of this file

drop_col_name = ["d3mIndex"] #input the column name that is useless in the dataset, eg. id-like column, name column, etc.
data, label = helper_func.dataPrep(data_name, label_name, drop_col_name)

# STEP 2: define your machine learning model and scorer
clf = svm.SVC(kernel='linear')
scorer = make_scorer(f1_score, average="macro") # score will be * -1, if greater_is_better is set to False

# STEP 3: go to use the Imputer !
imputer = Imputation(model=clf, scorer=scorer, strategies=["mean", "max", "min", "zero"])
best_imputation = imputer.fit(data, label)
print best_imputation

```


### TODO:
1. finish verbose func


## One-hot encoder
The encoder takes csv file or pandas DataFrame as input, then one-hot encode columns which are considered categorical. (currently:
take a column as category if:
* its dtype is not float and
* 95% of its data fall in 10 values.
* For the rest values (not top 10) with low frequency, put into one column "_others"

Note, currently: 
* For nonnumeric columns which don't fall into categories, they are converted into integer codes (0,1,2...), just as a temporary expedient.
* For column which has single unique value with some cells missing, the encoder only
  convert the original column into an indicator column "_nan" to tell if missing.
* For column which has two unique values and no cells missing, the encoder only convert
  the original column into binary (0/1) values.

### Usage:
```python
from dsbox.datapreprocessing.cleaner import encoder
# csv file as input: 
result = encoder.encode('yourDataset.csv')

# DataFrame as input:
data = pd.read_csv('yourDataset.csv')
result = encoder.encode(data)

# if label is given in the dataset
result = encoder.encode(data, label='name_of_label_column')

```

### TODO:
1. Deal with ID-like columns: identify (also let user decide?) and delete ? 
2. Find better way to distinguish categorical columns.
3. More functionality and more flexible implementation for user to config prefered setting.


## Discretizer
Take a column (pandas Series) as input, output a column with discretized values. For the discretize() function:
* **by**: "width": discretize by equal width; "frequency": discretize by equal frequency; "kmeans": discretize by kmeans clustering; "gmm": discretize by Gaussian mixure models clustering. default by="width".
* **num_bins**: number of bins. default num_bins=10.
* **labels**: list of values for the discretized bins, currently only for binning methods where orders of values are kept (by width and by frequency). default labels= [0,1,2...].


Note, currently: 
* Missing cells remain missing in the output column.

### Usage:
```python
from dsbox.datapreprocessing.cleaner import discretizer

data = pd.read_csv('yourDataset.csv')
col = data["column_name"]
# 10 bins, discretize by equal width
result = discretizer.discretize(col)
# 5 bins, discretize by gmm
result = discretizer.discretize(col,num_bins=5,by='gmm')
# or you can replace original column in the dataset with discretized values
data["column_name"] = result

```

### TODO:
- See if a better k, number of bins to choose can be found automatically. e.g. num_bins='auto'.
