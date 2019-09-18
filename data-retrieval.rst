Data retrieval
==============

Web scraping
------------

There is ample data on the selling prices of apartments in Sweden on the Hemnet website. Here, we are interested in the prices in the Stockholm area, and in the end we want to predict the final price of an apartment given input variables such as size in square metres, monthly fee and so on. To be able to get as much data as possible from the website, we use a web scraping tool in this project, that extracts the data from the website source. 

The tool we use is the Python module Scrapy_. With Scrapy_, we create a "web crawler", that will go through a select set of web pages and extract certain data from them. On the website we have the following information:

.. _Scrapy: https://scrapy.readthedocs.io/en/latest/

- address
- selling price
- sell date
- price per square metre
- size in square metres
- number of rooms
- monthly fee
- price increase compared to announced price

A few of these are less interesting to us. The price per square metre can be obtained with the size and the price. The price increase can be an interesting target, but is not a good predictor if the target is the selling price, as it can be known only after the selling price is known (and hence can not be used to predict the price). The sell date may be of interest, but will probably just be related to an overall price increase over time, which needs to be handled. If ignored, we need to make sure that we do not accidentally take training data from one time period and have test data from another test period if there are significant price differences over time. Finally, the address is not so useful in itself, but can be used to get the coordinates and create a feature such as the distance from the city centre (which we will do, see below).

In our web crawler we use CSS selectors to extract the data from between the corresponding html tags. For example, the following code extracts the monthly fee for each address/location::

   for location in response.css("div.sold-property-listing"):
       
      #Extracting the content with css selectors
      ...
      fee = location.css(".sold-property-listing__size .sold-property-listing__fee::text").extract()
      ...

Similar code then extracts the other features listed above. 

The extracted data is typically formatted as raw strings. In the case of numerical data, this can include some unnecessary non-numerical parts, we then use regular expressions to format the extracted data. 

As an example we take the selling price. This is given as the string ``"Slutpris [num]\xa0[num]\xa0[num] kr"`` where the ``\xa0`` are a type of space that is used as a thousand-separator so that ``[num][num][num]`` is the actual numerical value representing the price in kr. For example a price could be given as the string ``"Slutpris 3\xa0430\xa0000"``, which should be interpreted as the price 3430000 kr. To get the numerical value and format it as a number in the csv file resulting from the web scraping, we use for the price the following function::

   def format_price(price):
      #Format the price in the input into a numerical format
      if len(price) == 0: #No price supplied as input
         formatted_price = ["NaN"]
         return formatted_price
      
      new_entry = re.sub(r"Slutpris\s+(\d*)\xa0(\d*)\xa0(\d*) kr", r"\1\2\3", entry)
      formatted_price = float(new_entry)
      return formatted_price
      
In the function, ``re.sub(regexp, replacement, string_to_search)`` is a function from the ``re`` module which searches for a regular expression ``regexp`` in the provided string ``string_to_search`` and, if found, replaces it with the string ``replacement``. The three ``(\d*)`` catch the numbers into groups 1, 2 and 3, which are then assigned to the string ``new_entry`` as ``r"\1\2\3"``. 

From the web scraping, we then obtain a csv file with the following columns:

- address (*string*)
- selling price (*float*)
- sell date (*string*
- price per square metre (*float*)
- size in square metres (*float*)
- number of rooms (*float*)
- monthly fee (*float*)
- price increase compared to announced price (*float*)

Data trimming and engineering
-----------------------------

Some of the features in the web scraping data are not very useful as they are. To remedy this, we here describe how some of them can be used to create new, more useful features. 

The address is an example of a variable that is not very useful. However, the address gives us the location of the apartment, which is likely to be a good predictor for the price (central apartments tend to be more expensive). 

To get the location of an apartment given its address, we use a geocoding module, which is used to get latitude and longitude values for a provided address. We here use GeoPy_, with the geocoder ArcGIS_ (a more well-known geocoder is the one using Google Maps, the free version of that is however too limited to be useful for us here). 

.. _GeoPy: https://geopy.readthedocs.io/en/stable/
.. _ArcGIS: https://www.arcgis.com/index.html

The below codeblock will go through all addresses and find their latitudes and longitudes. The addresses are contained in the ``"address"`` column of the data frame ``house_prices``. The ``geocode()`` function returns a ``location`` object, which has attributes corresponding to the latitude and longitude.::

   from geopy import geocoders #geocoding is used to convert address to (lat,long) coordinates
   from geopy.geocoders import ArcGIS
   
   ag = ArcGIS()
   lat = []
   long = []
   nan = float("NaN") #assign variable for NaN, faster than using float(nan) every time
   addresses = house_prices["address"]+", Stockholm, Sweden"
   for address in addresses:
      try:
         location = ag.geocode(address)
         lat.append(location.latitude)
         long.append(location.longitude)
      except:
         print("Warning: trouble with address:", address)
         lat.append(nan)
         long.append(nan)


A relevant variable for predicting the price is likely the distance from the city centre. The code below gives that distance for the above obtained coordinates (lat,long) of each address. We use this to create a new feature ``dist_city_centre`` in the data frame that will be used instead of latitude and longitude. The Haversine formula gives the distance between two latitude and longitude points on a sphere (in this case, we could probably just have used the Pythagorean theorem here since distances are relatively small and the Earth's curvature is unlikely to be of relevance)::

   def distance_to_city_centre(lat,long,lat_cc,long_cc):
      from math import cos, sin, asin, sqrt
      # Constants
      A = 2.*6371 #2*R_earth
      deg2rad = np.pi/180. #to convert degrees to radians
      
      # Use Haversine formula to get distance in kilometres
      a = sin((lat-lat_cc)*deg2rad/2.)**2
      b = cos(lat*deg2rad)*cos(lat_cc*deg2rad)
      c = sin((long-long_cc)*deg2rad/2)**2
      return A*asin(sqrt(a+b*c))
      
   sthlm_loc = ag.geocode("Stockholm")
   lat_cc, long_cc = sthlm_loc.latitude, sthlm_loc.longitude #coordinates of Sthlm city centre
   dist_city_centre = []
   
   for lat, long in zip(house_prices_coord["lat"], house_prices_coord["long"]):
      dist_city_centre.append(distance_to_city_centre(lat, long, lat_cc, long_cc))
       
   house_prices_dist = house_prices_coord.join(pd.Series(dist_city_centre, name="dist_city_centre")).copy() #new data frame containing the new feature

The last thing we do with the data is to handle the sell date and make it more usable. We convert it to a numerical variable which represents the number of days since the sale compared to today's date. To do this, we use that the selling date is formatted as ``"[dd] [month] ([yyyy])"`` where ``[dd]`` is a number representing the day of the month, the month is given as a string in ``[month]`` and the year is given by ``[yyyy]``. By using regular expressions together with Python's ``datetime`` module, we can then obtain the number of days between today's date and the sale date and put in a new feature.

We finally save the processed data to a new csv file. This is because the geocoding in particular takes a very long time when the number of addresses becomes large. We are limited by the fact that we are not paying for a faster geocoding service.