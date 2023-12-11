---
title: 'Fraud Transactions Detection with Isolation Forest'
date: 2020-10-30
permalink: /posts/2020/10/fraud-detection-isolation-forest/
tags:
  - Isolation Forest
  - Anomaly Detection
  - Fraud Detection
  - Financial Transactions
---

Problem
======
Nowadays, more digital transactions performed over internet, higer chance your credit card information is leaked and thus hackers can performed fraud purchases on stolen credit cards.

We can deal with credit card frauds by using via anomaly detection algorithms. Such an algorithm typically models user profiling to determine baseline behaviors from her spending habits. We thus can identify patterns by grouping transactions, for instance, by location, type, amount, frequency, etc. It requires card issuers to build an AI-based anomaly detection that can produce high accuracy on fraud detection and can be automated in giving decisions.


IEEE-CIS Fraud Detection Dataset
======
[The IEEE-CIS dataset](https://www.kaggle.com/c/ieee-fraud-detection/data) is prepared by Vesta Corporation (available on Kaggle)for fraud detection. It is labeled dataset containing two types of CSV files: transactions and identity. Each type has a train and a test file. While train file of identity has 144,233 records and 41 features, train file of transactions has 590,540 records and 394 features, 181 days of transactions.

```python
import pandas as pd
import numpy as np
import os
import matplotlib.pyplot as plt
%matplotlib inline
from pandas.plotting import scatter_matrix
import seaborn as sns
from scipy.stats import pearsonr
import warnings
warnings.filterwarnings('ignore')
plt.style.use("ggplot")
# Load data
datapath = os.path.join("data", "ieee-fraud-detection")
df_iden_train = pd.read_csv(datapath + "/train_identity.csv")
df_iden_test = pd.read_csv(datapath + "/test_identity.csv")
df_trans_train = pd.read_csv(datapath + "/train_transaction.csv")
df_trans_test = pd.read_csv(datapath + "/test_transaction.csv")
```

## Joining transactions and identity datasets

```python
df_train = pd.merge(df_trans_train, df_iden_train, on='TransactionID', how='left',left_index=True,right_index=True)
df_test = pd.merge(df_trans_test, df_iden_test, on='TransactionID', how='left',left_index=True,right_index=True)
df_train.shape
```

It is a big dataset whose size of two train files joint is almost 2Gb with 590,540 records and 434 features.

```python
df_train.isFraud.value_counts()
df_train.isFraud.value_counts().plot.bar()
```

## Fraud Ratio

![Check fraud ratio](https://miro.medium.com/max/1400/1*6EeaWP4H-07dP6MAVOarEQ.webp)

The dataset contains 20,633 frauds out of 590,540 records, that is hence highly imbalanced dataset.

```python
df_train.isnull().sum()/len(df_train)*100    # missing ratio
```

This dataset also has high ratio of missing values, especially with features Addresses, Email, Device type, info and IDs.

## Let consider the correlation of different variables with the target variable

```python
# Check the correlation of features with the target
import seaborn as sns
corr = df_train.corrwith(df_train.isFraud).reset_index()
corr.columns = ['Index', 'Correlations']
corr = corr.set_index('Index')
corr = corr[abs(corr.Correlations) > 0.25]
corr = corr.sort_values(by=['Correlations'], ascending = False)
plt.figure(figsize=(4,10))
fig = sns.heatmap(corr, annot=True, fmt = "g", cmap="RdBu_r")
plt.title("Correlation of variables with Class")
plt.show()
```

![Check the correlation of variables](https://miro.medium.com/max/640/1*eMsajQYEOnWbBujvKurVlA.webp)


## Distribution of Log Transaction Amount

```python
#plot the amount feature
plt.figure(figsize=(10,8))
plt.title('Distribution of Amount in Log Transformation')
sns.distplot(np.log(df_train[df_train['isFraud']==0]['TransactionAmt']));
sns.distplot(np.log(df_train[df_train['isFraud']==1]['TransactionAmt']));
plt.legend(['legit','fraud'])
plt.show()
```

![Distribution of Log Transaction Amount](https://miro.medium.com/max/640/1*yHwsyHlyDrw77JjT7Ew6Tg.webp)


Frauds in IEEE-CIS dataset
======
We can further perform analysis on frauds of IEEE-CIS dataset. At first, let’s consider what types of product frauders performed:

```python
df_fraud = df_train[df_train.isFraud == 1]
plt.ylabel('type')
df_fraud.ProductCD.value_counts().plot.bar()
```

![Fraud distribution based on types of products purchased](https://miro.medium.com/max/640/1*J5vdZJGX8KFv25OyOL7NHA.webp)

Let’s see the distribution of frauds based on time purchased (hour or day):

```python
df_fraud['hour'] = (df_fraud['TransactionDT']//(3600))%24
df_test['hour'] = (df_test['TransactionDT']//(3600))%24
plt.figure(figsize=(20,10))
percentage = lambda i: len(i) / float(len(df_fraud['hour'])) * 100
ax = sns.barplot(x = df_fraud['hour'], y = df_fraud['hour'],  estimator = percentage)
ax.set(ylabel = "Percent")
plt.title('Fraud')
```

![Fraud distribution based on time performed in a day](https://miro.medium.com/max/786/1*DewYhU3nNTqNSF8l9XBIzw.webp)

```python
df_fraud['day'] = (df_fraud['TransactionDT']//(3600*24))%7
plt.figure(figsize=(15,10))
percentage = lambda i: len(i) / float(len(df_fraud['day'])) * 100
ax = sns.barplot(x = df_fraud['day'], y = df_fraud['day'],  estimator = percentage)
ax.set(ylabel = "Percent")
plt.title('Fraud')
```

![Fraud](https://miro.medium.com/max/786/1*DTWAQ8Tc4Sr8XFxL_L1TvA.webp)

From hour-based, we can see that more frauds are performed from 4:00pm till 2:00am and less frauds performed in the daylight time. There is no significant different among days in terms of frauds.

Now, let’s visualize frauds in the whole dataset

```python
plt.figure(figsize=(9,6))
sns.scatterplot(x="TransactionDT",y="TransactionAmt",hue="isFraud", data=df_train)
```

![Fraud](https://miro.medium.com/max/640/1*QzN8Ddehgtd_CjI6VvHgxA.webp)


Isolation Forest
======
Isolation Forest is a tree ensemble learning model, aims at detecting anomalies without profiling normal data points in advance. Isolation Forest is built on the basis of decision trees in which each tree divides the problem space into a set of partitions.

![Isolation Forest](https://miro.medium.com/max/786/1*xQmBQR6Fl7Mk4YZxjUVe4Q.webp)

In Isolation Forest, the partitions are created by first randomly selecting a feature and then selecting a random split value between the minimum and maximum value of the selected feature. Basically, each isolation tree is created by firstly randomly sampling N instances from the training dataset. Then, at each node:

1. Randomly choose a feature to split upon.
2. Randomly choose a split value from a uniform distribution spanning from the minimum value to the maximum value of the feature chosen in Step 2.

These steps are repeated until all N instances from dataset are “isolated” in leaf nodes of the isolation tree.

![Anomalies are more susceptible to isolation and hence have short path lengths](https://miro.medium.com/max/786/1*YqA0LxUifk_DznMTSLB7Vw.webp)

Basically, anomalies are less frequent than normal data and are different from them in terms of values. They lie further away from the normal data in the feature space. Thus, they could be identified closer to the root of the tree by using such a random partitioning.

Isolation Forest was implemented in the [sklearn](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.IsolationForest.html) python package. It has the following hyperparameters:

- **n_estimators**: *int, optional (default=100)*. The number of base estimators in the ensemble.
- **max_samples**: *int or float, optional (default=”auto”)*. The number of samples to draw from X to train each base estimator.
- **contamination**: *‘auto’ or float, optional (default=’auto’)*. The amount of contamination of the data set, i.e. the proportion of outliers in the data set. Used when fitting to define the threshold on the scores of the samples.
- **max_features**: *int or float, optional (default=1.0)*. The number of features to draw from X to train each base estimator.
- **bootstrap**: *bool, optional (default=False)*. If True, individual trees are fit on random subsets of the training data sampled with replacement. If False, sampling without replacement is performed.

```python
from sklearn.ensemble import IsolationForest
#split dataset
#fit model
clf = IsolationForest(max_samples = 256)
clf.fit()
```