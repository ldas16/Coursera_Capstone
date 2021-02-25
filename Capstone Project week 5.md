# 1. Introduction

### An important Gym Franchise wants to establish gyms in Central America. The have made studies that show that gyms are trending in Central America. 
### To start, they have decided to inaugurate two gyms in Costa Rica and then expand through the rest of the countries of Central America.
### They want to know in wich cities it would be a great idea to stablish without having a great competition at the beginning.
### So they decided to look for cities that are the most populated and at the same time, places where gyms are not among the five most common places.

# 2. Data

### For this project I'm going to import data from the webpage: https://simplemaps.com/data/world-cities.
### This webpage shows the information of many countries, including: city, latitude, longitude, country, country abbreviation, state or province and population, among others.
### This webpage lets to download the information in a csv file. So I'm going to use pandas to read that csv file.
### Then I'm going to filter the data to use only the information of the available cities from Costa Rica as requested for this project.
### After that, I'm going to use Foursquare API to fetch all the information of the common venues in every city.
### Then, I will use K-means method to clusterize the cities and focus only in the most populated cities that are close to each other.
### Finally, in the most populated clusters, I will look for those two most populated cities where gyms are not among the five most common places, to recomend stablish the gyms in those cities.¶

# 3. Methodology

### First, the information was imported from a webpage and then it was read with pandas.
### Then the Foursquare API was used to extract information about the most common venues of the cities of interest.
### After that, K-means method was used to clusterize the cities and focus only in the most populated cities that are close to each other.
### Finally, the clusters were used to look for those two most populated cities where gyms are not among the five most common places, to recomend stablish the gyms in those cities.¶

#### Import Libraries


```python
import numpy as np # library to handle data in a vectorized manner

import pandas as pd # library for data analsysis
pd.set_option('display.max_columns', None)
pd.set_option('display.max_rows', None)

import json # library to handle JSON files

!conda install -c conda-forge geopy --yes # uncomment this line if you haven't completed the Foursquare API lab
from geopy.geocoders import Nominatim # convert an address into latitude and longitude values

import requests # library to handle requests
from pandas.io.json import json_normalize # tranform JSON file into a pandas dataframe

# Matplotlib and associated plotting modules
import matplotlib.cm as cm
import matplotlib.colors as colors

# import k-means from clustering stage
from sklearn.cluster import KMeans

#!conda install -c conda-forge folium=0.5.0 --yes # uncomment this line if you haven't completed the Foursquare API lab
import folium # map rendering library

print('Libraries imported.')
```

    Collecting package metadata (current_repodata.json): done
    Solving environment: done
    
    ## Package Plan ##
    
      environment location: /home/jupyterlab/conda/envs/python
    
      added / updated specs:
        - geopy
    
    
    The following packages will be downloaded:
    
        package                    |            build
        ---------------------------|-----------------
        certifi-2020.12.5          |   py36h5fab9bb_1         143 KB  conda-forge
        geographiclib-1.50         |             py_0          34 KB  conda-forge
        geopy-2.1.0                |     pyhd3deb0d_0          64 KB  conda-forge
        openssl-1.1.1j             |       h7f98852_0         2.1 MB  conda-forge
        ------------------------------------------------------------
                                               Total:         2.4 MB
    
    The following NEW packages will be INSTALLED:
    
      geographiclib      conda-forge/noarch::geographiclib-1.50-py_0
      geopy              conda-forge/noarch::geopy-2.1.0-pyhd3deb0d_0
    
    The following packages will be UPDATED:
    
      certifi                          2020.12.5-py36h5fab9bb_0 --> 2020.12.5-py36h5fab9bb_1
      openssl                                 1.1.1i-h7f98852_0 --> 1.1.1j-h7f98852_0
    
    
    
    Downloading and Extracting Packages
    geopy-2.1.0          | 64 KB     | ##################################### | 100% 
    openssl-1.1.1j       | 2.1 MB    | ##################################### | 100% 
    certifi-2020.12.5    | 143 KB    | ##################################### | 100% 
    geographiclib-1.50   | 34 KB     | ##################################### | 100% 
    Preparing transaction: done
    Verifying transaction: done
    Executing transaction: done
    Libraries imported.


#### Import Data

#### Import Data from webpage: https://simplemaps.com/data/world-cities


```python
df=pd.read_csv('worldcities.csv')
df.head()
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
      <th>city</th>
      <th>city_ascii</th>
      <th>lat</th>
      <th>lng</th>
      <th>country</th>
      <th>iso2</th>
      <th>iso3</th>
      <th>admin_name</th>
      <th>capital</th>
      <th>population</th>
      <th>id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Tokyo</td>
      <td>Tokyo</td>
      <td>35.6897</td>
      <td>139.6922</td>
      <td>Japan</td>
      <td>JP</td>
      <td>JPN</td>
      <td>Tōkyō</td>
      <td>primary</td>
      <td>37977000.0</td>
      <td>1392685764</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Jakarta</td>
      <td>Jakarta</td>
      <td>-6.2146</td>
      <td>106.8451</td>
      <td>Indonesia</td>
      <td>ID</td>
      <td>IDN</td>
      <td>Jakarta</td>
      <td>primary</td>
      <td>34540000.0</td>
      <td>1360771077</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Delhi</td>
      <td>Delhi</td>
      <td>28.6600</td>
      <td>77.2300</td>
      <td>India</td>
      <td>IN</td>
      <td>IND</td>
      <td>Delhi</td>
      <td>admin</td>
      <td>29617000.0</td>
      <td>1356872604</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Mumbai</td>
      <td>Mumbai</td>
      <td>18.9667</td>
      <td>72.8333</td>
      <td>India</td>
      <td>IN</td>
      <td>IND</td>
      <td>Mahārāshtra</td>
      <td>admin</td>
      <td>23355000.0</td>
      <td>1356226629</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Manila</td>
      <td>Manila</td>
      <td>14.5958</td>
      <td>120.9772</td>
      <td>Philippines</td>
      <td>PH</td>
      <td>PHL</td>
      <td>Manila</td>
      <td>primary</td>
      <td>23088000.0</td>
      <td>1608618140</td>
    </tr>
  </tbody>
</table>
</div>



#### Eliminate unnecessary columns


```python
df = df.drop(['iso2','iso3','id','city_ascii','capital'], 1)
df.head()
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
      <th>city</th>
      <th>lat</th>
      <th>lng</th>
      <th>country</th>
      <th>admin_name</th>
      <th>population</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Tokyo</td>
      <td>35.6897</td>
      <td>139.6922</td>
      <td>Japan</td>
      <td>Tōkyō</td>
      <td>37977000.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Jakarta</td>
      <td>-6.2146</td>
      <td>106.8451</td>
      <td>Indonesia</td>
      <td>Jakarta</td>
      <td>34540000.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Delhi</td>
      <td>28.6600</td>
      <td>77.2300</td>
      <td>India</td>
      <td>Delhi</td>
      <td>29617000.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Mumbai</td>
      <td>18.9667</td>
      <td>72.8333</td>
      <td>India</td>
      <td>Mahārāshtra</td>
      <td>23355000.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Manila</td>
      <td>14.5958</td>
      <td>120.9772</td>
      <td>Philippines</td>
      <td>Manila</td>
      <td>23088000.0</td>
    </tr>
  </tbody>
</table>
</div>



#### Rename columns


```python
df.columns = ['City','Latitude','Longitude','Country','Province','Population']
df.head()
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
      <th>City</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Country</th>
      <th>Province</th>
      <th>Population</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Tokyo</td>
      <td>35.6897</td>
      <td>139.6922</td>
      <td>Japan</td>
      <td>Tōkyō</td>
      <td>37977000.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Jakarta</td>
      <td>-6.2146</td>
      <td>106.8451</td>
      <td>Indonesia</td>
      <td>Jakarta</td>
      <td>34540000.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Delhi</td>
      <td>28.6600</td>
      <td>77.2300</td>
      <td>India</td>
      <td>Delhi</td>
      <td>29617000.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Mumbai</td>
      <td>18.9667</td>
      <td>72.8333</td>
      <td>India</td>
      <td>Mahārāshtra</td>
      <td>23355000.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Manila</td>
      <td>14.5958</td>
      <td>120.9772</td>
      <td>Philippines</td>
      <td>Manila</td>
      <td>23088000.0</td>
    </tr>
  </tbody>
</table>
</div>



#### Filter the table to obtain the information of the available cities from Costa Rica


```python
df2 = df[df['Country'] == 'Costa Rica'].reset_index(drop=True)
df2.head()
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
      <th>City</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Country</th>
      <th>Province</th>
      <th>Population</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>San José</td>
      <td>9.9333</td>
      <td>-84.0833</td>
      <td>Costa Rica</td>
      <td>San José</td>
      <td>288054.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Cartago</td>
      <td>9.8667</td>
      <td>-83.9167</td>
      <td>Costa Rica</td>
      <td>Cartago</td>
      <td>221733.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Puerto Limón</td>
      <td>10.0022</td>
      <td>-83.0840</td>
      <td>Costa Rica</td>
      <td>Limón</td>
      <td>61072.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Liberia</td>
      <td>10.6338</td>
      <td>-85.4333</td>
      <td>Costa Rica</td>
      <td>Guanacaste</td>
      <td>45380.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Alajuela</td>
      <td>10.0278</td>
      <td>-84.2041</td>
      <td>Costa Rica</td>
      <td>Alajuela</td>
      <td>42975.0</td>
    </tr>
  </tbody>
</table>
</div>



#### Examine number of cities available


```python
df2.shape
```




    (18, 6)



#### Get the geographical coordinates of San Jose, capital of Costa Rica


```python
address = 'San Jose, CR'

geolocator = Nominatim(user_agent="cr_explorer")
location = geolocator.geocode(address)
latitude = location.latitude
longitude = location.longitude
print('The geograpical coordinates of Costa Rica are {}, {}.'.format(latitude, longitude))
```

    The geograpical coordinates of Costa Rica are 9.9325427, -84.0795782.


#### Create map of Costa Rica using latitude and longitude values


