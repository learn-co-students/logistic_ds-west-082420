
# Linear to Logistic regression
![img](img/linear_vs_logistic_regression.jpg)

## Learning goals

You will be able to:
* Describe the need for logistic regression
* Describe the mathematics behind logistic regression
* Interpret the parameters of a logistic regression model

## What do we know about linear regression?

- What are the requirements for the variables types?
- What assumptions do we have?
- How do we interpret the coefficients?
- What metrics do we use to evaluate our model?

And how will logistic regression be different?

![log](https://media.giphy.com/media/m8DnDYfRwEtvG/giphy.gif)

So far, we have used linear regression to predict continuous target variables: 

1. carbon offset 
2. NFL draft position 
3. home prices 
4. used car prices
5. Spotify streams

In exploring possible subjects, you almost surely came across data meant to predict classifications. The target was a binary variable: 

1. A patient has heart disease or not. 
2. A baby will be a boy or a girl.
3. A visa application will be approved or not.
4. A released prisoner will be imprisoned again or not.

## Scenarios 

*We will return to the scenarios below with real data at the bottom of the notebook*

#### Scenario 1: Predict income bracket
In this example, we want to find a relationship between age and monthly income. It is definitely reasonable to assume that, on average, older people have a higher income than younger people who are newer to the job market and have less experience.

#### Scenario 2: Predict likelihood of diabetes
This dataset is originally from the National Institute of Diabetes and Digestive and Kidney Diseases. The objective of the dataset is to diagnostically predict whether or not a patient has diabetes, based on certain diagnostic measurements included in the dataset. Several constraints were placed on the selection of these instances from a larger database. In particular, all patients here are females at least 21 years old of Pima Indian heritage. [reference here](https://www.kaggle.com/uciml/pima-indians-diabetes-database)

## Created income data


```python
# create data

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

np.random.seed(1234)

age = np.random.uniform(18, 65, 100)
income = np.random.normal((age/10), 0.5)
age = age.reshape(-1,1)
income.shape
```

Plot it!


```python
fig = plt.figure(figsize=(8,6))
fig.suptitle('age vs income', fontsize=16)
plt.scatter(age, income)
plt.xlabel("age", fontsize=14)
plt.ylabel("monthly income", fontsize=14)
plt.show()
```

In linear regression, you would try to find a relationship between age and monthly income. Conceptually, this means fitting a line that represents the relationship between age and monthly income, as shown below.


```python
fig = plt.figure(figsize=(8,6))
fig.suptitle('linear regression', fontsize=16)
plt.scatter(age, income)
plt.plot(age, age/10, c = "black")
plt.xlabel("age", fontsize=14)
plt.ylabel("monthly income", fontsize=14)
plt.show()
```

The idea is that you could use this line to make predictions in the future. In this case, the relationship is modeled as follows: the expected monthly income for someone who is, say, 40 years old, is 3000 (3 on the y-axis). Of course, the actual income will most likely be different, but this gives an indication of what the model predicts as the salary value.

## So how is this related to logistic regression?

Now, imagine you get a data set where no information on exact income is given (after all, people don't like to talk about how much they earn!), but you only have information on whether or not they earn more than 4000 USD per month. Starting from the generated data we used before, the new variable `income_bin` was transformed to 1 when someone's income is over 4000 USD, and 0 when the income is less than 4000 USD.


```python
# Your turn: Add code that transforms the income to a binary

income_bin = None

income_bin 
```

Let's have a look at what happens when we plot this.


```python
fig = plt.figure(figsize=(8,6))
fig.suptitle('age vs binary income', fontsize=16)
plt.scatter(age, income_bin)
plt.xlabel("age", fontsize=14)
plt.ylabel("monthly income (> or < 4000)", fontsize=14)
plt.show()
```

You can already tell that fitting a straight line will not be exactly desired here, but let's still have a look at what happens when you fit a regression line to these data. 


```python
from sklearn.linear_model import LogisticRegression
from sklearn.linear_model import LinearRegression

# create linear regression object
lin_reg = LinearRegression()
lin_reg.fit(age, income_bin)
# store the coefficients
coef = lin_reg.coef_
interc = lin_reg.intercept_
# create the line
lin_income = (interc + age * coef)
```


```python
fig = plt.figure(figsize=(8,6))
fig.suptitle('linear regression', fontsize=16)
plt.scatter(age, income_bin)
plt.xlabel("age", fontsize=14)
plt.ylabel("monthly income", fontsize=14)
plt.plot(age, lin_income, c = "black")
plt.show()
```

You can see that this doesn't make a lot of sense. This straight line cannot grasp the true structure of what is going on when using a linear regression model. Now, without going into the mathematical details for now, let's look at a logistic regression model and fit that to the dataset.


```python
# Create logistic regression object
regr = LogisticRegression(C=1e5)
# Train the model using the training sets
regr.fit(age, income_bin)
```


```python
# store the coefficients
coef = regr.coef_
interc = regr.intercept_
# create the linear predictor
lin_pred= (age * coef + interc)
# perform the log transformation
mod_income = 1 / (1 + np.exp(-lin_pred))
#sort the numbers to make sure plot looks right
age_ordered, mod_income_ordered = zip(*sorted(zip(age ,mod_income.ravel()),key=lambda x: x[0]))
```

### Look at dataset predictions

It is the **probability** of being in the target class


```python
np.set_printoptions(suppress=True)
print(mod_income[:6])
```

### Plot it!


```python
fig = plt.figure(figsize=(8,6))
fig.suptitle('logistic regression', fontsize=16)
plt.scatter(age, income_bin)
plt.xlabel("age", fontsize=14)
plt.ylabel("monthly income", fontsize=14)
plt.plot(age_ordered, mod_income_ordered, c = "black")
plt.show()
```

#### Review the new shape

This already looks a lot better! You can see that this function has an S-shape which plateaus to 0 in the left tale and 1 to the right tale. This is exactly what we needed here. Hopefully this example was a good way of showing why logistic regression is useful. Now, it's time to dive into the mathematics that make logistic regression possible.

That **S-shape** is what's known as a **sigmoid function**

![sigmoid](img/SigmoidFunction_701.gif)

## Logistic regression model formulation

### The model

As you might remember from the linear regression lesson, a linear regression model can be written as:

$$ \hat y = \hat\beta_0 + \hat\beta_1 x_1 + \hat\beta_2, x_2 +\ldots + \beta_n x_n $$

When there are $n$ predictors $x_1,\ldots,x_n$ and $n+1$ parameter estimates that are estimated by the model $\hat\beta_0, \hat\beta_1,\ldots, \hat\beta_n$. $ \hat y $ is an estimator for the outcome variable.

Translating this model formulation to our example, this boils down to:

$$ \text{income} = \beta_0 + \beta_1 \text{age} $$

When you want to apply this to a binary dataset, what you actually want to do is perform a **classification** of your data in one group versus another one. In our case, we want to classify our observations (the 100 people in our data set) as good as possible in "earns more than 4k" and "earns less than 4k". A model will have to make a guess of what the **probability** is of belonging to one group versus another. And that is exactly what logistic regression models can do! 

### Transformation

Essentially, what happens is, the linear regression is *transformed* in a way that the outcome takes a value between 0 and 1. This can then be interpreted as a probability (e.g., 0.2 is a probability of 20%). Applied to our example, the expression for a logistic regression model would look like this:

$$ P(\text{income} > 4000) = \displaystyle \frac{1}{1+e^{-(\hat \beta_0+\hat \beta_1 \text{age})}}$$

Note that the outcome is written as $P(\text{income} > 4000)$. This means that the output should be interpreted as *the probability that the monthly income is over 4000 USD*.

It is important to note that this is the case because the income variable was relabeled to be equal to 1 when the income is bigger than 4000, and 0 when smaller than 4000. In other words, The outcome variable should be interpreted as *the* **probability** *of the class label to be equal to 1*.

### Interpretation - with a side of more math
#### What are the odds?

As mentioned before, the probability of an income over 4000 can be calculated using:

$$ P(\text{income} > 4000) = \displaystyle \frac{1}{1+e^{-(\hat \beta_o+\hat \beta_1 \text{age})}}$$

You can show that, by multiplying both numerator and denominator by $e^{(\hat \beta_0+\hat \beta_1 \text{age})}$


$$ P(\text{income} > 4000) = \displaystyle \frac{e^{\hat \beta_0+\hat \beta_1 \text{age}}}{1+e^{\hat \beta_o+\hat \beta_1 \text{age}}}$$

As a result, you can compute $P(\text{income} \leq 4000)$ as:

$$ P(\text{income} < 4000) = 1- \displaystyle \frac{e^{\hat \beta_0+\hat \beta_1 \text{age}}}{1+e^{\hat \beta_o+\hat \beta_1 \text{age}}}= \displaystyle \frac{1}{1+e^{\hat \beta_0+\hat \beta_1 \text{age}}}$$



#### Odds ratio

This doesn't seem to be very spectacular, but combining these two results leads to an easy interpretation of the model parameters, triggered by the *odds*

$$ \dfrac{P(\text{income} > 4000)}{P(\text{income} < 4000)} = e^{\hat \beta_0+\hat \beta_1 \text{age}} $$

This expression can be interpreted as the *odds in favor of an income greater than 4000 USD*.

Taking the log of both sides leads to:
<br><br>
    $\ln{\dfrac{P(\text{income} > 4000)}{P(\text{income} < 4000)}} = \beta_0 + \beta_1*X_1 + \beta_2*X_2...\beta_n*X_n$
    
Here me can see why we call it logisitic regression.

Our linear function calculates the log of the probability we predict 1, divided by the probability of predicting 0.  In other words, the linear equation is calculating the **log of the odds** that we predict a class of 1.

## Generalized Linear Model
The strategy is to *generalize* the notion of linear regression; regression will become a special case. In particular, we'll keep the idea of the regression best-fit line, but now **we'll allow the model to be constructed from the dependent variable through some (non-trivial) function of the linear predictor**. 
This function is standardly called the **link function**. 

The equation from above: 
$\large\ln\left(\frac{p}{1 - p}\right) = \beta_0 + \beta_1x_1 + ... + \beta_nx_n$
<br>
is the characteristic link function is this logit function.

# Decision Boundary


![](img/decision_boundary_1.jpg)
![](img/decision_boundary_2.jpg)


```python
def sigmoid(log_odds):
    
    prob_class_1 = 1/(1+np.e**(log_odds*-1))
    
    return prob_class_1

sigmoid(0)
```




    0.5



#### Interpretting coefficients

This result, in combination with mathematical properties of exponential functions, leads to the fact that, applied to our example:

if *age* goes up by 1, the odds are multiplied by $e^{\beta_1}$

In our example, there is a positive relationship between age and income, this will lead a positive $\beta_1 > 0$, so $e^{\beta_1}>1$, and the odds will increase as *age* increases.

## Fitting the Model


Ordinary least squares does not make sense with regards to odds and binary outcomes.  The odds of the true value, 1, equals 1/(1-1). Instead of OLS, we frame the discussion as likelihood.  What is the likelihood that we see the labels given the features and the hypothesis. 

To maximize likelihood, we need to choose a probability distribution.  In this case, since the labels are binary, we use the Bernouli distribution. The likelihood equation for the Bernouli distribution is:

$ Likelihood=\prod\limits_{i=0}^N p_i^{y_i}(1-p_i)^{1-y_i}$

Taking the log of both sides leads to the log_likelihood equation:

$loglikelihood = \sum\limits_{i=1}^N y_i\log{p_i} + (1-y_i)\log(1-p_i) $

The goal of MLE is to maximize log-likelihood



![Maximum Likelihood](img/MLE.png)


There is no closed form solution like the normal equation in linear regression, so we have to use stocastic gradient descent.  To do so we take the derivative of the loglikelihood and set it to zero to find the gradient of the loglikelihood, then update our coefficients. Just like linear regression, we use a learning rate when updating them.

Math behind the gradient of log-likelihood is ESL section 4.4.1: https://web.stanford.edu/~hastie/ElemStatLearn//.

### Assumptions

- Binary logistic regression requires the dependent variable to be binary.
- For a binary regression, the factor level 1 of the dependent variable should represent the desired outcome.
- Only the meaningful variables should be included.
- The independent variables should be independent of each other. That is, the model should have little or no multicollinearity.
- The independent variables are linearly related to the log odds.
- Logistic regression requires quite large sample sizes.

# A real data example: Salaries (statsmodels)


```python
import statsmodels as sm
import numpy as np
import pandas as pd
import sklearn.preprocessing as preprocessing
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split
from scipy import stats
```


```python
salaries = pd.read_csv("salaries_final.csv", index_col = 0)
salaries.Target.value_counts()
```


```python
# !pip install patsy
```


```python
from patsy import dmatrices
y, X = dmatrices('Target ~ Age  + Race + Sex',
                  salaries, return_type = "dataframe")
```

#### Statsmodels method
[statsmodels logit documentation](https://www.statsmodels.org/dev/generated/statsmodels.discrete.discrete_model.Logit.html)


```python
import statsmodels.api as sm
logit_model = sm.Logit(y.iloc[:,1], X)
result = logit_model.fit()
```


```python
# statsmodels has a nice summary function - remember this
result.summary()
```


```python
# translate the coefficients to reflect odds
np.exp(result.params)
```


```python
pred = result.predict(X) > .7
sum(pred.astype(int))
```

Once you **get** a model with `Logit` you can use `LogitResults` to evaluate it's performance more in depth. 
[documentation](http://www.statsmodels.org/devel/generated/statsmodels.discrete.discrete_model.LogitResults.html)

## A Real Data Example: Diabetes (sklearn)




```python
diabetes = pd.read_csv('diabetes.csv')
diabetes.shape
```


```python
diabetes.head()
```


```python
import seaborn as sns
sns.pairplot(diabetes)
```


```python
diabetes.corr()
```


```python
diabetes.dtypes
```


```python
import sklearn.preprocessing as preprocessing

scaler = preprocessing.StandardScaler()
```


```python
X = diabetes.iloc[:,:-1]
```


```python
X.head()
```


```python
Y = diabetes.Outcome
```


```python
Y.head()
```


```python
X_scaled = scaler.fit_transform(X)
```


```python
type(X_scaled)
```


```python
X.columns
```


```python
X.corr()
```


```python
x_Df = pd.DataFrame(X_scaled, columns=X.columns)
```


```python
x_Df.corr()
```

## C parameter

Logistic regression in sklearn allows tuning of the regularization strength, i.e. Lasso/Ridge, via the C parameter.  

Like in regression, except now in MLE, the lasso adds a  term to the equation which penalizes models with too many coefficients, and ridge penalizes models with large coefficients. 

The strength of the penalty is the $\lambda$ term

C is the inverse of $\lambda$, so a small C results in a large penalty.


```python
logreg = LogisticRegression(C = 1**1000, penalty='l2')
model_log = logreg.fit(x_Df, Y)
model_log
```


```python
logreg.score(X_scaled, Y)
```


```python
#we can iterate through values of C to find the optimal parameter.
import warnings
warnings.filterwarnings('ignore')

best = 0
best_score = 0
for c in np.arange(.001, 2, .001):
    lr = LogisticRegression(C=c, penalty='l1')
    lr.fit(X_scaled, Y)
    if lr.score(X_scaled, Y) > best_score:
        best = c
        best_score = lr.score(X_scaled, Y)
print(best)
print(best_score)
```


```python
lr = LogisticRegression(C=.001, penalty='l2')
lr.fit(X_scaled, Y)
```


```python
lr.coef_
```


```python
#Or we can use grid-search.

# Create regularization penalty space
penalty = ['l1', 'l2']

# Create regularization hyperparameter space
C = np.arange(.1, 100, .5)
print(C)
# Create hyperparameter options
hyperparameters = dict(C=C, penalty=penalty)
```


```python
from sklearn.model_selection import GridSearchCV
# Create grid search using 5-fold cross validation
lr = LogisticRegression()
clf = GridSearchCV(lr, hyperparameters, cv=5, verbose=0)

grid = clf.fit(X_scaled, Y)
print(grid.best_estimator_.get_params()['penalty'])
print(grid.best_estimator_.get_params()['C'])
```


```python
lr = LogisticRegression(C=.6, penalty='l2')
lr.fit(X_scaled, Y)
lr.score(X_scaled, Y)
```


```python
model_log.coef_
```


```python
y_pred = lr.predict(X_scaled)
# Returns the probabilitis instead of the rounded predictions
y_proba =lr.predict_proba(X_scaled)
# Returns the accuracy
# lr.score(X_scaled, Y)
y_proba[:,1] > .5
```


```python
from sklearn.metrics import accuracy_score
```


```python
accuracy_score(Y, y_pred)
```


```python

```