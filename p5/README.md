# Enron dataset analysis: detect persons-of-interest (POIs)

### Student: Irina Truong
### Date: 03/06/2016

## Project goal

The goal of this project is to detect persons of interest in Enron
email dataset. The dataset contains financial and email data made
public as a result of the Enron scandal. There is also an additional
hand-generated list of persons of interest in the fraud case, which
means individuals who were indicted, reached a settlement or plea
deal with the government, or testified in exchange for prosecution
immunity.

## Data overview

* Total number of data points: 146
* POI: 18, non-POI: 128 (12% POI)
* Features (21 total):
    * salary : 35% empty
    * to_messages : 41% empty
    * deferral_payments : 73% empty
    * total_payments : 14% empty
    * exercised_stock_options : 30% empty
    * bonus : 44% empty
    * restricted_stock : 25% empty
    * shared_receipt_with_poi : 41% empty
    * restricted_stock_deferred : 88% empty
    * total_stock_value : 14% empty
    * expenses : 35% empty
    * loan_advances : 97% empty
    * from_messages : 41% empty
    * other : 36% empty
    * from_this_person_to_poi : 41% empty
    * poi : 0% empty
    * director_fees : 88% empty
    * deferred_income : 66% empty
    * long_term_incentive : 55% empty
    * email_address : 24% empty
    * from_poi_to_this_person : 41% empty
    
We have 20 impot features, out of which some are financial and some
are email-related:

 Payments (10)      |     Stock (4)             | Email-related features (6)
--------------------|---------------------------|---------------------------
salary              | exercised_stock_options   | to_messages
bonus               | restricted_stock          | shared_receipt_with_poi
long_term_incentive | restricted_stock_deferred | from_messages
deferred_income     | total_stock_value         | from_this_person_to_poi
deferral_payments   | -                         | email_address
loan_advances       | -                         | from_poi_to_this_person
other               | -                         | -
expenses            | -                         | -
director_fees       | -                         | -
total_payments      | -                         | -

## Bad data

* `TOTAL` data point was removed as an outlier
* `LOCKHART EUGENE E` record was removed because all values in that row were `NaN`
* `BELFER ROBERT` and `BHATNAGAR SANJAY` were updated with the correct data to correspond to PDF.

## Additional features / removed features

I added some features to the dataset:

**Financial totals**:

* `total_income` is a sum of `total_payments` and `total_stock_value`

**Email totals**:

* `total_poi_correspondence` is a sum of all emails related with POI: 
  `from_poi_to_this_person`, `from_this_person_to_poi`, `shared_receipt_with_poi`

**Email ratios**:
  
* `fraction_to_poi`: ratio of `from_this_person_to_poi` to `from_messages`
* `fraction_from_poi`: ratio of `from_poi_to_this_person` to `to_messages`
* `fraction_poi_correspondence`: ratio of `total_poi_correspondence` to
  all emails, `to` and `from`.
  
**No totals**:

I experimented with removing `total_payments` and `total_stock_value`
from the dataset, since they are already aggregates.

## Classifiers

I evaluated the following classifier algorithms (see `poi_algo.py`):

* `LogisticRegression`
* `LinearSVC`
* `SVC` with `rbf` kernel
* `KMeans` with 2 clusters
* `GaussianNB`
  
## Feature scaling

Because we have email features that are measured in tens or hundreds, 
and financial features that can be as high as a few millions, I used a
`MinMaxScaler` as part of pipeline to scale the features.

## Feature selection

Feature selection step is represented by a `SelectKBest` part of the
pipeline. I implemented a loop in the code that searches through and
evaluates all possible combinations of classifiers and sets of
features, and logs out features scored by `SelectKBest` for each
combination (see `output` directory).

The following feature sets were tried:

* all: financial totals, email totals, email ratios
* all-totals: financial totals, email totals
* finance-totals: financial totals
* email-totals: email totals
* email-ratios: email ratios
* none: no additional features
* no-totals: also remove original totals.

