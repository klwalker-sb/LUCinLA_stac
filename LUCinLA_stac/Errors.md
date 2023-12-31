# cleaning files
================================================================================================================================
use `tuyau clean` to clean out files that are no longer needed and properly delete corrupted files.

Following the time series step in the pipeline, `tuyau clean` can be used to delete the original download files and free up quite a bit of memory (downloads consume ~100GB of memory per cell) 
The parameter `clean:delete_record` is False by default. As long as this is not set to True, the record will be preserved in `processing.info` and any processed files will not be redownloaded. (Even if `processing.info` is deleted, it can be recreated (with less info) from the files in the brdf folder, so processed files need not be redownloaded.) 

the bash script `eostac_clean.sh` has the recommended parameter configuration for regular file cleaning. By default, this will clean out both the download folders and the nocoreg folder (the nocoreg folder contains a copy of each brdf prior to the coreg step. This can be used to check the accuracy of the coreg step, but does take up a bit of memory). 

In the case of corrupted files or other files that we do not want to include in the final processing, `tuyau clean` has other configuration options. `eostac_clean_removeFromDB.sh` has the recommended configuration to remove corrupted downloads. By setting `clean:delete_record` to True, the record of the file is deleted from `processing.info` when the file is deleted from the downloads folder so that is is able to be redownloaded. If the file has been run through the brdf stage, the corresponding brdf file is also deleted (which can have a variety of file names, depending on whether the coreg step has been run and whether it succeeded), and the brdf field is reset to NaN in `processing.info` so that it can be recreated.  If a brdf file is corrupted and not the download itself, can set `clean:remove_items` to '\[brdf]\' to remove only the brdf file and record. 
files to be cleaned can be specified with the parameters: 
`clean:sat_sensors` (options = S2A,S2B,LT05,LE07,LC08,LC09,S2(for all sentinel only),L(for all Landsat only) or All)
`clean:date_range`  (format: '\[YYYYMMDD, YYYYMMDD]\'. if a single image is to be cleaned, can use '\[YYYYMMDD]\'. if all files are to be cleaned (all dates), use '\[0]\')
can also use `clean:xlist` to provide a list of images to remove (format is path to .csv file with one image per row (no heading)). File names can be from download or brdf folder (if from download, will remove brdf as well)
if `clean:delete_record` is set to False, the record will remain in `processing.info` and will be flagged so that it is not reprocessed.
This is useful if you want to remove a list of images that are deemed to be low quality, for example.

# troubleshooting errors

### download errors

### brdf errors

#### out of input error
![alt](/Images/errors/brdf_out_of_input_error.jpg)
This means that the `scene.info` file is corrupted, in either the landsat or sentinel download file (indicated in the error script)
if you locate the `scene.info` file, you will probably see that it has zero bytes.
![alt](/Images/errors/brdf_out_of_input_check.jpg)
To get around this error, simply delete the corrupted `scene.info` file and rerun the download script. This will create a new `scene.info` file and will not redownload files that already exist. 