```python

map_cr = folium.Map(location=[latitude, longitude], zoom_start=9)

# add markers to map
for lat, lng, label in zip(df2['Latitude'], df2['Longitude'], df2['City']):
    label = folium.Popup(label, parse_html=True)
    folium.CircleMarker(
        [lat, lng],
        radius=5,
        popup=label,
        color='blue',
        fill=True,
        fill_color='#3186cc',
        fill_opacity=0.7,
        parse_html=False).add_to(map_cr)  
    
map_cr
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgPHNjcmlwdD5MX1BSRUZFUl9DQU5WQVMgPSBmYWxzZTsgTF9OT19UT1VDSCA9IGZhbHNlOyBMX0RJU0FCTEVfM0QgPSBmYWxzZTs8L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2FqYXguZ29vZ2xlYXBpcy5jb20vYWpheC9saWJzL2pxdWVyeS8xLjExLjEvanF1ZXJ5Lm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvanMvYm9vdHN0cmFwLm1pbi5qcyI+PC9zY3JpcHQ+CiAgICA8c2NyaXB0IHNyYz0iaHR0cHM6Ly9jZG5qcy5jbG91ZGZsYXJlLmNvbS9hamF4L2xpYnMvTGVhZmxldC5hd2Vzb21lLW1hcmtlcnMvMi4wLjIvbGVhZmxldC5hd2Vzb21lLW1hcmtlcnMuanMiPjwvc2NyaXB0PgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2xlYWZsZXRAMS4yLjAvZGlzdC9sZWFmbGV0LmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9ib290c3RyYXAvMy4yLjAvY3NzL2Jvb3RzdHJhcC10aGVtZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vZm9udC1hd2Vzb21lLzQuNi4zL2Nzcy9mb250LWF3ZXNvbWUubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9yYXdnaXQuY29tL3B5dGhvbi12aXN1YWxpemF0aW9uL2ZvbGl1bS9tYXN0ZXIvZm9saXVtL3RlbXBsYXRlcy9sZWFmbGV0LmF3ZXNvbWUucm90YXRlLmNzcyIvPgogICAgPHN0eWxlPmh0bWwsIGJvZHkge3dpZHRoOiAxMDAlO2hlaWdodDogMTAwJTttYXJnaW46IDA7cGFkZGluZzogMDt9PC9zdHlsZT4KICAgIDxzdHlsZT4jbWFwIHtwb3NpdGlvbjphYnNvbHV0ZTt0b3A6MDtib3R0b206MDtyaWdodDowO2xlZnQ6MDt9PC9zdHlsZT4KICAgIAogICAgICAgICAgICA8c3R5bGU+ICNtYXBfZDBiZTRhZDJiYTAyNGFmM2IwMGY0NjgzYzgxM2I2YzEgewogICAgICAgICAgICAgICAgcG9zaXRpb24gOiByZWxhdGl2ZTsKICAgICAgICAgICAgICAgIHdpZHRoIDogMTAwLjAlOwogICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICBsZWZ0OiAwLjAlOwogICAgICAgICAgICAgICAgdG9wOiAwLjAlOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICA8L3N0eWxlPgogICAgICAgIAo8L2hlYWQ+Cjxib2R5PiAgICAKICAgIAogICAgICAgICAgICA8ZGl2IGNsYXNzPSJmb2xpdW0tbWFwIiBpZD0ibWFwX2QwYmU0YWQyYmEwMjRhZjNiMDBmNDY4M2M4MTNiNmMxIiA+PC9kaXY+CiAgICAgICAgCjwvYm9keT4KPHNjcmlwdD4gICAgCiAgICAKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGJvdW5kcyA9IG51bGw7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgdmFyIG1hcF9kMGJlNGFkMmJhMDI0YWYzYjAwZjQ2ODNjODEzYjZjMSA9IEwubWFwKAogICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgJ21hcF9kMGJlNGFkMmJhMDI0YWYzYjAwZjQ2ODNjODEzYjZjMScsCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB7Y2VudGVyOiBbOS45MzI1NDI3LC04NC4wNzk1NzgyXSwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIHpvb206IDksCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICBtYXhCb3VuZHM6IGJvdW5kcywKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIGxheWVyczogW10sCiAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICB3b3JsZENvcHlKdW1wOiBmYWxzZSwKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgIGNyczogTC5DUlMuRVBTRzM4NTcKICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgICAgfSk7CiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciB0aWxlX2xheWVyX2Y5NGU2MDVlODZiZDQ0NGZiOWU0OTEwMTU0NDRhYjE1ID0gTC50aWxlTGF5ZXIoCiAgICAgICAgICAgICAgICAnaHR0cHM6Ly97c30udGlsZS5vcGVuc3RyZWV0bWFwLm9yZy97en0ve3h9L3t5fS5wbmcnLAogICAgICAgICAgICAgICAgewogICJhdHRyaWJ1dGlvbiI6IG51bGwsCiAgImRldGVjdFJldGluYSI6IGZhbHNlLAogICJtYXhab29tIjogMTgsCiAgIm1pblpvb20iOiAxLAogICJub1dyYXAiOiBmYWxzZSwKICAic3ViZG9tYWlucyI6ICJhYmMiCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2QwYmU0YWQyYmEwMjRhZjNiMDBmNDY4M2M4MTNiNmMxKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9lY2YzYzY3M2UxNmY0NmU3YTBlMzFhZDBjMzg5YjA5NyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzkuOTMzMywtODQuMDgzM10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9kMGJlNGFkMmJhMDI0YWYzYjAwZjQ2ODNjODEzYjZjMSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF85YjVmMWVlYTljZjY0MTk4OGUxZThhODBkZTUwMjg5MiA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9iNDMxNjY5MzUxMjI0ZDYyODNmYjQ3YmJhNjg4MjRiNCA9ICQoJzxkaXYgaWQ9Imh0bWxfYjQzMTY2OTM1MTIyNGQ2MjgzZmI0N2JiYTY4ODI0YjQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlNhbiBKb3PDqTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfOWI1ZjFlZWE5Y2Y2NDE5ODhlMWU4YTgwZGU1MDI4OTIuc2V0Q29udGVudChodG1sX2I0MzE2NjkzNTEyMjRkNjI4M2ZiNDdiYmE2ODgyNGI0KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2VjZjNjNjczZTE2ZjQ2ZTdhMGUzMWFkMGMzODliMDk3LmJpbmRQb3B1cChwb3B1cF85YjVmMWVlYTljZjY0MTk4OGUxZThhODBkZTUwMjg5Mik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl81YjIyYzc1MDg0Nzc0NjMxOTU4ZGMyOGEzZTU4NDEwYSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzkuODY2NywtODMuOTE2N10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9kMGJlNGFkMmJhMDI0YWYzYjAwZjQ2ODNjODEzYjZjMSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF9jYmVjZDhkYmVmMzA0NTIwOGU5NmQ4M2YwODk4MzZmMyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF80MzJiZTc5NTVmNmQ0YjQxYTQwNGVmZDU5MmZkZTA1NSA9ICQoJzxkaXYgaWQ9Imh0bWxfNDMyYmU3OTU1ZjZkNGI0MWE0MDRlZmQ1OTJmZGUwNTUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNhcnRhZ288L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2NiZWNkOGRiZWYzMDQ1MjA4ZTk2ZDgzZjA4OTgzNmYzLnNldENvbnRlbnQoaHRtbF80MzJiZTc5NTVmNmQ0YjQxYTQwNGVmZDU5MmZkZTA1NSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl81YjIyYzc1MDg0Nzc0NjMxOTU4ZGMyOGEzZTU4NDEwYS5iaW5kUG9wdXAocG9wdXBfY2JlY2Q4ZGJlZjMwNDUyMDhlOTZkODNmMDg5ODM2ZjMpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYjA1ZTJkMzZjYTBiNDcyMzhmN2EwNmZiZDFlNjI3NTIgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFsxMC4wMDIyLC04My4wODRdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfZDBiZTRhZDJiYTAyNGFmM2IwMGY0NjgzYzgxM2I2YzEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYTdkZDZlNmI0MTA3NDQxMDhkZGVmOWQ1N2Q3MzlmMzMgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNzRhNmRhYjAxOGI3NDZjNGE3ZGExMGMzOWE5Yzc4M2EgPSAkKCc8ZGl2IGlkPSJodG1sXzc0YTZkYWIwMThiNzQ2YzRhN2RhMTBjMzlhOWM3ODNhIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5QdWVydG8gTGltw7NuPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9hN2RkNmU2YjQxMDc0NDEwOGRkZWY5ZDU3ZDczOWYzMy5zZXRDb250ZW50KGh0bWxfNzRhNmRhYjAxOGI3NDZjNGE3ZGExMGMzOWE5Yzc4M2EpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfYjA1ZTJkMzZjYTBiNDcyMzhmN2EwNmZiZDFlNjI3NTIuYmluZFBvcHVwKHBvcHVwX2E3ZGQ2ZTZiNDEwNzQ0MTA4ZGRlZjlkNTdkNzM5ZjMzKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2ZmMDZjYTNlMGEyYTRmZTJiMWIzYTRlNTcyZTQ3Yjg1ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbMTAuNjMzOCwtODUuNDMzM10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9kMGJlNGFkMmJhMDI0YWYzYjAwZjQ2ODNjODEzYjZjMSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF83NTk0MmViZDk5MmE0OWRmOTFiYTcyOGE3NmRjMjY3OSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9kNTQ2NTc2M2EwZWY0ZGZjYmE5ZDczNDg2NmQzOTViOSA9ICQoJzxkaXYgaWQ9Imh0bWxfZDU0NjU3NjNhMGVmNGRmY2JhOWQ3MzQ4NjZkMzk1YjkiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkxpYmVyaWE8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzc1OTQyZWJkOTkyYTQ5ZGY5MWJhNzI4YTc2ZGMyNjc5LnNldENvbnRlbnQoaHRtbF9kNTQ2NTc2M2EwZWY0ZGZjYmE5ZDczNDg2NmQzOTViOSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9mZjA2Y2EzZTBhMmE0ZmUyYjFiM2E0ZTU3MmU0N2I4NS5iaW5kUG9wdXAocG9wdXBfNzU5NDJlYmQ5OTJhNDlkZjkxYmE3MjhhNzZkYzI2NzkpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMWY1ZDJjYjE5MjUyNGM4ZGFlMGMzM2QxOGIyZDU5MDkgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFsxMC4wMjc4LC04NC4yMDQxXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2QwYmU0YWQyYmEwMjRhZjNiMDBmNDY4M2M4MTNiNmMxKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2UwMDA5NjBiNTI4NzRmMDliOTIyYTI5NTEyOTM1NWUzID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2IxMzJjMGNhNjFiOTRlMTU4ZmFjOWFmNjYwNDVlNzZkID0gJCgnPGRpdiBpZD0iaHRtbF9iMTMyYzBjYTYxYjk0ZTE1OGZhYzlhZjY2MDQ1ZTc2ZCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QWxhanVlbGE8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2UwMDA5NjBiNTI4NzRmMDliOTIyYTI5NTEyOTM1NWUzLnNldENvbnRlbnQoaHRtbF9iMTMyYzBjYTYxYjk0ZTE1OGZhYzlhZjY2MDQ1ZTc2ZCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl8xZjVkMmNiMTkyNTI0YzhkYWUwYzMzZDE4YjJkNTkwOS5iaW5kUG9wdXAocG9wdXBfZTAwMDk2MGI1Mjg3NGYwOWI5MjJhMjk1MTI5MzU1ZTMpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfM2U2ODhiYzFlYjllNDc5MDg2YzkwOTUwYzZkY2IzNmMgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs5Ljk3NjQsLTg0LjgzMzldLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfZDBiZTRhZDJiYTAyNGFmM2IwMGY0NjgzYzgxM2I2YzEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfZmM0MmVjNGY4MWU3NDA1ZjljMWJmMjVlMDI2MDgxYTAgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfNGIzMmI0NzdjNTJmNGVjZWE3NzBkNmZiYjQ0OGE2YjcgPSAkKCc8ZGl2IGlkPSJodG1sXzRiMzJiNDc3YzUyZjRlY2VhNzcwZDZmYmI0NDhhNmI3IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5QdW50YXJlbmFzPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF9mYzQyZWM0ZjgxZTc0MDVmOWMxYmYyNWUwMjYwODFhMC5zZXRDb250ZW50KGh0bWxfNGIzMmI0NzdjNTJmNGVjZWE3NzBkNmZiYjQ0OGE2YjcpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfM2U2ODhiYzFlYjllNDc5MDg2YzkwOTUwYzZkY2IzNmMuYmluZFBvcHVwKHBvcHVwX2ZjNDJlYzRmODFlNzQwNWY5YzFiZjI1ZTAyNjA4MWEwKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2EwNDMxODVlYzE0ZjRlNGU5N2FmYTFiY2VkODE2NGJjID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbOS45NjA5LC04NC4wNzMxXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2QwYmU0YWQyYmEwMjRhZjNiMDBmNDY4M2M4MTNiNmMxKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2VmYTM1OTIzYWQ0OTQ3OTg5YzRhZDg3NWQxNTgxYmEwID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2U1ODkwYzk1N2VhYzQyNTZhNDQ3NTZkNDY1NDY4N2UzID0gJCgnPGRpdiBpZD0iaHRtbF9lNTg5MGM5NTdlYWM0MjU2YTQ0NzU2ZDQ2NTQ2ODdlMyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U2FuIEp1YW48L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwX2VmYTM1OTIzYWQ0OTQ3OTg5YzRhZDg3NWQxNTgxYmEwLnNldENvbnRlbnQoaHRtbF9lNTg5MGM5NTdlYWM0MjU2YTQ0NzU2ZDQ2NTQ2ODdlMyk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9hMDQzMTg1ZWMxNGY0ZTRlOTdhZmExYmNlZDgxNjRiYy5iaW5kUG9wdXAocG9wdXBfZWZhMzU5MjNhZDQ5NDc5ODljNGFkODc1ZDE1ODFiYTApOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfOTZlOThjZWFhMzllNGQ1ZjgwMzg5ZTgzODVjNmViNTMgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs5Ljk5ODUsLTg0LjExNjldLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfZDBiZTRhZDJiYTAyNGFmM2IwMGY0NjgzYzgxM2I2YzEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNTcyNzZmMTM5MzQxNDRjYjk1YWNlMzI4YmVhZjgzNzMgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMTlmMTY2ZWUyZmU1NDkzYWE3MzM0NThlNjlhNTNkNDMgPSAkKCc8ZGl2IGlkPSJodG1sXzE5ZjE2NmVlMmZlNTQ5M2FhNzMzNDU4ZTY5YTUzZDQzIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5IZXJlZGlhPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF81NzI3NmYxMzkzNDE0NGNiOTVhY2UzMjhiZWFmODM3My5zZXRDb250ZW50KGh0bWxfMTlmMTY2ZWUyZmU1NDkzYWE3MzM0NThlNjlhNTNkNDMpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfOTZlOThjZWFhMzllNGQ1ZjgwMzg5ZTgzODVjNmViNTMuYmluZFBvcHVwKHBvcHVwXzU3Mjc2ZjEzOTM0MTQ0Y2I5NWFjZTMyOGJlYWY4MzczKTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzEwYjA2NTU4N2VlMTQ0Nzk4YzNiMDBjNmZlMmRmOGJjID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbOS45MzIsLTg0LjE3Nl0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9kMGJlNGFkMmJhMDI0YWYzYjAwZjQ2ODNjODEzYjZjMSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF80ZTY4ZjQwNDBiZDA0OWRhYmUyM2NlNGQxNGJmZmRkMyA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9lOWJlMTY4NzA1Nzc0N2VjYmM0YTBjNzIwZGEzYjQ4NCA9ICQoJzxkaXYgaWQ9Imh0bWxfZTliZTE2ODcwNTc3NDdlY2JjNGEwYzcyMGRhM2I0ODQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlNhbnRhIEFuYTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNGU2OGY0MDQwYmQwNDlkYWJlMjNjZTRkMTRiZmZkZDMuc2V0Q29udGVudChodG1sX2U5YmUxNjg3MDU3NzQ3ZWNiYzRhMGM3MjBkYTNiNDg0KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzEwYjA2NTU4N2VlMTQ0Nzk4YzNiMDBjNmZlMmRmOGJjLmJpbmRQb3B1cChwb3B1cF80ZTY4ZjQwNDBiZDA0OWRhYmUyM2NlNGQxNGJmZmRkMyk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9lNGQyOWMzMDdhNjc0NWNjYThlY2YyM2RjOTA5YzUwZCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzkuMTQ5NywtODMuMzMzNF0sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9kMGJlNGFkMmJhMDI0YWYzYjAwZjQ2ODNjODEzYjZjMSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF85MjYwMjM1NzVlNTk0ZTY3YTU5YmQwY2Y4OTliZGMzMCA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9jYmI0Y2E3NzIwNzg0YzFiOTZkNjcxNGNiZTJiYWZkNiA9ICQoJzxkaXYgaWQ9Imh0bWxfY2JiNGNhNzcyMDc4NGMxYjk2ZDY3MTRjYmUyYmFmZDYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJ1ZW5vcyBBaXJlczwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfOTI2MDIzNTc1ZTU5NGU2N2E1OWJkMGNmODk5YmRjMzAuc2V0Q29udGVudChodG1sX2NiYjRjYTc3MjA3ODRjMWI5NmQ2NzE0Y2JlMmJhZmQ2KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2U0ZDI5YzMwN2E2NzQ1Y2NhOGVjZjIzZGM5MDljNTBkLmJpbmRQb3B1cChwb3B1cF85MjYwMjM1NzVlNTk0ZTY3YTU5YmQwY2Y4OTliZGMzMCk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9jMWVkMTZjZDcyMDY0NWZjYWQ0ZTUwM2M0NDdkMDBkNCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzEwLjMzMDUsLTg0LjQ0XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2QwYmU0YWQyYmEwMjRhZjNiMDBmNDY4M2M4MTNiNmMxKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzcwM2UzZDY1OWQ2ODQ1MjBhNDdjYzA5NDQ3ZGQ3MzViID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzA2MThjMzNjODA1YTQ2ODBiMGNjZjc3ZTZlNmEzM2IzID0gJCgnPGRpdiBpZD0iaHRtbF8wNjE4YzMzYzgwNWE0NjgwYjBjY2Y3N2U2ZTZhMzNiMyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UXVlc2FkYTwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNzAzZTNkNjU5ZDY4NDUyMGE0N2NjMDk0NDdkZDczNWIuc2V0Q29udGVudChodG1sXzA2MThjMzNjODA1YTQ2ODBiMGNjZjc3ZTZlNmEzM2IzKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2MxZWQxNmNkNzIwNjQ1ZmNhZDRlNTAzYzQ0N2QwMGQ0LmJpbmRQb3B1cChwb3B1cF83MDNlM2Q2NTlkNjg0NTIwYTQ3Y2MwOTQ0N2RkNzM1Yik7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8yZGM3YjgwOWM2Mzc0NjVmYmE4YmQwZDY5MmRkNTBlZiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzEwLjQzLC04NS4xXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2QwYmU0YWQyYmEwMjRhZjNiMDBmNDY4M2M4MTNiNmMxKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzcxNGI2ZTMwMWY2YjQzOTdhMDRjZTk3YTUzODUxNWY0ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzQxZmJkYzA5YzVjZTQ0OWRhZjg3Y2UwMGNmNmJjNTRlID0gJCgnPGRpdiBpZD0iaHRtbF80MWZiZGMwOWM1Y2U0NDlkYWY4N2NlMDBjZjZiYzU0ZSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q2HDsWFzPC9kaXY+JylbMF07CiAgICAgICAgICAgICAgICBwb3B1cF83MTRiNmUzMDFmNmI0Mzk3YTA0Y2U5N2E1Mzg1MTVmNC5zZXRDb250ZW50KGh0bWxfNDFmYmRjMDljNWNlNDQ5ZGFmODdjZTAwY2Y2YmM1NGUpOwogICAgICAgICAgICAKCiAgICAgICAgICAgIGNpcmNsZV9tYXJrZXJfMmRjN2I4MDljNjM3NDY1ZmJhOGJkMGQ2OTJkZDUwZWYuYmluZFBvcHVwKHBvcHVwXzcxNGI2ZTMwMWY2YjQzOTdhMDRjZTk3YTUzODUxNWY0KTsKCiAgICAgICAgICAgIAogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2Y5NWJlNmFlNTE4YjQyZmRhNTczYTg3OGQ4NTFmMTQyID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbOS45NzcxLC04NC43NDQzXSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2QwYmU0YWQyYmEwMjRhZjNiMDBmNDY4M2M4MTNiNmMxKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzgyOWIyMWM5MWUyMzQzNGE5NDY5NDkzZmJhOTk0Y2YxID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2QyOTBhODVjZTcyYjRkYzQ5OGQwY2UwNTZiZTA0YzQ0ID0gJCgnPGRpdiBpZD0iaHRtbF9kMjkwYTg1Y2U3MmI0ZGM0OThkMGNlMDU2YmUwNGM0NCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RWwgUm9ibGU8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzgyOWIyMWM5MWUyMzQzNGE5NDY5NDkzZmJhOTk0Y2YxLnNldENvbnRlbnQoaHRtbF9kMjkwYTg1Y2U3MmI0ZGM0OThkMGNlMDU2YmUwNGM0NCk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9mOTViZTZhZTUxOGI0MmZkYTU3M2E4NzhkODUxZjE0Mi5iaW5kUG9wdXAocG9wdXBfODI5YjIxYzkxZTIzNDM0YTk0Njk0OTNmYmE5OTRjZjEpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfOTY1YjE4ODZhZmNlNGE3ZTkxMTM5YzliMDYzNTFjMjMgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs5LjgyOTEsLTg0LjMwNDRdLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfZDBiZTRhZDJiYTAyNGFmM2IwMGY0NjgzYzgxM2I2YzEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfYjkyM2I1Nzk3YzhjNDRkMDljZmEwZWIwYzVlNWU1ZTUgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfZWZiOGMzOWZmZjI5NDkwMTlhZGM2NzEzMzk3OTllYTUgPSAkKCc8ZGl2IGlkPSJodG1sX2VmYjhjMzlmZmYyOTQ5MDE5YWRjNjcxMzM5Nzk5ZWE1IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5TYW50aWFnbzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYjkyM2I1Nzk3YzhjNDRkMDljZmEwZWIwYzVlNWU1ZTUuc2V0Q29udGVudChodG1sX2VmYjhjMzlmZmYyOTQ5MDE5YWRjNjcxMzM5Nzk5ZWE1KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzk2NWIxODg2YWZjZTRhN2U5MTEzOWM5YjA2MzUxYzIzLmJpbmRQb3B1cChwb3B1cF9iOTIzYjU3OTdjOGM0NGQwOWNmYTBlYjBjNWU1ZTVlNSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9iNTM0YjA1NTE0Y2Y0Y2RjYjlmMDEyMGVkNjQ1NmJlMiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzkuNTA4MywtODIuNjE0N10sCiAgICAgICAgICAgICAgICB7CiAgImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLAogICJjb2xvciI6ICJibHVlIiwKICAiZGFzaEFycmF5IjogbnVsbCwKICAiZGFzaE9mZnNldCI6IG51bGwsCiAgImZpbGwiOiB0cnVlLAogICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsCiAgImZpbGxPcGFjaXR5IjogMC43LAogICJmaWxsUnVsZSI6ICJldmVub2RkIiwKICAibGluZUNhcCI6ICJyb3VuZCIsCiAgImxpbmVKb2luIjogInJvdW5kIiwKICAib3BhY2l0eSI6IDEuMCwKICAicmFkaXVzIjogNSwKICAic3Ryb2tlIjogdHJ1ZSwKICAid2VpZ2h0IjogMwp9CiAgICAgICAgICAgICAgICApLmFkZFRvKG1hcF9kMGJlNGFkMmJhMDI0YWYzYjAwZjQ2ODNjODEzYjZjMSk7CiAgICAgICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBwb3B1cF81MjMyYTA5ZTI0NDQ0YWEzOWJmZmVhOTlmN2M3NDJkMSA9IEwucG9wdXAoe21heFdpZHRoOiAnMzAwJ30pOwoKICAgICAgICAgICAgCiAgICAgICAgICAgICAgICB2YXIgaHRtbF9iNWZkNDYzODY1ZmM0ZDE1OWNlMTkyOTljZTg1YzZlNSA9ICQoJzxkaXYgaWQ9Imh0bWxfYjVmZDQ2Mzg2NWZjNGQxNTljZTE5Mjk5Y2U4NWM2ZTUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlNpeGFvbGE8L2Rpdj4nKVswXTsKICAgICAgICAgICAgICAgIHBvcHVwXzUyMzJhMDllMjQ0NDRhYTM5YmZmZWE5OWY3Yzc0MmQxLnNldENvbnRlbnQoaHRtbF9iNWZkNDYzODY1ZmM0ZDE1OWNlMTkyOTljZTg1YzZlNSk7CiAgICAgICAgICAgIAoKICAgICAgICAgICAgY2lyY2xlX21hcmtlcl9iNTM0YjA1NTE0Y2Y0Y2RjYjlmMDEyMGVkNjQ1NmJlMi5iaW5kUG9wdXAocG9wdXBfNTIzMmEwOWUyNDQ0NGFhMzliZmZlYTk5ZjdjNzQyZDEpOwoKICAgICAgICAgICAgCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNjUwOGIxOGMxZjM1NGU0MGJkZjVjZjgyOGZhNzYzOTcgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFsxMS4wNzQyLC04NS42Mjk0XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2QwYmU0YWQyYmEwMjRhZjNiMDBmNDY4M2M4MTNiNmMxKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwX2IzZTgwOWQ4NTAzYjRmMzY4MTMxMmJhZmNjMmVlZjE1ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sX2YzZDBmMmFiODQ5ZjQ2ZGJiM2E5NmVlZTU4MjVkOTI2ID0gJCgnPGRpdiBpZD0iaHRtbF9mM2QwZjJhYjg0OWY0NmRiYjNhOTZlZWU1ODI1ZDkyNiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+TGEgQ3J1ejwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfYjNlODA5ZDg1MDNiNGYzNjgxMzEyYmFmY2MyZWVmMTUuc2V0Q29udGVudChodG1sX2YzZDBmMmFiODQ5ZjQ2ZGJiM2E5NmVlZTU4MjVkOTI2KTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyXzY1MDhiMThjMWYzNTRlNDBiZGY1Y2Y4MjhmYTc2Mzk3LmJpbmRQb3B1cChwb3B1cF9iM2U4MDlkODUwM2I0ZjM2ODEzMTJiYWZjYzJlZWYxNSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9mZDc0YmVmYWQ4Yjg0NDRkYWIwMmYzOGQwYjc3MjI5MCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzguNjUsLTgzLjE1XSwKICAgICAgICAgICAgICAgIHsKICAiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsCiAgImNvbG9yIjogImJsdWUiLAogICJkYXNoQXJyYXkiOiBudWxsLAogICJkYXNoT2Zmc2V0IjogbnVsbCwKICAiZmlsbCI6IHRydWUsCiAgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwKICAiZmlsbE9wYWNpdHkiOiAwLjcsCiAgImZpbGxSdWxlIjogImV2ZW5vZGQiLAogICJsaW5lQ2FwIjogInJvdW5kIiwKICAibGluZUpvaW4iOiAicm91bmQiLAogICJvcGFjaXR5IjogMS4wLAogICJyYWRpdXMiOiA1LAogICJzdHJva2UiOiB0cnVlLAogICJ3ZWlnaHQiOiAzCn0KICAgICAgICAgICAgICAgICkuYWRkVG8obWFwX2QwYmU0YWQyYmEwMjRhZjNiMDBmNDY4M2M4MTNiNmMxKTsKICAgICAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHBvcHVwXzE0Y2NiNzRmOGU1YzQ0M2RiMjIxNzg5M2JhOTZlN2I1ID0gTC5wb3B1cCh7bWF4V2lkdGg6ICczMDAnfSk7CgogICAgICAgICAgICAKICAgICAgICAgICAgICAgIHZhciBodG1sXzE1MGVkZjI1N2Y5MTRhNzI4NjRiOGY0NzZlY2I5ZjJhID0gJCgnPGRpdiBpZD0iaHRtbF8xNTBlZGYyNTdmOTE0YTcyODY0YjhmNDc2ZWNiOWYyYSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+R29sZml0bzwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfMTRjY2I3NGY4ZTVjNDQzZGIyMjE3ODkzYmE5NmU3YjUuc2V0Q29udGVudChodG1sXzE1MGVkZjI1N2Y5MTRhNzI4NjRiOGY0NzZlY2I5ZjJhKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2ZkNzRiZWZhZDhiODQ0NGRhYjAyZjM4ZDBiNzcyMjkwLmJpbmRQb3B1cChwb3B1cF8xNGNjYjc0ZjhlNWM0NDNkYjIyMTc4OTNiYTk2ZTdiNSk7CgogICAgICAgICAgICAKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9kMzcwYjcyNDVlYzQ0NTI4YTdhYWU3ZjVjMjM3ZDcxOCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzguOTYsLTgzLjUyMzldLAogICAgICAgICAgICAgICAgewogICJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwKICAiY29sb3IiOiAiYmx1ZSIsCiAgImRhc2hBcnJheSI6IG51bGwsCiAgImRhc2hPZmZzZXQiOiBudWxsLAogICJmaWxsIjogdHJ1ZSwKICAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLAogICJmaWxsT3BhY2l0eSI6IDAuNywKICAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsCiAgImxpbmVDYXAiOiAicm91bmQiLAogICJsaW5lSm9pbiI6ICJyb3VuZCIsCiAgIm9wYWNpdHkiOiAxLjAsCiAgInJhZGl1cyI6IDUsCiAgInN0cm9rZSI6IHRydWUsCiAgIndlaWdodCI6IDMKfQogICAgICAgICAgICAgICAgKS5hZGRUbyhtYXBfZDBiZTRhZDJiYTAyNGFmM2IwMGY0NjgzYzgxM2I2YzEpOwogICAgICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgcG9wdXBfNTg4OGM0YmI5YjAxNDQxOTk1YWUxODdiZTgxZjI3MWUgPSBMLnBvcHVwKHttYXhXaWR0aDogJzMwMCd9KTsKCiAgICAgICAgICAgIAogICAgICAgICAgICAgICAgdmFyIGh0bWxfMTgzMmE0NWQ4OWJlNDQyYjk4NzE5ZDY4MDNiNWVkYmYgPSAkKCc8ZGl2IGlkPSJodG1sXzE4MzJhNDVkODliZTQ0MmI5ODcxOWQ2ODAzYjVlZGJmIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DaXVkYWQgQ29ydMOpczwvZGl2PicpWzBdOwogICAgICAgICAgICAgICAgcG9wdXBfNTg4OGM0YmI5YjAxNDQxOTk1YWUxODdiZTgxZjI3MWUuc2V0Q29udGVudChodG1sXzE4MzJhNDVkODliZTQ0MmI5ODcxOWQ2ODAzYjVlZGJmKTsKICAgICAgICAgICAgCgogICAgICAgICAgICBjaXJjbGVfbWFya2VyX2QzNzBiNzI0NWVjNDQ1MjhhN2FhZTdmNWMyMzdkNzE4LmJpbmRQb3B1cChwb3B1cF81ODg4YzRiYjliMDE0NDE5OTVhZTE4N2JlODFmMjcxZSk7CgogICAgICAgICAgICAKICAgICAgICAKPC9zY3JpcHQ+ onload="this.contentDocument.open();this.contentDocument.write(atob(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



#### Define Foursquare Credentials and Version


```python
CLIENT_ID = '5OQC1AURHZFISWL3IKHBLWVJWLMZVE4IQ2LFERPPGEV4SJXG' # your Foursquare ID
CLIENT_SECRET = 'GAPAK1LZP3W4ZLAYDGHHYJD5KPLATRVLPGXHYQVNPHCKACSO' # your Foursquare Secret
VERSION = '20180605' # Foursquare API version
LIMIT = 100 # A default Foursquare API limit value

