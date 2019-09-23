Predicting the price of an apartment
====================================

Using the Python machine learning package scikit-learn_, we perform a regression analysis on the housing data described previously. We train regression models in order to predict the price of an apartment given its size, number of rooms, distance to the city centre and monthly fee. 

.. _scikit-learn: https://scikit-learn.org/stable/index.html

We begin by setting up a pipeline using the ``Pipeline()`` class of scikit-learn. This chains together various data preprocessing tools and makes it easy to perform a series of operations on a given data set. We impute missing data values with the mean value of the respective feature and scale the features to have zero mean and unit variance. The latter step typically improves the  performance of the models. 

We look at the following models:

- Decision tree regressor. 
- Random forest regressor. 
- Multilayer Perceptron regressor. Neural network regressor. 
- Gradient boosted tree regressor. This model does not come from scikit-learn, and is the ``XGBRegressor()`` estimator from the package XGBoost_.

.. _XGBoost: https://xgboost.ai

In all models we consider a five-fold cross-validation. In this procedure, the training data is split into five subsets, and each of these is used as a validation set once, with the remaining four sets used for training. This gives five validation scores, we aim to minimise the mean of these to end up with a good model. We use the *mean absolute error* for the score, which gives the mean value of the absolute value of the differences between predictions and ground truths (from the validation set) for a given cross validation fold. 

Decision tree
--------------

In this model, the training data is splito into successively smaller classes, in order to build up a set of binary rules that can be applied to other datasets. One ends up with leaves, at the end of each path through the tree. A tree with too many leaves is prone to overfitting, which will be reflected by bad cross validation scores. Therefore we scan over the maximum number of leaves, given by the parameter ``max_leaf_nodes``, in order to find the optimal number. We find that the best maximum number of leaves is 100, which also happens to be the default number for this estimator.

The best value that we find for the mean of the five cross validation scores using the training set is about 667 000 kr, a relatively large number. To get a better understanding of this number, we can divide by the mean of the prices in the training data. In this case we then get 0.194, i.e. on average the error in the predicted price is just below 20%. 


Random forest regressor
------------------------

Several randomly constructed decision trees are combined and averaged over, which typically gives a better estimation than a single tree.

Apart from the ``max_leaf_nodes`` parameter considered above for the deicision tree, we here consider also the ``n_estimators`` parameter, which gives the number of trees in the forest. We find that for both these parameters, the average of the cross validation scores tends to decrease as the parameters are increased. However, in both cases the curve flattens out at some value and is almost flat afterwards. We find that this happens for ``max_leaf_nodes`` of 1500 and ``n_estimators``  of 100. 

The best value we find for the average of the five cross validation scores for the random forest regressor is about 551 000 kr, which is about 0.160 when divided by the mean of the training data prices. 


Neural network
---------------

The number and size of the hidden layers in the neural network are important to determine how good the neural network regressor performs. We have considered a number of different choices for size of each layer and the depth of the network in order to find good numbers for these. In none of the cases we try, the neural network performs better than the random forest regressor, with scores ranging from extremely poor (<90%) to about 17.5% when divided by the average of the training data prices.

Gradient boosted tree regressor
---------------------------------

The XGBoost regressor is similar to a random forest regressor in that it combines an ensemble of decision trees to make stronger predictions, but different in how the training is performed. 

We first use the ``early_stopping_rounds`` parameter to find the best number for the ``n_estimators`` parameters (which plays the same role as for the random forest regressor). In this procedure we also optimise the ``learning_rate`` parameter, which is a step size parameter that can reduce overfitting. In the early stopping rounds procedure, one controls the training against a validation data set to check if predictions are improving when changing the earlier mentioned parameters. If the prediction on the validation set does not improve for the chosen value of ``early_stopping_rounds``, the training stops and the corresponding value for ``n_estimators`` is picked. In this procedure, we use 20% of the training data for validation, and the rest of the training data for training.

We loop over various learning rates and find that the value 0.13 of the learning rate is a good choice with a corresponding value of 246 for the value of ``n_estimators``. For this choice we get an average of 579 000 kr of the mean absolute error cross validation scores, or 0.169 when divided by the average of the training prices. 

Summary
---------

The best model in our comparison is the random forest regressor. The prediction is not very good however, reaching at best a mean absolute error of about 16% when divided by the average selling price. And in fact, all models except the decision tree reach about this value or just above for the error. Most likely the main improvement in this analysis would be to add more data. One could also think about the possibility to add more features, ideally features that are not very correlated with the ones used here, such as the average income in the area around an address, travel time with public transport to the city centre rather than just the distance or other interesting features. 