Metrics for all these are listed in the following table:

Features | Classifier | Accuracy | Precision | Recall | F1| F2
---------|------------|----------|-----------|--------|---|----
no-totals | gaussiannb | 0.83047 | 0.34565 | 0.30400 | 0.32349 | 0.31151
all | gaussiannb | 0.83320 | 0.34352 | 0.27550 | 0.30577 | 0.28686
all-totals | gaussiannb | 0.83320 | 0.34352 | 0.27550 | 0.30577 | 0.28686
email-ratios | gaussiannb | 0.83320 | 0.34352 | 0.27550 | 0.30577 | 0.28686
email-totals | gaussiannb | 0.83320 | 0.34352 | 0.27550 | 0.30577 | 0.28686
finance-totals | gaussiannb | 0.83320 | 0.34352 | 0.27550 | 0.30577 | 0.28686
none | gaussiannb | 0.83320 | 0.34352 | 0.27550 | 0.30577 | 0.28686
none | logreg | 0.74807 | 0.30515 | 0.69650 | 0.42437 | 0.55432
none | svc | 0.80720 | 0.29100 | 0.31050 | 0.30044 | 0.30639
no-totals | logreg | 0.74380 | 0.29081 | 0.64050 | 0.40000 | 0.51632
all-totals | linsvc.log | 0.73847 | 0.28261 | 0.62500 | 0.38923 | 0.50310
email-ratios | linsvc | 0.73847 | 0.28261 | 0.62500 | 0.38923 | 0.50310
no-totals | linsvc | 0.71813 | 0.28010 | 0.70950 | 0.40164 | 0.54301
no-totals | svc | 0.79253 | 0.27760 | 0.34700 | 0.30844 | 0.33048
all | svc | 0.71413 | 0.27665 | 0.70850 | 0.39792 | 0.53993
all-totals | svc.log | 0.71413 | 0.27665 | 0.70850 | 0.39792 | 0.53993
email-ratios | svc | 0.71413 | 0.27665 | 0.70850 | 0.39792 | 0.53993
email-totals | svc | 0.71413 | 0.27665 | 0.70850 | 0.39792 | 0.53993
finance-totals | svc | 0.71413 | 0.27665 | 0.70850 | 0.39792 | 0.53993
none | linsvc | 0.66540 | 0.27311 | 0.90850 | 0.41997 | 0.62001
all | logreg | 0.69793 | 0.26386 | 0.70700 | 0.38429 | 0.52923
email-ratios | logreg | 0.69793 | 0.26386 | 0.70700 | 0.38429 | 0.52923
email-totals | logreg | 0.69853 | 0.26386 | 0.70450 | 0.38392 | 0.52811
finance-totals | logreg | 0.69853 | 0.26386 | 0.70450 | 0.38392 | 0.52811
email-totals | linsvc | 0.69693 | 0.26347 | 0.70900 | 0.38418 | 0.52982
all-totals | logreg.log | 0.69573 | 0.26312 | 0.71200 | 0.38424 | 0.53087
all | linsvc | 0.69413 | 0.26204 | 0.71250 | 0.38317 | 0.53021
finance-totals | linsvc | 0.69413 | 0.26204 | 0.71250 | 0.38317 | 0.53021
no-totals | kmeans | 0.54473 | 0.15012 | 0.51800 | 0.23278 | 0.34763
email-ratios | kmeans | 0.50860 | 0.14322 | 0.53900 | 0.22630 | 0.34714
all | kmeans | 0.51013 | 0.13884 | 0.51400 | 0.21863 | 0.33368
none | kmeans | 0.52700 | 0.13870 | 0.48900 | 0.21611 | 0.32490
all-totals | kmeans.log | 0.50847 | 0.13857 | 0.51500 | 0.21838 | 0.33370
finance-totals | kmeans | 0.50847 | 0.13818 | 0.51300 | 0.21772 | 0.33258
email-totals | kmeans | 0.50507 | 0.13568 | 0.50500 | 0.21389 | 0.32699