print('Your credentails:')
print('CLIENT_ID: ' + CLIENT_ID)
print('CLIENT_SECRET:' + CLIENT_SECRET)
```

    Your credentails:
    CLIENT_ID: 5OQC1AURHZFISWL3IKHBLWVJWLMZVE4IQ2LFERPPGEV4SJXG
    CLIENT_SECRET:GAPAK1LZP3W4ZLAYDGHHYJD5KPLATRVLPGXHYQVNPHCKACSO


#### Function to extract the information from all the cities


```python
def getNearbyVenues(names, latitudes, longitudes, radius=500):
    
    venues_list=[]
    for name, lat, lng in zip(names, latitudes, longitudes):
        print(name)
            
        # create the API request URL
        url = 'https://api.foursquare.com/v2/venues/explore?&client_id={}&client_secret={}&v={}&ll={},{}&radius={}&limit={}'.format(
            CLIENT_ID, 
            CLIENT_SECRET, 
            VERSION, 
            lat, 
            lng, 
            radius, 
            LIMIT)
            
        # make the GET request
        results = requests.get(url).json()["response"]['groups'][0]['items']
        
        # return only relevant information for each nearby venue
        venues_list.append([(
            name, 
            lat, 
            lng, 
            v['venue']['name'], 
            v['venue']['location']['lat'], 
            v['venue']['location']['lng'],  
            v['venue']['categories'][0]['name']) for v in results])

    nearby_venues = pd.DataFrame([item for venue_list in venues_list for item in venue_list])
    nearby_venues.columns = ['City', 
                  'City Latitude', 
                  'City Longitude', 
                  'Venue', 
                  'Venue Latitude', 
                  'Venue Longitude', 
                  'Venue Category']
    
    return(nearby_venues)
