Exploring the data
==================

Features
--------

After retrieving and trimming the data, we have a csv file containing the following columns, with one row for each address:

- address (*string*)
- selling price (*float*)
- sell date (*string*
- price per square metre (*float*)
- size in square metres (*float*)
- number of rooms (*float*)
- monthly fee (*float*)
- price increase compared to announced price (*float*)
- days since sale (*float*)
- distance from city centre (*float*)

We begin by dividing the data into a training set and a test set, and will leave the test set untouched until the very last step. We use 20% of the data for testing, and hence 80% for training.


Target and predictors
---------------------

Out of the available data, we will consider the price as our target. Our predictors will be:

- size
- number of rooms
- monthly fee
- distance from city centre

We will not use the days since sale feature as a predictor, but it is still an interesting feature. Primarily we need to make sure that the price does not change too much over time. If that is the case, we need to somehow handle this in order to take the time dependence of the expected price into account. 

The remaining parameters are either not of interest to us or can not be used to predict the price.


Visualising the data
--------------------

To familiarise ourselves with the data, we here show some plots. First of all, we show below the position of the data points in latitude and longitude. The plot is centered on the Stockholm city centre. The color of each data point represents the price per square metre. We can see that, as expected, the price per square metre increases the closer the apartment is to the city centre. To get an understanding of the scale, the cluster at (long, lat) = (17.6, 59.2) represents Södertälje. The empty areas are mainly lakes and wooded areas.

.. image:: /_static/sales_latlong.pdf

We have mentioned that the price may change over time, something we are not currently handling. The below plot shows the median selling price as a function of the number of days since the sale. Ideally we would like this curve to be entirely flat, but we can see that the price shows some dependence on the selling date. The decrease around 50 days is the summer period, where few apartments are sold or bought, which apparently has the effect that the price goes down. 

.. image:: /_static/median_price_vs_selldate.pdf

To get an understanding of how the predictors and target correlate with each other we look at pairwise scatter plots. The plot below shows such plots for all combinations of predictors and target, with histograms of each corresponding feature along the diagonal. From this we can see that in particular size and fee have nice almost gaussian distributions. Price and distance to city centre have more skewed distribution with a tail extending towards the higher end values. We can also see that size and rooms are clearly correlated (not surprisingly). There is also a quite pronounced correlation between these and the monthly fee. The distance to city centre is not very correlated with either of these three variables, which seems reasonable. There is some correlation between the distance and the price however, which also is reasonable.

.. image:: /_static/pairplot_predictors_target.pdf

In terms of correlations, we are mainly interested in the correlations between the predictors and the target (although, if two predictors correlate very strongly, one could argue that using both doesn't provide more information than using only one, and one could possibly get by without one of them). Below we show a heatmap of the absolute value of the correlation matrix, which gives the values of how strong the correlations are in the above scatter plots. Darker colour means stronger correlation, either positive or negative. We can see that size, rooms and fee all correlate rather strongly with each other, as was visible in the above scatter plots. We will however still keep them as separate features.

.. image:: /_static/corrmatrix.pdf

In terms of correlations between target and predictors, we see in the table below that the size is mostly correlated with the price. Given the correlation between size and number of rooms, it is not so surprising that also rooms correlate strongly with price. The fee is more weakly correlated. There is a rather strong negative correlation between price and distance from city centre, which is not so surprising---more central apartments with smaller distance to the city centre will be more likely to sell at higher prices.

+------------------+---------------------+
| Predictor        |  Corr. with target  |
+==================+=====================+
| size             | 0.490572            |
+------------------+---------------------+
| rooms            | 0.432708            |
+------------------+---------------------+
| fee              | 0.165020            |
+------------------+---------------------+
| dist_city_centre | -0.337306           |
+------------------+---------------------+
