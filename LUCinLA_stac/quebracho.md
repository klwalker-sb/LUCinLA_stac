# Setting up a new project
======================================================================================================================================
The following skeleton, containing the grid file and two index files is necessary for a new project:
```
raid-cel/sandox/sandbox-cel/<country>_lc
    raster
        grids
        landsat_index
            index.csv.gz
        sentinel-2_index
            index.csv.gz
        srtm
    vector
        grids.gpkg
```

## Getting the indices:
The `index.csv.gz` files are complete catalogues of available imagery with basic parameters such as date and extent. You can use index files from other projects, but to get the most up-to-date indices, copy them from Google cloud:
for Landsat: gs://gcp-public-data-landsat/index.csv.gz and for Sentinel: gs://gcp-public-data-sentinel-2/index.csv.gz

## Creating the grid file:
Script to create gridfile (from Jordan Graesser's [EOSvault Github page](https://github.com/jgrss/eosvault))
```
import zones
import geopandas as gpd

df = gpd.read_file(<vector file with target bounds>)

left, bottom, right, top = df.total_bounds.tolist()

def round10(number, offset):
    #Round to the nearest 10
    return (round(number / 10.0) * 10.0) + offset

left = round10(left, -10)
bottom = round10(bottom, -10)
right = round10(right, 10)
top = round10(top, 10)

# Target grid size (in meters)
grid_size = 20000

# Target cell storage size (in meters)
cell_size = 10.0

#Create the Grids
df_grids = zones.grid((left, bottom, right, top),
                       grid_size, grid_size,
                       cell_size, cell_size,
                       crs=df.crs)
                       
#Take the intersecting grids only:
df_grids = df_grids.loc[[geom.intersects(df.geometry.values[0]) for geom in df_grids.geometry.values]]

#Add unique ids:
df_grids.loc[:, 'UNQ'] = range(1, df_grids.shape[0]+1)

#Save tp file:
df_grids.to_file('rgrids.gpkg', driver='GPKG')
```
Note: `zones.grid` can be found [here](https://github.com/jgrss/zones/blob/master/zones/base.py)

## Modifying the scripts:
Make sure to modify the lines in `~/project/config/config.yaml` to point to the correct input and output locations.
Also modify all downloading and processing scripts in `~/code/bash`

In the downloading scripts, you need to enter the crs in proj4js format. You can look this up at spatialreference.org, or can convert using earthpy (i.e: proj4 = et.epsg['32613'])
