(Pipeline)=
# Pipeline process
======================================================================

Jordan Graesser's `pytuyau` script drives the pipeline process, [see here](https://github.com/jgrss/pytuyau#readme) for more information on the script process.

:::{note}The CEL drive space on the cluster is setup such that all products through step #1 of Pipeline are saved on the sandbox space (`raid-cel/sandbox/sandbox-cel` or just `/home/sandbox-cel`). These files are not backed up for more than 10 days (if you lose important files, you can probably get them recovered within 10 days, but not after). Products from step#5 of Pipeline and above are saved to the downspout space (`raid-cel/r/downspout-cel` or just `/home/downspout-cel`), which has a separate long term backup.
:::

Pipeline template (with line numbers added for reference below):

    1-   #!/bin/bash -l
    2-  
    3-   #SBATCH -N 1 # number of nodes
    4-   #SBATCH -n 8 # number of cores
    5-   #SBATCH -t 0-08:00 # time (D-HH:MM)
    6-   #SBATCH -p basic 
    7-   #SBATCH -o slpipe.%N.%a.%j.out # STDOUT
    8-   #SBATCH -e slpipe.%N.%a.%j.err # STDERR
    9-   #SBATCH --job-name="slpipe-pre"
    10-  #SBATCH --array=910-914%4
    11-
    12-  #############################################
    13-  # Turn off NumPy parallelism and rely on dask
    14-  #############################################
    15-  export OPENBLAS_NUM_THREADS=1
    16-  export MKL_NUM_THREADS=1
    17-  # This should be sufficient for OpenBlas and MKL
    18-  export OMP_NUM_THREADS=1
    19-  ################################################
    20-
    21-  # Choices are {preprocess, fusion, topo, reconstruct, segment, classify}
    22-  # 1. preprocess
    23-  #    a. Checks 'no data' images
    24-  #    b. Co-registers Sentinel
    25-  # 5. reconstruct
    26-  #    - Reconstructs weekly time series using dynamic temporal smoothing (DTS)
    27-  # 7. segment
    28-  #    - Segments vegetation objects using SACFEI
    29-  # 8. classify
    30-  #    - Classifies land cover
    31-  # 9. assess
    32-  #    - Assesses land cover predictions
    33-  # 10. clean
    34-  #    - Removes unwanted files
    35-
    36-  STEP="preprocess"
    37-
    38-  GRIDS="${SLURM_ARRAY_TASK_ID}"
    39-
    40-  #########
    41-  # MASKING
    42-  #########
    43-
    44-  MASK_ONLY="no"
    45-  REF_RES=40.0
    46-  RESET_CLOUD_DB="False"
    47-
    48- #######
    49- # CLEAN
    50- #######
    51- 
    52- # masks, downloads, nodata, nocoreg, fusion, vis, vrts, old_seg
    53- #REMOVE_ITEMS="[downloads,masks,nodata,nocoreg,fusion,vis,vrts]"
    54- REMOVE_ITEMS="[]"
    55-
    56- ########
    57- # FUSION
    58- ########
    59- 
    60- WINDOW_SIZE=9
    61- MAX_YEARS=2
    62- MAX_DAYS=15
    63- FOVERWRITE="True"
    64-
    65- #############
    66- # RECONSTRUCT
    67- #############
    68- 
    69- RINPUT="ms"
    70- START_PAD="2010-03-01"
    71- START="2010-07-01"
    72- END_PAD="2020-11-01"
    73- END="2020-07-01"
    74- SKIP_INTERVAL=7
    75- SKIP_YEARS=10
    76- REC_VI="evi2"
    77- ROVERWRITE="False"
    78- SM_CHUNKS=512
    79- PREFILL_GAPS="True"
    80- PREFILL_MAX_DAYS=50
    81- DTS_MAX_WIN=61
    82- DTS_MIN_WIN=15
    83- PREFILL_YEARS=2
    84- DTS_t=5
    85- 
    86- ############
    87- # REINDEX VI
    88- ############
    89- REX_VI="evi2"
    90- 
    91- ##########
    92- # CLASSIFY
    93- ##########
    94-
    95- # sample | fit | fit_predict | predict
    96- CLSMETHOD="fit"
    97- CINPUT="ms"
    98- # temperate_gss_vi2; tropical_gss_vi2
    99- CLS_REGION_NAME="tropical_gss_vi2"
    100- CLS_MODEL_NAME="${CLS_REGION_NAME}_11_11"
    101- LC_VECTOR=".kml"
    102- KEEP_FEATURES="[1,1,1,1]"
    103- # update/overwrite the sample stack
    104- UPDATE_SAMPLES="True"
    105- # overwrite each grid sample dataset
    106- OVERWRITE_SAMPLES="False"
    107- OVERWRITE_MODEL="True"
    108- OVERWRITE_IMAGE="False"
    109- CLS_BANDS="[evi2,wi]"
    110- CLS_NITERS=2000
    111-
    112- # OUTPUT files
    113- TRAIN_SAMPLES="/training/south_america_extract_${CLS_REGION_NAME}.gpkg"
    114- MODEL_FILE="/training/south_america_${CLS_MODEL_NAME}.model"
    115-
    116- #########
    117- # SEGMENT
    118- #########
    119- 
    120- # fit | fit_predict | predict
    121- SEGMETHOD="predict"
    122- OVERWRITE_EDGE_MODEL="False"
    123- OVERWRITE_EDGE_IMAGE="True"
    124- OVERWRITE_SEG_IMAGE="True"
    125- SEG_REGION_NAME="temperate_gss_vi2"
    126- SEG_BANDS="[evi2,wi]"
    127- SEG_NITERS=2000
    128-
    129- SEG_TRAIN_PATH="/training/edges"
    130- SEG_MODEL_FILE="/training/south_america_extract_${SEG_REGION_NAME}.edges"
    131-
    132- ###############################
    133- # DO NOT MODIFY BELOW THIS LINE
    134- ###############################
    135- 
    136- # activate the virtual environment
    137- source ~/.nasaenv/bin/activate
    138- 
    139- CONFIG_UPDATES="grids:[${GRIDS}] 
    140- main_path:/raid-cel/sandbox/sandbox-cel/paraguay_lc/raster/grids
    141- backup_path:/raid-cel/r/downspout-cel/chile_lc/raster/grids
    142- num_workers:${SLURM_CPUS_ON_NODE} cloud_mask:reset_db:${RESET_CLOUD_DB} 
    143- cloud_mask:ref_res:${REF_RES} fusion:window_size:${WINDOW_SIZE} 
    144- fusion:fill_max_years:${MAX_YEARS} fusion:fill_max_days:${MAX_DAYS} 
    145- fusion:overwrite:${FOVERWRITE} reconstruct:input:${RINPUT} reconstruct:start_pad:${START_PAD} 
    146- reconstruct:end_pad:${END_PAD} reconstruct:start:${START} reconstruct:end:${END} 
    147- reconstruct:skip_interval:${SKIP_INTERVAL} reconstruct:skip_years:${SKIP_YEARS}
    148- reconstruct:vi:${REC_VI} reconstruct:overwrite:${ROVERWRITE} reconstruct:chunks:${SM_CHUNKS}
    149- reconstruct:smooth_kwargs:max_window:${DTS_MAX_WIN}
    150- reconstruct:smooth_kwargs:min_window:${DTS_MIN_WIN}
    151- reconstruct:smooth_kwargs:prefill_max_years:${PREFILL_YEARS}
    152- reconstruct:smooth_kwargs:prefill_gaps:${PREFILL_GAPS} 
    153- reconstruct:smooth_kwargs:prefill_max_days:${PREFILL_MAX_DAYS}
    154- reconstruct:smooth_kwargs:t:${DTS_t}
    155- reindex_vi:vi:${REX_VI} 
    156- classify:lc_vector:${LC_VECTOR} classify:train_samples:${TRAIN_SAMPLES}
    157- classify:model_file:${MODEL_FILE} classify:update_samples:${UPDATE_SAMPLES}
    158- classify:overwrite_samples:${OVERWRITE_SAMPLES} classify:overwrite_model:${OVERWRITE_MODEL}
    159- classify:overwrite_image:${OVERWRITE_IMAGE} classify:method:${CLSMETHOD} classify:input:${CINPUT}
    160- classify:image_bands:${CLS_BANDS} classify:image_bands_pred:${CLS_BANDS}
    161- classify:keep_features:${KEEP_FEATURES} classify:augment:n_iters:${CLS_NITERS}
    162- segment:method:${SEGMETHOD} segment:image_bands:${SEG_BANDS}
    163- segment:overwrite_model:${OVERWRITE_EDGE_MODEL} segment:overwrite_edges:${OVERWRITE_EDGE_IMAGE}
    164- segment:overwrite_seg:${OVERWRITE_SEG_IMAGE} segment:training_path:${SEG_TRAIN_PATH}
    165- segment:model_file:${SEG_MODEL_FILE} segment:augment:n_iters:${SEG_NITERS}
    166- clean:remove_items:${REMOVE_ITEMS}"
    167- 
    168- if [ "$MASK_ONLY" == "yes" ]; then
    169- if [ "$STEP" == "preprocess" ]; then
    170- tuyau $STEP --config-updates $CONFIG_UPDATES --mask-only
    171- else
    172- tuyau $STEP --config-updates $CONFIG_UPDATES
    173- fi
    174- else
    175- tuyau $STEP --config-updates $CONFIG_UPDATES
    176- fi
    177-
    178- deactivate