```

#### Code to run the above function on each city


```python
CR_venues = getNearbyVenues(names=df2['City'],
                                   latitudes=df2['Latitude'],
                                   longitudes=df2['Longitude']
                                  )
```

    San José
    Cartago
    Puerto Limón
    Liberia
    Alajuela
    Puntarenas
    San Juan
    Heredia
    Santa Ana
    Buenos Aires
    Quesada
    Cañas
    El Roble
    Santiago
    Sixaola
    La Cruz
    Golfito
    Ciudad Cortés


#### Check the size of the resulting dataframe


```python
print(CR_venues.shape)
CR_venues.head()
```

    (255, 7)





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
      <th>City</th>
      <th>City Latitude</th>
      <th>City Longitude</th>
      <th>Venue</th>
      <th>Venue Latitude</th>
      <th>Venue Longitude</th>
      <th>Venue Category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>San José</td>
      <td>9.9333</td>
      <td>-84.0833</td>
      <td>La Sorbetera de Lolo Mora</td>
      <td>9.934467</td>
      <td>-84.081841</td>
      <td>Ice Cream Shop</td>
    </tr>
    <tr>
      <th>1</th>
      <td>San José</td>
      <td>9.9333</td>
      <td>-84.0833</td>
      <td>Rincón Retana</td>
      <td>9.934561</td>
      <td>-84.082022</td>
      <td>Sandwich Place</td>
    </tr>
    <tr>
      <th>2</th>
      <td>San José</td>
      <td>9.9333</td>
      <td>-84.0833</td>
      <td>El Tostador</td>
      <td>9.934511</td>
      <td>-84.083321</td>
      <td>Café</td>
    </tr>
    <tr>
      <th>3</th>
      <td>San José</td>
      <td>9.9333</td>
      <td>-84.0833</td>
      <td>Mercado Central de San José</td>
      <td>9.934492</td>
      <td>-84.081830</td>
      <td>Market</td>
    </tr>
    <tr>
      <th>4</th>
      <td>San José</td>
      <td>9.9333</td>
      <td>-84.0833</td>
      <td>Soda Tala</td>
      <td>9.934671</td>
      <td>-84.081785</td>
      <td>Restaurant</td>
    </tr>
  </tbody>
</table>
</div>



#### Check how many venues were returned for each city


```python
CR_venues.groupby('City').count()
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
      <th>City Latitude</th>
      <th>City Longitude</th>
      <th>Venue</th>
      <th>Venue Latitude</th>
      <th>Venue Longitude</th>
      <th>Venue Category</th>
    </tr>
    <tr>
      <th>City</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Alajuela</th>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
    </tr>
    <tr>
      <th>Buenos Aires</th>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
    </tr>
    <tr>
      <th>Cartago</th>
      <td>29</td>
      <td>29</td>
      <td>29</td>
      <td>29</td>
      <td>29</td>
      <td>29</td>
    </tr>
    <tr>
      <th>Cañas</th>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>Ciudad Cortés</th>
      <td>6</td>
      <td>6</td>
      <td>6</td>
      <td>6</td>
      <td>6</td>
      <td>6</td>
    </tr>
    <tr>
      <th>El Roble</th>
      <td>5</td>
      <td>5</td>
      <td>5</td>
      <td>5</td>
      <td>5</td>
      <td>5</td>
    </tr>
    <tr>
      <th>Heredia</th>
      <td>28</td>
      <td>28</td>
      <td>28</td>
      <td>28</td>
      <td>28</td>
      <td>28</td>
    </tr>
    <tr>
      <th>La Cruz</th>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
      <td>4</td>
    </tr>
    <tr>
      <th>Liberia</th>
      <td>15</td>
      <td>15</td>
      <td>15</td>
      <td>15</td>
      <td>15</td>
      <td>15</td>
    </tr>
    <tr>
      <th>Puerto Limón</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>Puntarenas</th>
      <td>50</td>
      <td>50</td>
      <td>50</td>
      <td>50</td>
      <td>50</td>
      <td>50</td>
    </tr>
    <tr>
      <th>Quesada</th>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>San José</th>
      <td>42</td>
      <td>42</td>
      <td>42</td>
      <td>42</td>
      <td>42</td>
      <td>42</td>
    </tr>
    <tr>
      <th>San Juan</th>
      <td>38</td>
      <td>38</td>
      <td>38</td>
      <td>38</td>
      <td>38</td>
      <td>38</td>
    </tr>
    <tr>
      <th>Santa Ana</th>
      <td>24</td>
      <td>24</td>
      <td>24</td>
      <td>24</td>
      <td>24</td>
      <td>24</td>
    </tr>
    <tr>
      <th>Sixaola</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



