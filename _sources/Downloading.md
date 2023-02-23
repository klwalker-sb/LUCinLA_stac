# Downloading process
===============================================================================================================================

The `Eostac` script drives the download process, [see here](https://github.com/jgrss/eostac#readme) for more information on the script process.

To run the script:
(GetCells)=
## 1. Get gridcells to download
Use the shared Google Sheet to find cells to process. Mark a / in the square for the cell/step for anything you are currently processing and and X if the step had completed and been verified as successful.

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
(Downloading)=
## 2. Edit the eostac_dl.sh document for targeted grid cells and sensor product
```
cd ~/code/bash/
vim eostac_dl_py.sh
```
To edit a line, type `i`, edit as desired, then hit `Esc` key and type `:wq`. Hit `Enter` key
Go here for [more info on editing in vim](vimCommands)

This is the default script (with line numbering added for reference below):
```
1-   #!/bin/bash -l

2- #SBATCH -N 1 # number of nodes
3- #SBATCH -n 8 # number of cores
4- #SBATCH -t 0-20:00 # time (D-HH:MM)
5- #SBATCH -p basic
6- #SBATCH -o stacdl_py.%N.%a.%j.out # STDOUT
7- #SBATCH -e stacdl_py.%N.%a.%j.err # STDERR
8- #SBATCH --job-name="stacdl_py"
9- #SBATCH --array=500-510%2

10- GRID_ID=$(($SLURM_ARRAY_TASK_ID + 3000))

11- PROJECT_HOME="/raid-cel/sandbox/sandbox-cel/paraguay_lc/stac"
12- OUT_DIR="${PROJECT_HOME}/grid/00${GRID_ID}"
13- GRID_FILE="/raid-cel/sandbox/sandbox-cel/LUCinLA_grid_8858.gpkg"
14- EPSG=8858
15- L7STOPYR=2017
16- BUFFER=100

17- ### activate the virtual environment
18- conda activate venv.lucinsa38_dl

19- TIMESTAMP0=`date "+%Y-%m-%d %H:%M:%S"`

20- START_YEAR=2010
21- END_YEAR=2023
22- YEAR=$START_YEAR
23- while [ $YEAR -ne $END_YEAR ]
24- do
25-     for m in {1..11}
26-     do
27         CURRENT_MONTH=$(printf "%02d" $m)
28-        NEXT_ITER=$(($m+1))
29-        NEXT_MONTH=$(printf "%02d" $NEXT_ITER)
30-        START_DATE="${YEAR}-${CURRENT_MONTH}-01"

31-        if [[ $m -eq 11 ]]
32-        then
33-            END_DATE="${YEAR}-${NEXT_MONTH}-31"
34-        else
35-            END_DATE="${YEAR}-${NEXT_MONTH}-01"
36-        fi

37-        echo "Working on $START_DATE to $END_DATE "	
38-        TIMESTAMP=`date "+%Y-%m-%d %H:%M:%S"`
39-        echo "$TIMESTAMP "

40-        #eostac download --start-date $START_DATE --end-date $END_DATE --bounds $GRID_FILE --bounds-query UNQ==$GRID_ID --out-path 41-        $OUT_DIR --epsg $EPSG --bounds-buffer $BUFFER --L7stop_yr $L7STOPYR --max-items -1 -w 4 -t 2

42-        done
43-        YEAR=$(($YEAR+1))
44-     done

45- TIMESTAMP=`date "+%Y-%m-%d %H:%M:%S"`
46- TIMETOT=$(($(date -d "$TIMESTAMP" "+%s") - $(date -d "$TIMESTAMP0" "+%s") ))
47- echo" Done at $TIMESTAMP " 
48- echo" full process took: $(($TIMETOT/60)) minutes "
49- source deactivate

```
Most lines should stay as they are.  
**the lines that you need to change are:** 
* **Grid info:** `#SBATCH --array=` (line 9 here). Enter the last 3 digits of each gridcell you are processing.  
     You can enter a range (e.g. 898-908), But be mindful that you are not hogging all the computer bandwidth.
     If your grid id is >999, modify the number GRID_ID line (line 10 here) to sum to the id 
          (e.g. cell 3919 = 919 in array line and 3000 in GRID_ID equation) 
     (SLURM does not accept numbers greater than 999 in the array line directly)
     
:::{admonition}You can limit the number of cells that are processed at one time by adding %n
For example, 898-908%4 would process 4 cells at a time. When the first 4 finish, the next will start.
:::
::: {warning} servers throttle downloading loads, so more downloads will fail if too many are processed at once.
%2 is safest but can do more and rerun to catch errors.
:::

After making these edits, save your script and exit:
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
>    * (line 5): -p tells SLURM this job should be sent to the standard partition (there are also 'short', 'gpu', and 'largemem' >available).
>    * (line 6&7): -o and -e are output log files
>    * (line 8): -job-name is the name of the batch job
>    * (line 9): array is the list of gridcells for processing (1,2,3,etc or 1-3)
>* Line 12
>    * umask sets the permission for files that are creates. left digit is for user, middle digit is for group and right digit is for others. 0= no permissions denied (read/write/execute allowed), 2= write permission denied, 7= all permissions denied  
>* Lines 20-21
>    * User variables for start and end dates to stream. (For Paraguay, the dates are from 1-Jan-2000 to 31-Dec-2022. For Chile, 
    the dates are from 1-Jan-1985 to 31-Dec-2022)
>*Lines 23-44: The script runs a seperate call to the download script for each month from Jan of the start year to Dec of the end year. This is to not overwhelm the server when requesting downloads.
 
:::{note}
**For GRID IDs >1000**: SLURM processors usually do not allow array numbers >1000. To get around this, modify the GRID_ID on line 29 to: `GRID_ID=$(($SLURM_ARRAY_TASK_ID + 1000))` Then subtract 1000 from the array number on line 10.
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
* Sentinel 2: 40min - 1hr
* Sentinel 2 cloudmasks: 20min
* Landsat 8: 30-40min
* Landsat 7: 20-30min
* Landsat 5: 5-10min
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
![](/Images/cat_download_multi.png)

You can also generate a figure to see which cells downloaded fully: 
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
cat sldown.bellows.909.78.out 
```
If the .out file does not indicate any big problems and the slider at the bottom of the .err file is near 100% with only a few files missing in the fraction next to it (e.g. 124/127), you probably do not need to worry about larger issues. These cells do need to be rerun, however, to register correctly in the database for further processing.
**Repeat steps 2 and 3 above**. In adition to editing the grid cells to correspond to those that you are rerunning, in the ```download_eri.sh``` file, you will also change the following line:
```
yes | no
IGNORE_INCOMPLETE="no"

#to 
yes | no
IGNORE_INCOMPLETE="yes"
```
By setting "Ignore Incomplete" to "yes", the database will populate correctly even if all of the images do not download. If they do not load the second time, they probably aren't going to, so you can move on as long as only a few images are missing. Check the .err and .out files to make sure there are no major errors, though.
```{attention} Remember to reset "IGNORE_INCOMPLETE="no"  before beginning new downloads
```

Check .err and .out files as before.

## 6. Move .err and .out files to new directory to facilitate tracking of new downloads
All these files will soon make it difficult to find the new ones to monitor. Move them to an archive directory after checking to clean up the clutter.
```
#Make the directory (first time only)
mkdir /home/<username>/code/bash/archive/
#Move .err and .out files from /bash to /archive
cd ~/code/bash/
mv /home/<username>/code/bash/*.{err,out} /home/<username>/code/bash/archive/
```