
# Scraping Air Pollution Data from Thailand EPA

Thailand's environment protection agency(EPA) makes air pollution data available through their [website](http://www.aqmthai.com/). However, obtaining bulk records by hand can be tedious. This notebook explains how to automatically scrape data from this website using selenium and beautiful soup library, which can be applied to any website with similar structure.


## Scrap data from a single station.

The website provides three ways of obtaining pollution data: the first tab provides an hourly pollution report for all measurement stations, the second tab is for historical data from a specific day and hour for all measurement stations, the third tab allows a batch request for historical data from a specific station. Data a month back from today is available.

<img src="data/aqi_front4.png" width="500">

I am going to show how to scrap data using the third tab, which involves the follow steps: (1) select the station number from the area and province you are interested in, (2) pick the time, (3) pick the parameters, (4) ask to display the table, save the data from the displayed table as html, and (5) click "Next" to show the rest of the table until all the data is scrapped.


```python
#import modules
from pathlib import Path
import requests
from bs4 import BeautifulSoup
from selenium import webdriver 
from selenium.webdriver.support.select import Select
import time

from datetime import datetime, date
```


```python
# use Firefox to open the website
browser = webdriver.Firefox()
url='http://www.aqmthai.com/public_report.php'
browser.get(url)
```

First, select a station ID. From inspecting the HTML element, we see that it is listed under a select tag with id="stationId."

<img src="data/aqi_station.png" width="500">


```python
#Select stationId
sta_name = "11t"
station = Select(browser.find_element_by_css_selector('select[id="stationId"]'))
station.select_by_value(sta_name)
```

Next, we select the time range we are interested in. I am interested in all data, which go back a month, so I am going to use the default date, and pick the time from midnight to 23:00 on that day. Then I pick the parameter that I am interested in. The tricky part is that each station has different parameters, so it is important to read the displayed parameters before selecting the parameters. We ask selenium to click the display table button, under the name```bt_show_table```. Sometimes the website response can be slow, so we wait 10 seconds after each step before continuing.


```python
#select time
start_hr = Select(browser.find_element_by_id('startHour'))
start_hr.select_by_index(0)
start_min = Select(browser.find_element_by_id('startMin'))
start_min.select_by_index(0)
stop_hr = Select(browser.find_element_by_id('endHour'))
stop_hr.select_by_index(23)
stop_min = Select(browser.find_element_by_id('endMin'))
stop_min.select_by_index(59)

#select parameters to display
param = Select(browser.find_element_by_id('parameterSelected'))
for i in range(16):
    param.select_by_index(i)
    
#Retrive data 
button = browser.find_element_by_name('bt_show_table')
button.click()
time.sleep(10)
```

Save the data as HTML and click the "Next" button to display more data and save these data. In the next section, I will show how to extract the data from the saved HTML files. Note that the data scraping can be done at the step without downloading the HTML file.


```python
#download webpage as html file for scraping 
page = browser.page_source
with open('web/page1.html','w', encoding='utf-8') as f:
    f.write(page)
```


```python
nums =[str(i) for i in range(2,9)]
print(nums)

# click the next button to display the next page. There are 7 more pages to click 
for num in nums:
    next_button = browser.find_element_by_name('bt_next_page')
    next_button.click()
    time.sleep(10)
    page = browser.page_source
    
    with open('web/page'+num+'.html','w', encoding='utf-8') as f:
        f.write(page)  
```

## Extracting the data from HTML files

I use beautifulsoup to extract the data from the HTML files, and use pandas to assemble the data into a single table for exporting as a csv file. The data table in the HTML files is under the tag with id```table_mn_di```. The header is under the first tr tag and the data is in the following tr tag. 

<img src="data/aqi_table.png" width="500">

The header texts are string inside a children of the tr tag, so the ```soup_obj.stripped_strings``` command can easily extract all the values. For the data row, the measurement values are in the value attribute of the input tag. 



```python
from glob import glob
files = glob('web/*.html')
```


```python
with open('web/page1.html', encoding='utf-8') as f:
    result_soup = BeautifulSoup(f.read())
```


