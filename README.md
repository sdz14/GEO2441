## Practical 5 - Google Earth Engine for Time Series and Vegetation Indices

Today we will use Google Earth Engine in order to create and graph a simple time series, identify changes in land cover using vegetation indices, and construct and compare vegetation indices from raw satellite imagery. 

## Quick Earth Engine refresher

Google Earth Engine is a cloud based satellite image processing platform. It allows us to process very large volumes of satellite imagery, without the need to download and mosaic ("stitch") the images together manually ourselves. This saves a lot of time and removes the constraints of hard disk space.

It's Integrated Development Environment (IDE) uses the JavaScript programming language (with the option of using Python - which is beyond the scope of this practical), to manipulate and analyse the satellite imagery.

You can read more about Earth Engine *[here](https://developers.google.com/earth-engine/)

## Task 1 - 

For the first part of the practical, we will be using data from the Moderate Resolution Imaging Spectroradiometer (MODIS) instrument onboard NASA's Terra and Aqua satellites. MODIS provides us with quite coarse spatial resolution data, which is useful for monitoring land cover change on a larger (e.g. country-wide) scale. As a result of its somewhat low spatial resolution, it has a very good temporal resolution of 1-2 days. For more details about MODIS, refer to NASA's Terra satellite website *[here](https://terra.nasa.gov/about/terra-instruments/modis)*

Using MODIS data, we will derive and compare two Vegetation Indices and see how they relate to each other. Firstly, we will derive Normalized Difference Vegetation Index (NDVI), which is the de facto gold standard of VIs. Following this, we will derive a the Modified Soil-adjusted Vegetation Index (MSAVI), specifically, its second, improved, iteration MSAVI2. 

The mathematics behind both of the Vegetation Indices can be seen below, 

Normalized Difference Vegetation Index (NDVI):

![Normalized Difference Vegetation Index](https://raw.githubusercontent.com/sdz14/GEO2441/master/LaTeX%20Equations/normalised_diff_veg_index.png)

Modified Soil Adjusted Vegetation Index 2 (MSAVI2):

![Modified Soil Adjusted Vegetation Index 2](https://raw.githubusercontent.com/sdz14/GEO2441/master/LaTeX%20Equations/msavi2.png)

Vegetation Indices are quite simple and easy way to monitor vegetation cover, health and dynamics, and hence are very useful for environmental monitoring (Xue, J. and Baofeng, S., 2017).


The first task will be to derive both of the vegetation indices, create a landcover map for your chosen country and then proceed to plot the two vegetation indices against each other to identify their (if any) relationship. 


* Fire up Google Earth Engine by clicking *[here](https://code.earthengine.google.com/)*, log in and you will be greeted with a, hopefully, familiar sight. 

* We will need to import the country dataset. Luckily, Google Earth Engine provides the The United States Office of the Geographer's "LSIB: Large Scale International Boundary Polygons" dataset for us. 
Go ahead and import the dataset by: 

```javascript
var countryShps = ee.FeatureCollection("USDOS/LSIB_SIMPLE/2017")
                                  .filter(ee.Filter.eq("country_co", "US"));
```
Breaking the code down, we can see that we declared our variable using `var countryShps` and assigned it to `ee.FeatureCollection("USDOS/LSIB_SIMPLE/2017")`. After this, we "filtered" the dataset to only show the country we are interested in (in my case USA) by using a "filter" method `.filter(ee.Filter.eq("country_co", "US")`.

If you imagine the shapefiles we are using as a database with columns and rows corresponding to a result, the "country_co" argument tells Earth Engine to choose from the Country code column within the table, while the "US" argument picks out a country from the said column.

Go ahead and pick the country you are interested by changing out the second argument within the filter function. Refer here for the FIPS Country codes: https://en.wikipedia.org/wiki/List_of_FIPS_country_codes . 

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

* Now, onto the hard part. Let's create the NDVI image from the raw MODIS imagery. 

```javascript
var modisNDVI = modisData.expression(
                          "((NIR - RED) / (NIR + RED))", 
                          {"NIR": modisData.select("sur_refl_b02"),
                           "RED": modisData.select("sur_refl_b01")
                          })
                          .float();
```

Now lets, break the code snippet down:

Firstly, we declare a variable, `modisNDVI`, as before. Then we use the `.expression()` method and input the NDVI equation. Whilst still inside the `.expression()` method, we need to declare a "dictionary", using curly brackets {}. Inside this dictionary, we tell Earth Engine, which of the satellite image bands it needs to use for the expression. 
A dictionary is simply a "list" where a "key" corresponds to a value. For example, the key "NIR" corresponds to the Near Infrared band of our MODIS image. Hence we choose `"NIR" : modisData.select("sur_refl_b02")`, because the "sur_refl_b02" is the Near Infrared band of our MODIS image. Lastly, we need to make sure that the values which we create are understood and represented correctly by the computer, hence we use the `.float()` method to convert the numbers into "floating point" numbers.

Now, using: `Map.addLayer(modisNDVI, {min: ???, max: ???, palette: ['ffffff', '119701', '000000']}, "MODIS NDVI")` lets add the dataset we just created to the Map. Replace the `???` with the range of values you think NDVI should correspond to. 

If you are not sure, a quick Google search should point you to the right answer.

Did it work? You should have something similar to this: 

![Earth Engine Screenshot 2](https://raw.githubusercontent.com/sdz14/GEO2441/master/screenshots/earthengine_sc_ndvi.png)

* We are powering through, Now let's do the same thing we did above for MSAVI2. 

```javascript
var modisMSAVI = modisData.expression(
                "(2 * ??? + 1 - sqrt(pow((2 * ??? + 1), 2) - 8 * (??? - ???))) / 2",
                {???: modisData.select(???),
                 ???: modisData.select(???)
                })
                .???();
```

Here's the MSAVI equation again, to help you plug in the right parameters: 

![Modified Soil Adjusted Vegetation Index 2](https://raw.githubusercontent.com/sdz14/GEO2441/master/LaTeX%20Equations/msavi2.png)

1. Replace the `???` in the expression method with the relevant bands.
2. Define the relevant bands used in the expression in the dictionary (Hint: Make sure to wrap the parameters in quotes, like so "BAND"). 
3. Replace the `???` in `modisData.select()` with the relevant bands from the MODIS dataset. (Hint: inputting "MODIS/006/MOD09A1" into the search bar of Earth Engine will bring up the description and metadata for the MODIS dataset we are using - use the wavelength values to determine which wavebands are relevant - once again quick Google search for Electromagnetic Spectrum wavelengths is your friend).
4. Convert the resultant dataset to use floating point numbers.
5. Add your result to the Map, like you did with NDVI.

Once again, if all goes well, you should have something similar to this: 

![Earth Engine Screenshot 3](https://raw.githubusercontent.com/sdz14/GEO2441/master/screenshots/earthengine_msavi.png)


* Now, lets plot a graph showing how related well NDVI and MSAVI relate. 

Now, let's sample some pixels and store their values. To do this, we will use the "Add a Marker" tool, to create a "MultiPoint" geometry collection. It can be found in the left hand corner of the Map window underneath the text editor (the icon looks like a little pin). 
Now go ahead and lay down a whole lot of pins spaced out well across your NDVI/MSAVI study area. Try to get at least 50 (preferably more) pins down to get an accurate representation of the relationship. My sampling looks a little like this: 

![Earth Engine Screenshot 4](https://raw.githubusercontent.com/sdz14/GEO2441/master/screenshots/earthengine_sampling.png)

(Hint: Earth Engine will save your points under a default name "geometry". I highly recommend changing this to something more specific. You can find your saved geometries in the "Imports" section at the top of the text editor.)

* Okay, let's get the values which we sampled into a dictionary. 

Let's "reduce" the rasters which we have produced to a dictionary of NDVI/MSAVI pixel values. We can do this by assigning the following code to our declared pixel dictionary: 

`var ndviPixels = modisNDVI.reduceRegion(ee.Reducer.toList(), ???, 160);`
Replace `???` with the name of your geometry.
Now, let's do the same thing for our MSAVI pixels. 

* Right, almost done. We now must convert the dictionary of the pixel values into an array. 

We can do this quite simply by finding out what the name of the "object" we have created is. Firstly, print the result of the dictionary to the console by calling: 

`print("NDVI Pixels", ndviPixels);`

This should output a dictionary of the pixel values to the console, like so: 

![Earth Engine Screenshot 5](https://raw.githubusercontent.com/sdz14/GEO2441/master/screenshots/earthengine_print.png)

As you can see, the pixel values dictionary inherited the name of one of the bands we have used in our calculation of NDVI, though this might be different for you.

To convert this dictionary into an array which we can use to plot the pixel values, we need to call: 

```javascript
var ndviValues = ee.Array(ndviPixels.get("sur_refl_b02"));
```
Now, let's do the same for MSAVI values. 

* Finally, we're at the end of the journey. Let's plot NDVI/MSAVI against each other and see how well they relate. 

```javascript
var chart = ui.Chart.array.values(???, 0, ???).setSeriesNames(["??? vs ???"]).setOptions({
      title: 'MODIS NDVI vs MODIS MSAVI',
      hAxis: {'title': 'MODIS ???'},
      vAxis: {'title': 'MODIS ???'},
      pointSize: 3,
});

print(chart);
```

Now, let's replace the `???` with the relevant arguments. Let's plot the NDVI on the X Axis and the MSAVI on the Y Axis. 

Finished product: 

![Earth Engine Screenshot 6](https://raw.githubusercontent.com/sdz14/GEO2441/master/screenshots/earthengine_plot.png)

So, we can see that there is a general positive relationship between the two Vegetation Indices. Any idea why this could be? 

If you made it this far, congratulations. 

Take a breather for a few minutes, then open a new project...


#### References: 
1. Jinru Xue and Baofeng Su, “Significant Remote Sensing Vegetation Indices: A Review of Developments and Applications,” Journal of Sensors, vol. 2017, Article ID 1353691, 17 pages, 2017. [https://doi.org/10.1155/2017/1353691](https://doi.org/10.1155/2017/1353691).
2. Wikipedia - "List of FIPS country codes" - https://en.wikipedia.org/wiki/List_of_FIPS_country_codes
