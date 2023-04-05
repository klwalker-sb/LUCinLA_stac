# Processing
================================================================================================================================

Revisit the [Processing overview](processing_overview) for the conceptual workflow.

**Processing workflow**
1. Get grid cells to process. ([see here](GetCells))
2. Run downloading process ([see here](downloading)) to download all images for each cell from Google Earth Engine.
3. Check download status ([see here](checkDownload)) to make sure all images downloaded correctly
    * If incomplete, rerun step 2.
4. Run BRDF script ([see here](post))
5. Run Pipeline process ([see here](pipeline))
    * Coregistration
    * Time Series construction
--------------------------------------------------------------------------------------------------------------------------------    
    * Segment -- (do after Preprocess and Reconstruct are done for a large chunk of grids)
    * Classify -- (do after Segment is done for a large chunk of grids)
    * Assess  -- (do after validation data are available)
    * Clean

Processing products, at various stages, are housed under the following file structure:
:::{note}
sandbox-cel holds bulk of preprocessing data and is minimally backed up. 
downspout-cel hold final time series data and derived products and has dual backup in different locations.
:::
```
raid-cel/sandbox/sandbox-cel/paraguay_lc/stac
    /grids/
    --003001  
    --003002  
    --003003  
    ...etc
   within each grid directory:
   /grids/003001/  
   --brdf         # BRDF-adjusted NetCDF archive of NetCDF files  
   --brdf_masks   # 8-bit cloud/shadow masks  
   --brdf_nodata  
   --brdf_ref     # reference file for auto-geo-referencing S-2 images  
   --brdf_s2_nocoreg  # copies of un-adjusted S-2 images  
   --brdf_ts      # weekly time series of bands or spectral indices  
   --landsat      # original downloaded Landsat images
   --sentinel2    # original downloaded Sentinel2 images
raid-cel/r/downspout-cel/paraguay_lc/stac/
   /grids/003001/
   -- brdf_ts/ms  # each time series index that has been produced for this cell is in this folder
      --evi2      
      --wi
      ...
   -- comp        # folder containing composite outputs
```
