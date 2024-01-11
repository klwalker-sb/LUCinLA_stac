(classification_rf)=
# single-year classification
================================================================================================================================

## Building a single-year random-forest model from the smoothed time-series:


### Step 1: Choose the feature model (variable inputs to use in the model) 
Three types of features can be used in the model:
1) ***spectral-index stats (`si_vars`)***: Summary statistics and phenological variables for the modelling year generated from smoothed time-series data for each selected spectral image
2) ***singleton variables (`singleton_vars`)***: Ancillary raster datasets covering the full AOI for a single point in time (can be constant or annual, but do not provide time-series data within a given year.)
3) ***segmentation variables (`poly_vars`)***: Outputs from segmentation model, which, unlike singleton_vars, are already parsed by cell, but like singleton_vars, do not have a time-series element.  

##### spectral-index variables
Spectral-index stats contain both an index component and a statistic component. For the index component, see [vegetation indices](#veg_indices) for options. The default RF model uses all six indices that we generated smoothed time series for in the pipeline process (evi2, gcvi, wi, kndvi, nbr & ndmi). Indices can be readily dropped from new models, but adding new indices will require running the ts pipeline step to generate time series data for the index for all cells involved.

The statistic component regards how the time-series data for a spectral index are to be summarized into a single value, for example from  summary statistics or phenological variables for the modelling year. The default RF model uses (Max,Min,Amp,Avg,CV,Std,Jan,Feb,Mar,Apr,May,Jun,Jul,Aug,Sep,Oct,Nov,Dec) but variables can readily be dropped or added for new models. 

Variable options:
* Max = 
* Min = 
* (finishing this...)

##### singleton variables
Currently, the only singleton variable available is `forest_strata`, which provides vegetational ecozones

##### segmentation variables
(in progress...)

### Step 2: make variable stack for model
Feature_model
Run `LUCinSA_helpers make_var_stack` using the configuration in the Bash Script, `rf0_raster_var_stack.sh`
* `in_dir` is the path to the outer ts directory for a cell (i.e. the parent cell folder containing all the ts data) 
* feature_model` the name of the feature model (defining the spec_vars, si_vars, singleton_vars and poly_vars)> this is stored in the `feature_mod_dict`. If the feature model does not already exist, it will be added to the dictionary based on the parameters here. If it already exists, the parameters here will be filled with those already set in the dictionary)  
* `--feature_mod_dict` is the path to the dictionary containing all feature models
* `--spec_indices` the set of vegetation index to run.
* `--si_vars` sets the raster variables for the spec indices. To maintain structure and avoid confusion, the same set of si_vars should be used for all vegetation indices used in the model. (This dataset can be culled at step 4.) 
* `--singleton_vars`
* `--singleton_var_dict` is the path to the dictionary defining all singleton layers 
*`--poly_vars`
These same sets of VIs and SI_VARS should be used to make the variable dataframe in the next step.

### Step 3: make variable dataframe for all sample points
To extract the value of each raster variable to each sample point and construct the dataframe used to build the random forest model, run `LUCinSA_helpers make_var_dataframe` via the Bash script: `var_dataframe.sh`. 
The sample data can either be in point or polygon form, depending on whether `--Load_samp` is True.
* If points: 
    * `--load_samp` is set to True
    *`--ptfile` is used to supply the path to the point file, which needs to be in the form of .csv with and 'OID_' column containing unique identifires for each point (as integers) and 'XCoord' and 'YCoord'columns containing the x and y coordinates in the same coordinate system as grid_file
    
 * if polygons:
     * `--load_samp` is set to False
     * `--ground_polys` is used to supply the path to the polygon file. The polygon file should be a shapefile or geopackage.
     * `--newest` and `--oldest` can be used to filter polygons to those from a certain year or set of years based on an attribute in the polygon file with any of the following labels: 'FirstYrObs','ObsYr','Year' or 'Acquired' (if 'Acquired, it can be in the form YYYYMMDD). --newest and --oldest can be the same year if only a single year is desired. If no year filter is desired, set --newest and --oldest to 0. 
     * `--npts` sets the number of random points that are drawn from each polygon. The variable data will be extracted to these points, and the output will be a dataframe.
     * `--cells` indicates the cells within which to search for points/polygons. This can be either a list of unique cell ids or a path to a .csv file containing the list (no header). This helps narrow the search and reduce time, but can be broad -- Every cell in the list does not need to contain sample data.

### Step 4: choose sample model (sample points to use in training)
`Notebook 6a`
* class balance
* percent test/train
* poly-based augmentation

### Step 5: build random forest model
`LUCinSA_helpers rf_model` uses [RandomForestClassifier from scikit-learn](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html){cite:p}`scikit-learn` to fit a random forest model to the variable dataframe. Prior to fitting the model, 20% of the training data is held out to be used for model inspection in the next step (this is separate from any data held out for final map validation). The final model is saved in the `--out_dir` as {model_name}_RFmod.joblib
* `--df_in` is the path to the variable dataframe created in Step 3.
* `--model_name` is a unique string that will appear in all output names to distinguish between multiple models in configuration testing/optimization.
* `--importance_method` sets how feature importances are to be calculated. If `importance_method='Impurity'`, the default feature importances from the sklearn rf model are returned. If `importance_method='Permutation'`, a permutation test is run to calculate feature importances. As long as `importance_method is not 'None'`, the feature improtances are returned in the `--out_dir` as 'VarImportance_{model_name}.csv' 
* `--ran_hold` is an integer that can be used as a constant seed to hold randomization methods constant between different runs of a given model (for reproducibility). (NOTE: This is not working at the moment; repeated runs of the same model do give different results)
* `--lut` points to the lookup table with the numeric and descriptive labels for classes. Our LUT can be referenced (or downloaded) from the main directory of LUCinSA_helpers as `LUT.txt` (an abbreviated copy is provided [here](#LINK THIS) for reference) 
* `--lc_mod` indicates the classification system to to be used. Our full model `Lc_mod='All'` uses the 25 classes in the `LC25` column of the LUT. (Note that this is the classification system used during model building -- more detailed classes can always be regrouped to show less detail following model-building/classification, but not vice-versa.) 
Current classification models are:
* `crop_nocrop`=`LC2`: a two-class model for crop/nocrop 
* `crop_nocrop_medcrop`=`LC3`: a three class model for crop/nocrop/medcrop, 
* `crop_nocrop_medcrop_tree`=`LC4`: a four class model for crop/nocrop/medcrop/tree
* `'veg'`=`LC5`: a five class model for vegetation type (noveg/lowveg/medveg/highveg/mature)
* `cropType` = `LC_crops`: a model with specific crop classes and nocrop. 
* **class models can be added** by adding a column to the LUT and a correspoinding line to the `get_class_col` function in `rf.py`. All unique ids (`LC_UNQ` column) must have a corresponding value in the class column. If a category should be excluded from training (e.g. because it is not specific enough), assigning a value of 99 will cause points from that category to be dropped from the dataframe prior to modelling.      

### Step 5: inspect/optimize the model
`Notebook 6b`
When a model is created that is useful as a candidate for a final model or as a comparison case, its configuration can be stored in the dictionary called `Feature_Models.json`. This dictionary currently stores the lists of spec_indices, spec_index_vars, and singleton_vars so that they can be looked up in the future. An example of this dictionary is currently in the same folder as the LUT (in the main directory of LUCinSA_helpers), but should be moved to the shared project directory if multiple users are generating models.    

choose sample_model
### Step 6: classify surface to produce map
once a model has been selected, a single-year map can be created using `LUCinSA_helpers rf_classification` 

After classifying each desired cell, the cells can be mosaicked together into a single geotiff using: `LUCinSA_helpers mosaic`

 