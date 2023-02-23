(post)=
# Post-download processing 
================================================================================================================================

## 1. Edit the post_eri.sh document for targeted grid cells and sensor product
```
cd ~/code/bash/
vim post_eri.sh
```
To edit a line, type `i`, edit as desired, then hit `Esc` key and type `:wq`. Hit `Enter` key
Go here for [more info on editing in vim](vimCommands)

This is the default script (with line numbering added for reference below):
Default script
```
1- #!/bin/bash
2- 
3-  #SBATCH -N 1 # number of nodes
4-  #SBATCH -n 8 # number of cores
5-  #SBATCH -t 0-08:00 # time (D-HH:MM)
6-  #SBATCH -p basic
7-  #SBATCH -o slpst.%N.%a.%j.out # STDOUT
8-  #SBATCH -e slpst.%N.%a.%j.err # STDERR
9-  #SBATCH --job-name="pstsa"
10- #SBATCH --array=911
11- 
12- # Change permissions on output files
13- umask 002 
14- #####################################################
15- 
16- # landsat | sentinel-2
17- SATELLITE="sentinel-2"
18- ASSET_ID="COPERNICUS/S2"
19- 
20- # LT05 | LE07 | LC08
21- #LANDSAT_SENSOR="LC08"
22- #ASSET_ID="LANDSAT/${LANDSAT_SENSOR}/C01/T1_SR"
23- #SATELLITE="landsat"
24- 
25- # There are 2 threads per worker from the config file
26- NUM_WORKERS=4
27- BATCH_SIZE=100
28- #####################################################
29- 
30- # As an array job
31- GRID_ID=$SLURM_ARRAY_TASK_ID
32- 
33- MY_USERNAME=""
34- IN_DIR="/raid-cel/sandbox/sandbox-cel/paraguay_lc/raster/grids"
35- CONFIG_FILE="/home/${MY_USERNAME}/project/config/config_eri.yaml"
36- 
37- #############################################
38- # Turn off NumPy parallelism and rely on dask
39- #############################################
40- export OPENBLAS_NUM_THREADS=1
41- export MKL_NUM_THREADS=1
42- # This should be sufficient for OpenBlas and MKL
43- export OMP_NUM_THREADS=1
44- ################################################
45- 
46- # activate the virtual environment
47- source ~/.nasaenv/bin/activate
48- 
49- eosvault post --in-dir $IN_DIR --grid $GRID_ID --config-file $CONFIG_FILE --batch-size $BATCH_SIZE --max-workers 
50- $NUM_WORKERS --satellite $SATELLITE --asset-id $ASSET_ID
51- 
52- deactivate
```
Most lines should stay as they are.
**Do ensure that your username is entered between the quotes in line 37.
As with the downloading script, **you also need to change Grid info and Product info lines**:  
**Grid info:** `#SBATCH --array= ` (line 10 here). This is where you enter the gridcells you are processing.  
     You can enter a range (e.g. 898-908), But be mindful that you are not hogging all the computer bandwidth.
:::{admonition}You can limit the number of cells that are processed at one time by adding %n
For example, 898-908%4 would process 4 cells at a time. When the first 4 finish, the next will start.
:::
**Product info:** Lines following `# landsat | sentinel-2` (Lines 16-23 here). This is where you select the satellite product, by uncommenting the corresponding lines (remove the # at the beginning of the line) and commenting those that don't correspond (replacing the # at the beginning of the line). (Only one product can be run at a time).   

**<span style='color:purple'> For Sentinel: </span>** uncomment `SATELLITE="sentinel-2` (line 17 here) and `ASSET_ID="COPERNICUS/S2` (line 18 here).

**<span style='color:purple'> For Landsat: </span>** uncomment `LANDSAT_SENSOR="LC08"` `ASSET_ID="LANDSAT/${LANDSAT_SENSOR}/C01/T1_SR"`and `SATELLITE="landsat"` (lines 21-23 here). Also change the Landsat sensor (line 21 here) to correspond to the sensor you want to process: "LT05" for Landsat 5, "LE07" for Landsat 7, or "LC08" for Landsat 8 (as noted in line 20). You will do all three, separately.   

## 2. Run the process
```
#if not already in bash directory, navigate there:
cd ~/code/bash/
#submit the command:
sbatch post_eri.sh
``` 
:::{note}
Current run-time estimates for single grid cells:
* Sentinel 2: 
* Landsat 8:
* Landsat 7:
* Landsat 5:
:::