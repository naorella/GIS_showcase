This Project was made with the **[gis_dev_container](https://github.com/naorella/gis_dev_container)** project,  
by creating a virual environment with docker.

# An Analysis of Toronto Roads and the Effect of Calming Features

##### By: Nelson Alesandro


The following Jupyter Notebook leverages the GeoPandas Python libraries  
and its dependencies to create plots and summaries based on analytical observations.

The purpose is to explore the relationship that calming features within roads have on traffic counts and collision rates.

This Notebook will show a step by step proccess, as well as rationale, that were taken to reach our conclusion.


# Importing, Cleaning, and Merging Data

#### Importing Libraries

The first thing we need to do is import all the libraries that we will use.


```python
#Libraries for Data Manipulation
###########################
import pandas as pd
import geopandas as gpd
import numpy as np
from shapely.geometry import Point, Polygon
#see all dataframe columns when calling head on a dataframe
pd.set_option('display.max_columns', None)

#Libraries for geocoding
###########################
from geopy.geocoders import Nominatim
from geopy.geocoders import GoogleV3

#Libraries for Mapping
###########################
from mpl_toolkits.basemap import Basemap
import matplotlib.pyplot as plt
import folium

#Libraries for Plotting data
###########################
import matplotlib.pyplot as plt
import matplotlib.colors
import seaborn as sns
import matplotlib as mpl
import scipy.stats as pystat
import matplotlib.pyplot as plt
from mpl_toolkits.axes_grid1 import make_axes_locatable
import branca
import branca.colormap as cm
import scipy

#Ignore copy warnings for dataframes
#The warnigns are meant to bring attention that a copy of data
#not the orignal is being manipulated. However manipulating copies
#  is the intended action at times.
import warnings
pd.options.mode.chained_assignment = None  # default='warn'
warnings.simplefilter(action='ignore', category=FutureWarning)

#Tell matplot lib to generate plots inline, within the notebook
%matplotlib inline
```


#### Importing and Creating Roads Shapefile

We import the Centreline data from the Open Data portal from the City of Toronto.  
This data set contains a variety of line features within Toronto, including: walkways, lake shores, railroads, and roads.

We plot the data and call .head() to see a general overview of its shape and confirm it is the data we need.


```python
shape_path = "/workspace/GIS_project/data/Centreline - Version 2 - 4326/Centreline - Version 2 - 4326.shp"
centreline = gpd.read_file(shape_path)
centreline.plot()
```




    <Axes: >




    
![png](Analysis_files/Analysis_5_1.png)
    



```python
centreline.head()
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
      <th>_id1</th>
      <th>CENTREL2</th>
      <th>LINEAR_3</th>
      <th>LINEAR_4</th>
      <th>LINEAR_5</th>
      <th>ADDRESS6</th>
      <th>ADDRESS7</th>
      <th>PARITY_8</th>
      <th>PARITY_9</th>
      <th>LO_NUM_10</th>
      <th>HI_NUM_11</th>
      <th>LO_NUM_12</th>
      <th>HI_NUM_13</th>
      <th>BEGIN_A14</th>
      <th>END_ADD15</th>
      <th>BEGIN_A16</th>
      <th>END_ADD17</th>
      <th>BEGIN_A18</th>
      <th>END_ADD19</th>
      <th>BEGIN_A20</th>
      <th>END_ADD21</th>
      <th>LOW_NUM22</th>
      <th>HIGH_NU23</th>
      <th>LOW_NUM24</th>
      <th>HIGH_NU25</th>
      <th>LINEAR_26</th>
      <th>LINEAR_27</th>
      <th>LINEAR_28</th>
      <th>LINEAR_29</th>
      <th>LINEAR_30</th>
      <th>FROM_IN31</th>
      <th>TO_INTE32</th>
      <th>ONEWAY_33</th>
      <th>ONEWAY_34</th>
      <th>FEATURE35</th>
      <th>FEATURE36</th>
      <th>JURISDI37</th>
      <th>CENTREL38</th>
      <th>OBJECTI39</th>
      <th>MI_PRIN40</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>914600</td>
      <td>2141</td>
      <td>Morrison St</td>
      <td>Morrison Street</td>
      <td>None</td>
      <td>None</td>
      <td>N</td>
      <td>N</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>Morrison</td>
      <td>St</td>
      <td>None</td>
      <td>ET</td>
      <td>Morrison St</td>
      <td>13470555</td>
      <td>13470560</td>
      <td>0</td>
      <td>Not One-Way</td>
      <td>201500</td>
      <td>Local</td>
      <td>CITY OF TORONTO</td>
      <td>None</td>
      <td>1</td>
      <td>1</td>
      <td>LINESTRING (-79.50875 43.59744, -79.50987 43.5...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>914601</td>
      <td>2666</td>
      <td>Twelfth St</td>
      <td>Twelfth Street</td>
      <td>66-92</td>
      <td>65-89</td>
      <td>E</td>
      <td>O</td>
      <td>66</td>
      <td>92</td>
      <td>65</td>
      <td>89</td>
      <td>1040061</td>
      <td>1040085</td>
      <td>1040060</td>
      <td>1040082</td>
      <td>66</td>
      <td>92</td>
      <td>65</td>
      <td>89</td>
      <td>65</td>
      <td>89</td>
      <td>66</td>
      <td>92</td>
      <td>Twelfth</td>
      <td>St</td>
      <td>None</td>
      <td>None</td>
      <td>Twelfth St</td>
      <td>13470560</td>
      <td>13470530</td>
      <td>0</td>
      <td>Not One-Way</td>
      <td>201500</td>
      <td>Local</td>
      <td>CITY OF TORONTO</td>
      <td>None</td>
      <td>2</td>
      <td>2</td>
      <td>LINESTRING (-79.50987 43.5972, -79.51035 43.59...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>7862398</td>
      <td>2611</td>
      <td>Thirteenth St</td>
      <td>Thirteenth Street</td>
      <td>66-96</td>
      <td>65-91</td>
      <td>E</td>
      <td>O</td>
      <td>66</td>
      <td>96</td>
      <td>65</td>
      <td>91</td>
      <td>7862400</td>
      <td>7862422</td>
      <td>7862399</td>
      <td>7862415</td>
      <td>66</td>
      <td>96</td>
      <td>65</td>
      <td>91</td>
      <td>65</td>
      <td>91</td>
      <td>66</td>
      <td>96</td>
      <td>Thirteenth</td>
      <td>St</td>
      <td>None</td>
      <td>None</td>
      <td>Thirteenth St</td>
      <td>13470571</td>
      <td>13470538</td>
      <td>0</td>
      <td>Not One-Way</td>
      <td>201500</td>
      <td>Local</td>
      <td>CITY OF TORONTO</td>
      <td>None</td>
      <td>3</td>
      <td>3</td>
      <td>LINESTRING (-79.51087 43.59697, -79.51134 43.5...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>914587</td>
      <td>1962</td>
      <td>Lake Shore Blvd W</td>
      <td>Lake Shore Boulevard West</td>
      <td>None</td>
      <td>3180-3180</td>
      <td>N</td>
      <td>E</td>
      <td>0</td>
      <td>0</td>
      <td>3180</td>
      <td>3180</td>
      <td>0</td>
      <td>0</td>
      <td>1013460</td>
      <td>1013460</td>
      <td>0</td>
      <td>0</td>
      <td>3180</td>
      <td>3180</td>
      <td>0</td>
      <td>0</td>
      <td>3180</td>
      <td>3180</td>
      <td>Lake Shore</td>
      <td>Blvd</td>
      <td>W</td>
      <td>None</td>
      <td>Lake Shore Blvd W</td>
      <td>13470546</td>
      <td>13470552</td>
      <td>0</td>
      <td>Not One-Way</td>
      <td>201200</td>
      <td>Major Arterial</td>
      <td>CITY OF TORONTO</td>
      <td>None</td>
      <td>6</td>
      <td>6</td>
      <td>LINESTRING (-79.51805 43.59795, -79.51914 43.5...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>6735911</td>
      <td>1962</td>
      <td>Lake Shore Blvd W</td>
      <td>Lake Shore Boulevard West</td>
      <td>3197-3197</td>
      <td>3190-3190</td>
      <td>O</td>
      <td>E</td>
      <td>3197</td>
      <td>3197</td>
      <td>3190</td>
      <td>3190</td>
      <td>6735910</td>
      <td>6735910</td>
      <td>6735913</td>
      <td>6735913</td>
      <td>3197</td>
      <td>3197</td>
      <td>3190</td>
      <td>3190</td>
      <td>3197</td>
      <td>3197</td>
      <td>3190</td>
      <td>3190</td>
      <td>Lake Shore</td>
      <td>Blvd</td>
      <td>W</td>
      <td>None</td>
      <td>Lake Shore Blvd W</td>
      <td>13470552</td>
      <td>13470558</td>
      <td>0</td>
      <td>Not One-Way</td>
      <td>201200</td>
      <td>Major Arterial</td>
      <td>CITY OF TORONTO</td>
      <td>None</td>
      <td>7</td>
      <td>7</td>
      <td>LINESTRING (-79.51914 43.5977, -79.52024 43.59...</td>
    </tr>
  </tbody>
</table>
</div>



All we need however are the road features, and for the puproses of our observation we will exclude Expressways and their ramps.  
The reason for exlusion being that calming features like speed bumps or traffic islands are only on non-Expressway roads. So in order to get  
accurate analysis we want two sample groups that model the same amount of variables as closely as possible.


```python
#We only want non motorway roads. So find feature types, and select the ones we want
centreline.FEATURE36.unique()

```




    array(['Local', 'Major Arterial', 'River', 'Geostatistical line',
           'Major Shoreline', 'Collector', 'Minor Arterial', 'Major Railway',
           'Expressway', 'Laneway', 'Expressway Ramp', 'Pending',
           'Major Arterial Ramp', 'Other', 'Hydro Line', 'Walkway',
           'Minor Shoreline (Land locked)', 'Minor Railway', 'Trail',
           'Creek/Tributary', 'Collector Ramp', 'Busway', 'Ferry Route',
           'Access Road', 'Other Ramp', 'Minor Arterial Ramp'], dtype=object)




```python
road_list = ['Local', 'Major Arterial', 'Major Arterial Ramp', 'Collector', 'Collector Ramp', 'Minor Arterial', 'Minor Arterial Ramp']
roads = centreline.query('FEATURE36 in @road_list')
roads = roads[['CENTREL2', 'LINEAR_3', 'LINEAR_4','LINEAR_5',
                'LINEAR_26', 'LINEAR_30', 'FROM_IN31', 'TO_INTE32', 'FEATURE35',
                'FEATURE36', 'JURISDI37', 'OBJECTI39', 'MI_PRIN40', 'geometry']]
roads.head()
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
      <th>CENTREL2</th>
      <th>LINEAR_3</th>
      <th>LINEAR_4</th>
      <th>LINEAR_5</th>
      <th>LINEAR_26</th>
      <th>LINEAR_30</th>
      <th>FROM_IN31</th>
      <th>TO_INTE32</th>
      <th>FEATURE35</th>
      <th>FEATURE36</th>
      <th>JURISDI37</th>
      <th>OBJECTI39</th>
      <th>MI_PRIN40</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>914600</td>
      <td>2141</td>
      <td>Morrison St</td>
      <td>Morrison Street</td>
      <td>Morrison</td>
      <td>Morrison St</td>
      <td>13470555</td>
      <td>13470560</td>
      <td>201500</td>
      <td>Local</td>
      <td>CITY OF TORONTO</td>
      <td>1</td>
      <td>1</td>
      <td>LINESTRING (-79.50875 43.59744, -79.50987 43.5...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>914601</td>
      <td>2666</td>
      <td>Twelfth St</td>
      <td>Twelfth Street</td>
      <td>Twelfth</td>
      <td>Twelfth St</td>
      <td>13470560</td>
      <td>13470530</td>
      <td>201500</td>
      <td>Local</td>
      <td>CITY OF TORONTO</td>
      <td>2</td>
      <td>2</td>
      <td>LINESTRING (-79.50987 43.5972, -79.51035 43.59...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7862398</td>
      <td>2611</td>
      <td>Thirteenth St</td>
      <td>Thirteenth Street</td>
      <td>Thirteenth</td>
      <td>Thirteenth St</td>
      <td>13470571</td>
      <td>13470538</td>
      <td>201500</td>
      <td>Local</td>
      <td>CITY OF TORONTO</td>
      <td>3</td>
      <td>3</td>
      <td>LINESTRING (-79.51087 43.59697, -79.51134 43.5...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>914587</td>
      <td>1962</td>
      <td>Lake Shore Blvd W</td>
      <td>Lake Shore Boulevard West</td>
      <td>Lake Shore</td>
      <td>Lake Shore Blvd W</td>
      <td>13470546</td>
      <td>13470552</td>
      <td>201200</td>
      <td>Major Arterial</td>
      <td>CITY OF TORONTO</td>
      <td>6</td>
      <td>6</td>
      <td>LINESTRING (-79.51805 43.59795, -79.51914 43.5...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6735911</td>
      <td>1962</td>
      <td>Lake Shore Blvd W</td>
      <td>Lake Shore Boulevard West</td>
      <td>Lake Shore</td>
      <td>Lake Shore Blvd W</td>
      <td>13470552</td>
      <td>13470558</td>
      <td>201200</td>
      <td>Major Arterial</td>
      <td>CITY OF TORONTO</td>
      <td>7</td>
      <td>7</td>
      <td>LINESTRING (-79.51914 43.5977, -79.52024 43.59...</td>
    </tr>
  </tbody>
</table>
</div>



We re-project the roads we chose into EPSG:26917, the local mapping projection for the Toronto area,  
also known as NAD83 / UTM zone 17N.  
A benefit of the reprojection is that the units we use for distance are now in meters, as specified by metadata of the projections.

We plot once more to visually check that the new road feature matches what we wanted.


```python
road_26917 = roads.to_crs(epsg=26917)
road_26917.plot()
```




    <Axes: >




    
![png](Analysis_files/Analysis_11_1.png)
    



#### Importing Calming Features Shapefile

Next we import the calming features database. This dataset contains the shapefile of all calming features within Toronto,  
including the name of the road they are located on.


```python
file_path = "/workspace/GIS_project/data/Traffic Calming Database/Traffic Calming Database - 04.04.2023V2.shp"
calming_roads = gpd.read_file(file_path)
```

Once more we plot and check the data to make sure it lines up with what we expect.


```python
calming_roads = calming_roads.to_crs('epsg:26917')
calming_roads.plot()
```




    <Axes: >




    
![png](Analysis_files/Analysis_15_1.png)
    



```python
calming_roads.head(-5)
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
      <th>calm_id</th>
      <th>street</th>
      <th>intersecti</th>
      <th>intersec_1</th>
      <th>spd_hump</th>
      <th>traf_islan</th>
      <th>spd_cush</th>
      <th>Installed</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1.0</td>
      <td>Oakridge Drive</td>
      <td>Brimley Rd</td>
      <td>McCowan Rd</td>
      <td>8.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2004</td>
      <td>LINESTRING (641464.739 4842951.003, 641507.965...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3.0</td>
      <td>Atlas Avenue</td>
      <td>Gloucester Grove</td>
      <td>Eglinton Avenue West</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1999</td>
      <td>LINESTRING (625890.037 4839370.112, 625854.462...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4.0</td>
      <td>Balliol Street</td>
      <td>Mt. Pleasant Road</td>
      <td>Cleveland Street</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1998</td>
      <td>LINESTRING (630028.385 4839752.147, 630387.961...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>5.0</td>
      <td>Balmoral Avenue</td>
      <td>Avenue Road</td>
      <td>Yonge Street</td>
      <td>6.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1998</td>
      <td>LINESTRING (629505.289 4838240.561, 629455.886...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6.0</td>
      <td>Bartlett Avenue North</td>
      <td>Geary Avenue</td>
      <td>Davenport Road</td>
      <td>6.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1999</td>
      <td>LINESTRING (626022.276 4836406.392, 626006.544...</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>855</th>
      <td>2151.0</td>
      <td>Rainsford Rd</td>
      <td>Kingston Rd</td>
      <td>Queen St E</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>4.0</td>
      <td>2022</td>
      <td>LINESTRING (636495.803 4836447.656, 636480.886...</td>
    </tr>
    <tr>
      <th>856</th>
      <td>2159.0</td>
      <td>Roseheath Ave</td>
      <td>Danforth Ave</td>
      <td>Hanson St</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>4.0</td>
      <td>2022</td>
      <td>LINESTRING (635672.31 4837829.833, 635667.053 ...</td>
    </tr>
    <tr>
      <th>857</th>
      <td>2152.0</td>
      <td>Scarboro Beach Blvd</td>
      <td>Hubbard Blvd</td>
      <td>Queen St E</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>3.0</td>
      <td>2022</td>
      <td>LINESTRING (637568.753 4836791.877, 637582.565...</td>
    </tr>
    <tr>
      <th>858</th>
      <td>2153.0</td>
      <td>Scarborough Rd</td>
      <td>Pine Ave</td>
      <td>Queen St E</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>3.0</td>
      <td>2022</td>
      <td>LINESTRING (638193.782 4837424.426, 638322.386...</td>
    </tr>
    <tr>
      <th>859</th>
      <td>2156.0</td>
      <td>Bowmore Rd</td>
      <td>Gerrard St E</td>
      <td>Wrenson Rd</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>3.0</td>
      <td>2022</td>
      <td>LINESTRING (635823.673 4837499.543, 635838.7 4...</td>
    </tr>
  </tbody>
</table>
<p>860 rows × 9 columns</p>
</div>



#### Combining Roads with Calming Features

One of the advantages of directly working with Python libraries rather than manually doing the tasks via ArcGIS or QGIS, is that
repetative tasks that are time consuming can be done with relative ease.

In this case what we are doing is grouping each feature by it's street name, and combining them with each other if they are within 5 meters.  
Because we are automating it we can iterate through thousand of street names in a matter of seconds.

The reason for this method rather than merging on the street name values or proximity is due to edge cases.  
If we were to merge on street names, there is a chance the either feature will match on a duplicate street  
name somewhere else in the city.

Alternatively, if we were to match soley by proximity, then if a calming feature was accidently offset during creation it might instead
match to an adjacent road connected to the one its supposed to be on.

If we were to do this manually we would need to match calming features 1:m to any roads within 5 meters, and then filter out non matching street names.  
Although tedious, not difficult, but we will see this process again later in the notebook on a process less likely to be done manually.


```python
#split each dataframe into groups
calm_split = {k:d for k, d in calming_roads.groupby('street')}
road_split = {k:d for k, d in road_26917.groupby('LINEAR_5')}

#our new feature
combined_roads = pd.DataFrame()
for i in calm_split:
    #try is needed for when a road is present in one group, but not the other
    #causing a key value error
    try:
        #if shared road is present AND wihtin the distance then add to an already existing dataframe
        combined_roads = pd.concat([combined_roads, gpd.sjoin_nearest(road_split[i], calm_split[i], max_distance=5, how='right')])
    except:
        #otherwise we didn't need to add the data and can pass over it
        pass

#There can be multiple calming features per road, but we only care if a road has one or not, so we
#retrieve the max values on its properties.
#not using mode, since its likely a road may have two features, therefore returning an error on mode
combined_roads = combined_roads.drop(columns=['index_left', 'geometry'])
combined_roads = combined_roads.groupby('CENTREL2').agg('max')
#reset to remove groups
combined_roads = combined_roads.reset_index()

#only keep the columns we need
combined_roads = combined_roads[['CENTREL2', 'calm_id', 'spd_hump', 'traf_islan', 'spd_cush', 'Installed']]
combined_roads.head(-5)
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
      <th>CENTREL2</th>
      <th>calm_id</th>
      <th>spd_hump</th>
      <th>traf_islan</th>
      <th>spd_cush</th>
      <th>Installed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>127.0</td>
      <td>1329.0</td>
      <td>4.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2009</td>
    </tr>
    <tr>
      <th>1</th>
      <td>131.0</td>
      <td>1329.0</td>
      <td>4.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2009</td>
    </tr>
    <tr>
      <th>2</th>
      <td>250.0</td>
      <td>969.0</td>
      <td>7.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2005</td>
    </tr>
    <tr>
      <th>3</th>
      <td>377.0</td>
      <td>1273.0</td>
      <td>5.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2008</td>
    </tr>
    <tr>
      <th>4</th>
      <td>515.0</td>
      <td>1112.0</td>
      <td>5.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2006</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>740</th>
      <td>30139823.0</td>
      <td>473.0</td>
      <td>5.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2000</td>
    </tr>
    <tr>
      <th>741</th>
      <td>30141591.0</td>
      <td>49.0</td>
      <td>4.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1999</td>
    </tr>
    <tr>
      <th>742</th>
      <td>30142944.0</td>
      <td>156.0</td>
      <td>7.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2001</td>
    </tr>
    <tr>
      <th>743</th>
      <td>30142945.0</td>
      <td>156.0</td>
      <td>7.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2001</td>
    </tr>
    <tr>
      <th>744</th>
      <td>30145697.0</td>
      <td>1352.0</td>
      <td>3.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2011</td>
    </tr>
  </tbody>
</table>
<p>745 rows × 6 columns</p>
</div>



Thanks to matching calming features to streets, calming featrures now share a common ID with roads,  
in which we can now use to merge back into the roads feature.


```python
#add the calmed roads back into road using the new shared ids
road_26917 = road_26917.merge(combined_roads, how='left', on=['CENTREL2'], validate='1:1', suffixes=('_l', '_r'))
road_26917.head()
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
      <th>CENTREL2</th>
      <th>LINEAR_3</th>
      <th>LINEAR_4</th>
      <th>LINEAR_5</th>
      <th>LINEAR_26</th>
      <th>LINEAR_30</th>
      <th>FROM_IN31</th>
      <th>TO_INTE32</th>
      <th>FEATURE35</th>
      <th>FEATURE36</th>
      <th>JURISDI37</th>
      <th>OBJECTI39</th>
      <th>MI_PRIN40</th>
      <th>geometry</th>
      <th>calm_id</th>
      <th>spd_hump</th>
      <th>traf_islan</th>
      <th>spd_cush</th>
      <th>Installed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>914600</td>
      <td>2141</td>
      <td>Morrison St</td>
      <td>Morrison Street</td>
      <td>Morrison</td>
      <td>Morrison St</td>
      <td>13470555</td>
      <td>13470560</td>
      <td>201500</td>
      <td>Local</td>
      <td>CITY OF TORONTO</td>
      <td>1</td>
      <td>1</td>
      <td>LINESTRING (620365.778 4828243.398, 620275.681...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>914601</td>
      <td>2666</td>
      <td>Twelfth St</td>
      <td>Twelfth Street</td>
      <td>Twelfth</td>
      <td>Twelfth St</td>
      <td>13470560</td>
      <td>13470530</td>
      <td>201500</td>
      <td>Local</td>
      <td>CITY OF TORONTO</td>
      <td>2</td>
      <td>2</td>
      <td>LINESTRING (620275.681 4828215.213, 620235.062...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7862398</td>
      <td>2611</td>
      <td>Thirteenth St</td>
      <td>Thirteenth Street</td>
      <td>Thirteenth</td>
      <td>Thirteenth St</td>
      <td>13470571</td>
      <td>13470538</td>
      <td>201500</td>
      <td>Local</td>
      <td>CITY OF TORONTO</td>
      <td>3</td>
      <td>3</td>
      <td>LINESTRING (620195.613 4828188.199, 620155.247...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>914587</td>
      <td>1962</td>
      <td>Lake Shore Blvd W</td>
      <td>Lake Shore Boulevard West</td>
      <td>Lake Shore</td>
      <td>Lake Shore Blvd W</td>
      <td>13470546</td>
      <td>13470552</td>
      <td>201200</td>
      <td>Major Arterial</td>
      <td>CITY OF TORONTO</td>
      <td>6</td>
      <td>6</td>
      <td>LINESTRING (619614.26 4828286.233, 619526.218 ...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6735911</td>
      <td>1962</td>
      <td>Lake Shore Blvd W</td>
      <td>Lake Shore Boulevard West</td>
      <td>Lake Shore</td>
      <td>Lake Shore Blvd W</td>
      <td>13470552</td>
      <td>13470558</td>
      <td>201200</td>
      <td>Major Arterial</td>
      <td>CITY OF TORONTO</td>
      <td>7</td>
      <td>7</td>
      <td>LINESTRING (619526.218 4828257.126, 619438.216...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



#### Importing Traffic Data

The Open Data set includes traffic data in the form of Comma Seperated Values (CSV), seperated by decade.  
So we will need to combine them first.


```python
#read shapefiles
file_path_1 = "/workspace/GIS_project/data/traffic/raw-data-1990-1999.csv"
file_path_2 = "/workspace/GIS_project/data/traffic/raw-data-2000-2009.csv"
file_path_3 = "/workspace/GIS_project/data/traffic/raw-data-2010-2019.csv"
file_path_4 = "/workspace/GIS_project/data/traffic/raw-data-2020-2029.csv"
#combine the csvs into a single dataframe
traffic = pd.concat(
    map(pd.read_csv, [file_path_1, file_path_2, file_path_3, file_path_4]), ignore_index=True)
#display head data
traffic.head(-5)
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
      <th>_id</th>
      <th>count_id</th>
      <th>count_date</th>
      <th>location_id</th>
      <th>location</th>
      <th>lng</th>
      <th>lat</th>
      <th>centreline_type</th>
      <th>centreline_id</th>
      <th>px</th>
      <th>time_start</th>
      <th>time_end</th>
      <th>sb_cars_r</th>
      <th>sb_cars_t</th>
      <th>sb_cars_l</th>
      <th>nb_cars_r</th>
      <th>nb_cars_t</th>
      <th>nb_cars_l</th>
      <th>wb_cars_r</th>
      <th>wb_cars_t</th>
      <th>wb_cars_l</th>
      <th>eb_cars_r</th>
      <th>eb_cars_t</th>
      <th>eb_cars_l</th>
      <th>sb_truck_r</th>
      <th>sb_truck_t</th>
      <th>sb_truck_l</th>
      <th>nb_truck_r</th>
      <th>nb_truck_t</th>
      <th>nb_truck_l</th>
      <th>wb_truck_r</th>
      <th>wb_truck_t</th>
      <th>wb_truck_l</th>
      <th>eb_truck_r</th>
      <th>eb_truck_t</th>
      <th>eb_truck_l</th>
      <th>sb_bus_r</th>
      <th>sb_bus_t</th>
      <th>sb_bus_l</th>
      <th>nb_bus_r</th>
      <th>nb_bus_t</th>
      <th>nb_bus_l</th>
      <th>wb_bus_r</th>
      <th>wb_bus_t</th>
      <th>wb_bus_l</th>
      <th>eb_bus_r</th>
      <th>eb_bus_t</th>
      <th>eb_bus_l</th>
      <th>nx_peds</th>
      <th>sx_peds</th>
      <th>ex_peds</th>
      <th>wx_peds</th>
      <th>nx_bike</th>
      <th>sx_bike</th>
      <th>ex_bike</th>
      <th>wx_bike</th>
      <th>nx_other</th>
      <th>sx_other</th>
      <th>ex_other</th>
      <th>wx_other</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>2891</td>
      <td>1990-08-21</td>
      <td>4614</td>
      <td>ELINOR AVE AT LAWRENCE AVE (PX 2178)</td>
      <td>-79.300554</td>
      <td>43.744099</td>
      <td>2.0</td>
      <td>13451060.0</td>
      <td>2178.0</td>
      <td>1990-08-21T07:30:00</td>
      <td>1990-08-21T07:45:00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>324.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>9.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>4.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>2891</td>
      <td>1990-08-21</td>
      <td>4614</td>
      <td>ELINOR AVE AT LAWRENCE AVE (PX 2178)</td>
      <td>-79.300554</td>
      <td>43.744099</td>
      <td>2.0</td>
      <td>13451060.0</td>
      <td>2178.0</td>
      <td>1990-08-21T07:45:00</td>
      <td>1990-08-21T08:00:00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>309.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>11.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>5.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>2891</td>
      <td>1990-08-21</td>
      <td>4614</td>
      <td>ELINOR AVE AT LAWRENCE AVE (PX 2178)</td>
      <td>-79.300554</td>
      <td>43.744099</td>
      <td>2.0</td>
      <td>13451060.0</td>
      <td>2178.0</td>
      <td>1990-08-21T08:00:00</td>
      <td>1990-08-21T08:15:00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>340.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>10.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>4.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>2891</td>
      <td>1990-08-21</td>
      <td>4614</td>
      <td>ELINOR AVE AT LAWRENCE AVE (PX 2178)</td>
      <td>-79.300554</td>
      <td>43.744099</td>
      <td>2.0</td>
      <td>13451060.0</td>
      <td>2178.0</td>
      <td>1990-08-21T08:15:00</td>
      <td>1990-08-21T08:30:00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>321.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>8.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>2891</td>
      <td>1990-08-21</td>
      <td>4614</td>
      <td>ELINOR AVE AT LAWRENCE AVE (PX 2178)</td>
      <td>-79.300554</td>
      <td>43.744099</td>
      <td>2.0</td>
      <td>13451060.0</td>
      <td>2178.0</td>
      <td>1990-08-21T08:30:00</td>
      <td>1990-08-21T08:45:00</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>295.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>14.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>4.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>838841</th>
      <td>185100</td>
      <td>101686</td>
      <td>2024-07-09</td>
      <td>51246</td>
      <td>Sumach St / Shuter St</td>
      <td>-79.359505</td>
      <td>43.658301</td>
      <td>2.0</td>
      <td>30026352.0</td>
      <td>2627.0</td>
      <td>2024-07-09T15:15:00</td>
      <td>2024-07-09T15:30:00</td>
      <td>5.0</td>
      <td>8.0</td>
      <td>7.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>17.0</td>
      <td>0.0</td>
      <td>5.0</td>
      <td>69.0</td>
      <td>9.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>5.0</td>
      <td>4.0</td>
      <td>7.0</td>
      <td>19.0</td>
      <td>2.0</td>
      <td>3.0</td>
      <td>5.0</td>
      <td>19.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>838842</th>
      <td>185101</td>
      <td>101686</td>
      <td>2024-07-09</td>
      <td>51246</td>
      <td>Sumach St / Shuter St</td>
      <td>-79.359505</td>
      <td>43.658301</td>
      <td>2.0</td>
      <td>30026352.0</td>
      <td>2627.0</td>
      <td>2024-07-09T15:30:00</td>
      <td>2024-07-09T15:45:00</td>
      <td>3.0</td>
      <td>7.0</td>
      <td>7.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>22.0</td>
      <td>2.0</td>
      <td>4.0</td>
      <td>55.0</td>
      <td>4.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>33.0</td>
      <td>3.0</td>
      <td>10.0</td>
      <td>25.0</td>
      <td>3.0</td>
      <td>8.0</td>
      <td>3.0</td>
      <td>16.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>838843</th>
      <td>185102</td>
      <td>101686</td>
      <td>2024-07-09</td>
      <td>51246</td>
      <td>Sumach St / Shuter St</td>
      <td>-79.359505</td>
      <td>43.658301</td>
      <td>2.0</td>
      <td>30026352.0</td>
      <td>2627.0</td>
      <td>2024-07-09T16:00:00</td>
      <td>2024-07-09T16:15:00</td>
      <td>5.0</td>
      <td>14.0</td>
      <td>3.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>6.0</td>
      <td>20.0</td>
      <td>1.0</td>
      <td>16.0</td>
      <td>50.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>11.0</td>
      <td>8.0</td>
      <td>11.0</td>
      <td>28.0</td>
      <td>2.0</td>
      <td>12.0</td>
      <td>4.0</td>
      <td>22.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>838844</th>
      <td>185103</td>
      <td>101686</td>
      <td>2024-07-09</td>
      <td>51246</td>
      <td>Sumach St / Shuter St</td>
      <td>-79.359505</td>
      <td>43.658301</td>
      <td>2.0</td>
      <td>30026352.0</td>
      <td>2627.0</td>
      <td>2024-07-09T16:15:00</td>
      <td>2024-07-09T16:30:00</td>
      <td>1.0</td>
      <td>4.0</td>
      <td>6.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>5.0</td>
      <td>26.0</td>
      <td>3.0</td>
      <td>6.0</td>
      <td>53.0</td>
      <td>11.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>10.0</td>
      <td>6.0</td>
      <td>12.0</td>
      <td>37.0</td>
      <td>5.0</td>
      <td>8.0</td>
      <td>2.0</td>
      <td>36.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>838845</th>
      <td>185104</td>
      <td>101686</td>
      <td>2024-07-09</td>
      <td>51246</td>
      <td>Sumach St / Shuter St</td>
      <td>-79.359505</td>
      <td>43.658301</td>
      <td>2.0</td>
      <td>30026352.0</td>
      <td>2627.0</td>
      <td>2024-07-09T16:30:00</td>
      <td>2024-07-09T16:45:00</td>
      <td>5.0</td>
      <td>10.0</td>
      <td>3.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>3.0</td>
      <td>28.0</td>
      <td>2.0</td>
      <td>13.0</td>
      <td>52.0</td>
      <td>7.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>8.0</td>
      <td>7.0</td>
      <td>9.0</td>
      <td>30.0</td>
      <td>7.0</td>
      <td>17.0</td>
      <td>4.0</td>
      <td>34.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
<p>838846 rows × 60 columns</p>
</div>



#### Recalculating and Cleaning Traffic Data

The CSV's sperate traffic by mode of transportation, although we are only concerned with motor vehicles, such as cars, buses, and motorcycles, not pedestirans
or bikes.

We add up all the columns that have to motor traffic into a new column called 'motortraf'.  
We also calculate the year the count occured so that we can group them later.

Finally the new data, as well as essential data, is put into a new truncated dataframe, for easier manipulation.


```python
#calculate new field that adds up all motor traffic
traffic['motortraf'] = traffic.loc[:,'sb_cars_r':'eb_bus_l'].sum(axis=1) + traffic.loc[:,'nx_other':'wx_other'].sum(axis=1)
#create a year field
traffic['year'] = pd.to_datetime(traffic['count_date']).dt.year

#new, cleaner, dataframe
traffic_trunc = traffic[['count_id', 'year', 'location_id', 'location', 'lng', 'lat', 'centreline_type', 'centreline_id', 'motortraf']]
traffic_trunc.head()
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
      <th>count_id</th>
      <th>year</th>
      <th>location_id</th>
      <th>location</th>
      <th>lng</th>
      <th>lat</th>
      <th>centreline_type</th>
      <th>centreline_id</th>
      <th>motortraf</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2891</td>
      <td>1990</td>
      <td>4614</td>
      <td>ELINOR AVE AT LAWRENCE AVE (PX 2178)</td>
      <td>-79.300554</td>
      <td>43.744099</td>
      <td>2.0</td>
      <td>13451060.0</td>
      <td>333.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2891</td>
      <td>1990</td>
      <td>4614</td>
      <td>ELINOR AVE AT LAWRENCE AVE (PX 2178)</td>
      <td>-79.300554</td>
      <td>43.744099</td>
      <td>2.0</td>
      <td>13451060.0</td>
      <td>320.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2891</td>
      <td>1990</td>
      <td>4614</td>
      <td>ELINOR AVE AT LAWRENCE AVE (PX 2178)</td>
      <td>-79.300554</td>
      <td>43.744099</td>
      <td>2.0</td>
      <td>13451060.0</td>
      <td>350.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2891</td>
      <td>1990</td>
      <td>4614</td>
      <td>ELINOR AVE AT LAWRENCE AVE (PX 2178)</td>
      <td>-79.300554</td>
      <td>43.744099</td>
      <td>2.0</td>
      <td>13451060.0</td>
      <td>329.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2891</td>
      <td>1990</td>
      <td>4614</td>
      <td>ELINOR AVE AT LAWRENCE AVE (PX 2178)</td>
      <td>-79.300554</td>
      <td>43.744099</td>
      <td>2.0</td>
      <td>13451060.0</td>
      <td>309.0</td>
    </tr>
  </tbody>
</table>
</div>



The data is grouped by year and we calculate the sums of traffic for whole years, aggregating on each road.  
We use 'location_id', as it is the unique identifier for traffic count.


```python
#aggregate and group by year and location id
traffic_agg = traffic_trunc.groupby(['year','location_id']).agg({'count_id': pd.Series.mode,
                                                              'location': pd.Series.mode,
                                                              'lng': pd.Series.mode, 'lat': pd.Series.mode,
                                                              'centreline_type':pd.Series.mode,
                                                              'centreline_id':pd.Series.mode,
                                                              'motortraf': 'sum'})
#reset index so that we can use the data frame
traffic_agg = traffic_agg.reset_index()
traffic_agg.head(-5)
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
      <th>year</th>
      <th>location_id</th>
      <th>count_id</th>
      <th>location</th>
      <th>lng</th>
      <th>lat</th>
      <th>centreline_type</th>
      <th>centreline_id</th>
      <th>motortraf</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1990</td>
      <td>3943</td>
      <td>2896</td>
      <td>OCONNOR DR AT ST CLAIR AVE (PX 447)</td>
      <td>-79.312836</td>
      <td>43.705396</td>
      <td>2.0</td>
      <td>13457092.0</td>
      <td>25543.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1990</td>
      <td>3950</td>
      <td>2899</td>
      <td>KINGSTON RD AT BROOKLAWN AVE &amp; ST CLAIR AVE E ...</td>
      <td>-79.236019</td>
      <td>43.722032</td>
      <td>2.0</td>
      <td>13454337.0</td>
      <td>23234.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1990</td>
      <td>3976</td>
      <td>3139</td>
      <td>EGLINTON SQ AT O CONNOR DR &amp; VICTORIA PARK AVE...</td>
      <td>-79.302100</td>
      <td>43.723380</td>
      <td>2.0</td>
      <td>13454288.0</td>
      <td>23507.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1990</td>
      <td>3977</td>
      <td>3178</td>
      <td>DANFORTH AVE AT VICTORIA PARK AVE (PX 355)</td>
      <td>-79.288358</td>
      <td>43.691232</td>
      <td>2.0</td>
      <td>13459445.0</td>
      <td>22429.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1990</td>
      <td>3978</td>
      <td>3243</td>
      <td>EGLINTON AVE AT VICTORIA PARK AVE (PX 451)</td>
      <td>-79.302645</td>
      <td>43.724723</td>
      <td>2.0</td>
      <td>13454107.0</td>
      <td>38630.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>22833</th>
      <td>2024</td>
      <td>51160</td>
      <td>101526</td>
      <td>Allenvale Ave / Lauder Ave</td>
      <td>-79.445359</td>
      <td>43.692544</td>
      <td>2.0</td>
      <td>13459620.0</td>
      <td>656.0</td>
    </tr>
    <tr>
      <th>22834</th>
      <td>2024</td>
      <td>51161</td>
      <td>101529</td>
      <td>Oakwood Ave / Earlsdale Ave</td>
      <td>-79.438091</td>
      <td>43.685555</td>
      <td>2.0</td>
      <td>13460884.0</td>
      <td>9187.0</td>
    </tr>
    <tr>
      <th>22835</th>
      <td>2024</td>
      <td>51231</td>
      <td>101538</td>
      <td>Brunswick Ave / Lowther Ave</td>
      <td>-79.408303</td>
      <td>43.668037</td>
      <td>2.0</td>
      <td>13463884.0</td>
      <td>910.0</td>
    </tr>
    <tr>
      <th>22836</th>
      <td>2024</td>
      <td>51232</td>
      <td>101541</td>
      <td>Royal York Rd / Valiant Rd</td>
      <td>-79.515116</td>
      <td>43.655848</td>
      <td>2.0</td>
      <td>13466089.0</td>
      <td>10564.0</td>
    </tr>
    <tr>
      <th>22837</th>
      <td>2024</td>
      <td>51233</td>
      <td>101537</td>
      <td>Bryn Rd / Gracefield Ave</td>
      <td>-79.482577</td>
      <td>43.711408</td>
      <td>2.0</td>
      <td>13456520.0</td>
      <td>1332.0</td>
    </tr>
  </tbody>
</table>
<p>22838 rows × 9 columns</p>
</div>



After aggregating we need to map the new data into a geodataframe using the 'lng' and 'lat' fields as  
longitude and latitude.

This allows it to be plotted, and for use to perform spatial analysis.


```python
points = traffic_agg.apply(lambda row: Point(row.lng, row.lat), axis = 1)
#only keep traffic 2014 and after to line up with collisions
traffic_intersections = gpd.GeoDataFrame(traffic_agg, geometry=points)
#using metadate (filename) to initialize the data with the proper projection
traffic_intersections.crs = {'init': 'epsg:4326'}
traffic_26917 = traffic_intersections.to_crs(epsg=26917)
traffic_26917.plot()
```




    <Axes: >




    
![png](Analysis_files/Analysis_28_1.png)
    


#### Importing Collision Data

The collision data has information on collisions, with their location offeset to the nearest intersection.


```python
collision = gpd.read_file('/workspace/GIS_project/data/Traffic Collisions - 4326/Traffic Collisions - 4326.shp')
collision.head(-5)
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
      <th>_id1</th>
      <th>OCC_DAT2</th>
      <th>OCC_MON3</th>
      <th>OCC_DOW4</th>
      <th>OCC_YEA5</th>
      <th>OCC_HOU6</th>
      <th>DIVISIO7</th>
      <th>FATALIT8</th>
      <th>INJURY_9</th>
      <th>FTR_COL10</th>
      <th>PD_COLL11</th>
      <th>HOOD_1512</th>
      <th>NEIGHBO13</th>
      <th>LONG_WG14</th>
      <th>LAT_WGS15</th>
      <th>AUTOMOB16</th>
      <th>MOTORCY17</th>
      <th>PASSENG18</th>
      <th>BICYCLE19</th>
      <th>PEDESTR20</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>1388552400000</td>
      <td>January</td>
      <td>Wednesday</td>
      <td>2014</td>
      <td>4</td>
      <td>D43</td>
      <td>0</td>
      <td>YES</td>
      <td>NO</td>
      <td>NO</td>
      <td>157</td>
      <td>Bendale South (157)</td>
      <td>-79.25535525232044</td>
      <td>43.75352197370893</td>
      <td>YES</td>
      <td>NO</td>
      <td>YES</td>
      <td>NO</td>
      <td>NO</td>
      <td>MULTIPOINT (-79.25536 43.75352)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>1388552400000</td>
      <td>January</td>
      <td>Wednesday</td>
      <td>2014</td>
      <td>14</td>
      <td>D14</td>
      <td>0</td>
      <td>NO</td>
      <td>YES</td>
      <td>NO</td>
      <td>078</td>
      <td>Kensington-Chinatown (78)</td>
      <td>-79.40601573209595</td>
      <td>43.652310093633126</td>
      <td>YES</td>
      <td>NO</td>
      <td>NO</td>
      <td>NO</td>
      <td>NO</td>
      <td>MULTIPOINT (-79.40602 43.65231)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>14</td>
      <td>1388552400000</td>
      <td>January</td>
      <td>Wednesday</td>
      <td>2014</td>
      <td>1</td>
      <td>D14</td>
      <td>0</td>
      <td>NO</td>
      <td>NO</td>
      <td>YES</td>
      <td>086</td>
      <td>Roncesvalles (86)</td>
      <td>-79.4286372237863</td>
      <td>43.64220588807775</td>
      <td>YES</td>
      <td>NO</td>
      <td>NO</td>
      <td>NO</td>
      <td>NO</td>
      <td>MULTIPOINT (-79.42864 43.64221)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>15</td>
      <td>1388552400000</td>
      <td>January</td>
      <td>Wednesday</td>
      <td>2014</td>
      <td>14</td>
      <td>D13</td>
      <td>0</td>
      <td>NO</td>
      <td>NO</td>
      <td>YES</td>
      <td>101</td>
      <td>Forest Hill South (101)</td>
      <td>-79.4178019320394</td>
      <td>43.68673751185377</td>
      <td>YES</td>
      <td>NO</td>
      <td>NO</td>
      <td>NO</td>
      <td>NO</td>
      <td>MULTIPOINT (-79.4178 43.68674)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>16</td>
      <td>1388552400000</td>
      <td>January</td>
      <td>Wednesday</td>
      <td>2014</td>
      <td>2</td>
      <td>D23</td>
      <td>0</td>
      <td>NO</td>
      <td>NO</td>
      <td>YES</td>
      <td>007</td>
      <td>Willowridge-Martingrove-Richview (7)</td>
      <td>-79.56313850270357</td>
      <td>43.67441063320029</td>
      <td>YES</td>
      <td>NO</td>
      <td>NO</td>
      <td>NO</td>
      <td>NO</td>
      <td>MULTIPOINT (-79.56314 43.67441)</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>574219</th>
      <td>687130</td>
      <td>1719723600000</td>
      <td>June</td>
      <td>Sunday</td>
      <td>2024</td>
      <td>17</td>
      <td>D33</td>
      <td>0</td>
      <td>NO</td>
      <td>YES</td>
      <td>NO</td>
      <td>047</td>
      <td>Don Valley Village (47)</td>
      <td>-79.34698765381663</td>
      <td>43.775179913411826</td>
      <td>YES</td>
      <td>NO</td>
      <td>NO</td>
      <td>NO</td>
      <td>NO</td>
      <td>MULTIPOINT (-79.34699 43.77518)</td>
    </tr>
    <tr>
      <th>574220</th>
      <td>687131</td>
      <td>1719723600000</td>
      <td>June</td>
      <td>Sunday</td>
      <td>2024</td>
      <td>19</td>
      <td>D14</td>
      <td>0</td>
      <td>NO</td>
      <td>NO</td>
      <td>YES</td>
      <td>080</td>
      <td>Palmerston-Little Italy (80)</td>
      <td>-79.41706672764312</td>
      <td>43.65516659990851</td>
      <td>YES</td>
      <td>NO</td>
      <td>NO</td>
      <td>YES</td>
      <td>NO</td>
      <td>MULTIPOINT (-79.41707 43.65517)</td>
    </tr>
    <tr>
      <th>574221</th>
      <td>687132</td>
      <td>1719723600000</td>
      <td>June</td>
      <td>Sunday</td>
      <td>2024</td>
      <td>15</td>
      <td>D51</td>
      <td>0</td>
      <td>NO</td>
      <td>YES</td>
      <td>NO</td>
      <td>168</td>
      <td>Downtown Yonge East (168)</td>
      <td>-79.3777897470096</td>
      <td>43.65194899105262</td>
      <td>YES</td>
      <td>NO</td>
      <td>NO</td>
      <td>NO</td>
      <td>NO</td>
      <td>MULTIPOINT (-79.37779 43.65195)</td>
    </tr>
    <tr>
      <th>574222</th>
      <td>687133</td>
      <td>1719723600000</td>
      <td>June</td>
      <td>Sunday</td>
      <td>2024</td>
      <td>18</td>
      <td>D55</td>
      <td>0</td>
      <td>NO</td>
      <td>NO</td>
      <td>YES</td>
      <td>066</td>
      <td>Danforth (66)</td>
      <td>-79.32363123294031</td>
      <td>43.68336521316597</td>
      <td>YES</td>
      <td>NO</td>
      <td>NO</td>
      <td>NO</td>
      <td>NO</td>
      <td>MULTIPOINT (-79.32363 43.68337)</td>
    </tr>
    <tr>
      <th>574223</th>
      <td>687134</td>
      <td>1719723600000</td>
      <td>June</td>
      <td>Sunday</td>
      <td>2024</td>
      <td>20</td>
      <td>D43</td>
      <td>0</td>
      <td>YES</td>
      <td>NO</td>
      <td>NO</td>
      <td>142</td>
      <td>Woburn North (142)</td>
      <td>-79.23296484605089</td>
      <td>43.77953670123865</td>
      <td>YES</td>
      <td>NO</td>
      <td>NO</td>
      <td>NO</td>
      <td>NO</td>
      <td>MULTIPOINT (-79.23296 43.77954)</td>
    </tr>
  </tbody>
</table>
<p>574224 rows × 21 columns</p>
</div>



#### Cleaning Collision Data

Some of the fields contain both numeric and string values, so we need to standardize the data into common types.

We are also only concerned with whether or not a motor collision occured and if there were injuries, not how many injuries were in a single crash. So the fields
are turned into a binary value, rather than 'YES', 'NO', or a number > 1.

We ignore NaN values, as they wont provide location data, they type of crash, or if injuries were present.


```python
#create new field that we can sum up for total crashes in a year
collision['motorcol'] = collision.apply(lambda x: 1 if x['AUTOMOB16'] == 'YES' else 0, axis=1)
#change the number of fatalities into whether a fatal crash happened or not
collision['fatals'] = collision.apply(lambda x: 1 if x['FATALIT8'] > 0 else 0, axis=1)
#binary field for whether the crash had injuries, so we can sum up total amount of collisions with injuries
collision['injuries'] = collision.apply(lambda x: 1 if x['INJURY_9'] == 'YES' else 0, axis=1)
#aggregate and group by year and geometry
```

We sum up the collisions by year and location.

There is no location ID like there was with traffic, so we will have to rely on the geometry it self, which can be problamatic at times  
if data at two different location are meant to represent the same interestion. 

We hope to resolve this when we combine this data set with the traffic data set.


```python
collision_agg = collision.groupby(['OCC_YEA5','geometry']).agg({'_id1': 'max',
                                                            'HOOD_1512': pd.Series.mode,
                                                            'NEIGHBO13': pd.Series.mode,
                                                            'motorcol':'sum',
                                                            'fatals':'sum',
                                                            'injuries': 'sum'})
collision_agg = collision_agg.reset_index()
collision_agg = collision_agg.rename(columns={'OCC_YEA5':'year'})
collision_agg.head(-5)
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
      <th>year</th>
      <th>geometry</th>
      <th>_id1</th>
      <th>HOOD_1512</th>
      <th>NEIGHBO13</th>
      <th>motorcol</th>
      <th>fatals</th>
      <th>injuries</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2014</td>
      <td>MULTIPOINT (-79.60451 43.64709)</td>
      <td>60064</td>
      <td>011</td>
      <td>Eringate-Centennial-West Deane (11)</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2014</td>
      <td>MULTIPOINT (-79.60308 43.65156)</td>
      <td>59852</td>
      <td>011</td>
      <td>Eringate-Centennial-West Deane (11)</td>
      <td>6</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2014</td>
      <td>MULTIPOINT (-79.59005 43.64551)</td>
      <td>41959</td>
      <td>011</td>
      <td>Eringate-Centennial-West Deane (11)</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2014</td>
      <td>MULTIPOINT (-79.5812 43.65095)</td>
      <td>61195</td>
      <td>011</td>
      <td>Eringate-Centennial-West Deane (11)</td>
      <td>5</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2014</td>
      <td>MULTIPOINT (-79.57949 43.6514)</td>
      <td>64462</td>
      <td>013</td>
      <td>Etobicoke West Mall (13)</td>
      <td>12</td>
      <td>0</td>
      <td>3</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>102844</th>
      <td>2024</td>
      <td>MULTIPOINT (-79.3792 43.64252)</td>
      <td>684870</td>
      <td>166</td>
      <td>St Lawrence-East Bayfront-The Islands (166)</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>102845</th>
      <td>2024</td>
      <td>MULTIPOINT (-79.37683 43.64115)</td>
      <td>685827</td>
      <td>166</td>
      <td>St Lawrence-East Bayfront-The Islands (166)</td>
      <td>16</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>102846</th>
      <td>2024</td>
      <td>MULTIPOINT (-79.37468 43.64195)</td>
      <td>686995</td>
      <td>166</td>
      <td>St Lawrence-East Bayfront-The Islands (166)</td>
      <td>15</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>102847</th>
      <td>2024</td>
      <td>MULTIPOINT (-79.37302 43.64252)</td>
      <td>686910</td>
      <td>166</td>
      <td>St Lawrence-East Bayfront-The Islands (166)</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>102848</th>
      <td>2024</td>
      <td>MULTIPOINT (-79.37999 43.63997)</td>
      <td>682244</td>
      <td>166</td>
      <td>St Lawrence-East Bayfront-The Islands (166)</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>102849 rows × 8 columns</p>
</div>




```python
#plot our new data
#assign the proper projection for the area
collision_26917 = gpd.GeoDataFrame(collision_agg)
collision_26917 = collision_26917.to_crs(epsg=26917)
collision_26917.plot()
```




    <Axes: >




    
![png](Analysis_files/Analysis_35_1.png)
    


#### Combining Traffic Data with Collision Data




Here we have another example of dividing features into groups by a value, in this case by the year, and combining them by proximity.

The difference with this example is that if it were to be done manually, the time needed to join them would be significant.  
To be done manually would invole filtering each data set by a year, extracting the data to new layers, and then performing the join  
between the two. Repeat this process for each year, and then combined all the years together for the finalized dataset.

Rather than do that, we seperate by value and iterate through the years, discarding data if a year is not present.

We join right onto the collision data set, so that each collision point now has a location_id that we can use to combine points that  
meant to represent the same location.



```python
#split each dataframe into groups
col_split = {k:d for k, d in collision_26917.groupby('year')}
traf_split = {k:d for k, d in traffic_26917.groupby('year')}

#our new, empty, dataset
combined = pd.DataFrame()
for i in traf_split:
    #try is needed for when a road is present in on group, but not the other
    #causing a key value error
    try:
        #if shared road is present AND wihtin the distance then add to an already existing dataframe
        #we merge on right because we want each collision point to have a location_id
        combined = pd.concat([combined, gpd.sjoin_nearest(traf_split[i], col_split[i], max_distance=10, how='right')])
    except:
        #otherwise we didn't need to add the data and can pass over it
        combined = pd.concat([combined, traf_split[i]])


#reset index to remove groups
combined = combined.reset_index()

#concating the data leads to some misformating, this fixes it, by grabbing the field with data
combined['year'] = np.nanmax([combined['year'], combined['year_left'], combined['year_right']], axis=0)
#drop un-needed columns
combined = combined.drop(columns=['year_right', 'year_left', 'index_left'])

combined.head(-4)
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
      <th>index</th>
      <th>year</th>
      <th>location_id</th>
      <th>count_id</th>
      <th>location</th>
      <th>lng</th>
      <th>lat</th>
      <th>centreline_type</th>
      <th>centreline_id</th>
      <th>motortraf</th>
      <th>geometry</th>
      <th>_id1</th>
      <th>HOOD_1512</th>
      <th>NEIGHBO13</th>
      <th>motorcol</th>
      <th>fatals</th>
      <th>injuries</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>1990.0</td>
      <td>3943.0</td>
      <td>2896</td>
      <td>OCONNOR DR AT ST CLAIR AVE (PX 447)</td>
      <td>-79.312836</td>
      <td>43.705396</td>
      <td>2.0</td>
      <td>13457092.0</td>
      <td>25543.0</td>
      <td>POINT (635935.311 4840535.754)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>1990.0</td>
      <td>3950.0</td>
      <td>2899</td>
      <td>KINGSTON RD AT BROOKLAWN AVE &amp; ST CLAIR AVE E ...</td>
      <td>-79.236019</td>
      <td>43.722032</td>
      <td>2.0</td>
      <td>13454337.0</td>
      <td>23234.0</td>
      <td>POINT (642085.246 4842512.259)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>1990.0</td>
      <td>3976.0</td>
      <td>3139</td>
      <td>EGLINTON SQ AT O CONNOR DR &amp; VICTORIA PARK AVE...</td>
      <td>-79.302100</td>
      <td>43.723380</td>
      <td>2.0</td>
      <td>13454288.0</td>
      <td>23507.0</td>
      <td>POINT (636759.408 4842550.798)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>1990.0</td>
      <td>3977.0</td>
      <td>3178</td>
      <td>DANFORTH AVE AT VICTORIA PARK AVE (PX 355)</td>
      <td>-79.288358</td>
      <td>43.691232</td>
      <td>2.0</td>
      <td>13459445.0</td>
      <td>22429.0</td>
      <td>POINT (637940.014 4839003.071)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>1990.0</td>
      <td>3978.0</td>
      <td>3243</td>
      <td>EGLINTON AVE AT VICTORIA PARK AVE (PX 451)</td>
      <td>-79.302645</td>
      <td>43.724723</td>
      <td>2.0</td>
      <td>13454107.0</td>
      <td>38630.0</td>
      <td>POINT (636712.455 4842699.059)</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>118091</th>
      <td>102845</td>
      <td>2024.0</td>
      <td>4207.0</td>
      <td>101620</td>
      <td>BAY ST AT HARBOUR SQ &amp; QUEENS QUAY (PX 2416)</td>
      <td>-79.376841</td>
      <td>43.641152</td>
      <td>2.0</td>
      <td>13468022.0</td>
      <td>9567.0</td>
      <td>MULTIPOINT (630919 4833297.275)</td>
      <td>685827.0</td>
      <td>166</td>
      <td>St Lawrence-East Bayfront-The Islands (166)</td>
      <td>16.0</td>
      <td>0.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>118092</th>
      <td>102846</td>
      <td>2024.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>MULTIPOINT (631090.311 4833389.868)</td>
      <td>686995.0</td>
      <td>166</td>
      <td>St Lawrence-East Bayfront-The Islands (166)</td>
      <td>15.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>118093</th>
      <td>102847</td>
      <td>2024.0</td>
      <td>38125.0</td>
      <td>101517</td>
      <td>FREELAND ST AT QUEENS QUAY E</td>
      <td>-79.373042</td>
      <td>43.642491</td>
      <td>2.0</td>
      <td>13467854.0</td>
      <td>7821.0</td>
      <td>MULTIPOINT (631223.027 4833455.654)</td>
      <td>686910.0</td>
      <td>166</td>
      <td>St Lawrence-East Bayfront-The Islands (166)</td>
      <td>4.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>118094</th>
      <td>102848</td>
      <td>2024.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>MULTIPOINT (630666.802 4833161.666)</td>
      <td>682244.0</td>
      <td>166</td>
      <td>St Lawrence-East Bayfront-The Islands (166)</td>
      <td>3.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>118095</th>
      <td>102849</td>
      <td>2024.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>MULTIPOINT (630657.159 4833172.821)</td>
      <td>685426.0</td>
      <td>166</td>
      <td>St Lawrence-East Bayfront-The Islands (166)</td>
      <td>8.0</td>
      <td>0.0</td>
      <td>1.0</td>
    </tr>
  </tbody>
</table>
<p>118096 rows × 17 columns</p>
</div>



#### Cleaning Collisions with Traffic Locaiton ID

By assigning each collision point a location ID that is used by the traffic dataset, we can recombine points that represent the same
location back into each other, after we merge back into the traffic dataset.


```python
#keep only data we need
traffic_collision = combined[['year', 'location_id', 'HOOD_1512', 'NEIGHBO13', 'centreline_id', 'motorcol', 'fatals', 'injuries']]

#add the collision data back into traffic data using the new shared ids
format_traf_col = traffic_26917.merge(traffic_collision, how='left', on=['year', 'location_id'], validate='1:m', suffixes=('_l', '_r'))

#aggregate by year and location id once more to combine collision data together on the same location
#the function grabs the first value in a datafram, since we dont care which we grab when aggregating.
def first_val(x):
    return x.iloc[0]

col_traf_agg = format_traf_col.groupby(['year','location_id']).agg({'count_id': first_val,
                                                                    'location': first_val,
                                                                    'centreline_id_r': first_val,
                                                                    'HOOD_1512': first_val,
                                                                    'NEIGHBO13': first_val,
                                                                    'motortraf': 'sum',
                                                                    'motorcol':'sum',
                                                                    'fatals':'sum',
                                                                    'injuries': 'sum',
                                                                    'geometry': first_val})
#reset to remove groups
col_traf_agg = col_traf_agg.reset_index()
#fix formatting
col_traf_agg = col_traf_agg.rename(columns={'centreline_id_r':'centreline_id_r'})

#our new data
traffic_collision = gpd.GeoDataFrame(col_traf_agg)
traffic_collision = traffic_collision.set_crs(epsg=26917)

#checking to see we still have data after the merge
col_traf_map = traffic_collision.query('motortraf > 0 & motorcol > 0')
col_traf_map.plot()
```




    <Axes: >




    
![png](Analysis_files/Analysis_40_1.png)
    


## Joining Traffic and Collision Point Data to Road line Data

We need to join collisions to roads. Since collisions are offset to the nearest intersection there will be multiple roads  
sharing the same intersection counts. 

Since we do not know which direction a car was coming from for collisions we assumed the same for traffic when combining the two.  
We will also assume that any intersection connected to a road contributes to its overall traffic and collisions.

This can be problamatic in cases where a minor road is attached to a major road, and the majority of collision  
at an intersection are caused by the major road. This may lead to inflated counts for minor roads, but unless a minor road  
is exlcusively surrounding by high traffic roads, it will still have an overall lower count when compared to major roads.

Major roads surrounded by minor roads will not be as influenced as we will assume that major roads   
will be connected to other major roads, and therefore have greater traffic counts when compared to minor roads.


```python
#create buffer around each road
road_26917_buffer = road_26917.copy()
road_26917_buffer['geometry'] = road_26917_buffer['geometry'].buffer(5)
road_26917_buffer.plot(color='blue')

```




    <Axes: >




    
![png](Analysis_files/Analysis_42_1.png)
    



```python
road_26917_buffer.head()
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
      <th>CENTREL2</th>
      <th>LINEAR_3</th>
      <th>LINEAR_4</th>
      <th>LINEAR_5</th>
      <th>LINEAR_26</th>
      <th>LINEAR_30</th>
      <th>FROM_IN31</th>
      <th>TO_INTE32</th>
      <th>FEATURE35</th>
      <th>FEATURE36</th>
      <th>JURISDI37</th>
      <th>OBJECTI39</th>
      <th>MI_PRIN40</th>
      <th>geometry</th>
      <th>calm_id</th>
      <th>spd_hump</th>
      <th>traf_islan</th>
      <th>spd_cush</th>
      <th>Installed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>914600</td>
      <td>2141</td>
      <td>Morrison St</td>
      <td>Morrison Street</td>
      <td>Morrison</td>
      <td>Morrison St</td>
      <td>13470555</td>
      <td>13470560</td>
      <td>201500</td>
      <td>Local</td>
      <td>CITY OF TORONTO</td>
      <td>1</td>
      <td>1</td>
      <td>POLYGON ((620277.174 4828210.441, 620276.699 4...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>914601</td>
      <td>2666</td>
      <td>Twelfth St</td>
      <td>Twelfth Street</td>
      <td>Twelfth</td>
      <td>Twelfth St</td>
      <td>13470560</td>
      <td>13470530</td>
      <td>201500</td>
      <td>Local</td>
      <td>CITY OF TORONTO</td>
      <td>2</td>
      <td>2</td>
      <td>POLYGON ((620230.306 4828339.006, 620230.177 4...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7862398</td>
      <td>2611</td>
      <td>Thirteenth St</td>
      <td>Thirteenth Street</td>
      <td>Thirteenth</td>
      <td>Thirteenth St</td>
      <td>13470571</td>
      <td>13470538</td>
      <td>201500</td>
      <td>Local</td>
      <td>CITY OF TORONTO</td>
      <td>3</td>
      <td>3</td>
      <td>POLYGON ((620150.489 4828311.727, 620150.362 4...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>914587</td>
      <td>1962</td>
      <td>Lake Shore Blvd W</td>
      <td>Lake Shore Boulevard West</td>
      <td>Lake Shore</td>
      <td>Lake Shore Blvd W</td>
      <td>13470546</td>
      <td>13470552</td>
      <td>201200</td>
      <td>Major Arterial</td>
      <td>CITY OF TORONTO</td>
      <td>6</td>
      <td>6</td>
      <td>POLYGON ((619527.788 4828252.379, 619527.315 4...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6735911</td>
      <td>1962</td>
      <td>Lake Shore Blvd W</td>
      <td>Lake Shore Boulevard West</td>
      <td>Lake Shore</td>
      <td>Lake Shore Blvd W</td>
      <td>13470552</td>
      <td>13470558</td>
      <td>201200</td>
      <td>Major Arterial</td>
      <td>CITY OF TORONTO</td>
      <td>7</td>
      <td>7</td>
      <td>POLYGON ((619439.831 4828222.367, 619439.359 4...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



By using a copy of the roads dataset, we can recreate the roads with a buffer of 5 meters. We then join any points within them  
combining any traffic and collision date within their range.


```python
#use buffer to get all point within the polygon
buffer_with_data = gpd.sjoin(road_26917_buffer, traffic_collision, how='right')
buffer_with_data = buffer_with_data[['CENTREL2', 'year', 'motortraf', 'motorcol','fatals','injuries']]

#aggregate
buffer_agg = buffer_with_data.groupby(['CENTREL2', 'year']).agg('sum')
buffer_agg = buffer_agg.reset_index()
#join back to roads
all_roads = road_26917.merge(buffer_agg, how='left', on=['CENTREL2'], validate='1:m', suffixes=('_l', '_r'))
```

plot the data to visually check that it is identical to the road_26917 dataframe, as it should be.


```python
all_roads.plot()
```




    <Axes: >




    
![png](Analysis_files/Analysis_47_1.png)
    


#### Calculating Collision Rate

For any road with both traffic counts and collison data we will find the rate of collision.
We will do it per 100,000 so that the values are readable, especially for roads that see very low  
rates of collision.


```python
#calculate the collision rate for each road
#collision rate is per 100,000 cars
all_roads['col_rate'] = all_roads['motorcol']/all_roads['motortraf']*100000

#for all calmed roads, calculate the collision rate before it was installed and after
all_roads['colrt_bfr'] = all_roads.query('Installed - year == 1 ')['col_rate']
all_roads['colrt_aft'] = all_roads.query('year - Installed == 1 ')['col_rate']

all_roads.head(-5)
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
      <th>CENTREL2</th>
      <th>LINEAR_3</th>
      <th>LINEAR_4</th>
      <th>LINEAR_5</th>
      <th>LINEAR_26</th>
      <th>LINEAR_30</th>
      <th>FROM_IN31</th>
      <th>TO_INTE32</th>
      <th>FEATURE35</th>
      <th>FEATURE36</th>
      <th>JURISDI37</th>
      <th>OBJECTI39</th>
      <th>MI_PRIN40</th>
      <th>geometry</th>
      <th>calm_id</th>
      <th>spd_hump</th>
      <th>traf_islan</th>
      <th>spd_cush</th>
      <th>Installed</th>
      <th>year</th>
      <th>motortraf</th>
      <th>motorcol</th>
      <th>fatals</th>
      <th>injuries</th>
      <th>col_rate</th>
      <th>colrt_bfr</th>
      <th>colrt_aft</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>914600</td>
      <td>2141</td>
      <td>Morrison St</td>
      <td>Morrison Street</td>
      <td>Morrison</td>
      <td>Morrison St</td>
      <td>13470555</td>
      <td>13470560</td>
      <td>201500</td>
      <td>Local</td>
      <td>CITY OF TORONTO</td>
      <td>1</td>
      <td>1</td>
      <td>LINESTRING (620365.778 4828243.398, 620275.681...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>914601</td>
      <td>2666</td>
      <td>Twelfth St</td>
      <td>Twelfth Street</td>
      <td>Twelfth</td>
      <td>Twelfth St</td>
      <td>13470560</td>
      <td>13470530</td>
      <td>201500</td>
      <td>Local</td>
      <td>CITY OF TORONTO</td>
      <td>2</td>
      <td>2</td>
      <td>LINESTRING (620275.681 4828215.213, 620235.062...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7862398</td>
      <td>2611</td>
      <td>Thirteenth St</td>
      <td>Thirteenth Street</td>
      <td>Thirteenth</td>
      <td>Thirteenth St</td>
      <td>13470571</td>
      <td>13470538</td>
      <td>201500</td>
      <td>Local</td>
      <td>CITY OF TORONTO</td>
      <td>3</td>
      <td>3</td>
      <td>LINESTRING (620195.613 4828188.199, 620155.247...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>914587</td>
      <td>1962</td>
      <td>Lake Shore Blvd W</td>
      <td>Lake Shore Boulevard West</td>
      <td>Lake Shore</td>
      <td>Lake Shore Blvd W</td>
      <td>13470546</td>
      <td>13470552</td>
      <td>201200</td>
      <td>Major Arterial</td>
      <td>CITY OF TORONTO</td>
      <td>6</td>
      <td>6</td>
      <td>LINESTRING (619614.26 4828286.233, 619526.218 ...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2019.0</td>
      <td>8614.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>23.218017</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>914587</td>
      <td>1962</td>
      <td>Lake Shore Blvd W</td>
      <td>Lake Shore Boulevard West</td>
      <td>Lake Shore</td>
      <td>Lake Shore Blvd W</td>
      <td>13470546</td>
      <td>13470552</td>
      <td>201200</td>
      <td>Major Arterial</td>
      <td>CITY OF TORONTO</td>
      <td>6</td>
      <td>6</td>
      <td>LINESTRING (619614.26 4828286.233, 619526.218 ...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2022.0</td>
      <td>7573.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>13.204807</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>99061</th>
      <td>60005935</td>
      <td>8574</td>
      <td>Neilson Rd</td>
      <td>Neilson Road</td>
      <td>Neilson</td>
      <td>Neilson Rd</td>
      <td>13445271</td>
      <td>13445040</td>
      <td>201300</td>
      <td>Minor Arterial</td>
      <td>CITY OF TORONTO</td>
      <td>259825</td>
      <td>259825</td>
      <td>LINESTRING (644396.132 4849419.184, 644326.929...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2018.0</td>
      <td>9464.0</td>
      <td>7.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>73.964497</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>99062</th>
      <td>60005967</td>
      <td>7835</td>
      <td>Ellesmere Rd</td>
      <td>Ellesmere Road</td>
      <td>Ellesmere</td>
      <td>Ellesmere Rd</td>
      <td>13445401</td>
      <td>13445166</td>
      <td>201200</td>
      <td>Major Arterial</td>
      <td>CITY OF TORONTO</td>
      <td>259827</td>
      <td>259827</td>
      <td>LINESTRING (644439.968 4849293.103, 645091.375...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1990.0</td>
      <td>15880.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>99063</th>
      <td>60005967</td>
      <td>7835</td>
      <td>Ellesmere Rd</td>
      <td>Ellesmere Road</td>
      <td>Ellesmere</td>
      <td>Ellesmere Rd</td>
      <td>13445401</td>
      <td>13445166</td>
      <td>201200</td>
      <td>Major Arterial</td>
      <td>CITY OF TORONTO</td>
      <td>259827</td>
      <td>259827</td>
      <td>LINESTRING (644439.968 4849293.103, 645091.375...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1994.0</td>
      <td>18448.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>99064</th>
      <td>60005967</td>
      <td>7835</td>
      <td>Ellesmere Rd</td>
      <td>Ellesmere Road</td>
      <td>Ellesmere</td>
      <td>Ellesmere Rd</td>
      <td>13445401</td>
      <td>13445166</td>
      <td>201200</td>
      <td>Major Arterial</td>
      <td>CITY OF TORONTO</td>
      <td>259827</td>
      <td>259827</td>
      <td>LINESTRING (644439.968 4849293.103, 645091.375...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1996.0</td>
      <td>17301.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>99065</th>
      <td>60005967</td>
      <td>7835</td>
      <td>Ellesmere Rd</td>
      <td>Ellesmere Road</td>
      <td>Ellesmere</td>
      <td>Ellesmere Rd</td>
      <td>13445401</td>
      <td>13445166</td>
      <td>201200</td>
      <td>Major Arterial</td>
      <td>CITY OF TORONTO</td>
      <td>259827</td>
      <td>259827</td>
      <td>LINESTRING (644439.968 4849293.103, 645091.375...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2001.0</td>
      <td>16623.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>99066 rows × 27 columns</p>
</div>



# Analysis

This Section of the notebook is concerned with the analysis of that data.  
We now have all the data we need, so we will be extracting the data  
based on the maps we will make the analysis we want.

Specifically we want data from 2023, as it is the most recent full years worth of data.
We also want max Traffic and Collision Rate from all time.

'all_traffic' will have the highest motor counts per road from any year, as well as the year it was achieved.


```python
#copy data from all_roads
all_traffic = all_roads[['CENTREL2', 'LINEAR_5', 'motortraf','geometry']]
#aggregate data to retrieve the highest motorcount (traffic) from any year
all_traffic = all_traffic.groupby(['CENTREL2']).agg({'LINEAR_5':pd.Series.mode, 'motortraf':'max', 'geometry': pd.Series.mode, })
all_traffic = all_traffic.reset_index()
#turn the dataframe back into a geodata fram after aggregating
all_traffic = gpd.GeoDataFrame(all_traffic)
all_traffic = all_traffic.set_crs(epsg=26917)
#retrieve the year that the motor count came from, by using the traffic count and road id
all_traffic = all_traffic.merge(all_roads[['CENTREL2','year', 'motortraf']], how='left', on=['motortraf', 'CENTREL2'])
all_traffic = all_traffic[['CENTREL2', 'LINEAR_5', 'year', 'motortraf','geometry']]
```

Like 'all_traffic', 'all_col_rate' will have the highest collision rate per road from any year, as well as the year it was achieved.


```python
#copy data from all_roads
all_col_rate = all_roads[['CENTREL2', 'LINEAR_5', 'col_rate','geometry']]
#aggregate data to retrieve the highest motorcount (traffic) from any year
all_col_rate = all_col_rate.groupby(['CENTREL2']).agg({'LINEAR_5':pd.Series.mode, 'col_rate':'max', 'geometry': pd.Series.mode, })
all_col_rate = all_col_rate.reset_index()
#turn the dataframe back into a geodata fram after aggregating
all_col_rate = gpd.GeoDataFrame(all_col_rate)
all_col_rate = all_col_rate.set_crs(epsg=26917)
#retrieve the year that the motor count came from, by using the traffic count and road id
all_col_rate = all_col_rate.merge(all_roads[['CENTREL2','year', 'col_rate']], how='left', on=['col_rate', 'CENTREL2'])
all_col_rate = all_col_rate[['CENTREL2', 'LINEAR_5', 'year', 'col_rate','geometry']]
```

'normal_roads' and 'calmed_roads' will have data only for that type of road for mapping purposes.

'normal_data' and 'calm_data' represent the same features, but for plotting purposes. 'data' being the two combined.


```python
#return only normal roads and format dataframe
normal_roads = all_roads.query('year == 2023 & Installed.isna()')
normal_roads['type'] = 'normal'
normal_roads['col_rate'] = round(normal_roads['col_rate'],2)

#return only calmed roads and format dataframe
calmed_roads = all_roads.query('year == 2023 & Installed < 2023')
calmed_roads['type'] = 'calmed'
calmed_roads['col_rate'] = round(calmed_roads['col_rate'],2)

#dataframes for the matplot
normal_data = normal_roads[['col_rate', 'type']]
calm_data = calmed_roads[['col_rate', 'type']]
#combined if needed
data = pd.concat([normal_data, calm_data])

#geo dataframes combined if needed
#map_data = pd.concat([normal_roads, calmed_roads])
#return columns that we want to show
#map_data = map_data[['LINEAR_5','year', 'FEATURE36','type', 'calm_id', 'Installed', 'motortraf', 'motorcol', 'col_rate', 'fatals', 'injuries', 'geometry']]

```

We gather the statistical summuries of the both data sets, which help us interpret the plots.


```python
data.groupby('type').describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead tr th {
        text-align: left;
    }

    .dataframe thead tr:last-of-type th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th colspan="8" halign="left">col_rate</th>
    </tr>
    <tr>
      <th></th>
      <th>count</th>
      <th>mean</th>
      <th>std</th>
      <th>min</th>
      <th>25%</th>
      <th>50%</th>
      <th>75%</th>
      <th>max</th>
    </tr>
    <tr>
      <th>type</th>
      <th></th>
      <th></th>
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
      <th>calmed</th>
      <td>39.0</td>
      <td>58.858205</td>
      <td>139.940272</td>
      <td>0.0</td>
      <td>0.00</td>
      <td>29.58</td>
      <td>55.68</td>
      <td>862.07</td>
    </tr>
    <tr>
      <th>normal</th>
      <td>2444.0</td>
      <td>93.091919</td>
      <td>424.878205</td>
      <td>0.0</td>
      <td>14.91</td>
      <td>51.07</td>
      <td>106.40</td>
      <td>13413.17</td>
    </tr>
  </tbody>
</table>
</div>




```python
#set our graph style
sns.set_style('whitegrid')
```

A frequency Histogram helps visualize the distributions of how often  
certain collison rates occur. Here we see that most roads have a low collision rate,  
which is expected, if not apprecaited.

Both datasets share a visually similar distribution.  
We also see that the frequency slowly ramps down for both, visually similar to a right  
skewed normal distribution graph.


```python
sns.displot(data=data, x="col_rate",hue='type', kde=True)
plt.yscale("symlog")
plt.xlim(0, 1600)
```




    (0.0, 1600.0)




    
![png](Analysis_files/Analysis_61_1.png)
    


Once again, in the box plots we see that both data sets are skewed right,  
with a greater distribution around lower collision rates. 


```python
ax = sns.boxplot(x='col_rate',y='type', hue=data['type'],data=data)
plt.xscale('log')
```


    
![png](Analysis_files/Analysis_63_0.png)
    


Violin plots with a strip plot provide similar data to boxplots,  
but helps better visualize where the data groups around.


```python
ax = sns.violinplot(x='col_rate',y='type', hue=data['type'], data=data)
ax = sns.stripplot(x='col_rate',y='type', hue=data['type'], data=data)
plt.xscale('log')
```


    
![png](Analysis_files/Analysis_65_0.png)
    


Probability plots are also great tools to identify distributions of data.  
In this case we also see a heavy distribution of low collison rate streets.

More importantly we see that the data does not follow a normal distribution.  
The further from the mean you go the more exteme the collision rates should be,   
Whether lower or higher. This would result in a straigt line, as data would be evenly  
distributed from z scores of -3 to 3.

This alligns with what we saw in the above graphs, that data is bunched towards  
the lower values. But in both cases they follow a similar distribution


```python
ax1 = plt.subplot(211)
res = scipy.stats.probplot(normal_data['col_rate'], plot=plt) 

ax2 = plt.subplot(212)
res2 = scipy.stats.probplot(calm_data['col_rate'], plot=plt)

ax1.set_title("Probility Plot for Normal Roads")
ax1.set_ylabel('Collision Rate per 100,000')

ax2.set_title("Probility Plot for Calmed Roads")
ax2.set_ylabel('Collision Rate per 100,000')

ax1.set_ylim([0, 800]) #to zoom in and see data
ax2.set_ylim([0, 300]) #to zoom in and see data

plt.tight_layout()

```


    
![png](Analysis_files/Analysis_67_0.png)
    


### Two Sample T-test

We will perform a two sample right tail t-test. What this means is that we will assert that:
- Null hypothesis: Both means are the same, or
- Alternative hypothesis: The mean for collision rates on normal roads is higher

It is two sample as both normal road data and calm road data are samples of all road data found in Toronto,  
something that is unknown, since not every road has data.

With the information we have we know that the data skews positively and that from the  
statistics summary the variation data differs to much from one another, so the t-test  
must be an unequal variance t-test, also known as Welch’s t-test.

Our alpha will be set to 0.05, meaning we are 95% confident.  
If the p-value from our result is lower than the alpha we reject  
the null hypothesis and accept the alternative hypothesis.



```python
alpha= 0.05

t_statistic, p_value = pystat.ttest_ind(normal_data['col_rate'], calm_data['col_rate'], alternative='greater', equal_var=False)


# Output the results
print(f"t-statistic: {t_statistic}")
print(f"P-value: {p_value}")

if p_value < alpha:
    print("There is a significant difference between the collison rates.")
else:
    print("There is no significant difference between the collsion rates.")
```

    t-statistic: 1.4264065905488366
    P-value: 0.07998370304737284
    There is no significant difference between the collsion rates.


From these results we can assert that the mean of both groups are not different enough that they represent different  
populations. Because the p_value > 0.05, the average of collision rates for normal roads is equal to that of calmed roads.

But just because calmed roads have an equal rate of collisions, it doesn't mean that calming features don't have an effect.  
There could be a variety of factors. We can map the data and look for reasons as to why.


```python
popup_data = ["LINEAR_5",'JURISDI37','calm_id', 'spd_hump', 'traf_islan','spd_cush', "type", 'year', 'motortraf','Installed', 'motorcol', "col_rate"]
m = normal_roads.explore(
    column="col_rate",
    scheme="naturalbreaks",  # use mapclassify's natural breaks scheme
    legend=True, 
    cmap = 'spring_r',
    tooltip=["LINEAR_5", "type",'motortraf', 'motorcol', "col_rate"],
    popup=popup_data,  # show popup (on-click)
    name="Normal Roads Collision Rates in 2023",  # name of the layer in the map
    legend_kwds={'colorbar':True,
                'caption':'Normal Roads Collision Rate per 100,000',
                'scale':False,
                },
    style_kwds={'weight': 5}
)

m = normal_roads.explore(
    column="motortraf",
    scheme="naturalbreaks",  # use mapclassify's natural breaks scheme
    legend=True, 
    cmap = 'spring_r',
    tooltip=["LINEAR_5", "type",'motortraf', 'motorcol', "col_rate"],
    popup=popup_data,  # show popup (on-click)
    name="Normal Roads Traffic Count in 2023",  # name of the layer in the map
    legend_kwds={'colorbar':True,
                'caption':'Normal Roads Traffic Count',
                'scale':False,
                },
    style_kwds={'weight': 5},
    m=m
)

m = calmed_roads.explore(
    column="col_rate", 
    scheme="naturalbreaks",  # use mapclassify's natural breaks scheme
    legend=True,  
    cmap='summer_r',
    tooltip=["LINEAR_5", "type",'motortraf', 'motorcol', "col_rate"],
    popup=popup_data,  # show popup (on-click)
    name="Calmed Roads Collision Rates in 2023",  # name of the layer in the map
    legend_kwds={'colorbar':True,
                'caption':'Calmed Roads Collision Rate per 100,000',
                'scale':False,
                },
    style_kwds={'weight': 5},
    m=m
)

m = calmed_roads.explore(
    column="motortraf",
    scheme="naturalbreaks",  # use mapclassify's natural breaks scheme
    legend=True,  # show legend
    cmap = 'summer_r',
    tooltip=["LINEAR_5", "type",'motortraf', 'motorcol', "col_rate"],
    popup=popup_data,  # show popup (on-click)
    name="Calmed Roads Traffic Count in 2023",  # name of the layer in the map
    legend_kwds={'colorbar':True,
                'caption':'Calmed Roads Traffic Count',
                'scale':False,
                },
    style_kwds={'weight': 5},
    m=m
)



folium.LayerControl().add_to(m)


#file is too large for notebook
m.save('Toronto_Traffic_Collisions.html')
```

The map 'Toronto_Traffic_Collisions.html' shows that many of the calmed roads with high collision rates happen to be near roads with high traffic, taken from the all_traffic layer.

What is the correlation between the two?


```python
correlation_data = calmed_roads[['motortraf', 'motorcol', 'col_rate', 'fatals', 'injuries']]
correlation_data.apply(lambda x : pd.factorize(x)[0]).corr(method='pearson', min_periods=1)
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
      <th>motortraf</th>
      <th>motorcol</th>
      <th>col_rate</th>
      <th>fatals</th>
      <th>injuries</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>motortraf</th>
      <td>1.000000</td>
      <td>0.312909</td>
      <td>0.669447</td>
      <td>NaN</td>
      <td>0.066424</td>
    </tr>
    <tr>
      <th>motorcol</th>
      <td>0.312909</td>
      <td>1.000000</td>
      <td>0.707355</td>
      <td>NaN</td>
      <td>0.562448</td>
    </tr>
    <tr>
      <th>col_rate</th>
      <td>0.669447</td>
      <td>0.707355</td>
      <td>1.000000</td>
      <td>NaN</td>
      <td>0.352848</td>
    </tr>
    <tr>
      <th>fatals</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>injuries</th>
      <td>0.066424</td>
      <td>0.562448</td>
      <td>0.352848</td>
      <td>NaN</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
correlation_data = normal_roads[['motortraf', 'motorcol', 'col_rate', 'fatals', 'injuries']]
correlation_data.apply(lambda x : pd.factorize(x)[0]).corr(method='pearson', min_periods=1)
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
      <th>motortraf</th>
      <th>motorcol</th>
      <th>col_rate</th>
      <th>fatals</th>
      <th>injuries</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>motortraf</th>
      <td>1.000000</td>
      <td>0.091351</td>
      <td>0.715786</td>
      <td>-0.039840</td>
      <td>-0.035064</td>
    </tr>
    <tr>
      <th>motorcol</th>
      <td>0.091351</td>
      <td>1.000000</td>
      <td>0.208260</td>
      <td>0.047986</td>
      <td>0.566223</td>
    </tr>
    <tr>
      <th>col_rate</th>
      <td>0.715786</td>
      <td>0.208260</td>
      <td>1.000000</td>
      <td>-0.011151</td>
      <td>0.092918</td>
    </tr>
    <tr>
      <th>fatals</th>
      <td>-0.039840</td>
      <td>0.047986</td>
      <td>-0.011151</td>
      <td>1.000000</td>
      <td>0.036139</td>
    </tr>
    <tr>
      <th>injuries</th>
      <td>-0.035064</td>
      <td>0.566223</td>
      <td>0.092918</td>
      <td>0.036139</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>



We can see for both types of roads, motor traffic does indeed have a high correlation with collision rate. Another observation is that  
although the number of collisions is moderately correlated to traffic in normal roads, there is a higher correlation in calmed roads. The same  
is true for traffic and collision rates.

In both types of roads, the correlation between collison and injuries are nearly the same, with them being strongly correlated. This seems intuive  
until you consider that in both types fatalities are weakly correlated. This could be due to the fact that those who get transported  
to hospitals are not recorded as fatalities, so only the extreme cases which may happen rarely remain.  

Ultimately from these observations we can infer that calmed roads in general are safer, with a lower average for collision rates.   
This does not extend to the observation that our t-test was wrong, but rather they may have been other factors at play the influenced    
the outcome. 

But if we were to implment road calming features, which roads would we choose?


```python
#we made the normal roads traffic again
m = normal_roads.explore(
    column="motortraf",
    scheme="naturalbreaks",  # use mapclassify's natural breaks scheme
    legend=True,  # show legend
    cmap = 'spring_r',
    tooltip=["LINEAR_5", "type","motortraf", "motorcol", "col_rate"],
    popup=True,  # show popup (on-click)
    name="2023 Normal Roads Traffic Count",  # name of the layer in the map
    legend_kwds={'colorbar':True,
                'caption':'Traffic Count',
                'scale':False,
                },
    style_kwds={'weight': 5}
)
m = calmed_roads.explore(
    column="motortraf",
    scheme="naturalbreaks",  # use mapclassify's natural breaks scheme
    legend=True,  # show legend
    cmap = 'summer_r',
    tooltip=["LINEAR_5", "type",'motortraf', 'motorcol', "col_rate"],
    popup=True,  # show popup (on-click)
    name="Calmed Roads Traffic Count",  # name of the layer in the map
    legend_kwds={'colorbar':True,
                'caption':'2023 Calmed Roads Traffic Count',
                'scale':False,
                },
    style_kwds={'weight': 5},
    m=m
)

folium.LayerControl().add_to(m)


#file is too large for notebook
m.save('Toronto_Traffic.html')
```

The above map 'Toronto_Traffic.html' highlights high traffic roads both with and without calming features.  
The normal roads without calming features could be considered candidates.

And finally what were the highest traffic counts and collision rates for roads throughout the years?\
How do calmed roads compare to this?


```python
m = all_traffic.explore(
    column="motortraf",
    scheme="naturalbreaks",  # use mapclassify's natural breaks scheme
    legend=True, 
    cmap = 'spring_r',
    tooltip=["LINEAR_5",'year', 'motortraf'],
    popup=True,  # show popup (on-click)
    name="Highest Traffic Counts",  # name of the layer in the map
    legend_kwds={'colorbar':True,
                'caption':'Highest Traffic Count',
                'scale':False,
                },
    style_kwds={'weight': 5}
)

m = calmed_roads.explore(
    column="motortraf",
    scheme="naturalbreaks",  # use mapclassify's natural breaks scheme
    legend=True,  # show legend
    cmap = 'summer_r',
    tooltip=["LINEAR_5", "type",'motortraf'],
    popup=True,  # show popup (on-click)
    name="2023 Calmed Roads Traffic Count",  # name of the layer in the map
    legend_kwds={'colorbar':True,
                'caption':'Calmed Roads Traffic Count',
                'scale':False,
                },
    style_kwds={'weight': 5},
    m=m
)
folium.LayerControl().add_to(m)
m.save('Highest_Traffic_Toronto.html')
```


```python
m = all_col_rate.explore(
    column="col_rate",
    scheme="naturalbreaks",  # use mapclassify's natural breaks scheme
    legend=True, 
    cmap = 'spring_r',
    tooltip=["LINEAR_5",'year', 'col_rate'],
    popup=True,  # show popup (on-click)
    name="Highest Collision Rates",  # name of the layer in the map
    legend_kwds={'colorbar':True,
                'caption':'Highest Traffic Count',
                'scale':False,
                },
    style_kwds={'weight': 5}
)

m = calmed_roads.explore(
    column="col_rate",
    scheme="naturalbreaks",  # use mapclassify's natural breaks scheme
    legend=True,  # show legend
    cmap = 'summer_r',
    tooltip=["LINEAR_5", "type",'col_rate'],
    popup=True,  # show popup (on-click)
    name="2023 Calmed Roads Collision Rates",  # name of the layer in the map
    legend_kwds={'colorbar':True,
                'caption':'Calmed Roads Traffic Count',
                'scale':False,
                },
    style_kwds={'weight': 5},
    m=m
)
folium.LayerControl().add_to(m)
m.save('Highest_Collision_Rate_Toronto.html')
```

For 'Highest_Collision_Rate_Toronto.html' and 'Highest_Traffic_Toronto.html':  

Some Calmed Roads from 2023 coincide with the highest traffic counts of all time,  
such as on Balmoral Avenue and Merton Street, but mostly every other calmed road has decreased from its peak.

When examining Collision Rates, the same two roads have lower collision rates in 2023, than they do at their peak.  
This shows evidence that calmed features are doing their job and the with increased traffic, these two roads are  
now safer. Some exceptions being Vesta Drive and Mayfair Avenue, which now have higher collison rates, although their traffic  
counts are down from their peak. 



## Conclusion

So how do our observations stack up against our t-test result? There may have been a variety of factors at play, 
but the main one to consider is our decision to aggregate traffic and collsion data with road data, regardless of direction.

A major observation in each of the maps above is that many minor roads off of major roads had a higher number of traffic  
and collision. This is to be expected, but at times they were on par with the major roads. This is most likely an error
with our assumption that minor roads would not be as effectinv by the grouping of traffic and collision data.

It is difficult to recitfy this as collision data is on a per interseciton bases, and the directional data for traffic  
is not geographcially tied, but rather a seperate field for whther or not a vehicle was entering or leaving the intersection.  

There can be room for improvenment in our analysis, but as it stands this notebook offers a showcase of the methods, and reasoning  
used to reach our conslusion, that the mean rate of collision of both groups are equal.



###### This Notebook contains original analysis by Nelson Alesandro, 2024.

###### Data within this notebook is from the City of Torontos Open Data set.
