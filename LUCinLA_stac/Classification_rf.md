(classification_rf)=
# single-year classification
================================================================================================================================

## Building a single-year random-forest model from the smoothed time-series:

### Step 1: Choose the feature model (variable inputs to use in the model) 
Three types of features can be used in the model:
1) ***spectral-index stats (`si_vars`)***: Summary statistics and phenological variables for the modelling year generated from smoothed time-series data for each selected spectral image
2) ***singleton variables (`singleton_vars`)***: Ancillary raster datasets covering the full AOI for a single point in time (can be constant or annual, but do not provide time-series data within a given year.)
3) ***segmentation variables (`poly_vars`)***: Outputs from segmentation model (or other object-extraction method), which, unlike singleton_vars, are already parsed by cell, but like singleton_vars, do not have a time-series element.  

Choose a unique ***`feature_model`*** name for the desired combination of features

Feature models are stored in the dictionary (`feat_mod_dict`) at "/home/downspout-cel/paraguay_lc/Feature_Models.json".
This allows quick lookup by name of the feature set (spec_indices, si_vars, singleton_vars, and poly_vars) and the full band sequence of the resulting stack (in case the internal band names get stripped).

##### spectral-index variables
Spectral-index stats contain both an index component and a statistic component. For the index component, see [vegetation indices](#veg_indices) for options. The default RF model uses all six indices that we generated smoothed time series for in the pipeline process (evi2, gcvi, wi, kndvi, nbr & ndmi). Indices can be readily dropped from new models, but adding new indices will require running the ts pipeline step to generate time series data for the index for all cells involved.

The statistic component regards how the time-series data for a spectral index are to be summarized into a single value, for example from  summary statistics or phenological variables for the modelling year. The default RF model uses (Max,Min,Amp,Avg,CV,Std,Jan,Feb,Mar,Apr,May,Jun,Jul,Aug,Sep,Oct,Nov,Dec) but variables can readily be dropped or added for new models. 

Variable options:
* **Max** = Maximum Value for year
* **Min** = Minimum Value for year
* **Amp** = Amplitude across year = (Max-Min)
* **Avg** = Mean Value for year
* **Std** = Standard Deviation for year
* **CV** =  Coefficient of Variation for year = (1000*Std/Avg)
* **MaxDate** = Day of year of maximum value
* **MinDate** = Day of year of minimum value
* **MaxDateCos** = Cosine of MaxDate(scaled to 360) (so that calendar is cyclical and 31 Jan is a close to 1 Dec as it is to 30 Jan)
* **MinDateCos** = Cosine of MinDate(scaled to 360)

* **Jan** = Value for smoothed time series on 20 Jan
* **Feb** = Value for smoothed time series on 20 Feb
* ... etc.
* **Dec** = Value for smoothed time series on 20 Dec

Note that the year is defined by the parameters `start_yr` and `start_mo`, such that it starts on the first of start_mo of start_yr (so a year can be 1/Jan-31/Dec, 1/Apr-31/Mar, 1/Jun-31/May, etc. to best align with the local crop calendar and avoid splitting the primary cropping cycle)

New variables can be added within `ts_composite.py` of LUCinSA_helpers.

##### singleton variables
Singleton variables are ancillary raster datasets covering the whole area and time period. The only singleton variable currently available is `forest_strata`, which provides vegetational ecozones ("estratos_corregidos2014_6" -- TODO: get citation info from Ata).

A dictionary for singleton variables exists at: "/home/downspout-cel/paraguay_lc/singleton_var_dict.json"
This contains the name of the variable, path to the dataset, and name of the attribute to use.
To add another singleton variable, simply add the dataset at the desired location in the shared folder and add the data (manually) to the dictionary.

##### segmentation variables
Segmentation variables are outputs and summary variables from the segmentation process (TODO: link to segmentation processing description). Current variables are:

* **pred_area** =
* **pred_dist** =
* **pred_ext** =
* **pred_APR** =
* **AvgNovDec_FieldStd** =

### Step 2: make variable stack for model
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
* `--singleton_vars` is the set of singleton rasters to include (e.g. \['forest_strata'])
* `--singleton_var_dict` is the path to the dictionary defining all singleton layers (e.g. "/home/downspout-cel/paraguay_lc/singleton_var_dict.json")
* `--poly_vars` is the list of segmentation variables to include (if passed from BASH script, needs to be list inside string (e.g: "\[var1,var2,var3]")
* `--poly_path` is the path to the folder containing the segmentation variables (e.g. "/home/downspout-cel/paraguay_lc/Segmentations/RF_feats/")
* `--scratch_dir` is an optional path to the scratch directory (to save storage with processing temp files (which may be tied uo in lengthy backup period in time series directory))

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
 * `--grid_file` is the path to the file containing the geographic info for the  
 * `--feature_model` is the name of the feature model (the same used in step 2 above, used to find the stored variable stack)
 * `--start_yr` is the first year of the mapping period (the same used in step 2 above, used here to find the stored variable stack)
 * `--feature_model_dict`is the path to the dictionary with the feature model, to look up the band order (e.g. "/home/downspout-cel/${COUNTRY}_lc/Feature_Models.json")


### Step 4: choose sample model (sample points to use in training)
`Notebook 6b`
* class balance
* percent test/train
* poly-based augmentation

### Step 5: build random forest model
`LUCinSA_helpers rf_model` uses [RandomForestClassifier from scikit-learn](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html){cite:p}`scikit-learn` to fit a random forest model to the variable dataframe. Prior to fitting the model, 20% of the training data is held out to be used for model inspection in the next step (this is separate from any data held out for final map validation). The final model is saved in the `--out_dir` as {model_name}_RFmod.joblib

* `--df_in` is the path to the variable dataframe created in Step 3 (and modified in step 4).
* `--feature_model` is the same feature model name from steps 2-4 above (used here to create a consistent model name)
* `--sample_model` is a unique name for the sample model created in step 4 above (used here as part of the model name)
* `--start_yr` is the year of the mapping period (used here as the third part of the model name)
* `--importance_method` sets how feature importances are to be calculated. If `importance_method='Impurity'`, the default feature importances from the sklearn rf model are returned. If `importance_method='Permutation'`, a permutation test is run to calculate feature importances. As long as `importance_method is not 'None'`, the feature improtances are returned in the `--out_dir` as 'VarImportance_{model_name}.csv' 
* `--ran_hold` is an integer that can be used as a constant seed to hold randomization methods constant between different runs of a given model (for reproducibility). (NOTE: This is not working at the moment; repeated runs of the same model do give different results)
* `--lut` points to the lookup table with the numeric and descriptive labels for classes. Our LUT can be referenced (or downloaded) from the main directory of LUCinSA_helpers as `LUT.txt` (an abbreviated copy is provided [here](# TODO LINK THIS) for reference) 
* `--lc_mod` indicates the classification system to to be used. Our full model `Lc_mod='All'` uses the 25 classes in the `LC25` column of the LUT. (Note that this is the classification system used during model building -- more detailed classes can always be regrouped to show less detail following model-building/classification, but not vice-versa.) 
    Current classification models are:
        * **crop_nocrop = LC2**: a two-class model for crop/nocrop 
        * **crop_nocrop_medcrop = LC3**: a three class model for crop/nocrop/medcrop, 
        * **crop_nocrop_medcrop_tree = LC4**: a four class model for crop/nocrop/medcrop/tree
        * **veg' = LC5**: a five class model for vegetation type (noveg/lowveg/medveg/highveg/mature)
        * **cropType = LC_crops**: a model with specific crop classes and nocrop. 
        * **class models can be added** by adding a column to the LUT and a correspoinding line to the `get_class_col` function in `rf.py`. All unique ids (`LC_UNQ` column) must have a corresponding value in the class column. If a category should be excluded from training (e.g. because it is not specific enough), assigning a value of 99 will cause points from that category to be dropped from the dataframe prior to modelling.      

### Step 5: inspect/optimize the model
`Notebook 6c`


### Step 6: classify surface to produce map
once a model has been selected, a single-year map can be created using `LUCinSA_helpers rf_classification` 

* `--in_dir` is the path to the main time-series directory (i.e.: "/home/raid-cel/r/downspout-cel/paraguay_lc/stack/grids") 
* `--df_in` is the path to the variable dataframe created in Step 3 (and modified in step 4)
* `--feature_model` is the same feature model name from steps 2-5 above (used here to create a consistent model name)
* `--sample_model` is a unique name for the sample model from steps 4-5 above (used here as part of the model name)
* `--start_yr` is the year of the mapping period (used here as the third part of the model name)
* `--cell_list` is the list of cells to run. This can be single cell fed from the SBATCH array line or a path to a single .csv file or folder containg .csv files with the list of cells to run (no header). With the later option, the files can then be named numerically and run in an array via the SBATCH line. (examples are given in `rf3_classify_image.sh`).   
* optinal parametters from step 5 can be included if the rf model has not already been built
* optional parameters from step 2 can be included if variable stacks should be made for cells without stacks (i.e those that are not part of the sample)

### Step 7: Mosaic into map
After classifying each desired cell, the cells can be mosaicked together into a single geotiff using: `LUCinSA_helpers mosaic`

 