Most lines should stay as they are.\
**the lines that you need to change are:**\
 * **Grid info:** `#SBATCH --array=` (line 10 here). This is where you enter the gridcells you are processing.\
You can enter a range (e.g. 898-908), But be mindful that you are not hogging all the computer bandwidth.
:::{admonition}You can limit the number of cells that are processed at one time by adding %n
For example, 898-908%4 would process 4 cells at a time. When the first 4 finish, the next will start.
:::
 * **Step:** (line 36 here). Run script for one step, then change to other. 
For initial processing, should be 'preprocess' or 'reconstruct'
 * **Rec_VI** (If running **Reconstruct**), set vegetation index here (see options in next section). To get multiple vegetation indices for a cell, run reconstruct multiple times.

## Choices for vegetation indices:
Current options for vegetation indices are:
 * avi = ?? (removed?)
 * evi2 = 2.5 * ( NIR - RED) / ( NIR + 2.4 * RED + 1.0 )
     * Enhanced Vegetation Index, good for areas of high LAI (leaf-area index), where NDVI tends to saturate
 * gcvi= scaled index using Green & NIR
     * Green Chlorophyll Vegetation Index, useful for crop classification
 * kndvi= tanh(( NIR - RED) / ( NIR + 2.4 * RED + 1.0 ))^2)
     * See [Camps-Valls et al 2021, A unified vegetation index for quantifying the terrestrial biosphere](https://www.science.org/doi/10.1126/sciadv.abc7447)
 * nbr = (NIR - SWIR2) / (NIR + SWIR2) + 1e-9)
     * Normalized Burn Index
 * wi = 0 if (red + SWIR1) > 0.5, else 1.0 - ((red + SWIR1) / 0.5))

