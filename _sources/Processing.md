# Processing
================================================================================================================================

Revisit the [Processing overview](processing_overview) for the conceptual workflow.

**Processing workflow**
1. Get grid cells to process. ([see here](GetCells))
2. Run downloading process ([see here](downloading)) to download all images for each cell from Google Earth Engine.
    * Sentinel 2, Surface reflectance
    * Sentinel 2, Cloud Masks
    * Landsat 5
    * Landsat 7
    * Landsat 8
3. Check download status ([see here](checkDownload)) to make sure all images downloaded correctly
    * If incomplete, rerun step 2 for incomplete product, with IGNORE_INCOMPLETE = "YES". ([see here](incomplete))
4. Run post downloading process. ([see here](post))
    * Sentinel 2, Surface reflectance
    * Landsat 5
    * Landsat 7
    * Landsat 8
5. Run Pipeline process ([see here](pipeline))
    * Preprocess
    * Reconstruct
    * Segment -- (do after Preprocess and Reconstruct are done for a large chunk of grids)
    * Classify -- (do after Segment is done for a large chunk of grids)
    * Assess  -- (do after validation data are available)
    * Clean

Processing products, at various stages, are housed under the following file structure:
```
raid-cel/sandbox/sandbox-cel/paraguay_lc/raster/grids/
    /grids  
    --000001  
    --000002  
    --000003  
    ...etc
#And within each grid directory:
   /grids/000001  
   --brdf         # BRDF-adjusted NetCDF archive of NetCDF files  
   --brdf_masks   # 8-bit cloud/shadow masks  
   --brdf_nodata  
   --brdf_ref     # reference file for auto-geo-referencing S-2 images  
   --brdf_s2_nocoreg  # copies of un-adjusted S-2 images  
   --brdf_ts      # weekly time series of bands or spectral indices  
   --cls          # land cover classification  
   --gee          # raw data stream from Earth Engine  
   --seg          # segmentation outputs
```