#### Let's find out how many unique categories can be curated from all the returned venues


```python
print('There are {} uniques categories.'.format(len(CR_venues['Venue Category'].unique())))
```

    There are 88 uniques categories.


#### Analyze Each Neighborhood


```python
# one hot encoding
CR_onehot = pd.get_dummies(CR_venues[['Venue Category']], prefix="", prefix_sep="")

# add city column back to dataframe
CR_onehot['City'] = CR_venues['City'] 

# move city column to the first column
fixed_columns = [CR_onehot.columns[-1]] + list(CR_onehot.columns[:-1])
CR_onehot = CR_onehot[fixed_columns]

CR_onehot.head()
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
      <th>City</th>
      <th>American Restaurant</th>
      <th>Art Gallery</th>
      <th>Arts &amp; Crafts Store</th>
      <th>Asian Restaurant</th>
      <th>Athletics &amp; Sports</th>
      <th>Auto Garage</th>
      <th>Bakery</th>
      <th>Bar</th>
      <th>Bed &amp; Breakfast</th>
      <th>Beer Garden</th>
      <th>Big Box Store</th>
      <th>Bistro</th>
      <th>Boutique</th>
      <th>Boxing Gym</th>
      <th>Brewery</th>
      <th>Burger Joint</th>
      <th>Burrito Place</th>
      <th>Bus Station</th>
      <th>Bus Stop</th>
      <th>Café</th>
      <th>Caribbean Restaurant</th>
      <th>Chinese Restaurant</th>
      <th>Church</th>
      <th>Coffee Shop</th>
      <th>Convenience Store</th>
      <th>Creperie</th>
      <th>Deli / Bodega</th>
      <th>Department Store</th>
      <th>Dessert Shop</th>
      <th>Diner</th>
      <th>Electronics Store</th>
      <th>Event Space</th>
      <th>Falafel Restaurant</th>
      <th>Fast Food Restaurant</th>
      <th>Food</th>
      <th>Food &amp; Drink Shop</th>
      <th>Fried Chicken Joint</th>
      <th>Gift Shop</th>
      <th>Grocery Store</th>
      <th>Gym</th>
      <th>Gym / Fitness Center</th>
      <th>Gymnastics Gym</th>
      <th>Harbor / Marina</th>
      <th>Historic Site</th>
      <th>Hotel</th>
      <th>Ice Cream Shop</th>
      <th>Italian Restaurant</th>
      <th>Japanese Restaurant</th>
      <th>Juice Bar</th>
      <th>Karaoke Bar</th>
      <th>Latin American Restaurant</th>
      <th>Market</th>
      <th>Mediterranean Restaurant</th>
      <th>Mexican Restaurant</th>
      <th>Museum</th>
      <th>Music Venue</th>
      <th>Other Repair Shop</th>
      <th>Park</th>
      <th>Peruvian Restaurant</th>
      <th>Pet Store</th>
      <th>Pharmacy</th>
      <th>Pizza Place</th>
      <th>Plaza</th>
      <th>Pool</th>
      <th>Pub</th>
      <th>Racetrack</th>
      <th>Restaurant</th>
      <th>Salad Place</th>
      <th>Sandwich Place</th>
      <th>Seafood Restaurant</th>
      <th>Shoe Store</th>
      <th>Shop &amp; Service</th>
      <th>Shopping Mall</th>
      <th>Shopping Plaza</th>
      <th>Snack Place</th>
      <th>Soccer Stadium</th>
      <th>Sports Bar</th>
      <th>Steakhouse</th>
      <th>Supermarket</th>
      <th>Sushi Restaurant</th>
      <th>Taco Place</th>
      <th>Theater</th>
      <th>Train Station</th>
      <th>Tree</th>
      <th>Vegetarian / Vegan Restaurant</th>
      <th>Video Store</th>
      <th>Wings Joint</th>
      <th>Women's Store</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>San José</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>San José</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>San José</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>San José</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>San José</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



#### Let's examine the new dataframe size


```python
CR_onehot.shape
```




    (255, 89)



#### Let's group rows by City and by taking the mean of the frequency of occurrence of each category


```python
CR_grouped = CR_onehot.groupby('City').mean().reset_index()
CR_grouped
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
      <th>City</th>
      <th>American Restaurant</th>
      <th>Art Gallery</th>
      <th>Arts &amp; Crafts Store</th>
      <th>Asian Restaurant</th>
      <th>Athletics &amp; Sports</th>
      <th>Auto Garage</th>
      <th>Bakery</th>
      <th>Bar</th>
      <th>Bed &amp; Breakfast</th>
      <th>Beer Garden</th>
      <th>Big Box Store</th>
      <th>Bistro</th>
      <th>Boutique</th>
      <th>Boxing Gym</th>
      <th>Brewery</th>
      <th>Burger Joint</th>
      <th>Burrito Place</th>
      <th>Bus Station</th>
      <th>Bus Stop</th>
      <th>Café</th>
      <th>Caribbean Restaurant</th>
      <th>Chinese Restaurant</th>
      <th>Church</th>
      <th>Coffee Shop</th>
      <th>Convenience Store</th>
      <th>Creperie</th>
      <th>Deli / Bodega</th>
      <th>Department Store</th>
      <th>Dessert Shop</th>
      <th>Diner</th>
      <th>Electronics Store</th>
      <th>Event Space</th>
      <th>Falafel Restaurant</th>
      <th>Fast Food Restaurant</th>
      <th>Food</th>
      <th>Food &amp; Drink Shop</th>
      <th>Fried Chicken Joint</th>
      <th>Gift Shop</th>
      <th>Grocery Store</th>
      <th>Gym</th>
      <th>Gym / Fitness Center</th>
      <th>Gymnastics Gym</th>
      <th>Harbor / Marina</th>
      <th>Historic Site</th>
      <th>Hotel</th>
      <th>Ice Cream Shop</th>
      <th>Italian Restaurant</th>
      <th>Japanese Restaurant</th>
      <th>Juice Bar</th>
      <th>Karaoke Bar</th>
      <th>Latin American Restaurant</th>
      <th>Market</th>
      <th>Mediterranean Restaurant</th>
      <th>Mexican Restaurant</th>
      <th>Museum</th>
      <th>Music Venue</th>
      <th>Other Repair Shop</th>
      <th>Park</th>
      <th>Peruvian Restaurant</th>
      <th>Pet Store</th>
      <th>Pharmacy</th>
      <th>Pizza Place</th>
      <th>Plaza</th>
      <th>Pool</th>
      <th>Pub</th>
      <th>Racetrack</th>
      <th>Restaurant</th>
      <th>Salad Place</th>
      <th>Sandwich Place</th>
      <th>Seafood Restaurant</th>
      <th>Shoe Store</th>
      <th>Shop &amp; Service</th>
      <th>Shopping Mall</th>
      <th>Shopping Plaza</th>
      <th>Snack Place</th>
      <th>Soccer Stadium</th>
      <th>Sports Bar</th>
      <th>Steakhouse</th>
      <th>Supermarket</th>
      <th>Sushi Restaurant</th>
      <th>Taco Place</th>
      <th>Theater</th>
      <th>Train Station</th>
      <th>Tree</th>
      <th>Vegetarian / Vegan Restaurant</th>
      <th>Video Store</th>
      <th>Wings Joint</th>
      <th>Women's Store</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Alajuela</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.250000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.25</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.25</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.250000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Buenos Aires</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.250000</td>
      <td>0.25</td>
      <td>0.250000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.250000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Cartago</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.034483</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.068966</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.034483</td>
      <td>0.000000</td>
      <td>0.034483</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.034483</td>
      <td>0.034483</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.034483</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.034483</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.034483</td>
      <td>0.034483</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.034483</td>
      <td>0.000000</td>
      <td>0.034483</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.034483</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.068966</td>
      <td>0.034483</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.034483</td>
      <td>0.103448</td>
      <td>0.034483</td>
      <td>0.00</td>
      <td>0.034483</td>
      <td>0.00</td>
      <td>0.034483</td>
      <td>0.034483</td>
      <td>0.034483</td>
      <td>0.034483</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.034483</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.034483</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Cañas</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Ciudad Cortés</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.166667</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.166667</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.166667</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.166667</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.166667</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.166667</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>El Roble</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.2</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.4</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.2</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.2</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Heredia</td>
      <td>0.00</td>
      <td>0.035714</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.035714</td>
      <td>0.000000</td>
      <td>0.035714</td>
      <td>0.071429</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.035714</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.035714</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.035714</td>
      <td>0.035714</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.071429</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.035714</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.035714</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.107143</td>
      <td>0.035714</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.035714</td>
      <td>0.000000</td>
      <td>0.107143</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.035714</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.035714</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.035714</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.035714</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.035714</td>
      <td>0.000000</td>
      <td>0.035714</td>
      <td>0.0</td>
      <td>0.035714</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>7</th>
      <td>La Cruz</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.500000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.250000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.250000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Liberia</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.133333</td>
      <td>0.133333</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.066667</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.20</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.133333</td>
      <td>0.000000</td>
      <td>0.066667</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.066667</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.133333</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.066667</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Puerto Limón</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>1.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Puntarenas</td>
      <td>0.02</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.040000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.02000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.06</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.02</td>
      <td>0.02</td>
      <td>0.02</td>
      <td>0.000000</td>
      <td>0.04000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.060000</td>
      <td>0.02</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.02</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.100000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.0</td>
      <td>0.040000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.020000</td>
      <td>0.02</td>
      <td>0.060000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.140000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.020000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.02</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Quesada</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.5</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.500000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>12</th>
      <td>San José</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.02381</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.02381</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.02381</td>
      <td>0.095238</td>
      <td>0.047619</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.02381</td>
      <td>0.000000</td>
      <td>0.02381</td>
      <td>0.02381</td>
      <td>0.071429</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.047619</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.00</td>
      <td>0.047619</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.023810</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.023810</td>
      <td>0.00</td>
      <td>0.071429</td>
      <td>0.000000</td>
      <td>0.095238</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.047619</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.023810</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>13</th>
      <td>San Juan</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.026316</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.052632</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.052632</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.052632</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.026316</td>
      <td>0.026316</td>
      <td>0.000000</td>
      <td>0.052632</td>
      <td>0.026316</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.026316</td>
      <td>0.026316</td>
      <td>0.026316</td>
      <td>0.00</td>
      <td>0.078947</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.026316</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.026316</td>
      <td>0.000000</td>
      <td>0.052632</td>
      <td>0.000000</td>
      <td>0.026316</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.078947</td>
      <td>0.00</td>
      <td>0.052632</td>
      <td>0.000000</td>
      <td>0.078947</td>
      <td>0.026316</td>
      <td>0.026316</td>
      <td>0.000000</td>
      <td>0.026316</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.026316</td>
      <td>0.026316</td>
      <td>0.000000</td>
      <td>0.026316</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.026316</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Santa Ana</td>
      <td>0.00</td>
      <td>0.041667</td>
      <td>0.041667</td>
      <td>0.000000</td>
      <td>0.041667</td>
      <td>0.041667</td>
      <td>0.000000</td>
      <td>0.041667</td>
      <td>0.000000</td>
      <td>0.041667</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.041667</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.041667</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.041667</td>
      <td>0.041667</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.041667</td>
      <td>0.041667</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.041667</td>
      <td>0.041667</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.166667</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.041667</td>
      <td>0.000000</td>
      <td>0.041667</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.041667</td>
      <td>0.083333</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.041667</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Sixaola</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>1.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.00000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
  </tbody>