This table is sorted first by precision, and then by recall, because
precision in general was by far the worst metric out of the two. We are
really only interested in the first 8 records here, because all others
have precision lower than the `0.3` threshold specified in project
requirements. 

We can see that logistic regression with no additional features performed
best in terms of recall (**0.69650**), while still satisfying
the requirement of precision (**0.30515**). So this combination is
what I used in `poi_id.py` to dump classifier and data in the end. I
think if we were really looking for people to investigate more closely,
false positives are acceptable, and the less false negatives we have
the better. So there's no need to worry too much about low precision.

Here is a table of features selected by `SelectKBest` when using
logistic regression, and their scores:

 # |                   Feature |     Score | Selected
---|---------------------------|-----------|---------
1  |         total_stock_value | 22.782108 |     True
2  |   exercised_stock_options | 22.610531 |     True
3  |                     bonus | 21.060002 |     True
4  |                    salary | 18.575703 |     True
5  |           deferred_income | 11.561888 |     True
6  |       long_term_incentive | 10.072455 |     True
7  |            total_payments |  9.380237 |     True
8  |          restricted_stock |  8.964964 |     True
9  |   shared_receipt_with_poi |  8.746486 |     True
10 |             loan_advances |  7.242730 |     True
11 |                  expenses |  5.550684 |     True
12 |   from_poi_to_this_person |  5.344942 |     True
13 |                     other |  4.219888 |     True
14 |   from_this_person_to_poi |  2.426508 |     True
15 |             director_fees |  2.112762 |     True
16 |               to_messages |  1.698824 |     True
17 | restricted_stock_deferred |  0.743493 |     True
18 |         deferral_payments |  0.221214 |    False
19 |             from_messages |  0.164164 |    False

Total stock value seems to be the most important feature, followed by
exercised_stock_options, bonus and salary. Email features are not high
on the list, but they were included in the set anyway. Only deferral
payments and from_messages were not included by `GridSearchCV`.

## Scoring the grid search

This was challenging, because we can usually ask `GridSearchCV` to
optimize for one of the metrics: either precision or recall. But we have
a requirement in this project that both should be at least `0.3`. To solve
this, I added a custom scorer in `poi_id.py`:

```python
def precision_vs_recall_score(actual, predicted):
    """We want both precision and recall to be as high as possible, 0.3 at
    least."""
    pr = precision_score(actual, predicted)
    rc = recall_score(actual, predicted)
    if (pr < 0.3) or (rc < 0.3):
        return min(pr, rc)
    return max(pr, rc)
```

The logic of this is as follows: we calculate both precision and recall.
Then if one of them is lower than our threshold, we want to penalize this
combination in grid search and make it appear low on our list, so we return
**the worst of the two**. Otherwise, we want the higher metrics to score
high, so we return **the best of the two** (for this dataset, usually 
it'd be recall).

## Conclusion

This was a challenging project, because the dataset is so small, and we
only have a few POIs that we can learn from. I found that additional
features (totals, ratios) were not even helpful - probably because
they are highly correlated with features we already have in dataset. In
the end, I picked Logistic Regression and ended up with recall 0.69650
and precision of 0.30515.

## Links

I had to look at some other students' code to help me understand and
implement this project. From the Machine Learning class only, I was not
able to fully understand Pipelines and GridSearchCV. Repositories:

http://fch808.github.io/Identifying_Fraud_at_Enron.html
https://jaycode.github.io/enron/identifying-fraud-from-enron-email.html
https://github.com/allanbreyes/udacity-data-science/blob/master/p4/README.md
https://github.com/yielder/identifying-fraud-from-enron-email

I also used `sklearn` extensive online documentation:

http://scikit-learn.org/stable/documentation.html