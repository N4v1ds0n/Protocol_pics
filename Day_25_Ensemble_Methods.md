# Day 25 - Ensemble Methods

![not always](https://cdn.analyticsvidhya.com/wp-content/uploads/2017/02/12152925/meme.jpg)

*Protocol by Aljoscha*

# 1. Ensemble Methods - what, what for and how?

## A. what?
A collection of models working together on same dataset

## B. what for?
- less sensitive to overfitting
- lower error

## C. How?
- I: different models same data
- II: bagging: same model on different parts of the data
- III: boosting: seq. adjusts for importance of observations
- IV: stacking: training in parallel and combining

Results are combined via majority voting

# 2. Majority Voting - 

## A. When does it work?
- models are be better than random
- predictions from the different models are uncorrelated
- a sufficient amount of models

## B. How does it work?
predictions from different models are aggregated
- Hard Voting - mode
- Soft Voting - average (weighted by probability)

# I. Different models
use different methods e.g.:
- decision trees
- random forest
- neural networks
- support vector machine (SVM)
- naive bayes (NB)
- k nearest neighbors (KNN)
- Logistic Regression
- ...

# II. Bagging - Random Forest
tries to de-correlate predictions via
- randomly chosen subset of input variables
- randomly chosen subset of data cases (bootstrap)

Creating and training a random forest in extremely easy in Scikit-Learn. The cell below is all you need.
```
from sklearn.ensemble import RandomForestClassifier

# Create the model with 100 trees
model = RandomForestClassifier(n_estimators=100, 
                               random_state=RSEED, 
                               max_features = 'sqrt',
                               n_jobs=-1, verbose = 1)

# Fit on training data
model.fit(train, train_labels)
```
[documentation](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html)

Can be used for getting feature importance:

```
features = list(train.columns)
fi_model = pd.DataFrame({'feature': features,
                   'importance': model.feature_importances_}).\
                    sort_values('importance', ascending = False)
fi_model.head(10)
```
after "brew install graphviz" Trees can be visualized:
```
# Save tree as dot file
export_graphviz(tree, 'image/tree_real_data.dot', rounded = True, 
                feature_names = features, max_depth = 6,
                class_names = ['poor health', 'good health'], filled = True)

# Convert to png
call(['dot', '-Tpng', 'image/tree_real_data.dot', '-o', 'image/tree_real_data.png', '-Gdpi=200'])

# Visualize
Image(filename='image/tree_real_data.png')
```
<img src="https://raw.githubusercontent.com/JodaFlame/ds-random-forest/main/image/shorttree.png?token=AUAONYR7MRJMSM24MHIO33DA2IZJO" alt="drawing" width="400"/>

## conlusion

- Accuracy: Random Forest outperform Decision Trees
- Variance: Is higher for Decision Trees

## Thanks to Franziska and Peter for helping me with notebook 3 :)
here a nice plot they did:
```
sns.lmplot(data=loans, x='fico', y='int.rate', col='not.fully.paid', hue='credit.policy')
```
![thx](https://raw.githubusercontent.com/JodaFlame/ds-random-forest/main/image/test.png?token=AUAONYQHGM6RSPWBZGTWLCDA2I3XE)

# III. Boosting
Boosting builds multiple incremental models to decrease the bias, while keeping variance small.

1. Every instances has the same weight
2. Weights of instances incorrectly predicted are increased.
3. Next round: weights are updated ...and so on...
4. All models predict
5. Majority Vote (weighted average)

Scikit-Learn offers a nice implementation of AdaBoost with SAMME (a specific algorithm for Multi classification).
```
from sklearn.ensemble import AdaBoostClassifier
from sklearn.datasets import make_classification
X,Y = make_classification(n_samples=100, n_features=2, n_informative=2,
                          n_redundant=0, n_repeated=0, random_state=102)
clf = AdaBoostClassifier(n_estimators=4, random_state=0, algorithm='SAMME')
clf.fit(X, Y) 
```
Again, you can find Gradient Boosting function in Scikit-Learn’s library.
```
# for regression
from sklearn.ensemble import GradientBoostingRegressor
model = GradientBoostingRegressor(n_estimators=3,learning_rate=1)
model.fit(X,Y)

# for classification
from sklearn.ensemble import GradientBoostingClassifier
model = GradientBoostingClassifier()
model.fit(X,Y)
```

[explained in short here](https://towardsdatascience.com/boosting-algorithms-explained-d38f56ef3f30)

# IV. Stacking

1. We split the training data into K-folds just like K-fold cross-validation.
2. A base model is fitted on the K-1 parts and predictions are made for Kth part.
3. We do for each part of the training data.
4. The base model is then fitted on the whole train data set to calculate its performance on the test set.
5. We repeat the last 3 steps for other base models.
6. Predictions from the train set are used as features for the second level model.
7. Second level model is used to make a prediction on the test set.

![stacking](https://media.geeksforgeeks.org/wp-content/uploads/20190515104518/stacking.png)

[explained in short here](https://www.geeksforgeeks.org/stacking-in-machine-learning/)