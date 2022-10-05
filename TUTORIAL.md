This is a guide for users on how to run the codes we developed, if you would like to test it in another metropolitan area in Brazil or across the globe. 
We divided this tutorial according to the scripts on [GEE](https://code.earthengine.google.com/?accept_repo=users/lorrainetoliveira/cfp-project).
It is important to state again that the features used will surely differ from place to place. The pool of datasets we used is derived from months of research together with local experts. Some features will likely be used to other contexts such as DEM or distance to natural areas, the interpretation of the features values in relation to deprivation might change. 

**PS1: The words feature and variable are used interchangebly in this tutorial.**

**PS2: This tutorial will soon have a Portuguese version.**

## 1a_prep_AGSNbuffer
This code is twofold. First, it loads the two main base layers of the model the layer with the extent of the study area and the layer with the extent of the local deprived settlements. Second, it excludes non-deprived pixels from the training sample, that might be noise source, by applying a buffer function. 
Here the user need to upload the local layers and determine the scale of analysis (*pixel resolution*), because the buffer would need to be half of such value to exclude pixels only partially inside the deprived settlements extent. We used 20x20m grid, therefore our buffer is 10m. The code will export the resulting layer (after buffer) as an asset.

**Attention: please check every time 'scale' or 'SCALE' appears in any of the following scripts, inside a function or for export purposes. We used 20m but the user needs to adapt it accordingly.**

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
Here the user might need to adapt the radius of the kernel used. We used 5 (in pixels, which means 100m), that we found reasonable considering the spatial structure of the Metropolitan Area of São Paulo. Depending on the region, a smaller kernel might be more efficient to show smaller urban blocks, or larger ones in a more sprawled context. 
![image](https://user-images.githubusercontent.com/101252763/194064200-05770fa7-86ae-4096-9c68-5b6ac74f752d.png)

## 2c_input_KDE
This script calculates a heatmap from the dangling points extracted previously under *'1c_prep_deadends'*. Similar as the previous script, here the user might need to adapt the kernel radius. Visualizing the results with the *'Map.addLayer'* function always helps with this decision. 

## 2d_input_MSI
This is a novel approach just as the extraction of dead ends street points. From the building footprint layer (WSF 2019) we calculated the Mean Shape Index formula for each pixel and combined the resulting values per settlement. 
The WSF datasets are also acquired by tiles and you might need to download more than we needed for our case study. So another variable should be imported to the user's assets, loaded into the code as a new variable and included in the mosaic. 
![image](https://user-images.githubusercontent.com/101252763/194071687-626b5d5c-7616-4ae7-a2e0-076c266cfb7b.png)
Besides that, the user will need to adapt the scale and the radius of the reducers of the applied filters (we used them for noise removal). Here we used 2 fos max reducers and 4 for min reducers. Use the displayed map to identify the most suitable radius, making sure to check how the filter impacts areas with small and large settlements.
![image](https://user-images.githubusercontent.com/101252763/194072106-faf2ab15-0f48-4dd1-b5a3-957089ac0fbe.png)

## 2e_input_GISdata
This code calculates the Euclidean distance to features (street network, water bodies, natural areas and urban facilities). Nothing must be added besides the scale of analysis after importing the OSM merged layers (line, points and polygons) to the script. 
This code can be adapted to include other features such as distance to industrial facilities, sewage plants or water wells. This depends on the context and the availability (and completeness) of the OSM data.

## 3_input_Inspection
This code is basically to visualize all input data, inspecting source of inconsistencies or any weird patterns. If other input features are added, the user will need to look into standard visualization parameters for such dataset. 
![image](https://user-images.githubusercontent.com/101252763/194074306-aa9ad661-9bbd-4597-bd46-705c69721247.png)

## 3_input_Zscore / 3_input_ZscoreAutomated
This script calculates the z-score, measuring the distance between a data point and the mean using standard deviations. It helps finding and removing atypical values that can disturb the model performance. 
We created a manual and an automated version. In the manual version, the user will need to choose the variable on the top of the code to select the input feature, and on the end of the code for proper visualizing and exporting. 
![image](https://user-images.githubusercontent.com/101252763/194075380-180bdb3e-151c-4a71-83bf-e12a3b18be4f.png)
The user also need to select the z-score threshold. The standard rule is -3 < z < 3. 
![image](https://user-images.githubusercontent.com/101252763/194075562-53d7d920-3007-4732-b1a5-e0340a0595a3.png)

## 4_model_Filter
This code implements the k-means algorithm, visualize the results and applies a post classification filter to deal with the noise derived from the own classification process and to deal with the voids derived from the z-score calculation.
The first thing the user can "play with" is to removing and including new variables. By adding // to the line code the algorithm do not read that variable. The user also need to change (if there are new variables or variables removed) from the image collection. 
![image](https://user-images.githubusercontent.com/101252763/194076455-19454e7c-bd12-4dcc-a20d-87b452756dfb.png)

Under the training part, the user can change the number of clusters. This is a very important step that has no proper decision methods besides trial and error. Other programming languages offer tools for this e.g.,Python: elbow method, but we could not apply into this javascript yet. 
![image](https://user-images.githubusercontent.com/101252763/194077592-5941794a-c5e4-4b0d-b78d-999e83e151be.png)

To visualize the graphs, the user will need to remove or include the variables name and rename the bands accordingly. 
![image](https://user-images.githubusercontent.com/101252763/194078171-dda79b2f-6553-4305-ad95-83f87baa95f7.png)

And remove or include the number of graphs according to the number of clusters. Besides changing on line 172-175 of the code, the user should remove all the lines regarding the creation of the histograms that are not selected. For instance, we used 4 four clusters, but if there are only 2 clusters for a certain context, the histograms 3 and 4 must be erased. The user can also select the variable they would like to see on the graph, even only one at a time.
![image](https://user-images.githubusercontent.com/101252763/194078332-657da266-5ee5-4820-95af-34281e41a05a.png)

Finally, we used spatial filters for a better visualization of the output map. The user can change the kernel radius (line 265) and check the influence on the final result by visualizing it on the map.
![image](https://user-images.githubusercontent.com/101252763/194079649-8bb5df27-6e03-4372-aaa6-215f7206e9ac.png)

The screenshot below shows the final map result and the first cluster map.
![image](https://user-images.githubusercontent.com/101252763/194081061-8f86305a-7e4c-40b4-9a21-3e4070865316.png)
Scrooling down the console tab, the other histograms can be seen and by clicking on the gray arrow on the right corner of the graph, it opens in another tab with download options in different formats.
![image](https://user-images.githubusercontent.com/101252763/194081642-1c409717-2e03-4ce8-b2b3-3970e6f94b2d.png)

## 5_UserInterface
The user interface is the source script of our [app](https://pedrassoli-julio.users.earthengine.app/view/ibm-nd-app-beta-v2), that compiles this batch of 10 scripts.  
