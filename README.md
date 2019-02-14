## Practical 5 - Google Earth Engine for Time Series and Vegetation Indices

Today we will use Google Earth Engine in order to create and graph a simple time series, identify changes in land cover using vegetation indices, and construct and compare vegetation indices from raw satellite imagery. 

## Quick Earth Engine refresher

Google Earth Engine is a cloud based satellite image processing platform. It allows us to process very large volumes of satellite imagery, without the need to download and mosaic ("stitch") the images together manually ourselves. This saves a lot of time and removes the constraints of hard disk space.

It's Integrated Development Environment (IDE) uses the JavaScript programming language (with the option of using Python - which is beyond the scope of this practical), to manipulate and analyse the satellite imagery.


### Task 1 - 
Fire up Google Earth Engine by clicking *[here](https://code.earthengine.google.com/)*, where you will be greeted with a, hopefully, familiar sight. 

Normalized Difference Vegetation Index (NDVI):

![Normalized Difference Vegetation Index](https://raw.githubusercontent.com/sdz14/GEO2441/master/LaTeX%20Equations/normalised_diff_veg_index.png)

Enhanced Vegetation Index (EVI):

![Enhanced Vegetation Index](https://raw.githubusercontent.com/sdz14/GEO2441/master/LaTeX%20Equations/enhanced_veg_index.png)
