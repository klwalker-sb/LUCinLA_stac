(post)=
# Post-download processing 
================================================================================================================================

## 1. Edit the eostac_brdf.sh document for targeted grid cells and sensor product
```
cd ~/code/bash/
vim eostac_brdf_py.sh
```
To edit a line, type `i`, edit as desired, then hit `Esc` key and type `:wq`. Hit `Enter` key
Go here for [more info on editing in vim](vimCommands)

This is the default script (with line numbering added for reference below):
Default script
```
1-  #!/bin/bash -l

2-  #SBATCH -N 1 # number of nodes
3-  #SBATCH -n 8 # number of cores
4-  #SBATCH -t 0-48:00 # time (D-HH:MM)
5-  #SBATCH -p basic
6-  #SBATCH -o stac_brdf_py.%N.%a.%j.out # STDOUT
7-  #SBATCH -e stac_brdf_py.%N.%a.%j.err # STDERR
8-  #SBATCH --job-name="brdf_py"
9-  #SBATCH --array=932

    ####Run as array job:
10- GRID_ID=$(($SLURM_ARRAY_TASK_ID + 3000))

    ####Set permissions on output files
11- umask 002

    ###Settables:
12-  PROJECT_DIR="/raid-cel/sandbox/sandbox-cel/paraguay_lc"
13-  PROJECT_PATH="${PROJECT_DIR}/stac/grid/00${GRID_ID}/"
14-  OUT_PATH="${PROJECT_DIR}/stac/grid/00${GRID_ID}/brdf"
15-  GRID_FILE="/raid-cel/sandbox/sandbox-cel/LUCinLA_grid_8858.gpkg"
16-  COEFFS="~/tmp/eostac/files/coefficients"
17-  EPSG=8858
18-  BUFFER=100

    ###################################################################
    ### activate the virtual environment
19-  conda activate venv.lucinsa38_dl

20-  START_YEAR=2000
21-  END_YEAR=2023
22-  y=$START_YEAR
23-  while [ $y -ne $END_YEAR ]
24-  do
25-    START_DATE="${y}-1-01"
26-    END_DATE="${y}-12-31"
27-    echo  Working on $START_DATE to $END_DATE >&2 
28-    TIMESTAMP=`date "+%Y-%m-%d %H:%M:%S"`
29-    echo $TIMESTAMP >&2

30-    eostac brdf --start-date $START_DATE --end-date $END_DATE --project-path $PROJECT_PATH --out-path $OUT_PATH --threads 8 --apply- 
       bandpass --coeffs-path $COEFFS

31-    y=$(($y+1))
32-  done

33-  TIMETOT=$(($(date -d "$TIMESTAMP" "+%s") - $(date -d "$TIMESTAMP0" "+%s") ))
34-  TIMESTAMP=`date "+%Y-%m-%d %H:%M:%S"`
35-  echo done at $TIMESTAMP >&2
36-  echo full process took: $({$TIMETOT}/60) minutes >&2

37-  conda deactivate
````
Most lines should stay as they are.
As with the downloading script, **you need to change Grid info lines**:  
**Grid info:** `#SBATCH --array= ` (line 9 here). This is where you enter the last 3 digits of the gridcells you are processing.  
     You can enter a range (e.g. 898-908), But be mindful that you are not hogging all the computer bandwidth.
:::{admonition}You can limit the number of cells that are processed at one time by adding %n
For example, 898-908%4 would process 4 cells at a time. When the first 4 finish, the next will start.
:::
**Grid_ID_prefix** (line 10 here): GRID_ID=$(($SLURM_ARRAY_TASK_ID + 3000)) change final number so that the final sum = your actual grid id


## 2. Run the process
```
#if not already in bash directory, navigate there:
cd ~/code/bash/
#submit the command:
sbatch eostac_brdf_py.sh
``` 
:::{note}
Current run-time estimates for single grid cells:
14-20hrs
:::