```python
def page_data(result_soup):
    ''' Take a beautifulsoup object, extract the table head, 
    extract air pollution data, and return a dataframe.
    '''
    # find  <div> with id = 'table_mn_div'
    table = result_soup.find_all(attrs = {'id':'table_mn_div'})[0]
    table = table.table.tbody
    # find  header <tr> tag
    print('get header text')
    head = table.find_all('tr')[0]
    head_text = [text for text in head.stripped_strings]
    # find all data rows <tr> tag
    print('get body data')
    body = table.find_all('tr')[1:]
    
    matrix = []
    matrix = np.hstack(head_text)

    for row in body:
        data_s = row.find_all('input')
        # the last <input> tag is empty
        if len(data_s) != 0:
            row_data = [data['value'] for data in data_s]
            matrix = np.vstack((matrix, row_data))
            
    print('build the data dataframe')
    page_df = pd.DataFrame(matrix[1:,:], columns=matrix[0,:])
    return page_df
```


```python
# test the function on a single file
with open('web/page1.html', encoding='utf-8') as f:
    result_soup = BeautifulSoup(f.read())
    
page_df = page_data(result_soup)
df = pd.DataFrame()
df = pd.concat([page_df,page_df])
```

    obtain header text
    obtain body data
    build a dataframe
    


```python
# create an empty data frame
df = pd.DataFrame()
```


```python
for file in files:
    with open(file, encoding='utf-8') as f:
        result_soup = BeautifulSoup(f.read())
    
    page_df = page_data(result_soup)
    df = pd.concat([df,page_df])
    
```

    obtain header text
    obtain body data
    build a dataframe
    obtain header text
    obtain body data
    build a dataframe
    obtain header text
    obtain body data
    build a dataframe
    obtain header text
    obtain body data
    build a dataframe
    obtain header text
    obtain body data
    build a dataframe
    obtain header text
    obtain body data
    build a dataframe
    obtain header text
    obtain body data
    build a dataframe
    obtain header text
    obtain body data
    build a dataframe
    


```python
df.head().T
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Date</th>
      <td>2019,01,17,00,00,00</td>
      <td>2019,01,17,01,00,00</td>
      <td>2019,01,17,02,00,00</td>
      <td>2019,01,17,03,00,00</td>
      <td>2019,01,17,04,00,00</td>
    </tr>
    <tr>
      <th>11t_CO (ppm)</th>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <th>11t_NO (ppb)</th>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <th>11t_NOX (ppb)</th>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <th>11t_NO2 (ppb)</th>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <th>11t_SO2 (ppb)</th>
      <td>3</td>
      <td>4</td>
      <td>3</td>
      <td>3</td>
      <td>3</td>
    </tr>
    <tr>
      <th>11t_THC (ppm)</th>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <th>11t_CH4 (ppm)</th>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <th>11t_THCNM (ppm)</th>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <th>11t_O3 (ppb)</th>
      <td>3</td>
      <td>6</td>
      <td>11</td>
      <td>18</td>
      <td>17</td>
    </tr>
    <tr>
      <th>11t_PM10 (ug/m3)</th>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <th>11t_WS (m/s)</th>
      <td>0.28</td>
      <td>0.20</td>
      <td>0.13</td>
      <td>0.18</td>
      <td>0.30</td>
    </tr>
    <tr>
      <th>11t_WD (Degree)</th>
      <td>10.42</td>
      <td>78.22</td>
      <td>56.39</td>
      <td>58.55</td>
      <td>84.87</td>
    </tr>
    <tr>
      <th>11t_TEMP (Degree)</th>
      <td>28.39</td>
      <td>23.96</td>
      <td>21.43</td>
      <td>26.81</td>
      <td>26.89</td>
    </tr>
    <tr>
      <th>11t_RH (%)</th>
      <td>51.29</td>
      <td>51.70</td>
      <td>53.26</td>
      <td>56.92</td>
      <td>56.35</td>
    </tr>
    <tr>
      <th>11t_SRAD (w/m2)</th>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <th>11t_NRAD (w/m2)</th>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <th>11t_BP (mmHG)</th>
      <td>756.77</td>
      <td>756.59</td>
      <td>756.09</td>
      <td>755.92</td>
      <td>756.14</td>
    </tr>
    <tr>
      <th>11t_SD (Degree)</th>
      <td>45.43</td>
      <td>52.96</td>
      <td>62.94</td>
      <td>55.29</td>
      <td>44.66</td>
    </tr>
    <tr>
      <th>11t_RAIN (m/s)</th>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
    </tr>
    <tr>
      <th>11t_PM2.5 (ug/m3)</th>
      <td>53</td>
      <td>48</td>
      <td>54</td>
      <td>59</td>
      <td>56</td>
    </tr>
  </tbody>
</table>
</div>




```python
# save the data as a csv file
df.to_csv('data/aqmthai.csv')
```

## Scraping data from multiple measurement stations

One might be interested in the air pollution in your own province or see the variation among stations in the same province.  Bangkok itself has five stations. Some stations are on busy streets and some are in the outskirt of the city. Perhaps the air pollution in the outskirt is less severe. With a small adjustment to the above code, one can obtain the air pollution data from the stations that you are interested in. 

First, start by opening the website using selenium webdriver:


```python
browser = webdriver.Firefox()
url='http://www.aqmthai.com/public_report.php'
browser.get(url)
```

Turn the code in the previous section into functions:


```python
def display_pages(sta_index):
    # select station
    station = Select(browser.find_element_by_css_selector('select[id="stationId"]'))
    station.select_by_index(sta_index)

    
    #select parameters to display
    param = Select(browser.find_element_by_id('parameterSelected'))
    #values = ['CO', 'O3', 'PM10', 'WS','WD', 'TEMP','SRAD','NRAD','RAIN', 'SD']
    time.sleep(10)
    
    # each staton has different parameter options 
    html = browser.page_source
    soup = BeautifulSoup(html)
    select = soup.find_all(attrs={'id':'parameterSelected'})[0]
    options = len(select.find_all('option'))
    print(options)
    
    #select parameters
    for i in range(options):
        param.select_by_index(i)
        
    #select time
    start_hr = Select(browser.find_element_by_id('startHour'))
    start_hr.select_by_index(0)
    start_min = Select(browser.find_element_by_id('startMin'))
    start_min.select_by_index(0)
    stop_hr = Select(browser.find_element_by_id('endHour'))
    stop_hr.select_by_index(23)
    stop_min = Select(browser.find_element_by_id('endMin'))
    stop_min.select_by_index(59)
    
    #Retrive data 
    button = browser.find_element_by_name('bt_show_table')
    button.click()
    time.sleep(10)
