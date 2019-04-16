# Identifying the Sources of Winter Air Pollution in Bangkok


![Air Pollution Map near Bangkok in January 2019](https://github.com/worasom/aqi_thailand/blob/master/readme/bkk_jan2019.png)

Air pollution is a serious environmental threat in many Asian countries. In Thailand, this issue has recently gained prominence due to the high levels of air pollution in Bangkok during the winter of 2019. Air pollution is reported through the air quality index (AQI), with higher values indicating worse air pollution. The main pollutant is PM2.5 particle. Being informed about the current outdoor air quality can help you to protect yourself.

Tourists and locals probably have questions like: Is air pollution getting worse? Is there a time of day or day of week dependence? What about provinces outside Bangkok? What are the sources of the pollutions? 

## Hypothesis:

1. The source is from local traffic. The cold winter air causes a temperature inversion effect that traps the pollution in Bangkok. This means the pollution will depends highly on the weather pattern and traffic.
2. The source is from seasonal agricultural burning. Bangkok is not an agricultural region, so it is unclear how much burning is going to affect Bangkok.
3. Pollution from other provinces or countries. Some NGOs blamed the pollution on near by power plants.

## Analysis Procedures

•     Mined publicly available weather and air pollution data - NASA’s fire map, Berkeley’s Earth air pollution, and traffic data (Requests, wget, selenium, BeautifulSoup libraries) 
    - Blog: https://medium.com/@worasom/scraping-air-pollution-data-from-thailand-epa-a866f291c06
    - scrape weather data [notebook](https://github.com/worasom/aqi_thailand/blob/master/scraping_weather_v3.ipynb)
    - scrape PM25 data from Berkeley Earth website [notebook](https://github.com/worasom/aqi_thailand/blob/master/webscraping-PM25.ipynb)
    - scrape pollution data from Thai EPA website [notebok](https://github.com/worasom/aqi_thailand/blob/master/scraping-AQI.ipynb)
    
•   Data Visualization and feature engineer:
    - Created visualization of how the air pollution relates to geographical locations, agricultural burning, weather patterns (matplot.pyplotlib, Bokeh libraries).  Blog:  https://towardsdatascience.com/identifying-the-sources-of-winter-air-pollution-in-bangkok-part-i-d4392ea608dc
    -  Made extensive use of feature engineering and cleaning: included influence of neighboring provinces with time lag, distance weighed fire map data, extracted feature from weather patterns, traffic index, and date-time feature engineering. ([notebook](https://github.com/worasom/aqi_thailand/blob/master/pm25-ml2.ipynb)

• Build a machine learning model

   - Experimented with different ML algorithms, use auto ML(TPOT), vector AR. Tried including different sets of features. Experiments with regression and classification. Tried to model the change instead of the raw value. Tried to model the daily average.
   - Built a machine learning pipeline to identify the major contributors to PM2.5 air pollution: used autoML to find the best models and hyper parameters (TPOT library)([notebook](https://github.com/worasom/aqi_thailand/blob/master/Auto_TPOT.ipynb) 
   - Remove redundant feature using hierarchical clustering (scipy library) 
   -  Try with other approach: predict the pollution level using neural network [notebook](https://github.com/worasom/aqi_thailand/blob/master/pm25_NN.ipynb). Model the change in pollution level [notebook](https://github.com/worasom/aqi_thailand/blob/master/pm25_diff_model.ipynb)

• Final model 
    - Use the hourly PM2.5 data (raw)
    - Extremely Randomized Trees [notebook](https://github.com/worasom/aqi_thailand/blob/master/pm25_insight-2.ipynb) and Random forrest regressor [notebook](https://github.com/worasom/aqi_thailand/blob/master/pm25-ml2.ipynb).
    - Train, validation, test split by time
    - Hyper parameter tuning and features selecting
    - Achieve 68 - 73% R-squared on the validaton set and 99% R-squared for the training

• Identify the air pollution sources using feature of importance
• Model Interpretation: use tree interpreter to visualize the contribution of each feature for specific cases



# Winter haze in Bangkok

This section on visulaizing the behavior of the air pollution in Bangkok. The original blog can be found in  https://towardsdatascience.com/identifying-the-sources-of-winter-air-pollution-in-bangkok-part-i-d4392ea608dc. 

![PM 2.5 pollution in Bangkok over time](https://github.com/worasom/aqi_thailand/blob/master/readme/fig1.png)


Let us start by looking for general trends. This data is from the Berkeley Earth database. The air pollution in Bangkok seems to be seasonal, with surges in the winter. It appears that this is not a new problem, but public awareness has only increased recently. The air quality in Bangkok is actually still better than many Asian countries.

    
If you’re concerned about air quality, as a tourist visiting Bangkok, which months should you avoid? As a resident, what is the best time to go for a jog? Is the air pollution better during the holiday when everyone is away? To answer these questions, I plotted the daily average for different months using data from 2017 and 2018.

![Daily average of PM2.5 pollution in 2017 and 2018](https://github.com/worasom/aqi_thailand/blob/master/readme/bkk_dailyavg.png)

We again see the seasonal trend of the PM2.5 pollution. The pollution starts around late October and lasts until early April for both 2017 and 2018. The air quality seems to be better during the New Year break in 2017, but data is missing in 2018, thus inconclusive. On average tourists should avoid Bangkok between late October and early April. You may be thinking, “But that’s when I have my holiday! Where else I can go?” May I suggest Phuket or Samui Island? The south of Thailand has low AQI all year long, as you can see in the map below.

With the start of the monsoon season, the weather in Bangkok will get better, and public attention on this issue will likely fade. However, without immediate policy measures, the problem will come back again next winter. In fact, the Bangkok region is not the only place that suffers from particle air pollution. Below, I show a map of particle pollution in different months.

![particle pollution map](https://github.com/worasom/aqi_thailand/blob/master/readme/monthly_map.gif)

The seasonal pollution is not limited to the central region. While the air pollution in Bangkok peaked in January, the pollution moved to the northern provinces. In fact, their situation is much worse as judged by the maximum pollution level across the country shown below. Note that this is a plot of the maximum from monthly averages, to get rid of outliers.

![maximum particle pollution in each province](https://github.com/worasom/aqi_thailand/blob/master/readme/Thailand_max_monthavg.png)


# High PM2.5, Who Are the Culprits ?

Blog: https://towardsdatascience.com/identifying-the-sources-of-winter-air-pollution-in-bangkok-part-ii-72539f9b767a

My analysis procedure is as follows: Build a machine learning model(ML) to predict the air pollution level in Bangkok using environmental factor such as weather, traffic index, and fire maps. Include date-time features such as local hour, and weekday versus weekend in the model to capture other effects from human activities. Identify dominant sources of pollution using the feature of importance provided by the ML model.

If the source of the pollution is local, then the AQI will depend on factors such as weather patterns (wind speed, humidity, average temperature), local traffic, and hour of day. If the pollution is from agricultural burning, the AQI will depend on active fires with some time lag to account for geographical separation. Fire activities are included based on the distance from Bangkok. On the other hand, if the pollution not correlated with the fire map, then the model should put more weight on weather patterns, such as wind direction and wind speed.


## Autoregression process

Because the particle pollution tend to linger in the atmosphere, the current value will depend on the previous value. This means that the previous level must be included in the model, but from how far back?  Using PACF suggests using the data with lag one hour, so the level with lag one is included in the model. 

![](https://github.com/worasom/aqi_thailand/blob/master/readme/fig2.png)


## Agricultural Burning 


Farmers in Southeast Asia pick January — March as their burning season. For the north and northeastern provinces in Thailand, these burning activities are large enough to make these provinces among the most polluted places in the world during this time. For Bangkok, one might argue that because the region is heavily industrial rather than agricultural, it may not be affected as much by agricultural burning. But this is not the case.

Because of the tiny size of PM 2.5 particles, they remain suspended in the atmosphere for prolonged periods and can travel over very long distances. From the weather data, the average wind speed is 10 km/hour. The reported PM 2.5 level is a rolling average over 24 hours. A rough estimate is that the current PM 2.5 reading may be from sources as far as 240 km away. The picture below shows the fire map measured by NASA’s satellites, indicative of agricultural burning, on Jan 8, 2018 and on Feb 8, 2018. The yellow circle indicates the area within 240 km of Bangkok. The number of fires on Jan 8, which has an acceptable level of pollution, is much lower than the number of fires on Feb 8, which has an unhealthy level of pollution.
![Fire spots from NASA’s satellites](https://github.com/worasom/aqi_thailand/blob/master/readme/firemap2018.png)

This is how I calculate the fire features
It obviously need to depend on the distance from Bangkok
As a rough estimate, if we use the average wind speed of 10 km/hour and PM 2.5 is a rolling average over 24 hours. We get 240 km as the unit of interest. 

1. Create another time column add time delta based on the distance from Bangkok using 10 km/hr wind speed
2. Count the amount of fire in three zones: 240 km and 240km - 480 km and 480 -720 km zone. Count the number of fire with in 24 hours period within that zone. 

Here is the fire spots data in 240 km zone. You can see that it has the seasonal pattern similar to the pollution data. 


![The number of fires aligns with spikes in PM 2.5 levels](https://github.com/worasom/aqi_thailand/blob/master/readme/hotspot_time.png)

## Weather Pattern

The temperature inversion effect often occurs during winter because the temperature is cooler near the ground. The hotter air on top traps the cool air from flowing. This stagnant atmospheric condition allows the PM 2.5 particles to remain suspended in the air for longer. On the other hand, higher humidity or rain will help remove particles from the atmosphere. This is one reason why in the past when the air pollution was high, the government has sprayed water in the air. Unfortunately, this mitigation does not appear to be effective, since the volume of water is minuscule compared to actual rain. How much influence does weather pattern have on air pollution? Let’s compare the weather in winter versus other seasons.

![compare the weather pattern in winter and other seasons](https://github.com/worasom/aqi_thailand/blob/master/readme/weather1.png)

Temperature, wind speed and humidity are all lower in winter, but not by a large amount. Now, let’s look at the relationship of each of these with the PM 2.5 level.

![Effect of temperature, wind speed, and humidity on PM 2.5 level in winter](https://github.com/worasom/aqi_thailand/blob/master/readme/weather2.png)


Temperature, wind speed and humidity are all lower in winter, but not by a large amount. Now, let’s look at the relationship of each of these with the PM 2.5 level.

![Effect of temperature, wind speed, and humidity on PM 2.5 level in winter](https://github.com/worasom/aqi_thailand/blob/master/readme/wind_nowind.png)

On windy days, the pollution is clearly better. The median of the distribution for PM 2.5 levels is lower on windy days compared to on days without wind.

In fact, the pollution level also depends on the wind direction, as seen in this plot. I selected only four major wind directions for simplicity.

![PM2.5 relationship with the wind direction in winter](https://github.com/worasom/aqi_thailand/blob/master/readme/wind.png)

On the days where the wind comes from the south, the pollution level is lower likely because the Thai gulf is to the south of Bangkok. The clean ocean wind improves the air quality. Wind from the other three directions pass overland. However, having any wind is better than the stagnant atmospheric conditions on calm days.

The shift in the median PM 2.5 level is smaller between rainy days and days with no rain. There are fewer rainy days during the winter season, so the data is somewhat noisy, but a difference can be observed in the cumulative density function.

![PM2.5 relationship with the wind direction in winter](https://github.com/worasom/aqi_thailand/blob/master/readme/rain_norain.png)

## Traffic Index 

One of the sources of PM 2.5 particles is car engine exhaust. While campaigning for more public transportation usage is in general good for the environment, the effectiveness toward reducing PM 2.5 pollution is unclear because it has low correlation with the PM2.5 level.

Before working on the data, some data are missing and seem to change scale. Filling the missing data using the previous year. 


![Clean up traffic data](https://github.com/worasom/aqi_thailand/blob/master/readme/traffic1.png)

![Clean up traffic data](https://github.com/worasom/aqi_thailand/blob/master/readme/traffic.png)

We have seen that PM 2.5 levels are related to the time of day. The pollution is lower around 3 pm, but remains high during the night time. When plotted against the traffic data, the relationship with the pollution level is very noisy. There does not seem to be a strong correlation. Including the time of day and weekday versus weekend information into the model might make the relationship more clear.



# Machine Learning Model

The picture below show a dedrogram of all input features calculated from Spearman correlation. The dendrogram helps to identify redundant features that can be removed from the model. The number of fires within various distances and the level of PM 2.5 are closely related. Other features are further away. I ended up using all of these features in the model.

![dendrogram of input features](https://github.com/worasom/aqi_thailand/blob/master/readme/dendogram.png)

 ## Training, validation, and test sets
 
 Time series data should not be randomly split since the model might learn from an example of data. This is how I split the data for training, validation and test set. 
The blue region are the training set. I will use this for hyper parameter tuning, test on the validation set, retrain the train+validation together and test on the test set. 

 
 ![time series split](https://github.com/worasom/aqi_thailand/blob/master/readme/timeseries.png)
 
 ## Model performances
 
I choose Extremely Randomized Trees. This is like a random forest with the threshold for splitting is random instead of choosing the best split in randomforest. 
After hyperparameter tuning, I look at the feature of importance based on the validation set. I wrote my own function to calculate feature of importance. The feature of importance is based on permuting the data and see how it affect the loss. 
Removed less important feature. I play around with the threshold a bit. Retrain the model. I got about 0.78 R2 on the validation set and 0.57 R2 on the test set.

This is the model performance on training, validation and test set. 

 ![](https://github.com/worasom/aqi_thailand/blob/master/readme/model_per.png)

This is the prediction value compared to the actual. 

 ![](https://github.com/worasom/aqi_thailand/blob/master/readme/model_per2.png)

Then I retrained the model using both training and validation set. It hast 0.57 R2 on the test set.

This is the final model performance on training, validation and test set. 

 ![](https://github.com/worasom/aqi_thailand/blob/master/readme/model_per3.png)

This is the final model prediction value compared to the actual. 

 ![](https://github.com/worasom/aqi_thailand/blob/master/readme/model_per4.png)
 
The feature of importance based on the training set. 

 ![feature of importance](https://github.com/worasom/aqi_thailand/blob/master/readme/importance2.png)
 

The fire features have the highest importances and followed by humidity, temperature and hour of day. 


# Model Interpretation 

The influence of each feature is illustrated below using a tree interpreter for the data on Jan 22, 2019 at 7 am with 68 PM 2.5 level

 ![Model Interpretation](https://github.com/worasom/aqi_thailand/blob/master/readme/model_inter.png)
 
 We start with the average value of 24. The PM 2.5 level for the previous hour was 63, thus the model adds a value 29.43. There were 402 fires within a 240 km radius, thus the model adds 3 to the pollution level. Humidity and Tempearture add intotal of 3. The morning rush hour (7 am) adds 0.5 to the model. These six top factors account for most of the total 63 predicted for the PM 2.5 level. The remaining features to the right are less important and thus increase the predicted pollution value less. The model still under estimate the pollution level.
 
  ![Model Interpretation](https://github.com/worasom/aqi_thailand/blob/master/readme/model_inter2.png)
  
  On a good day such as October 10, 2018 at 1 am the PM 2.5 level was 12. The pollution level in the hour before was low, thus the model subtracts a value of 11. There were few files, and the model substract 1.3. The weather and traffic were good. The combination of many factors results in a low predicted PM 2.5 level of 12.
  
 
# Conclusions and Future Work 
 

In summary, 
- Study effect of  weather, fire spots, traffic on air pollution.
- Use ExtraTreeRegressor to identify features most contribute to the air pollution level in Bangkok. 
- Most important feature are fires from agricultural burning in winter season.
- The model still achieve about 0.58 - 0.78 R-squared on the validation and test set. 

The fact that there are complex interaction amount features. The level of influence of the fire depends on the wind direction and wind speed suggest that using a neural network maybe a better approach. This will be the subject of future work. 

This analysis confirms the suspicion that many people have — agricultural burning is the root cause of PM 2.5 pollution in Thailand. Burning activities as far as 240 km away from Bangkok, an area which extends into Myanmar, and Cambodia, can cause air problems in Bangkok. Solving this problem will not be easy. It will require a collaborative international effort among the Southeast Asian countries.
 



