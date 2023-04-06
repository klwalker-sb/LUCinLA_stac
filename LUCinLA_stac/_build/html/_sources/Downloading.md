# Downloading process
===============================================================================================================================

The `Eostac` script drives the download process, [see here](https://github.com/jgrss/eostac#readme) for more information on the script process.

To run the script:
(GetCells)=
## 1. Get gridcells to download
Use the [shared Google Sheet](https://docs.google.com/spreadsheets/d/1J2WRKW5qUcrvJCeXlSy4zFaJ5I4dqCRd/edit?usp=sharing&ouid=106313920977371808954&rtpof=true&sd=true) to find cells to process. Mark a / in the square for the cell/step for anything you are currently processing and and X if the step has been completed and verified as successful.

====================== alternative method (we aren't using this currently)
Find a block of cells in need of processing in the `cell_blocks.txt` file in `raid-cel/sandbox/sandbox-cel/paraguay_lc` (anything without an X next to it is free)
(if working ion another country, replace 'paraguay' with your country name) 
```
cd ~/../../
cd raid-cel/sandbox/sandbox-cel/paraguay_lc/
vim cell_blocks.txt
#"check out" a block of cells by putting an X and your initials next to it. Note your range(s) somewhere.
#save the file:
:wq
[Enter]
```
===========================================================================================================
(Downloading)=
## 2. Edit the downloading script for targeted grid cells
```
cd ~/code/bash/
vim eostac_dl_py.sh
# edit gridcell lines (lines 9 & 10)
```
To edit a line, type `i`, edit as desired, 
When done editing, save and quit: 
hit `Esc` key and type `:wq`, followed by `Enter` key
Go here for [more info on editing in vim](vimCommands)

This is the default script (with line numbering added for reference below):
```
1-  #!/bin/bash -l
 
2-  #SBATCH -N 1 # number of nodes
3-  #SBATCH -n 4 # number of cores
4-  #SBATCH -t 0-20:00 # time (D-HH:MM)
5-  #SBATCH -p basic
6-  #SBATCH -o stacdl2_py.%N.%a.%j.out # STDOUT
7-  #SBATCH -e stacdl2_py.%N.%a.%j.err # STDERR
8-  #SBATCH --job-name="stacdl2_py"
9-  #SBATCH --array=860
    ##############################################
    ### As an array job:
10- GRID_ID=$(($SLURM_ARRAY_TASK_ID + 3000))

11-  ##Set permissions on output files
12-  umask 002

13-  ##Settables:

14-  PROJECT_HOME="/raid-cel/sandbox/sandbox-cel/paraguay_lc/stac"
15-  LANDSAT_DIR="${PROJECT_HOME}/grid/00${GRID_ID}/landsat"
16-  SENTINEL_DIR="${PROJECT_HOME}/grid/00${GRID_ID}/sentinel2"
17-  FILETYPE='.tif'
18-  OUT_DIR="${PROJECT_HOME}/grid/00${GRID_ID}"
19-  L7STOPYR=2017
20-  GRID_FILE="/raid-cel/sandbox/sandbox-cel/LUCinLA_grid_8858.gpkg"
21-  EPSG=8858
22-  BUFFER=100

     ################################################
     ### activate the virtual environment
23-  conda activate venv.lucinsa38_dl

     ## if directories are not empty, run script to check for corrupt files
24-  if [ -n "$LANDSAT_DIR" ]
25-  then
26-     eostac check --out-path $LANDSAT_DIR --file-type $FILETYPE
27-  fi
28-  if [ -n "$SENTINEL_DIR" ]
29-  then
30-     eostac check --out-path $SENTINEL_DIR --file-type $FILETYPE
31-  fi

32-  TIMESTAMP0=`date "+%Y-%m-%d %H:%M:%S"`

33-  START_YEAR=2000
34-  END_YEAR=2023
35-  YEAR=$START_YEAR
36-  while [ $YEAR -ne $END_YEAR ]
37-  do
38-       for m in {1..11}
39-       do
40-            CURRENT_MONTH=$(printf "%02d" $m)
41-            NEXT_ITER=$(($m+1))
42-            NEXT_MONTH=$(printf "%02d" $NEXT_ITER)
43-            START_DATE="${YEAR}-${CURRENT_MONTH}-01"
                
44-            if [[ $m -eq 11 ]]
45-            then
46-              END_DATE="${YEAR}-${NEXT_MONTH}-31"
47-            else
48-              END_DATE="${YEAR}-${NEXT_MONTH}-01"
49-            fi

50-            echo Working on ${START_DATE} to ${END_DATE} >&2	
51-            TIMESTAMP=`date "+%Y-%m-%d %H:%M:%S"`
52-            echo $TIMESTAMP >&2

53-            eostac download --start-date $START_DATE --end-date $END_DATE --bounds $GRID_FILE --bounds-query UNQ==$GRID_ID --out-path   
               $OUT_DIR --epsg $EPSG --bounds-buffer $BUFFER --l7-stop_year $L7STOPYR --max-items -1 -w 4 -t 2 

54-       done
55-       YEAR=$(($YEAR+1))
56-  done

57-  TIMESTAMP=`date "+%Y-%m-%d %H:%M:%S"`
58-  TIMETOT=$(($(date -d "$TIMESTAMP" "+%s") - $(date -d "$TIMESTAMP0" "+%s") ))
59-  echo Done at $TIMESTAMP >&2
60-  echo full process took: $(($TIMETOT/60)) minutes >&2
61-  conda deactivate

```
Most lines should stay as they are.  
**the lines that you need to change are:** 
* **Grid info:** `#SBATCH --array=` (line 9 here). Enter the last 3 digits of each gridcell you are processing.  
     You can enter a range (e.g. 898-908), But be mindful that you are not hogging all the computer bandwidth.
     If your grid id is >999, modify the number GRID_ID line (line 10 here) to sum to the id 
          (e.g. cell 3919 = 919 in array line and 3000 in GRID_ID equation) 
     (this is because SLURM does not accept numbers greater than 999 in the array line directly)
     
:::{admonition}You can limit the number of cells that are processed at one time by adding %n
For example, 898-908%4 would process 4 cells at a time. When the first 4 finish, the next will start.
:::
::: {warning} servers throttle downloading loads, so more downloads will fail if too many are processed at once.
%2 is safest but can do more and rerun to catch errors.
:::

After making any edits, save your script and exit:
``` 
#(If in insert mode), use [Esc] to exit insert mode 
:wq
```

For further information on the rest of the script (parts you will NOT likely edit):
>* Line 1
>    * This is the "shebang" line, used in all Unix scripts, that tells the interpreter to execute the script.
>* Lines 3-10
>    * These are all configuration flags for SLURM.
>    * #SBATCH tells SLURM these are not user comments
>    * (line 2): We aren't doing any distributed computing across nodes, so we will always use -N 1.
>    * (line 3): -n 8 are the number of CPUs for concurrent or parallel processing (controlled in the Python scripts).
>    * (line 4): -t is the estimated max runtime (increase this if you get timeout errors)
>    * (line 5): -p tells SLURM this job should be sent to the standard partition (there are also 'short', 'gpu', and 'largemem').
>    * (line 6&7): -o and -e are output and error log files
>    * (line 8): -job-name is the name of the batch job
>    * (line 9): array is the list of gridcells for processing (1,2,3,etc or 1-3)
>* Line 12
>    * umask sets the permission for files that are creates. left digit is for user, middle digit is for group and right digit is for  
> others. 0= no permissions denied (read/write/execute allowed), 2= write permission denied, 7= all permissions denied  
>* Lines 14-22: General parameters for script. Do not need to change.
>     * (line 19): L7STOPYR is the year after which Landsat-7 images should no longer be downloaded/processed
>     * (line 21): EPSG is the coordinate system. 8858 is Equal Earth for the Americas.
>     * (line 22): BUFFER the distance beyond the grid cell boundary to extend processing (grids will overlap by this much)
> Lines 23-31 run checks for corrupt files if download directories already exist (used when script is rerun after hanging)
> Line 33 gets the current time stamp for progress reporting
> Lines 34-35: User variables for start and end dates to stream. (For Paraguay, the dates are from 1-Jan-2000 to 31-Dec-2022. For Chile, 
    the dates are from 1-Jan-1985 to 31-Dec-2022)
>*Lines 35-56: The script runs a seperate call to the download script for each month from Jan of the start year to Dec of the end year. > This is to not overwhelm the server when requesting downloads.
 
:::{note}
**For GRID IDs >999**: SLURM processors usually do not allow array numbers >999. To get around this, modify the GRID_ID on line 10 to: `GRID_ID=$(($SLURM_ARRAY_TASK_ID + X))` where  $SLURM_ARRAY_TASK_ID is your array number on line 9 and X is a number added to that to total the actual cell number
:::


## 3. Run the process
```
#if not already in bash directory, navigate there:
cd ~/code/bash/
#submit the command:
sbatch eostac_dl.py.sh
``` 
:::{note}
Current run-time estimates for single grid cells:
~ 5hrs
:::
To check on job:
```
cat sldown.bellows.909.78.err (with 909 = grid running and 78 = job number)
```
To check your position in line (works as long as one is running anything big outside of SLURM):
```
squeue -u [username]
```
to see if there are a lot of jobs before you:
```
squeue
```
see [other slurm commands here](SlurmCommands)

(checkDownload)=
## 4. Check downloading status:
Often some images do not download properly the first time. All images for a grid cell need to be downloaded (or exempted) for the remaining processing steps to work.

To check whether all images processed for a specific gridcell and task:
```
cat sldown.bellows.909.78.err (with 909 = grid running and 78 = job numb
# or, if multiple gridcells run:
ls ~/code/bash/
# then for each .err file found:
cat [filename]
```
If all images downloaded, you should see something like this (precentage bars to 100% for downloading and no errors):
![alt](/Images/cat_download_multi.png)

You can also generate a figure to see which cells downloaded fully: 
-------NOTE: We are not currently using this -------------------
```
#Activate virtual environment:
source .nasaenv/bin/activate
#Run status command (choose a cell number to zoom around for zoom-grid, usually in the middle of your line):
#For Paraguay:
eosvault status --config-file ~/project/config/config_eri_pry.yaml --out-dir ~/code/bash --grid-file /raid-cel/sandbox/sandbox-cel/paraguay_lc/vector/pry_grids.gpkg --zoom-grid <id # of cell to zoom to> --zoom-offset 200000
#For Chile:
eosvault status --config-file ~/project/config/config_eri_chile.yaml --out-dir ~/code/bash --grid-file /raid-cel/sandbox/sandbox-cel/chile_lc/vector/chl_grids.gpkg --zoom-grid <id # of cell to zoom to> --zoom-offset 200000

#Deactivate virtual environment:
deactivate
#Note you can zoom to the full figure (without cell numbers) by dropping the last two cell flags.
```
To view the Download Progress figure:
Download file to view on local desktop by opening a separate terminal and typing:
```
rsync -raz --progress <username>@ssh.eri.ucsb.edu:~/code/bash/grid_status_eosvault.png Desktop/gridstat.png
```
Alternatively, you can view the file though an FPT such as WinSCP.

The figure will look something like this:
![](/Images/grid_status.png)
The key indicates which cells loaded completely for each product. Those not listed need to be redownloaded. (Turquoise (y) means all are good)
(incomplete)=
## 5. Rerun cells that did not load completely
For cells that did not load completely, check the .err and .out files:
```
#i.e for grid cell 909, if 78 is job number for an incomplete process:
cat sldown.bellows.909.78.err
```
If the .err file indicates that the downloading process timed out for any period, you need to rerun the same download script. Repeat the run/check process until no error messages are given (note: yellow warnings about deleted files at the beginning of the script are fine. Errors for the very last period of the script may also need to be ignored, but should be noted in the downloading column of the cell spreadsheet.

## 6. Move .err and .out files to new directory to facilitate tracking of new downloads
All these files will soon make it difficult to find the new ones to monitor. Move them to an archive directory after checking to clean up the clutter.
```
#Make the directory (first time only)
mkdir /home/<username>/code/bash/archive/downloads
#Move .err and .out files from /bash to /archive
cd ~/code/bash/
mv /home/<username>/code/bash/stacdl2_py.*.{err,out} /home/<username>/code/bash/archive/downloads
```