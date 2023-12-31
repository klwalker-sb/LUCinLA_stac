(cleaning)=
# cleaning files
================================================================================================================================
use `tuyau clean` to clean out files that are no longer needed and to properly delete corrupted files.

### As part of the regular pipeline procedure:
Following the time series step in the pipeline, `tuyau clean` can be used to delete the original download files and free up quite a bit of memory (downloads consume ~100GB of memory per cell for a 2000-2022 time series) 

**Make sure the parameter `clean:delete_record` is set to `False`** (its default). As long as `clean:delete_record` is not set to True, the record will be preserved in the `processing.info` database and any processed files will not be redownloaded. (Even if `processing.info` is deleted, it can be recreated (albeit with less info) from the files in the brdf folder, so processed files need not be redownloaded with future updates) 

The recommended parameter configuration for regular file cleaning can be found in the bash script `eostac_clean.sh` (in sandbox-cel/paraguay_lc/templates). By default, this will clean out both the download folders and the nocoreg folder (the nocoreg folder contains a copy of each brdf prior to the coreg step. This can be used to check the accuracy of the coreg step, but does take up a bit of memory). 

### As a mechanism to delete corrupted files:
In the case of corrupted files or other files that should be reprocessed after removal, `clean:delete_record` should be set to `True`. This ensures that the file record is deleted from `processing.info` when the file is deleted from the downloads folder so that is is able to be redownloaded. If the file has been run through the brdf stage, the corresponding brdf file is also deleted (which can be tricky manually as it can have a variety of file names, depending on whether the coreg step has been run and whether it succeeded). The brdf field is reset to NaN in `processing.info` so that it can be recreated. If a brdf file is corrupted and not the download itself, can set `clean:remove_items` to '\[brdf]\' to remove only the brdf file and record. 
:::{warning} if `clean:delete_record` is set to `True` in `tuyau clean`, the brdf file and record will be deleted even if `brdf` is not in `clean:remove_items`. This allows/forces the item to be reprocessed in future runs of the downloading/brdf processes. :::

`eostac_clean_removeFromDB.sh` (in sandbox-cel/paraguay_lc/templates) has the recommended configuration to remove corrupted downloads. 
files to be cleaned can be specified with the parameters: 
*`clean:sat_sensors` (options = S2A,S2B,LT05,LE07,LC08,LC09,S2(for all sentinel only),L(for all Landsat only) or All)
*`clean:date_range`  (format: '\[YYYYMMDD, YYYYMMDD]\'. if a single image is to be cleaned, can use '\[YYYYMMDD]\'. if all files are to be cleaned (all dates), use '\[0]\')
*`clean:xlist` can be used to provide a list of images to remove (format is path to .csv file with one image per row (no heading)). File names can be from download or brdf folder (if from download, will remove brdf as well)

### As a mechanism to remove unwanted images:
This is useful if you want to remove a list of images that are deemed to be low quality, for example. This list can be generated using the LUCinSA_helpers create_thubnails method followed by the thumbnail_filter notebook. 

in `tuyau clean`, set `clean:delete_record` to `False` and set `clean:xlist` to the path to your list of images to remove (formated as .csv file with one image per row (no heading)). File names can be from download or brdf folder (if from download, will remove brdf as well)
When `clean:delete_record` is set to False, the record will remain in `processing.info` and will be flagged so that it is not reprocessed with future updates.