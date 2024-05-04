# Evaluating Climate Model Output Charter


## Problem Description



* Climate models are computational models that solve differential equations across the Earth’s surface for hundreds of different climate variables in order to simulate the Earth’s climate.
* There are tens of thousands of climate models that have been created over decades of climate research. The output of these models are consolidated into large databases such as CMIP (Climate Model Intercomparison Project) in order to aggregate and coordinate model output according to different sets of assumptions about how human greenhouse gas emissions could change in the future. These model outputs are used not just for climate research, but also to guide policy decisions intended to moderate human impact.
* Climate models separate their projections into the future and their historical simulations up to the present into separate datasets. CMIP6 standardizes climate models under its project such that historical simulations last from 1850 to 2014, representing the impact of humanity on the global climate since the industrial revolution. Analyzing historical simulations and verifying climate model performance before projections begin is a necessary task that varies considerably from model-building, and is well-suited for a data science approach
* The goal of my project is to create accessible data science tools that climate scientists and researchers can utilize to evaluate climate model historical simulations. This will benefit not just analysis of climate models from CMIP6, but analysis of climate models that are currently being designed and tested


## Scope



* The data science tools that I have produced are available in the same repository as this project charter, and involve the following:
    * **Integrating Observational Data:** Climate model output data is limited in its capacity to reliably capture real-world features. The ability to reorganize and regrid observational data into the same format as climate model output data is a key part of preliminary climate model evaluation. Specifically, this integration allows for summary analysis to take place by comparing climate model output data to observational data for any period of time between 2014 and 1960.
    * **Summary Analysis:** Aggregated information about climate model output performance relative to real-world data can be quickly and easily produced for a variety of climate models through my project. Specifically, climate model output data can be compared to observational climate data from the years 1994 to 2014. This time period is chosen specifically to account for the El Niño Southern Oscillation or ENSO cycle, which affects just about every climate variable in some way. A period of at least 20 years is a standard in climate science in order to ensure that the effects of each part of the ENSO cycle are accounted for.
    * **Regional Clustering:** In order to capture deeper insights into the patterns that are present in each climate variable, it is necessary to create a data science model that can recognize such patterns that are often invisible to human reviewers. As such, an unsupervised spectral clustering algorithm will be optimized on the grid format of the climate models and used to identify regional clusters that have the same temporal behavior for a particular climate variable.
    * **Clustering Comparisons:** Different climate variables may exhibit different regional clusterings, and so these clusterings must initially be created from observational data. A comparison can then be made between climate model output data and observational data in order to determine the similarity of regional variability across the Earth’s surface.
* Aspects of this project that were explored but considered outside the scope include:
    * **Graph-Based Representations:** While a graph-based approach to climate model evaluation was extremely promising, it ultimately resulted in data formats that were too large to work with or train on reliably. Specifically, the need for a fully-connected graph resulted in tens of thousands of edges for even coarse climate models, and I struggled to find a temporal graph neural network architecture that had successfully trained on datasets as large as mine.
    * **Growing Neural Gas:** This is an unsupervised neural network learning approach to understanding the topology of graph or grid-based data, but unfortunately the learning process of this model was very slow and demanded more computing resources than I had access to. While it was promising in the abstract, a major consideration of this project was the accessibility of the deliverables to not just climate scientists but also climate students and researchers without high performance computers, and so this approach was rejected.


## Metrics



* **Summary Analysis:** Different metrics for this tool are readily available, but RMSE is suitable for the majority of climate variables in order to capture a conservative magnitude difference on a global scale, without being overly sensitive to outlier regions.
* **Spectral Clustering:** When training the spectral clustering algorithm on observational data, accurate regional clustering metrics are not available for all climate variables. The most readily available metric, therefore, will be a comparison of the spectral clustering algorithm observed climate data against the Köppen Climate Classification Map, which indicates 30 unique land climate regions based on atmospheric temperature and precipitation. A major difference between the Köppen map and my clustering application is that my clusters will include the world’s oceans, and so it will be important to verify that the clustering algorithm can accurately distinguish coastal regions and oceanic regions. Beyond that, a referential comparison will be performed of the specific Köppen map regions identified by the clustering analysis.
* **Clustering Comparison:** When comparing clusterings of climate model output data to clusterings of observational data, the clusters may not directly line up due to the nature of the spectral clustering algorithm. For example, cluster 12 for observational data may be in a completely different area from cluster 12 in the climate model output data. Therefore, the Jaccard similarity between the grid points that constitute clusters will be used to compare the similarity between clusters. Following this, the “cluster of best fit” for each observational cluster will be the climate model cluster that has the highest Jaccard similarity with it. The average Jaccard similarity score from these “clusters of best fit” will then be used as a performance metric for the entire climate model output data. Other similarity metrics were considered and tested, such as 


