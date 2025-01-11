(classificationSteps)=
# modelling / classification steps
================================================================================================================================

### Step 1: Choose the feature model (variable inputs to use in the model) 
see [here](featureMod) for description of feature models and options 

### Step 2: Make variable stack for model
This stacks all variables into a single data stack for each cell, which makes for more efficient calculations (especially at the point of surface-level classification)

:::{warning} Making a physical variable stack can be computationally expensive (up to 1 GB stored data per cell and 1-2 GB temp data in backup limbo (unless using scratch_dir option). The stack does not need to be physically made if there is an existing stack that contains all of the desired bands. Steps 3 and 5 have option to build a smaller virtual stack based on a larger data stack.
:::

Run `LUCinSA_helpers make_var_stack` using the configuration in the Bash Script, `rf0_raster_var_stack.sh`
* `--in_dir` is the path to the main time-series directory (i.e.: "/home/raid-cel/r/downspout-cel/paraguay_lc/stack/grids")
* `--cell_list` is the list of cells to run. This can be single cell fed from the SBATCH array line or a path to a single .csv file or folder containg .csv files with the list of cells to run (no header). With the later option, the files can then be named numerically and run in an array via the SBATCH line. (examples are given in `rf0_raster_var_stack.sh`).   
* `--start_yr` is the year of the mapping period (first year if the cropping calendar spans two years).
* `--start_mo` is the first month of the corpping calendar, in numerical form (1 if calendar is Jan-Dec. 6 if calendar is Jun-May)
* `--feature_model` is the name of the feature model (defining the spec_vars, si_vars, singleton_vars and poly_vars). this is stored in the * `--feature_mod_dict` is the path to the dictionary containing all feature models. If the feature model does not already exist, it will be added to the dictionary based on the parameters here. If it already exists, the parameters here will be filled with those already set in the dictionary)   
* `--spec_indices` the set of vegetation index to include (e.g \[evi2,wi,gcvi,kndvi,ndmi,nbr] or a smaller subset. If passed from BASH script, needs to be list inside string (e.g: "\[var1,var2,var3]")
* `--si_vars` sets the raster variables to include for the spec indices. (\[max, min, ...]) To maintain structure and avoid confusion, the same set of si_vars should be used for all vegetation indices used in the model. (This dataset can be culled at step 4.) If passed from BASH script, needs to be list inside string (e.g: "\[var1,var2,var3]")
* `--pheno_vars` the set of season specific phenological variables to incude (e.g. \[sosd_wet, eosd_wet, los_wet, ...]
* `--spec_indices_pheno` the set of vegetation indices to get phenological variables for
* `--singleton_vars` is the set of singleton rasters to include (e.g. \['forest_strata'])
* `--singleton_var_dict` is the path to the dictionary defining all singleton layers (e.g. "/home/downspout-cel/paraguay_lc/singleton_var_dict.json")
* `--poly_vars` is the list of segmentation variables to include (if passed from BASH script, needs to be list inside string (e.g: "\[var1,var2,var3]")
* `--poly_path` is the path to the folder containing the segmentation variables (e.g. "/home/downspout-cel/paraguay_lc/Segmentations/RF_feats/")
* --`combo_vars` is a list of all components (spec index + si_var) of any remaining variables that have not already been included. This is in case a particular variable should be included for only a single index or subset of indices. (e.g. evi2_cv_year)
* `--scratch_dir` is an optional path to the scratch directory (to save storage with processing temp files (which may be tied up in lengthy backup period in time series directory))

These same sets of spec_indices, si_var, singleton_vars, and poly_vars should be used to make the variable dataframe in the next step.

### Step 3: make variable dataframe for all sample points
To extract the value of each raster variable to each sample point and construct the dataframe used to build the random forest model, run `LUCinSA_helpers make_var_dataframe` via the Bash script: `rf1_var_data_frame.sh`. 

The sample data can either be in point or polygon form, indicated by whether `--Load_samp` is True.
* If points: 
    * `--load_samp` is set to True
    * `--ptfile` is used to supply the path to the point file, which needs to be in the form of .csv with and 'OID_' column containing unique identifiers for each point (as integers) and 'XCoord' and 'YCoord'columns containing the x and y coordinates in the same coordinate system as grid_file
    
 * if polygons:
     * `--load_samp` is set to False
     * `--ground_polys` is used to supply the path to the polygon file. The polygon file should be a shapefile or geopackage.
     * `--newest` and `--oldest` can be used to filter polygons to those from a certain year or set of years based on an attribute in the polygon file with any of the following labels: 'FirstYrObs','ObsYr','Year' or 'Acquired' (if 'Acquired, it can be in the form YYYYMMDD). --newest and --oldest can be the same year if only a single year is desired. If no year filter is desired, set --newest and --oldest to 0. 
     * `--npts` sets the number of random points that are drawn from each polygon. The variable data will be extracted to these points, and the output will then be a dataframe, as with a point sample.
     
other parameters:
 * `--in_dir` is the path to the main time-series directory (i.e.: "/home/raid-cel/r/downspout-cel/paraguay_lc/stack/grids") 
 * `--out_dir` is the path to the folder in which to store the output dataframe
 * `--cell_list` are the cells within which to search for points/polygons. This can be either a list of unique cell ids or a path to a .csv file containing the list (no header). This helps narrow the search and reduce time, but can be broad (if a cell on the list does not contain sample data, it is skipped over).
 * `--grid_file` is the path to the file containing the geographic info for each cell, to speed processing
 * `--feature_model` is the name of the feature model (the same used in step 2 above, used to find the stored variable stack)
 * `--start_yr` is the first year of the mapping period (the same used in step 2 above, used here to find the stored variable stack)
 * `--feature_model_dict`is the path to the dictionary with the feature model, to look up the band order (e.g. "/home/downspout-cel/${COUNTRY}_lc/Feature_Models.json")


### Step 4: choose sample model (sample points to use in training)
Sample models indicate the subset of points to be used for training the model. See [here](#sampleMod) for sample model description and options. To facilitate replication, sample models are stored as .csv files containing the observation data for the selected points. These are stored in `{country}_lc/vector/pts_training/pt_subsets`. 

To create a new sample set:
see `LUCinSA_helpers Notebook 6b`
TOOD: copy sample balancing methods from CollectCube

(fixedHo)=
#### Step4b: Separate a fixed holdout from the sample set to se for model optimization
A fixed holdout set can be used to remove the exact same set of testing points from all sample sets to facilitate model testing and optimization (a separate holdout should already be removed from all data and never seen until the end for the purpose of final model validation). The fixed holdout sets are stored in `/home/downspout-cel/paraguay_lc/vector/pts_calval/fixed_HOs.` Holdout sets need to be generated for each feature model, but should contain the sampe points (with different variable sets in accordance with the feature model) holdout sets have been created in Collect_Cube (TODO: consolidate code). These include:
* `{feature_model}_HOLDOUT_smallCrop`,
* `{feature_model}_HOLDOUT_bigCrop`,
* `{feature_model}_HOLDOUT_noCrop`,
* `{feature_model}_HOLDOUT_medCrop`
(noteL these can all can be combined as `{feature_model}_HOLDOUT_all` and future steps will parse into the above sets)

(classificationRf)=
## Building a single-year random-forest model from the smoothed time-series:

### Step 5: build random forest model
`LUCinSA_helpers rf_model` uses [RandomForestClassifier from scikit-learn](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html){cite:p}`scikit-learn` to fit a random forest model to the variable dataframe. Prior to fitting the model, a percentage (= to `thresh`) of the training data is held out to be used for model inspection in the next step (this is separate from any data held out for final map validation). 
The final model is saved in the `--out_dir` as `{feature_model}_{sample_model}_{schematic_model}_{training_yrs}_RFmod.joblib`
Variable imporatnce and holdout results can also be saved under same model name.  

* `--df_in` is the path to the variable dataframe created in Step 3 (and modified in step 4). If using a fixed holdout, this it the TRAINING data left after the holdout sets are all extracted.
* `--out_dir` is the directory to which the model will be saved
* `--feature_model` is the same feature model name from steps 2-4 above (used here to create a consistent model name)
* `--sample_model` is a unique name for the sample model created in step 4 above (used here as part of the model name)
* `--lc_mod` is the classification model to be used [see_here](classMod) (used here to create a consistent model name)
* `--lut` points to the lookup table with the numeric and descriptive labels for classes. Our LUT can be referenced (or downloaded) from the main directory of LUCinSA_helpers as `LUT.txt` (an abbreviated copy is provided [here](# TODO LINK THIS) for reference)  
* `--train-yrs` is the year or years from which training data are supplied. (can be a single integer or list)
* `--importance_method` sets how feature importances are to be calculated. If `importance_method='Impurity'`, the default feature importances from the sklearn rf model are returned. If `importance_method='Permutation'`, a permutation test is run to calculate feature importances. As long as `importance_method is not 'None'`, the feature improtances are returned in the `--out_dir` as 'VarImportance_{model_name}.csv' 
* `--ran_hold` is an integer that can be used as a constant seed to hold randomization methods constant between different runs of a given model (for reproducibility). (NOTE: This is not working at the moment; repeated runs of the same model do give different results)
* `--thresh` is the threshold for selecting the test set. If using a fixed holdout, set `--thresh = 0`
Optional parameters:
* `--fixed_ho` is directory containing fixed holdout sets if to be used. [see here](fixedHo) 
* `--feature_mod_dict` is the dictionary containing the data for each feature model. only use here if the bands in the variable data frame (df_in) do not match the bands for the feature model in the dictionary (which should not be the case if the above steps were followed)
*`--update_model_dict` set to `True` if the above is relevant
*`--runnum` can be used to assign a new number to each model for model iteration/optimization
* `--existing_mod` can be set to point to an existing model (for example the best in an iteration set) to generate the other outputs from this function without overwriting the actual rf model.

             
### Step 5: inspect/optimize the model
`Notebook 6c`


### Step 6: classify surface to produce map
once a model has been selected, a single-year map can be created using `LUCinSA_helpers rf_classification` 

* `--in_dir` is the path to the main time-series directory (i.e.: "/home/raid-cel/r/downspout-cel/paraguay_lc/stack/grids") 
* `--df_in` is the path to the variable dataframe created in Step 3 (and modified in step 4)
* `--feature_model` is the same feature model name from steps 2-5 above (used here to create a consistent model name)
* `--sample_model_name` is a unique name for the sample model from steps 4-5 above (used here as part of the model name)
* `--train-yrs` is the year or years from which training data are supplied. (can be a single integer or list)
* `--start_yr` is the first year of the mapping period (if mapping period spans two years), otherwise simply the mapping year
* `--start_mo` is the first month of the mapping period (usually =1 for Jan if in N. hemisphere, 7 for July if in S. hemisphere)
* `--cell_list` is the list of cells to run. This can be single cell fed from the SBATCH array line or a path to a single .csv file or folder containg .csv files with the list of cells to run (no header). With the later option, the files can then be named numerically and run in an array via the SBATCH line. (examples are given in `rf3_classify_image.sh`).   
* `--feature_mod_dict` is the dictionary containing the specs for each feature model
* `--rf_mod` is the path to the rf model to be used
* `--img_out` is the name of the classified image if an alternative to the default is desired
* optinal parametters from step 5 can be included if the rf model has not already been built
* optional parameters from step 2 can be included if variable stacks should be made for cells without stacks (i.e those that are not part of the sample)
                      

### Step 7: Mosaic into map
After classifying each desired cell, the cells can be mosaicked together into a single geotiff using: `LUCinSA_helpers mosaic`

 