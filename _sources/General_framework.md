(General_framework)=
# General framework
================================================================================================================================

The framework outlined in this guide enables large quantities of satellite data to be ingested and processed to generate carefully calibrated, gap-filled weekly time-series data over large study areas (country to multi-country regions) at 10m resolution. 

Characteristics of our processing include:

* Use of standardized SpatioTemporal Asset Catalog **(STAC) methods** to search for and store satellite imagery
        A previous version of our processing protocol [here](https://klwalker-sb.github.io/LUCinSA/intro.html) ingested imagery from 
        Google Earth Engine. New methods using STAC-based sources now allow for faster ingestion and processing as well as better 
        standardization. (see [this site](https://stacspec.org/en/about/) for much more information about STAC)

* array-based processing methods using **Xarray and Dask** along with physical gridding to maximize parallel computations on a modest 
        HPC computing environment
        
* The opensource repo **[Geowombat,](https://geowombat.readthedocs.io/en/latest/)** as our primary toolkit 
    for processing raster data at scale (Read more about Geowombat [here](https://geowombat.readthedocs.io/en/latest/))
    
* Multi-sensor image standardization including BRDF correction and coregistration

* Time-series smoothing

* Optional object-based classification using segmentation output of a convolutional neural network trainied with time-series data

* User-friendly **quality checks** at various points in the processing