## Architecture



* A partial database of CMIP6 is stored within Google Cloud Storage
    * This database is maintained by the Climate Data Science Lab at Lamont Doherty Earth Observatory, and is publicly available and free to use
    * The database has 523774 rows, each representing the time-series output of one climate variable from one climate model going through one simulation. For the purposes of my project, I will be filtering for historical simulations that are the first member of their climate model ensemble (some research teams have ensembles of climate model outputs rather than just one; they are all technically the same model, but the first member is the earliest point at which the climate model begins its historical simulation).
    * CMIP6 data is converted from Zarr stores to xarray DataArrays. Then, these DataArrays contains the following four indices:
        * **lat**: Latitude
        * **lev**: Elevation
        * **lon**: Longitude
        * **time**: Timestamp
    * Climate model output data varies in grid size wildly between institutions. Sometimes the grid will be 1 degree latitude x 1 degree longitude, other times it will be 2 x 2.5 or 1 x 1.5. The process of regridding data to fit different grid sizes is a non-trivial task that requires accounting for the curvature of the Earth and how that affects the interpolation of grid squares with different surface areas. This regridding process is purely used to preprocess observational data and not climate model output data, which is preserved in its original grid at all times.
    * The timestamps that I will be using are on monthly timescales. While climate model simulations do often run on hourly, daily, or weekly timescales, the monthly data is an aggregation that limits the size of the datasets while providing enough information about yearly climate variable trends.
* A dataset of observed climate data is available from ERA5
    * The dataset is maintained by the Copernicus Climate Change Service, and specific data (for particular variables and time periods) is accessible via API
    * The dataset contains monthly climate observations for the past 40 years across hundreds of climate variables, many of which are identical to the ones projected by CMIP models under a slightly different name. For example, “Near-Surface Air Temperature” within the CMIP6 database refers to atmospheric temperature measured in Kelvin between 1.5 and 2 meters above the surface of the Earth. The ERA5 dataset contains a similar climate variable called “2 meter temperature” that is also measured in Kelvin. These two are a natural fit and constituted the primary climate variable that I worked with across climate models, especially because they simplify the elevation coordinate.
    * ERA5 atmospheric data is on a scale of 0.25 degrees latitude x 0.25 degrees longitude, and must be regridded to be usable. Regridding ERA5 data can be handled using the Climate Data Store (CDS) API provided by the Copernicus Climate Change Service, and this is a suitable way to preprocess the data for isolated instances. In cases where ERA5 data needs to be regridded dynamically, the xESMF python library will be used.
* As previously stated, a major consideration for this project is the accessibility and understandability of the analysis. Therefore, code will be provided as part of this project’s github repository that will allow an individual user to run the model in their own Google Colab environment without any other prerequisites. Integration of further functionality will be an ongoing effort as I tweak the Google Colab implementation of this project. For example, xESMF is notoriously difficult to install within virtual environments and thus constitutes a significant hurdle for the accessibility of regridding.


## Model



* **Spectral Clustering:** This clustering algorithm was chosen over traditional k-means clustering due to its ability to capture low-resolution visualizations of higher-dimensional relationships within datasets. Specifically, this clustering algorithm has the ability to treat temporal dependencies (presented as sequences within our data) as additional features along which to gauge proximity with neighboring grid squares. However, some important considerations needed to be made within the model parameters, namely nearest-neighbor connectivity and the number of clusters to use. Considering external limitations, the number of neighbors needed to be at least 12 because any fewer did not result in a fully-connected graph, and it could not be more than 80 due to errors within the Python modules being utilized. As for the number of clusters, at least 30 clusters are needed to attempt to represent 30 distinct regions from the Köppen Climate Classification Map. However, too many clusters can also harm performance, and testing a number of clusters above 70 resulted in excessive computational load. Some direct computational methods of considering the ideal number of clusters can be utilized after finding a good inflection point for nearest-neighbor connectivity.


## Evaluation



