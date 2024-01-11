(General_framework)=
# General framework
================================================================================================================================

The framework outlined in this guide enables large quantities of satellite data to be ingested and processed to generate carefully calibrated, gap-filled weekly time-series data over large study areas (country to multi-country regions) at 10m resolution. 

Characteristics of our processing include:

* Use of standardized SpatioTemporal Asset Catalog **(STAC) methods** to search for and store satellite imagery (with alternative option [here](https://klwalker-sb.github.io/LUCinSA/intro.html) to ingest imagery from Google Earth Engine.) New methods using STAC-based sources now allow for faster ingestion and processing as well as better standardization. (see [this site](https://stacspec.org/en/about/) for much more information about STAC)

* array-based processing methods using **rasterio, Xarray, and Dask** along with physical gridding to maximize parallel computations on a modest HPC computing environment
        
* The opensource repo **[Geowombat](https://geowombat.readthedocs.io/en/latest/)** as our primary toolkit 
    for processing raster data at scale (Read more about Geowombat [here](https://geowombat.readthedocs.io/en/latest/))
    
* Multi-sensor image standardization including BRDF correction {cite:p}`BRDF` and coregistration (based on AROSICS {cite:p}`arosics`,`arosics2` in following with the methods used to produce the harmonized Landsat and Sentinel data set {cite:p}`HLS`

* Time-series smoothing, following methods from {cite:t}`TS_smoother`

* Optional object-based classification using segmentation output of a convolutional neural network trainied with time-series data

* User-friendly **quality checks** at various points in the processing