</table>
</div>



#### Let's print each neighborhood along with the top 5 most common venues


```python
num_top_venues = 5

for City in CR_grouped['City']:
    print("----"+City+"----")
    temp = CR_grouped[CR_grouped['City'] == City].T.reset_index()
    temp.columns = ['venue','freq']
    temp = temp.iloc[1:]
    temp['freq'] = temp['freq'].astype(float)
    temp = temp.round({'freq': 2})
    print(temp.sort_values('freq', ascending=False).reset_index(drop=True).head(num_top_venues))
    print('\n')
```

    ----Alajuela----
                   venue  freq
    0  Food & Drink Shop  0.25
    1               Pool  0.25
    2                Bar  0.25
    3         Restaurant  0.25
    4  Other Repair Shop  0.00
    
    
    ----Buenos Aires----
                           venue  freq
    0                 Steakhouse  0.25
    1                  Juice Bar  0.25
    2                Karaoke Bar  0.25
    3  Latin American Restaurant  0.25
    4          Other Repair Shop  0.00
    
    
    ----Cartago----
                    venue  freq
    0         Pizza Place  0.10
    1  Mexican Restaurant  0.07
    2              Bakery  0.07
    3       Women's Store  0.03
    4                 Gym  0.03
    
    
    ----Cañas----
             venue  freq
    0        Hotel   1.0
    1  Art Gallery   0.0
    2          Pub   0.0
    3         Pool   0.0
    4        Plaza   0.0
    
    
    ----Ciudad Cortés----
               venue  freq
    0           Park  0.17
    1       Pharmacy  0.17
    2  Grocery Store  0.17
    3    Bus Station  0.17
    4    Snack Place  0.17
    
    
    ----El Roble----
                     venue  freq
    0             Bus Stop   0.4
    1    Other Repair Shop   0.2
    2                 Tree   0.2
    3        Big Box Store   0.2
    4  American Restaurant   0.0
    
    
    ----Heredia----
                venue  freq
    0  Ice Cream Shop  0.11
    1             Gym  0.11
    2     Coffee Shop  0.07
    3             Bar  0.07
    4          Market  0.04
    
    
    ----La Cruz----
                   venue  freq
    0                Bar  0.50
    1              Hotel  0.25
    2         Restaurant  0.25
    3  Other Repair Shop  0.00
    4                Pub  0.00
    
    
    ----Liberia----
                    venue  freq
    0  Chinese Restaurant  0.20
    1               Hotel  0.13
    2                 Bar  0.13
    3     Bed & Breakfast  0.13
    4          Restaurant  0.13
    
    
    ----Puerto Limón----
                     venue  freq
    0      Harbor / Marina   1.0
    1  American Restaurant   0.0
    2    Other Repair Shop   0.0
    3                  Pub   0.0
    4                 Pool   0.0
    
    
    ----Puntarenas----
                      venue  freq
    0    Seafood Restaurant  0.14
    1        Ice Cream Shop  0.10
    2            Restaurant  0.06
    3  Fast Food Restaurant  0.06
    4    Chinese Restaurant  0.06
    
    
    ----Quesada----
                     venue  freq
    0       Gymnastics Gym   0.5
    1               Market   0.5
    2  American Restaurant   0.0
    3    Other Repair Shop   0.0
    4                  Pub   0.0
    
    
    ----San José----
                           venue  freq
    0             Sandwich Place  0.10
    1                Coffee Shop  0.10
    2       Fast Food Restaurant  0.07
    3                 Restaurant  0.07
    4  Latin American Restaurant  0.05
    
    
    ----San Juan----
                           venue  freq
    0                        Pub  0.08
    1  Latin American Restaurant  0.08
    2             Sandwich Place  0.08
    3       Fast Food Restaurant  0.05
    4                  Pet Store  0.05
    
    
    ----Santa Ana----
                          venue  freq
    0                Restaurant  0.17
    1                Steakhouse  0.08
    2  Mediterranean Restaurant  0.04
    3                 Pet Store  0.04
    4       Peruvian Restaurant  0.04
    
    
    ----Sixaola----
                     venue  freq
    0          Bus Station   1.0
    1  American Restaurant   0.0
    2           Restaurant   0.0
    3                  Pub   0.0
    4                 Pool   0.0
    
    


