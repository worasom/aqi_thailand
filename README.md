# Identifying the Sources of Winter Air Pollution in Bangkok

Mining and visualizing Bangkok air pollution data 


•      Mined publicly available weather and air pollution data - NASA’s fire map, Berkeley’s Earth air pollution, and traffic data (Requests, wget, selenium, BeautifulSoup libraries) Blog: https://medium.com/@worasom/scraping-air-pollution-data-from-thailand-epa-a866f291c06
    - scrape weather data [notebook](https://github.com/worasom/aqi_thailand/blob/master/scraping_weather_v3.ipynb)
    - scrape PM25 data from Berkeley Earth website [notebook](https://github.com/worasom/aqi_thailand/blob/master/webscraping-PM25.ipynb)
    - scrape pollution data from Thai EPA website [notebok](https://github.com/worasom/aqi_thailand/blob/master/scraping-AQI.ipynb)
•       Created visualization of how the air pollution relates to geographical locations, agricultural burning, weather patterns (matplot.pyplotlib, Bokeh libraries).  
Blog:  https://towardsdatascience.com/identifying-the-sources-of-winter-air-pollution-in-bangkok-part-i-d4392ea608dc

•   Made extensive use of feature engineering and cleaning: included influence of neighboring provinces with time lag, distance weighed fire map data, extracted feature from weather patterns, and date-time feature engineering. ([notebook](https://github.com/worasom/aqi_thailand/blob/master/pm25-ml2.ipynb)

•   Remove redundant feature using hierarchical clustering (scipy library) ([notebook](https://github.com/worasom/aqi_thailand/blob/master/pm25-ml2.ipynb)

•   Build a random forest regression model to identify  major contribution to PM2.5 particle ([notebook](https://github.com/worasom/aqi_thailand/blob/master/pm25-ml2.ipynb) 

•   Built a machine learning pipeline to identify the major contributors to PM2.5 air pollution: used autoML to find the best models and hyper parameters (TPOT library)([notebook](https://github.com/worasom/aqi_thailand/blob/master/Auto_TPOT.ipynb)
•   Achieve 68 - 73% R-squared on the validaton set and 99% R-squared for the training set.

•  Try with other approach: predict the pollution level using neural network [notebook](https://github.com/worasom/aqi_thailand/blob/master/pm25_NN.ipynb). Model the change in pollution level [notebook](https://github.com/worasom/aqi_thailand/blob/master/pm25_diff_model.ipynb)
