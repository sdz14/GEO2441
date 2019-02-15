## Practical 5 - Google Earth Engine for Time Series and Vegetation Indices

Today we will use Google Earth Engine in order to create and graph a simple time series, identify changes in land cover using vegetation indices, and construct and compare vegetation indices from raw satellite imagery. 

## Quick Earth Engine refresher

Google Earth Engine is a cloud based satellite image processing platform. It allows us to process very large volumes of satellite imagery, without the need to download and mosaic ("stitch") the images together manually ourselves. This saves a lot of time and removes the constraints of hard disk space.

It's Integrated Development Environment (IDE) uses the JavaScript programming language (with the option of using Python - which is beyond the scope of this practical), to manipulate and analyse the satellite imagery.


### Task 1 - 

For the first part of the practical, we will be using data from the Moderate Resolution Imaging Spectroradiometer (MODIS) instrument onboard NASA's Terra and Aqua satellites. MODIS provides us with quite coarse spatial resolution data, which is useful for monitoring land cover change on a larger (e.g. country-wide) scale. As a result of its somewhat low spatial resolution, it has a very good temporal resolution of 1-2 days. For more details about MODIS, refer to NASA's Terra satellite website *[here](https://terra.nasa.gov/about/terra-instruments/modis)*

Using MODIS data, we will derive and compare two Vegetation Indices and see how they relate to each other. Firstly, we will derive Normalized Difference Vegetation Index (NDVI), which is the de facto gold standard of VIs. Following this, we will derive a the Modified Soil-adjusted Vegetation Index (MSAVI), specifically, its second, improved, iteration MSAVI2. 

The mathematics behind both of the Vegetation Indices can be seen below, 

Normalized Difference Vegetation Index (NDVI):

![Normalized Difference Vegetation Index](https://raw.githubusercontent.com/sdz14/GEO2441/master/LaTeX%20Equations/normalised_diff_veg_index.png)

Modified Soil Adjusted Vegetation Index 2 (MSAVI2):

![Modified Soil Adjusted Vegetation Index 2](https://raw.githubusercontent.com/sdz14/GEO2441/master/LaTeX%20Equations/msavi2.png)

Vegetation Indices are quite simple and easy way to monitor vegetation cover, health and dynamics, and hence are very useful for environmental monitoring.


The first task will be to derive both of the vegetation indices, create a landcover map for your chosen country and then proceed to plot the two vegetation indices against each other to identify their (if any) relationship. 


* Fire up Google Earth Engine by clicking *[here](https://code.earthengine.google.com/)*, log in and you will be greeted with a, hopefully, familiar sight. 

* We will need to import the country dataset. Luckily, Google Earth Engine provides the The United States Office of the Geographer's "LSIB: Large Scale International Boundary Polygons" dataset for us. 
Go ahead and import the dataset by: 

```javascript
var countryShps = ee.FeatureCollection("USDOS/LSIB_SIMPLE/2017")
                                  .filter(ee.Filter.eq("country_co", "US"));
```
Breaking the code down, we can see that we declared our variable using `var countryShps` and assigned it to `ee.FeatureCollection("USDOS/LSIB_SIMPLE/2017")`. After this, we "filtered" the dataset to only show the country we are interested in (in my case USA) by using a "filter" method `.filter(ee.Filter.eq("country_co", "US")`. 

If you imagine the shapefiles we are using as a database table with columns and rows corresponding to a result, the "country_co" argument tells Earth Engine to choose from the Country code column within the table, while the "US" argument picks out a country from the said column.

Right, if you did everything correctly, you should have an imported dataset, filtered to your country of interest, which we will use to crop (or "clip" in GIS lingo) our MODIS imagery. 

* Let's import our MODIS imagery, filter it, and clip it to our country of interest. 

```javascript
var modisData = ee.ImageCollection("MODIS/006/MOD09A1")
                                  .filter(ee.Filter.date('2010-10-01', '2010-11-01'))
                                  .median()
                                  .clip(countryShps);
```

Like last time, we declared our variable "modisData", and filtered it to only use imagery between the dates we specified. Then we picked the median image, using the `.median()` method, and lastly, we "clipped" it to the country which we are interested in. Perfect! Hopefully the universe is on your side and if you enter the following code: 

```javascript
Map.addLayer(modisData, {min: -100, max: 16000}, "MODIS Dataset");
```

You should have something that looks like this:

![Earth Engine Screenshot 1](https://raw.githubusercontent.com/sdz14/GEO2441/master/screenshots/earthengine_sc.png)