:::{note}To add a new vegetation index to the code, edits need to be made to both the `vis.py` and `check_reconstruction.py` scripts in `~/tmp/pytuyau/pytuyau/steps`. 
ie, to add NDVI:
   in `check_reconstruction.py` around line 169, add:
   ```
   elif params['reconstruct']['vi'] == 'ndvi':
       band_names = ['nir', 'red'] 
   ```
   in `vis.py` around line 6, add name or your index to the list:
   ```
   AVAIL_VIS = ['avi', 'evi2', 'gcvi', 'kndvi', 'nbr', 'wi','ndvi']
   ```
   in `vis.py` around line 27 add:
   ```
   elif params['reconstruct']['vi'] == 'ndvi':
       vi_data = data_src.gw.ndvi(nodata=0, scale_factor=1)
   ```
   in `vis.py` under other index methods (around line 75) add:
   ```
   def ndvi(self, data):
        return self._norm(data[1], data[0])
   ```
   (note that `_norm` is defined above that as `(b2 - b1) / ((b2 + b1) + 1e-9)` there are currently another option of  
   ` _scale_min_max: ((((max_out - min_out) * (xv - min_in)) / (max_in - min_in)) + min_out)\ .clip(min_out, max_out)`, or can 
   create a different equation without self argument (use `@staticmethod` on line above index definition)
   
   After modifying script, reinstall:
   ```
   cd ~
   source .nasaenv/bin/activate
   cd tmp/pytuyau
   python setup.py build && pip install .
   ```
:::

## Run the process

    #if not already in bash directory, navigate there:
    cd ~/code/bash/
    #submit the command:
    sbatch pipeline_eri.sh

Current run-time estimates for single grid cells:
\* Preprocess
\* Reconstruct
\* Segment
\* Classify
\* Assess
\* Clean


## Get grid pipeline status

To generate the Pipeline Progress figure:

    #Activate virtual environment:
    source .nasaenv/bin/activate
    #Run status command:
    tuyau status --config-updates status:project_path:/raid-cel/sandbox/sandbox-cel/paraguay_lc/raster/grids status:out_path:<PNG location> status:grid_file:/raid-cel/sandbox/sandbox-cel/paraguay_lc/vector/pry_grids.gpkg status:zoom:True
    #Deactivate virtual environment:
    deactivate

To view the Processing Progress figure:
Download file to view on local computer:

    rsync -raz --progress <username>@ssh.eri.ucsb.edu:<ERI path> <local path>

Alternatively, you can view the file though an FPT such as WinSCP.