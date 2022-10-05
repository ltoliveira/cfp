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
* To develop a transparent and open-source workflow to capture intra-urban deprivation in the Metropolitan Area of São Paulo
* To critically address scalability challenges and possible data biases and inconsistencies
* To assess the accountability of the approach to prospective users


## Accessing the platform 
We are using the GEE platform to manage and analyse the data, because it has a great open catalog, speedy processing, no environmental maintenance or preparation required and stream all laters and outputs to the local computer for visualization purposes. With the link below you can access the batch of codes and the input data repository: https://code.earthengine.google.com/?accept_repo=users/lorrainetoliveira/cfp-project

**PS:** If you don't have yet, please remember to [sign up](https://signup.earthengine.google.com/) for an Earth Engine account.


## Methodology 
1. Literature review: Relevant variables that characterise deprived areas in Brazil and especifically in the State of São Paulo. 
2. User requirements: Through workshop with different potential users, we framed the user needs such as scale of analysis, aggregation level of the final output, output formats and addressed possible gaps in data (availability, update consistency, spatial coverage).
3. Data collection: All datasets are open source, spatial, quantitative, available for the entire study area and frequently updated. 
- Most datasets were acquired directly from GEE catalog. The exceptions are: the layer of deprived areas in Brazil ("AGSN layer"), the extent of the Metropolitan Area of São Paulo ("MASP layer"), the building footprints ("WSF layer") from 2019 - GEE only contains the one from 2015, shapefiles from Open Street Maps ("osmpoints","osmlines","osmpolygons"). 
4. Data processing: Here we do not only extracted the input features for the model in a single workflow - sometimes having to create the extraction function that did not existed - , but we also investigate data inconsistencies (completeneness, biases, etc). 
5. Modelling: K-means algorithm [wekaKMeans](https://developers.google.com/earth-engine/apidocs/ee-clusterer-wekakmeans) was used and optimized multiple times with multiple strategies such as inspecting each [input feature](https://code.earthengine.google.com/c5737a03110fb8bb66a88da589bb3327) and including each one sistematically , [removing boundary pixels](https://www.sciencedirect.com/science/article/abs/pii/S0143622812001592?via%3Dihub) from the training sample for less noise , detecting outliers [z-score](https://datascience.eu/mathematics-statistics/what-is-a-z-score/), normalization process and post classification filters.
6. Validation: Model was validated with local specialists from academia, NGO and governmental institutions in a three hour workshop.
8. Documentation: Besides the [GEE repo](https://code.earthengine.google.com/?accept_repo=users/lorrainetoliveira/cfp-project), this repository is also used for documentation and will be soon translated to Portuguese, so that it can also may serve as a tutorial guide for users.


### Model Input 



### Which code does what?


## Results
Map
Graphs
App (beta version)
