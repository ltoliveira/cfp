# CFP Project documentation


This repository was developed by Lorraine Trento Oliveira, Dr. Julio Pedrassoli, Dr. Monika Kuffer
as part of the CFP project funded by Notre Dame-IBM Tech Ethics Lab 
(Access call at https://techethicslab.nd.edu/news/notre-dame-ibm-technology-ethics-lab-announces-funding-for-proposals/)
and its aim is twofold: archive for metadata documentation and tutorial of how to use the [batch of codes](https://code.earthengine.google.com/?accept_repo=users/lorrainetoliveira/cfp-project) developed by our group.  

## Introduction to the project
We scaled the unsupervised machine-learning model developed by Ms. Oliveira (https://essay.utwente.nl/88986/)
to classify deprived as areas in the city of São Paulo to the Metropolitan Area of São Paulo using solely open data sources
working on the automation level of the script and dealing with scalability challenges. 

### Objectives
- To develop a transparent and open-source workflow to capture intra-urban deprivation in the Metropolitan Area of São Paulo
- To critically address scalability challenges and possible data biases and inconsistencies
- To assess the accountability of the approach to prospective users


## Accessing the platform 
We are using the GEE platform to manage and analyse the data, because it has a great open catalog, speedy processing, no environmental maintenance or preparation required and stream all laters and outputs to the local computer for visualization purposes. With the link below you can access the batch of codes and the input data repository: https://code.earthengine.google.com/?accept_repo=users/lorrainetoliveira/cfp-project

**PS:** If you don't have yet, please remember to [sign up](https://signup.earthengine.google.com/) for an Earth Engine account.


## Methodology 
1. Literature review: Relevant variables that characterise deprived areas in Brazil and especifically in the State of São Paulo. 
2. User requirements: Through workshop with different potential users, we framed the user needs such as scale of analysis, aggregation level of the final output, output formats and addressed possible gaps in data (availability, update consistency, spatial coverage).
3. Data collection: All datasets are open source, spatial, quantitative, available for the entire study area and frequently updated. 
- Most datasets were acquired directly from GEE catalog. The exceptions are: the layer of deprived areas in Brazil ("AGSN layer"), the extent of the Metropolitan Area of São Paulo ("MASP layer"), the building footprints ("WSF layer") from 2019 - GEE only contains the one from 2015, shapefiles from OpenStreetMap. ("osmpoints","osmlines","osmpolygons"). 
4. Data processing: Here we do not only extracted the input features for the model in a single workflow - sometimes having to create the extraction function that did not existed - , but we also investigate data inconsistencies (completeneness, biases, etc). 
5. Modelling: K-means algorithm [wekaKMeans](https://developers.google.com/earth-engine/apidocs/ee-clusterer-wekakmeans) was used and optimized multiple times with multiple strategies such as inspecting each [input feature](https://code.earthengine.google.com/c5737a03110fb8bb66a88da589bb3327) and including each one sistematically , [removing boundary pixels](https://www.sciencedirect.com/science/article/abs/pii/S0143622812001592?via%3Dihub) from the training sample for less noise , detecting outliers [z-score](https://datascience.eu/mathematics-statistics/what-is-a-z-score/), normalization process and post classification filters.
6. Validation: Model was validated with local specialists from academia, NGO and governmental institutions in a three hour workshop.
8. Documentation: Besides the [GEE repo](https://code.earthengine.google.com/?accept_repo=users/lorrainetoliveira/cfp-project), this repository is also used for documentation and will be soon translated to Portuguese, so that it can also serve as a tutorial guide for users in Brazilian municipalities.


### Model Input 
This table shows the input data used for the Metropolitan Area of São Paulo. These features can be completed different depending on the context. The same country, even the same city, has different deprivation levels and those features should manifest their spatial characteristics so that the model can properly capture it. We processed each dataset and extracted each feature inside GEE. We explain what is the function of each code in the next section.
![image](https://user-images.githubusercontent.com/101252763/194051051-e5c4f731-ab79-4c15-856d-7279cd717e85.png)


### Which code does what?
- 1a_prep_AGSNbuffer: This script created to reduce the possible noise on the boundaries of the settlement layer caused by heterogeneous non deprived pixels inside the sampling area.
- 1b_prep_OSMmerge: OSM datasets require a merging process after downloading from HOTOSM platform. For the Metropolitan region of Sao Paulo (MASP), four region's of interest (ROI) were used to encompass the entire extent.
- 1c_prep_deadends: This code was created to locate and extract the dead end streets (in specific the dangling points). This process is not automated in any GIS software which makes this a novel approach. The output of this code will be used as input to derive the density of dead end streets in the ROI (Kernel Density Estimation analysis).
- 2a_input_RSdata: This code extracts the data from Remote Sensing based datasets.
- 2b_input_GLCM: This code extracts texture metrics (Gray-Level Co-occurrence Matrix -GLCM). It measures the spatial relationship of the pixels by calculating statistics derived from the pixels brightness. 
- 2c_input_KDE: This code calculates the Kernel Density Estimation to derive the feature *Density of dead end streets* as the second step of '1c_prep_deadends'.
- 2d_input_MSI: This script calculates the Mean Shape Index, a landscape metric that quantifies the complexity of the urban fabric. 
- 2e_input_GISdata: This code provide an accessbility analysis of the study area by calculating the Euclidean distance to features (shapefiles from HOTOSM).
- 3_input_Inspection: This code inspects the model input before feeding into the model, looking into possible inconsistencies (noise, data patterns, masked values etc)
- 3_input_Zscore: This code calculates the z-score, a measure that describes the distance between a data point and the mean using standard deviations.
- 3_input_ZscoreAutomated: This script is an automated version of the previous one. 
- 4_model_Filter: This code implements the k-means algorithm, visualize the results and applies a post classification filter.
- 5_UserInterface: This is the user interface prototype used for the development of the app.

## Results
The main model outputs are the clustering map - that can be automatically exported to the user's Google Drive - the histograms of the features per resulting cluster type - that can be easily download as .csv files. 
We have also developed an [App - beta version](https://pedrassoli-julio.users.earthengine.app/view/ibm-nd-app-beta-v2). The user can select the city the inside the Metropolitan Area of São Paulo, the app provides the output map and graphs for download. 
![image](https://user-images.githubusercontent.com/101252763/194055090-68fe5243-093b-4ac9-9c8d-71a9bf0d8ef9.png)

