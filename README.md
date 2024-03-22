# Project Charter
## Problem Description

* The Earth’s climate is changing over time, and humanity is having an impact on the Earth’s climate on a global scale. In the present day, climate models simulate the Earth’s climate and use that to predict the trajectory of the climate into the future. These predictions are called “projections” to distinguish their long-term nature from short-term weather predictions
* There are tens of thousands of climate models that have been created over decades of climate research. The output of these models are consolidated into large databases such as CMIP (Climate Model * * * * * Intercomparison Project) in order to aggregate model output into different sets of predictions. CMIP6 is the most recent iteration of CMIP, and it uses a variable called SSP (Shared Socioeconomic Pathways) to project different scenarios. SSP is determined based on possible rates of greenhouse gas emission from humanity along with many other socioeconomic factors. Ultimately, it is a representation of the spectrum of effect humanity’s greenhouse gas emissions can have on the climate by 2100, between the low end goal (1.5 degrees C of warming) set by the Paris Agreement and doing nothing while maintaining our current emissions. Climate model projections with specific SSP values are called “scenarios”
* In the face of the escalating risk of climate change, these climate models are the only tool that we have to guide policy decisions. As a result, we need to ensure two things:
  * We need confidence in the output of these simulations. Improving credibility in climate simulations involves improving the climate models
  * We need as much meaningful information as we can from our existing climate models. This means developing better and more appropriate techniques towards drawing information out of these models
* Consolidating thousands of different climate models has typically been done by taking a plain average of every model’s output under different SSP values, and using those averages as the overall projections for “high emission”, “moderate emission”, “low emission” scenarios. However, there's a growing consensus among climate scientists that there is no "model democracy" because not all models are equally accurate, and some models have outlier predictions based on their makeup
* A framework for evaluating climate model output is therefore necessary. Manual approaches to climate model evaluation by climate scientists takes a very long time. A data science solution to climate model evaluation can make this process faster, easier, and more informative for climate scientists to build better models

## Scope
* I am not a climate scientist and do not have climate science expertise; as a result, the scope of my contribution will be limited to data science. I will be working strictly with the output of existing climate models rather than building or tweaking those models
* The data science framework that I will produce will involve the following:
  * Each climate model output in CMIP6 has a historical simulation and a scenario projection. The historical output simulates the climate by forcing known historical conditions up until 2014. The scenario projection then projects how the climate will develop according to different SSP values from the end of its historical simulation up until 2100. We can separate the database of all climate models into one historical simulation and a set of SSP scenarios that are part of the same model output
  * Between 2014 and today, we have observed climate data that has been collected and validated from all around the world
  * A machine learning model will be developed as a performance metric in order to grade each climate model’s output for each scenario with regards to the actual observed climate data
  * The performance metric models will be used to create a time-series ensemble machine learning model that uses this metric as a weighting scheme for voting. This metric will start at 1 for each model, indicating a plain average estimation
  * The ensemble model will then be trained on a subset of observed data, adjusting the weights for each model based on its performance against the observed data
  * Once the model has been trained, a post-hoc analysis of the weighting scheme will be done to gather the lowest-weighted models and/or the worst-performing variables across models, each of which are possible model bias or blindspots
* The ensemble model will be presented as a “more accurate” projection of the future climate based on observed trends than a simple plain average
* The secondary analysis of potential systemic bias in models will be limited to identifying the features rather than speculating on the source of bias (that’s for climate scientists)

## Metrics
* The qualitative objective is to provide a sensible framework for climate model evaluation in aggregate
* The quantitative objective is to improve ensemble predictions made by CMIP6 through the use of machine learning
* The predictions will be compared via time-series comparison metrics (such as SMAPE) of the predictions made via plain average and via weighted ensemble


## Architecture
* A partial database of CMIP6 is stored within Google Cloud Storage
  * This database is maintained by the Climate Data Science Lab at Lamont Doherty Earth Observatory, and is publicly available and free to use
  * The database has 523774 rows, each representing the time-series output of one climate variable from one climate model going through one simulation. The relevant features are:
    * activity_id: For my purposes, either CMIP for historical simulations or ScenarioMIP for scenario projections based on SSP
    * institution_id: The name of the institution where the model comes from
    * source_id: The unique name of the climate model
    * experiment_id: For my purposes, either ‘historical’ or ‘ssp###’ for the SSP value used
    * member_id: CMIP’s ensemble member ID
    * table_id: Specifying realm (atmosphere, ocean, ice, land, etc) and frequency of output (hourly, daily, monthly, annual, etc)
    * variable_id: The climate variable this model output contains data for
    * grid_label: Indicates whether the model output is on the native grid or is regridded. Gridding is important because on the Earth’s surface, each square created by longitude/latitude is not equal due to curvature, so storing data as (latitude, longitude) needs to account for the area disparity
z_store: The path to the Zarr store where the model output is stored
  * Then, the actual model output is found using the following four indices:
    * lat: Latitude
    * lev: Elevation
    * lon: Longitude
    * time: Timestamp

* A dataset of observed climate data is available from ERA5
  * The dataset is maintained by the Copernicus Climate Change Service, and specific data (for particular variables and time periods) is accessible via request
  * The dataset contains hourly climate observations for the past 40 years across hundreds of climate variables, many of which are identical to the ones projected by CMIP models
  * Due to the nature of the dataset, retrieving any part of it and looking through it requires a waiting period and foresight into what data will be necessary
  * May need to be regridded to fit the format of CMIP

* Python will be used for all aspects of this project, using several modules specifically to access Google Cloud Storage, read and convert Zarr stores to dataframes, and analyze climate data

## Plan
* Phase 0 (complete)
  * Accessing the partial CMIP6 climate model database accessible via Google Cloud Storage
  * Filling in gaps of domain knowledge to interpret and manipulate CMIP6 data
* Phase 1 (in progress)
  * Identify a few climate variables that have readily accessible observed data for the past 10-20 years
  * Select a partition of the partial CMIP6 database that contains models that simulated the variables from the previous step
  * Sanity check each variable from observed data to ensure it is compatible with the CMIP6 format; regrid if necessary
  * Weed out potential model bias and improper methodology before continuing
* Phase 2
  * Establish a prototype of the ensemble model training and evaluation workflow, using a limited subset of models and climate variables
  * Demo prototype; analyze time to run, limitations, model performance, etc.
* Phase 3
  * Iterate on prototype ensemble model using lessons learned from previous phase
  * Weigh cost/benefit of expanding the ensemble model to more variables and climate models; expand if reasonable
  * Finalize ensemble model with maximal data and complexity based on limitations
* Phase 4
  * Run ensemble model; analyze performance and evaluate against plain average
  * Collect model data and visualizations across noteworthy climate variables
  * Perform post-hoc analysis on worst-performing variables and climate models

## Personnel
* Malvin Prifti
* Dr. Rebecca Beadling (consulted for domain expertise in climate modeling)

## Communication
* Moving forward, I will be meeting with Dr. Beadling weekly on Fridays to discuss progress, ask domain questions, and orient towards applications that are useful for climate science
* I will also be available via email and in class to discuss project details

