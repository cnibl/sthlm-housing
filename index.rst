.. Stockholm housing price prediction documentation master file, created by
   sphinx-quickstart on Tue Sep 17 23:47:13 2019.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Stockholm housing price predictions
===================================

In this project we look at the prices of sold apartments in the Stockholm area. The data is scraped from the web and then trimmed and engineered. We then begin by exploring the resulting dataset with some visualisations. Lastly we train regression models in order to predict the price of an apartment given its size, distance from city centre and other variables.

The main goal of this project is not to perform the most accurate prediction, but rather to go through how to get the data from the web with a web scraping tool and how to process the scraped data into something usable.

.. toctree::
   :maxdepth: 2
   :caption: Contents:

   data-retrieval
   data-exploration
   regression-analysis


.. Indices and tables
.. ==================
.. 
.. * :ref:`genindex`
.. * :ref:`modindex`
.. * :ref:`search`
