This is a guide for users on how to run the codes we developed, if you would like to test it in another metropolitan area in Brazil or across the globe. 
We divided this tutorial according to the scripts on [GEE](https://code.earthengine.google.com/?accept_repo=users/lorrainetoliveira/cfp-project).
It is important to state again that the features used will surely differ from place to place. The pool of datasets we used is derived from months of research together with local experts. Some features will likely be used to other contexts such as DEM or distance to natural areas, the interpretation of the features values in relation to deprivation might change. 

## 1a_prep_AGSNbuffer
This code is twofold. First, it loads the two main base layers of the model the layer with the extent of the study area and the layer with the extent of the local deprived settlements. Second, it excludes non-deprived pixels from the training sample, that might be noise source, by applying a buffer function. 
Here the user need to upload the local layers and determine the unit of analysis (grid), because the buffer would need to be half of such value to exclude pixels only partially inside the deprived settlements extent. We used 20x20m grid, therefore our buffer is 10m. 
The code will export the resulting layer (after buffer) as an asset.

## 1b_prep_OSMmerge 
We provided this code to process the OSM datasets that are download outside the GEE catalog. As the Region of Interest (ROI) is quite large, the HOTOSM platform required four tiles to encompass the entire study area' extent. This script merged the tiles per feature type (points, lines and polygons) and exported them as assets as shown in the screenshot. If the ROI is larger, requiring more than than four tiles, another variables would need to be added. For instance, following our code *'var osmline3 = OSML5.merge(OSML6);'*. If the ROI of the user has only one tile, this script can be skipped. 
![image](https://user-images.githubusercontent.com/101252763/194058050-2158ad1a-3462-4a30-b14e-fa52588f4a46.png)

## 1c_prep_deadends
This code was created to locate and extract the dead end streets (in specific the dangling points) from HOTOSM line feature layer. This process is not automated in any GIS software which makes this a novel approach. The only change the user needs to do here is, by testing,  to substitute the variable 'osmSimplified' for 'osmRandomColumn'. This will depend on the complexity and completeness of the road network (OSM line feature) of the study area.  
![image](https://user-images.githubusercontent.com/101252763/194060643-882b3358-6b98-479f-98ed-8811a276549d.png)
The script will export the resulting point layer as an asset that is further used at '2c_input_KDE' to calculate the kernel density estimation of the dangling points.

## 2a_input_RSdata
This script extracted the features from Remote Sensing data, respectively  Normalized Diffence Vegetation Index (NDVI), DEM, Slope, Population count, Night-time lights (VIIRS-DNB Band). Here the user will need to change the acquisition date (*FilterDate'*) accordingly. Since the deprived settlments layer from Brazil is from 2019, we tried to maintain temporal consistency. 
The user might also need to change the *'CLOUDY_PIXEL_PERCENTAGE'* threshold depending on the imagery availability for the study area, always trying to maintain the lowest cloud coverage.
![image](https://user-images.githubusercontent.com/101252763/194063095-11e6155b-a7a9-4061-959e-185482d660e9.png)
From here one, all code outputs from the 2a, 2b, 2c,2d and 2e batches are exported as assets to be used as model input.

## 2b_input_GLCM
This code extracts the five texture metrics (Gray-Level Co-occurrence Matrix - GLCM) used in this study. GEE already provides a function to calculate such metrics.  
Here the user might need to adapt the radius of the kernel used. We used 5 (in pixel units, which means 100m), that we found reasonable considering the spatial structure of the Metropolitan Area of São Paulo. Depending on the region, a smaller kernel might be more efficient to show smaller urban blocks, or larger ones in a more sprawled context. 
![image](https://user-images.githubusercontent.com/101252763/194064200-05770fa7-86ae-4096-9c68-5b6ac74f752d.png)

## 2c_input_KDE
This script calculates a heatmap from the dangling points extracted previously under *'1c_prep_deadends'*. Similar as the previous script, here the user might need to adapt the kernel radius. Visualizing the results with the *'Map.addLayer'* function always helps with this decision. 

## 2d_input_MSI
This is a novel approach just as the extraction of dead ends street points. From the building footprint layer (WSF 2019) we calculated the Mean Shape Index formula for each pixel and combined the resulting values per settlement. 

## 2e_input_GISdata
## 3_input_Inspection
## 3_input_Zscore
## 4_model_Filter
## 5_UserInterface
