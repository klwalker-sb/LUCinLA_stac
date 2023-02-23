# Land-Use Change in Latin America (LUCinLA) Computing Guide
================================================================================================================================
Guide to computer setup and processing for remote-sensing projects on Land-Use Change in Latin America in Robert Heilmayr's lab

| :------------------------------------: | :------------------------------------: | 
|   ![](/Images/3977_L8_2014_forest.jpg)              |  ![](/Images/3977_L8_2014_soy.jpg)               |

   ![](/Images/Forest_to_Soy_KNDVI.jpg) 



This guide focuses on setup details specific to the computing environment available to us at UC Santa Barbara and is mostly geared toward students with little to no experience with High-Performance Computing who need to quickly jump into data processing. While the computing environment is quite specific, the more general framework for ingesting, processing, and checking large quantities of imagery using primarily open-source resources could be of interest to others with similar computing resources.  

The framework outlined in this guide enables large quantities of satellite data to be ingested and processed to generate carefully calibrated, gap-filled weekly time-series data over large study areas (country to multi-country regions) at 10m resolution. 

Characteristics of our processing include:
* Use of standardized SpatioTemporal Asset Catalog (STAC) methods to search for and store satellite imagery
* array-based processing methods and a gridded structure to maximize parallel computations on a modest HPC computing environment
* The opensource repo [Geowombat,](https://geowombat.readthedocs.io/en/latest/) as our primary toolkit for processing raster data at scale.
* Multi-sensor image standardization including BRDF correction and coregistration
* Time-series smoothing
* Optional segmentation for polygon-based classification