#### Function to sort the venues in descending order


```python
def return_most_common_venues(row, num_top_venues):
    row_categories = row.iloc[1:]
    row_categories_sorted = row_categories.sort_values(ascending=False)
    
    return row_categories_sorted.index.values[0:num_top_venues]
```

#### Let's create the new dataframe and display the top 5 venues for each City.


```python
num_top_venues = 5

indicators = ['st', 'nd', 'rd']

# create columns according to number of top venues
columns = ['City']
for ind in np.arange(num_top_venues):
    try:
        columns.append('{}{} Most Common Venue'.format(ind+1, indicators[ind]))
    except:
        columns.append('{}th Most Common Venue'.format(ind+1))

# create a new dataframe
Cities_venues_sorted = pd.DataFrame(columns=columns)
Cities_venues_sorted['City'] = CR_grouped['City']

for ind in np.arange(CR_grouped.shape[0]):
    Cities_venues_sorted.iloc[ind, 1:] = return_most_common_venues(CR_grouped.iloc[ind, :], num_top_venues)

Cities_venues_sorted.head()
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
      <th>City</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Alajuela</td>
      <td>Restaurant</td>
      <td>Food &amp; Drink Shop</td>
      <td>Bar</td>
      <td>Pool</td>
      <td>Women's Store</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Buenos Aires</td>
      <td>Juice Bar</td>
      <td>Karaoke Bar</td>
      <td>Latin American Restaurant</td>
      <td>Steakhouse</td>
      <td>Women's Store</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Cartago</td>
      <td>Pizza Place</td>
      <td>Bakery</td>
      <td>Mexican Restaurant</td>
      <td>Women's Store</td>
      <td>Gym</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Cañas</td>
      <td>Hotel</td>
      <td>Harbor / Marina</td>
      <td>Coffee Shop</td>
      <td>Convenience Store</td>
      <td>Creperie</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Ciudad Cortés</td>
      <td>Snack Place</td>
      <td>Grocery Store</td>
      <td>Pharmacy</td>
      <td>Bus Station</td>
      <td>Park</td>
    </tr>
  </tbody>
</table>
</div>



#### Cluster Neighborhoods


```python
# set number of clusters
kclusters = 5

CR_grouped_clustering = CR_grouped.drop('City', 1)

# run k-means clustering
kmeans = KMeans(n_clusters=kclusters, random_state=0).fit(CR_grouped_clustering)

# check cluster labels generated for each row in the dataframe
kmeans.labels_[0:10] 
```




    array([0, 0, 0, 3, 0, 0, 0, 0, 0, 1], dtype=int32)



#### Let's create a new dataframe that includes the cluster as well as the previous information


```python
# add clustering labels
Cities_venues_sorted.insert(0, 'Cluster Labels', kmeans.labels_)

CR_merged = df2

# merge CR_grouped with df2 to add previous information for each city
CR_merged = CR_merged.join(Cities_venues_sorted.set_index('City'), on='City')

CR_merged.head() # check the last columns!
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
      <th>City</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Country</th>
      <th>Province</th>
      <th>Population</th>
      <th>Cluster Labels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>San José</td>
      <td>9.9333</td>
      <td>-84.0833</td>
      <td>Costa Rica</td>
      <td>San José</td>
      <td>288054.0</td>
      <td>0.0</td>
      <td>Coffee Shop</td>
      <td>Sandwich Place</td>
      <td>Fast Food Restaurant</td>
      <td>Restaurant</td>
      <td>Convenience Store</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Cartago</td>
      <td>9.8667</td>
      <td>-83.9167</td>
      <td>Costa Rica</td>
      <td>Cartago</td>
      <td>221733.0</td>
      <td>0.0</td>
      <td>Pizza Place</td>
      <td>Bakery</td>
      <td>Mexican Restaurant</td>
      <td>Women's Store</td>
      <td>Gym</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Puerto Limón</td>
      <td>10.0022</td>
      <td>-83.0840</td>
      <td>Costa Rica</td>
      <td>Limón</td>
      <td>61072.0</td>
      <td>1.0</td>
      <td>Harbor / Marina</td>
      <td>Wings Joint</td>
      <td>Coffee Shop</td>
      <td>Convenience Store</td>
      <td>Creperie</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Liberia</td>
      <td>10.6338</td>
      <td>-85.4333</td>
      <td>Costa Rica</td>
      <td>Guanacaste</td>
      <td>45380.0</td>
      <td>0.0</td>
      <td>Chinese Restaurant</td>
      <td>Hotel</td>
      <td>Restaurant</td>
      <td>Bar</td>
      <td>Bed &amp; Breakfast</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Alajuela</td>
      <td>10.0278</td>
      <td>-84.2041</td>
      <td>Costa Rica</td>
      <td>Alajuela</td>
      <td>42975.0</td>
      <td>0.0</td>
      <td>Restaurant</td>
      <td>Food &amp; Drink Shop</td>
      <td>Bar</td>
      <td>Pool</td>
      <td>Women's Store</td>
    </tr>
  </tbody>
</table>
</div>




