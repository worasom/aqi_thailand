# Identifying the Sources of Winter Air Pollution in Bangkok


![Optional Text](https://github.com/worasom/aqi_thailand/blob/master/readme/bkk_jan2019.png)

Air pollution is a serious environmental threat in many Asian countries. In Thailand, this issue has recently gained prominence due to the high levels of air pollution in Bangkok during the winter of 2019. Air pollution is reported through the air quality index (AQI), with higher values indicating worse air pollution. The main pollutant is PM2.5 particle. Being informed about the current outdoor air quality can help you to protect yourself.

Tourists and locals probably have questions like: Is air pollution getting worse? Is there a time of day or day of week dependence? What about provinces outside Bangkok? What are the sources of the pollutions? 

## Hypothesis:
The source is from local traffic. The cold winter air causes a temperature inversion effect that traps the pollution in Bangkok. This means the pollution will depends highly on the weather pattern and traffic.
The source is from seasonal agricultural burning. Bangkok is not an agricultural region, so it is unclear how much burning is going to affect Bangkok.

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

![Optional Text](https://github.com/worasom/aqi_thailand/blob/master/readme/fig1.png)


Let us start by looking for general trends. This data is from the Berkeley Earth database. The air pollution in Bangkok seems to be seasonal, with surges in the winter. It appears that this is not a new problem, but public awareness has only increased recently. The air quality in Bangkok is actually still better than many Asian countries.

    
If you’re concerned about air quality, as a tourist visiting Bangkok, which months should you avoid? As a resident, what is the best time to go for a jog? Is the air pollution better during the holiday when everyone is away? To answer these questions, I plotted the daily average for different months using data from 2017 and 2018.

![Optional Text](https://github.com/worasom/aqi_thailand/blob/master/readme/bkk_dailyavg.png)

We again see the seasonal trend of the PM2.5 pollution. The pollution starts around late October and lasts until early April for both 2017 and 2018. The air quality seems to be better during the New Year break in 2017, but data is missing in 2018, thus inconclusive. On average tourists should avoid Bangkok between late October and early April. You may be thinking, “But that’s when I have my holiday! Where else I can go?” May I suggest Phuket or Samui Island? The south of Thailand has low AQI all year long, as you can see in the map below.

With the start of the monsoon season, the weather in Bangkok will get better, and public attention on this issue will likely fade. However, without immediate policy measures, the problem will come back again next winter. In fact, the Bangkok region is not the only place that suffers from particle air pollution. Below, I show a map of particle pollution in different months.

![Optional Text](https://github.com/worasom/aqi_thailand/blob/master/readme/monthly_map.gif)

The seasonal pollution is not limited to the central region. While the air pollution in Bangkok peaked in January, the pollution moved to the northern provinces. In fact, their situation is much worse as judged by the maximum pollution level across the country shown below. Note that this is a plot of the maximum from monthly averages, to get rid of outliers.

![Optional Text](https://github.com/worasom/aqi_thailand/blob/master/readme/Thailand_max_monthavg.png)