* Below is a table containing evaluations for different numbers of clusters used and number of neighbors considered when training the spectral clustering model. Specifically, each score has one number representing the number of Köppen map regions identified by the clustering (out of 30 possible regions) along with a second number representing the number of continental coastlines that do not have an overlapping ocean and land region (out of 7 possible continents):

**Clustering Parameter Optimization**

**(# Köppen map regions identified, # Continents with clear regional coastlines)**


<table>
  <tr>
   <td>
   </td>
   <td><strong>k = 30</strong>
   </td>
   <td><strong>k = 35</strong>
   </td>
   <td><strong>k = 40</strong>
   </td>
   <td><strong>k = 45</strong>
   </td>
   <td><strong>k = 50</strong>
   </td>
   <td><strong>k = 55</strong>
   </td>
   <td><strong>k = 60</strong>
   </td>
   <td><strong>k = 65</strong>
   </td>
   <td><strong>k = 70</strong>
   </td>
  </tr>
  <tr>
   <td><strong>n = 12</strong>
   </td>
   <td>5, 0
   </td>
   <td>6, 1
   </td>
   <td>6, 1
   </td>
   <td>7, 1
   </td>
   <td>10, 1
   </td>
   <td>8, 1
   </td>
   <td>9, 1
   </td>
   <td>10, 1
   </td>
   <td>9, 1
   </td>
  </tr>
  <tr>
   <td><strong>n = 29</strong>
   </td>
   <td>10, 1
   </td>
   <td>14, 2
   </td>
   <td>15, 2
   </td>
   <td>15, 2
   </td>
   <td>15, 2
   </td>
   <td>15, 2
   </td>
   <td>14, 2
   </td>
   <td>15, 2
   </td>
   <td>16, 2
   </td>
  </tr>
  <tr>
   <td><strong>n = 46</strong>
   </td>
   <td>12, 2
   </td>
   <td>17, 3
   </td>
   <td>18, 3
   </td>
   <td>21, 4
   </td>
   <td>22, 4
   </td>
   <td>22, 3
   </td>
   <td>21, 4
   </td>
   <td>20, 4
   </td>
   <td>22, 4
   </td>
  </tr>
  <tr>
   <td><strong>n = 63</strong>
   </td>
   <td>18, 3
   </td>
   <td>19, 3
   </td>
   <td>20, 4
   </td>
   <td>23, 5
   </td>
   <td>25, 6
   </td>
   <td>23, 5
   </td>
   <td>24, 5
   </td>
   <td>21, 4
   </td>
   <td>23, 4
   </td>
  </tr>
  <tr>
   <td><strong>n = 80</strong>
   </td>
   <td>20, 3
   </td>
   <td>20, 4
   </td>
   <td>21, 4
   </td>
   <td>23, 4
   </td>
   <td>25, 6
   </td>
   <td>23, 5
   </td>
   <td>24, 5
   </td>
   <td>23, 4
   </td>
   <td>23, 5
   </td>
  </tr>
</table>




* As a result, parameters of n = 64 and k = 50 were chosen due to balancing computational load with performance. This set of parameters was then utilized to train a spectral clustering model on both the observational ERA5 dataset and an example model in the NOAA-GFDL CM4’s climate model output.
* **Clustering Comparison:** The average Jaccard similarity between the clustering for the GFDL’s climate model output and the ERA5 dataset was 0.1814, with each of the 50 observational clusters producing the following Jaccard similarity scores with respect to their climate model clusters of best fit: \
jaccard_indices = [

    0.0991, 0.1773, 0.2052, 0.1367, 0.1032, 0.0968, 0.1667, 0.1260, 0.1967, 0.1626,


    0.1665, 0.0700, 0.1712, 0.1896, 0.1593, 0.1195, 0.3802, 0.1318, 0.1090, 0.0953,


    0.2355, 0.0837, 0.1606, 0.1741, 0.0956, 0.4005, 0.0914, 0.2313, 0.1560, 0.1713,


    0.1622, 0.1555, 0.2347, 0.0745, 0.2635, 0.0902, 0.1627, 0.3010, 0.1135, 0.1902,


    0.1948, 0.0954, 0.1219, 0.1798, 0.0806, 0.1380, 0.1813, 0.1123, 0.1393, 0.1513]



## Conclusion



* This project was a really difficult undertaking in a field that I had very little experience in before this semester. I am incredibly grateful for the advice and climate science domain knowledge that Dr. Beadling provided me with throughout the past few months. Without her, my project would not have been possible. In the future, I would like to iterate on this work with more wisdom, time, and energy.
