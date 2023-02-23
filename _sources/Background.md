# Project background
================================================================================================================================

LUCinLA employs a framework developed by Jordan Graesser to synthesize large quantities of satellite imagery from multiple sensors to generate high-resolution (sub-10m) landcover maps focused on agriculture/vegetation.

Current LUCinLA projects in Robert Heilmayr's Conservation Economics Lab (CEL) include:
* A NASA-funded grant on property values and land-use change in Paraguay and Uruguay:  The supporting map, produced by the procedure outlined in this guide, will be a high-resolution maps of fields, identified to crop-type, across a large region spanning Paraguay and Uruguay (focusing on the Pampas, Chaco, and Alto Parana Atlantic Forest regions).
* Forest mapping in Chile, to assess affect of payments to private landowners on forest cover (and forest type in terms of natural vs. plantation) and ultimately carbon stocks.

These projects involve large study areas but small units-of-analysis (field sizes are often <2ha). The framework outlined in this guide enables large quantities of satellite data to be ingested from Google Earth Engine and processed on a high-capacity cluster computer to generate carefully calibrated, gap-filled weekly time-series data over the entire study region at 10m resolution. These data cubes are then converted to high-resolution vegetation maps via spectral analysis and pattern recognition methods including boundary identification (segmentation) and sequential learning to identify seasonal cycles.  
