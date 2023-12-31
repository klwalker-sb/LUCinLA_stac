(errors)=
# troubleshooting errors

### download errors

### brdf errors

#### out of input error
![alt](/Images/errors/brdf_out_of_input_error.jpg)
This means that the `scene.info` file is corrupted, in either the landsat or sentinel download file (indicated in the error script)
if you locate the `scene.info` file, you will probably see that it has zero bytes.
![alt](/Images/errors/brdf_out_of_input_check.jpg)
To get around this error, simply delete the corrupted `scene.info` file and rerun the download script. This will create a new `scene.info` file and will not redownload files that already exist. 