```python
CR_merged.dropna()
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
      <th>City</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Country</th>
      <th>Province</th>
      <th>Population</th>
      <th>Cluster Labels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>San José</td>
      <td>9.9333</td>
      <td>-84.0833</td>
      <td>Costa Rica</td>
      <td>San José</td>
      <td>288054.0</td>
      <td>0.0</td>
      <td>Coffee Shop</td>
      <td>Sandwich Place</td>
      <td>Fast Food Restaurant</td>
      <td>Restaurant</td>
      <td>Convenience Store</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Cartago</td>
      <td>9.8667</td>
      <td>-83.9167</td>
      <td>Costa Rica</td>
      <td>Cartago</td>
      <td>221733.0</td>
      <td>0.0</td>
      <td>Pizza Place</td>
      <td>Bakery</td>
      <td>Mexican Restaurant</td>
      <td>Women's Store</td>
      <td>Gym</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Puerto Limón</td>
      <td>10.0022</td>
      <td>-83.0840</td>
      <td>Costa Rica</td>
      <td>Limón</td>
      <td>61072.0</td>
      <td>1.0</td>
      <td>Harbor / Marina</td>
      <td>Wings Joint</td>
      <td>Coffee Shop</td>
      <td>Convenience Store</td>
      <td>Creperie</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Liberia</td>
      <td>10.6338</td>
      <td>-85.4333</td>
      <td>Costa Rica</td>
      <td>Guanacaste</td>
      <td>45380.0</td>
      <td>0.0</td>
      <td>Chinese Restaurant</td>
      <td>Hotel</td>
      <td>Restaurant</td>
      <td>Bar</td>
      <td>Bed &amp; Breakfast</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Alajuela</td>
      <td>10.0278</td>
      <td>-84.2041</td>
      <td>Costa Rica</td>
      <td>Alajuela</td>
      <td>42975.0</td>
      <td>0.0</td>
      <td>Restaurant</td>
      <td>Food &amp; Drink Shop</td>
      <td>Bar</td>
      <td>Pool</td>
      <td>Women's Store</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Puntarenas</td>
      <td>9.9764</td>
      <td>-84.8339</td>
      <td>Costa Rica</td>
      <td>Puntarenas</td>
      <td>41528.0</td>
      <td>0.0</td>
      <td>Seafood Restaurant</td>
      <td>Ice Cream Shop</td>
      <td>Chinese Restaurant</td>
      <td>Restaurant</td>
      <td>Fast Food Restaurant</td>
    </tr>
    <tr>
      <th>6</th>
      <td>San Juan</td>
      <td>9.9609</td>
      <td>-84.0731</td>
      <td>Costa Rica</td>
      <td>San José</td>
      <td>24944.0</td>
      <td>0.0</td>
      <td>Latin American Restaurant</td>
      <td>Pub</td>
      <td>Sandwich Place</td>
      <td>Fast Food Restaurant</td>
      <td>Pet Store</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Heredia</td>
      <td>9.9985</td>
      <td>-84.1169</td>
      <td>Costa Rica</td>
      <td>Heredia</td>
      <td>22700.0</td>
      <td>0.0</td>
      <td>Gym</td>
      <td>Ice Cream Shop</td>
      <td>Coffee Shop</td>
      <td>Bar</td>
      <td>Bistro</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Santa Ana</td>
      <td>9.9320</td>
      <td>-84.1760</td>
      <td>Costa Rica</td>
      <td>San José</td>
      <td>11320.0</td>
      <td>0.0</td>
      <td>Restaurant</td>
      <td>Steakhouse</td>
      <td>Shop &amp; Service</td>
      <td>Brewery</td>
      <td>Ice Cream Shop</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Buenos Aires</td>
      <td>9.1497</td>
      <td>-83.3334</td>
      <td>Costa Rica</td>
      <td>Puntarenas</td>
      <td>45000.0</td>
      <td>0.0</td>
      <td>Juice Bar</td>
      <td>Karaoke Bar</td>
      <td>Latin American Restaurant</td>
      <td>Steakhouse</td>
      <td>Women's Store</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Quesada</td>
      <td>10.3305</td>
      <td>-84.4400</td>
      <td>Costa Rica</td>
      <td>Alajuela</td>
      <td>31106.0</td>
      <td>4.0</td>
      <td>Gymnastics Gym</td>
      <td>Market</td>
      <td>Event Space</td>
      <td>Coffee Shop</td>
      <td>Convenience Store</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Cañas</td>
      <td>10.4300</td>
      <td>-85.1000</td>
      <td>Costa Rica</td>
      <td>Guanacaste</td>
      <td>20306.0</td>
      <td>3.0</td>
      <td>Hotel</td>
      <td>Harbor / Marina</td>
      <td>Coffee Shop</td>
      <td>Convenience Store</td>
      <td>Creperie</td>
    </tr>
    <tr>
      <th>12</th>
      <td>El Roble</td>
      <td>9.9771</td>
      <td>-84.7443</td>
      <td>Costa Rica</td>
      <td>Puntarenas</td>
      <td>15759.0</td>
      <td>0.0</td>
      <td>Bus Stop</td>
      <td>Big Box Store</td>
      <td>Tree</td>
      <td>Other Repair Shop</td>
      <td>Event Space</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Sixaola</td>
      <td>9.5083</td>
      <td>-82.6147</td>
      <td>Costa Rica</td>
      <td>Limón</td>
      <td>10234.0</td>
      <td>2.0</td>
      <td>Bus Station</td>
      <td>Women's Store</td>
      <td>Church</td>
      <td>Convenience Store</td>
      <td>Creperie</td>
    </tr>
    <tr>
      <th>15</th>
      <td>La Cruz</td>
      <td>11.0742</td>
      <td>-85.6294</td>
      <td>Costa Rica</td>
      <td>Guanacaste</td>
      <td>9195.0</td>
      <td>0.0</td>
      <td>Bar</td>
      <td>Hotel</td>
      <td>Restaurant</td>
      <td>Women's Store</td>
      <td>Event Space</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Ciudad Cortés</td>
      <td>8.9600</td>
      <td>-83.5239</td>
      <td>Costa Rica</td>
      <td>Puntarenas</td>
      <td>3850.0</td>
      <td>0.0</td>
      <td>Snack Place</td>
      <td>Grocery Store</td>
      <td>Pharmacy</td>
      <td>Bus Station</td>
      <td>Park</td>
    </tr>
  </tbody>
</table>
</div>



#### Examine Clusters

#### Cluster 1


```python
CR_merged.loc[CR_merged['Cluster Labels'] == 0, CR_merged.columns[[1] + list(range(5, CR_merged.shape[1]))]]
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
      <th>Latitude</th>
      <th>Population</th>
      <th>Cluster Labels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>9.9333</td>
      <td>288054.0</td>
      <td>0.0</td>
      <td>Coffee Shop</td>
      <td>Sandwich Place</td>
      <td>Fast Food Restaurant</td>
      <td>Restaurant</td>
      <td>Convenience Store</td>
    </tr>
    <tr>
      <th>1</th>
      <td>9.8667</td>
      <td>221733.0</td>
      <td>0.0</td>
      <td>Pizza Place</td>
      <td>Bakery</td>
      <td>Mexican Restaurant</td>
      <td>Women's Store</td>
      <td>Gym</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10.6338</td>
      <td>45380.0</td>
      <td>0.0</td>
      <td>Chinese Restaurant</td>
      <td>Hotel</td>
      <td>Restaurant</td>
      <td>Bar</td>
      <td>Bed &amp; Breakfast</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10.0278</td>
      <td>42975.0</td>
      <td>0.0</td>
      <td>Restaurant</td>
      <td>Food &amp; Drink Shop</td>
      <td>Bar</td>
      <td>Pool</td>
      <td>Women's Store</td>
    </tr>
    <tr>
      <th>5</th>
      <td>9.9764</td>
      <td>41528.0</td>
      <td>0.0</td>
      <td>Seafood Restaurant</td>
      <td>Ice Cream Shop</td>
      <td>Chinese Restaurant</td>
      <td>Restaurant</td>
      <td>Fast Food Restaurant</td>
    </tr>
    <tr>
      <th>6</th>
      <td>9.9609</td>
      <td>24944.0</td>
      <td>0.0</td>
      <td>Latin American Restaurant</td>
      <td>Pub</td>
      <td>Sandwich Place</td>
      <td>Fast Food Restaurant</td>
      <td>Pet Store</td>
    </tr>
    <tr>
      <th>7</th>
      <td>9.9985</td>
      <td>22700.0</td>
      <td>0.0</td>
      <td>Gym</td>
      <td>Ice Cream Shop</td>
      <td>Coffee Shop</td>
      <td>Bar</td>
      <td>Bistro</td>
    </tr>
    <tr>
      <th>8</th>
      <td>9.9320</td>
      <td>11320.0</td>
      <td>0.0</td>
      <td>Restaurant</td>
      <td>Steakhouse</td>
      <td>Shop &amp; Service</td>
      <td>Brewery</td>
      <td>Ice Cream Shop</td>
    </tr>
    <tr>
      <th>9</th>
      <td>9.1497</td>
      <td>45000.0</td>
      <td>0.0</td>
      <td>Juice Bar</td>
      <td>Karaoke Bar</td>
      <td>Latin American Restaurant</td>
      <td>Steakhouse</td>
      <td>Women's Store</td>
    </tr>
    <tr>
      <th>12</th>
      <td>9.9771</td>
      <td>15759.0</td>
      <td>0.0</td>
      <td>Bus Stop</td>
      <td>Big Box Store</td>
      <td>Tree</td>
      <td>Other Repair Shop</td>
      <td>Event Space</td>
    </tr>
    <tr>
      <th>15</th>
      <td>11.0742</td>
      <td>9195.0</td>
      <td>0.0</td>
      <td>Bar</td>
      <td>Hotel</td>
      <td>Restaurant</td>
      <td>Women's Store</td>
      <td>Event Space</td>
    </tr>
    <tr>
      <th>17</th>
      <td>8.9600</td>
      <td>3850.0</td>
      <td>0.0</td>
      <td>Snack Place</td>
      <td>Grocery Store</td>
      <td>Pharmacy</td>
      <td>Bus Station</td>
      <td>Park</td>
    </tr>
  </tbody>
</table>
</div>



#### Cluster 2


```python
CR_merged.loc[CR_merged['Cluster Labels'] == 1, CR_merged.columns[[1] + list(range(5, CR_merged.shape[1]))]]
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
      <th>Latitude</th>
      <th>Population</th>
      <th>Cluster Labels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>10.0022</td>
      <td>61072.0</td>
      <td>1.0</td>
      <td>Harbor / Marina</td>
      <td>Wings Joint</td>
      <td>Coffee Shop</td>
      <td>Convenience Store</td>
      <td>Creperie</td>
    </tr>
  </tbody>
</table>
</div>



#### Cluster 3


```python
CR_merged.loc[CR_merged['Cluster Labels'] == 2, CR_merged.columns[[1] + list(range(5, CR_merged.shape[1]))]]
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
      <th>Latitude</th>
      <th>Population</th>
      <th>Cluster Labels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>14</th>
      <td>9.5083</td>
      <td>10234.0</td>
      <td>2.0</td>
      <td>Bus Station</td>
      <td>Women's Store</td>
      <td>Church</td>
      <td>Convenience Store</td>
      <td>Creperie</td>
    </tr>
  </tbody>
</table>
</div>



#### Cluster 4


```python
CR_merged.loc[CR_merged['Cluster Labels'] == 3, CR_merged.columns[[1] + list(range(5, CR_merged.shape[1]))]]
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
      <th>Latitude</th>
      <th>Population</th>
      <th>Cluster Labels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>11</th>
      <td>10.43</td>
      <td>20306.0</td>
      <td>3.0</td>
      <td>Hotel</td>
      <td>Harbor / Marina</td>
      <td>Coffee Shop</td>
      <td>Convenience Store</td>
      <td>Creperie</td>
    </tr>
  </tbody>
</table>
</div>



#### Cluster 5


```python
CR_merged.loc[CR_merged['Cluster Labels'] == 4, CR_merged.columns[[1] + list(range(5, CR_merged.shape[1]))]]
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
      <th>Latitude</th>
      <th>Population</th>
      <th>Cluster Labels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>10</th>
      <td>10.3305</td>
      <td>31106.0</td>
      <td>4.0</td>
      <td>Gymnastics Gym</td>
      <td>Market</td>
      <td>Event Space</td>
      <td>Coffee Shop</td>
      <td>Convenience Store</td>
    </tr>
  </tbody>
</table>
</div>



# Results

### From Cluster 1, it is recommended to establish the gyms in the cities of San Jose and Liberia.

# Discussion section

### The results may not represent the reality, because in the selected country the common venues does not necessarily include all the exact venues that exist in the area. For that reason it is recommended to made a second analysis.

# Conclusion

### According to the requirements defined at the beginning of the project, it was possible to find and recommend the best two places to establish the gyms.