```


```python
def get_num_param():
    html = browser.page_source
    soup = BeautifulSoup(html)
    select = soup.find_all(attrs={'id':'parameterSelected'})[0]
    return len(select.find_all('option'))
```


```python
def go_through_page(sta_index):
    #download webpage as html file for scraping 
    page = browser.page_source
    # include the file name in the station index
    save_page(page, f'web2/{str(sta_index)}page1.html')
        
    nums =[str(i) for i in range(2,9)]
    # click the next button to display the next page. There are 7 more pages 
    for num in nums:
        next_button = browser.find_element_by_name('bt_next_page')
        next_button.click()
        time.sleep(5)
        page = browser.page_source
        save_page(page, f'web2/{str(sta_index)}page'+num+'.html')
```


```python
def save_page(page, filename):
    with open(filename,'w', encoding='utf-8') as f:
        f.write(page)
```

Select the station number by index. Here, I select all stations. 


```python
for sta_index in range(1,61):
    browser.get(url)
    display_pages(sta_index)
    go_through_page(sta_index)
    print('done with station', sta_index)
```

    10
    done with 1
    13
    done with 2
    13
    done with 3
    15
    done with 4
    20
    done with 5
    16
    done with 6
    17
    done with 7
    15
    done with 8
    14
    done with 9
    14
    done with 10
    19
    done with 11
    17
    done with 12
    16
    done with 13
    16
    done with 14
    10
    done with 15
    12
    done with 16
    13
    done with 17
    19
    done with 18
    26
    done with 19
    20
    done with 20
    21
    done with 21
    10
    done with 22
    15
    done with 23
    11
    done with 24
    17
    done with 25
    16
    done with 26
    17
    done with 27
    20
    done with 28
    12
    done with 29
    17
    done with 30
    17
    done with 31
    17
    done with 32
    16
    done with 33
    16
    done with 34
    14
    done with 35
    15
    done with 36
    10
    done with 37
    20
    done with 38
    12
    done with 39
    16
    done with 40
    13
    done with 41
    13
    done with 42
    12
    done with 43
    11
    done with 44
    10
    done with 45
    12
    done with 46
    13
    done with 47
    14
    done with 48
    14
    done with 49
    16
    done with 50
    16
    done with 51
    8
    done with 52
    8
    done with 53
    10
    done with 54
    34
    done with 55
    16
    done with 56
    12
    done with 57
    14
    done with 58
    14
    done with 59
    14
    done with 60
    

Done! To summarized, I show how to scrap air pollution data from Thailand EPA. I use selenium to select parameters and display data and beautifulsoup to scrap web content. Now, I can start working with the data. The next post is going be about visualization these data.
