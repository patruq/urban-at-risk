# Identifying At-Risk Areas in Urban Settings

### Can public safety data be used as an accurate classifier for city neighborhoods?

-------------------------------------------------

***Executive Summary***

As municipalities around the country modernize existing information systems for the 21st century, data on critical municipal functions are becoming increasingly available to the public.

In exploring how sparse local government resources may be best allocated towards specific locations and areas that are in the most need of intervention, traditional approaches, like examining crime statistics to deduce problem area “hotspots”, may not be providing us with the most complete picture of urban health.

The problem I set out to answer was, How does safety data inform us on the identification of areas suffering from risks to public safety. Here, the geographic size-area being predicted is at the zip code level. There were several reasons for this, logistical constraints on initial project implementation being one of them. The hope is that this body of work, which works at a “higher level” of prediction analysis, can, with continued research, work as a prediction model for more geographically-specific locations.


### Notes on the Data:
- There were 4 datasets explored in this project: 3 pertaining to public safety information, and one used the “base” dataset upon which to the others were joined together.

- All data obtained pertained to calnedar year 2019

- The property assessment dataset served as the backdrop of the project- using this data, I was able to tie together each PW violation, fire incident, and applicable 311 request to a given property in Boston over the entirety of 2019.

- Each of the public safety datasets provided detailed information across a number of metrics localized to specific locations around the city.

- All of the data was publicly provided via Analyze Boston, an online portal for exploring data directly collected by the city on various aspects of city management. As we can see, each contains a lot of data to sift through. Cleaning these datasets into a workable database of public safety issues mapped to location proved to be a formidable task.

****
**Property Assessment**
- Unique properties registered with the city of Boston. Contains many metrics on features of property, land use type, property values, ownership, and amenities.

**Public Works Violations (PW)**
- Descriptions, locations, and status of PW violations around Boston.

**Fire Incident Reports**
- Details of each fire incidence by location in Boston, including estimated content and property loss.

**311 Requests**
- Catalogued citizen requests to the city of Boston for services.
- Organized by request location, subject, and source.

### Predictor and Target Variable:

- The predictor variable is a collection of features selected on basis or relevance to public safety, reviewed below.
- The target variable is `zipcode`.

***Contents:***

- <b>Data Cleaning</b>
  - Feature Selection with regards to missing values was an important first step,  and required a lot of analysis in looking for patterns in the missingness of the data. Each of the datasets contained missing values throughout a number of their respective columns. The PW violations data seen here had comparatively few NaN values (represented in this heatmap) of null values throughout its columns,

  - Looking at the property assessment dataset, there were many more missing values throughout many of its columns.

  - In each of the datasets used in this analysis, no missing value rows were present in location-relevant columns, or in violation, incident, or request description - and no systematic imputation was necessary.

  - Given how many features were present in the raw data, narrowing down on relevant features was also time consuming. In the Property Assessment data, many of the columns pertained to real estate attributes- Number of bathrooms, types of interior and exterior finishes- and were necessary to the overall aims of the project and were dropped from the final model.
  - The core features from each raw dataset were: Owner_occupied property, land use, average_bldg value, av_total value, gross tax, estimated loss due to fire, reason and source of 311 request, and value-description-status of PW violation.

- <b>Feature Engineering</b>
  - Feature Engineering took an untold number of hours testing different approaches towards creating features on each dataset before combining into one “master” dataframe.

  - My initial approach was to create latitude / longitude coordinates to join each dataset along, using a custom function that takes in user inputs for columns of “street number”, “street name”, and “st name suffix”, and uses an openstreetmap.org API via Nominatim to generate coordinates. This approach also used geopandas to extract the lat/long from the API and build workable coordinates. HOWEVER, in order to stay compliant with the API’s terms of service, only one set of coordinates could be generated per second- to accommodate I tried splitting the 97515 individual properties in the Property Assessment dataframe into 15 smaller samples for geo-processing, but this still was taking days to process.

  - Ulitmately, to circumvent this unforseen issue, I ended up combining the individual st_num, st_name, and zipcode columns. Once these full addresses were created, the dataframes were “joinable” by a shared column of the full address

  - Some other steps included:
    - OneHotEncoding relevant categorical columns, which drasticly grew the dimensionality of the overall feature set- primarily among the  descriptive and nominal column types, land use type (with 17 unique land use types) and description of PW violation columns (which had 40 unique description types) greatly increased dataset attributes. And to expedite this process, I used a function to read in a dataframe and onehotencode relevant columns.

    - I also extensively used groupby’s to create columns of totals and averages  for different metrics at the Street and Zipcode levels. Although the final/combined dataframe would be organized row by row as individual properties, the code-architecture is in place to evaluate the dataset on the individual street or zipcode levels.

- <b>Exploratory Data Analysis</b>
  - Overall, the number of properties within a given zipcode varies across the 34 unique zipcode areas in Boston (as a reminder, these property types vary in uses types: commercial, residential, etc). Several zipcodes, 02112 (just outside of Fanueil Hall), and 02201 (the Old South Meeting House) are tiny areas within the city. Some initital investigating into this yielded that a small number of zipcodes have little to no residential population and/or are simply historic districts. This is one possible discovery that will need some additional research.

  - As for outliers in the data, although a number of columns contain a fair amount of them, they represent fairly smalla small percentage of the overall data. Important outliers that I specifically looked into were: outsized damage and loss due to fire, and pw_violation_fine_values. These outliers represent locations with outsized amounts of citations issued by the city in response to health, safety, and cleanliness at a given property.

  - Also, of particular interest, was grouping fire_incidents summed at the street level within gives zipcodes. Sorted by number of fire incidents by every street in a zipcode, the higher the number of fire incidents , the higher the total PW violations by street in a given zipcode. It is worth noting also that, as that sum number of total fire incidents goes down, the total number of requests to the city for service becomes nearly zero (Only one outlier zipcode had more than 3 requests by street total, most being nearly zero).

- <b>Modeling</b>
    - So, perhaps both intuitively and unintuitively, a simple logistic regression can predict location (as zipcode) with near perfect accuracy.
      - Its worth noting that the difference between the train score and test score on my last tests were differenced only by 0.0013- indicating of course next to no overfitting from the model.
    - Intuitively speaking, as I think we have certain common sense understandings of how cities work, that are useful in estimating larger-scale areas most likely in need of assistance.
    - Unintuitively speaking, I think that a model able to perform this well on a complicated dataset most likely needs to be re-examined in its implementation, particularly with the label or target class.
    - The least populated classes in Y (of which pertain to those zipcodes I noted earlier as only being very sparesly in the dataset), needed to be dropped to have enough instances across all classes to predict on. I definitely want to go back to this and find a possible better solution than just excluding them outright from the model.
    - As for the model itself, I gridsearch’d over a Logistic Regression model, primarily tuning the hyperparameters of C (alpha). In the final iteration of the model building process, a Logistic Regression with a Lasso Penalty, liblinear solver, and a regularization strength of 0.85.
      - The Lasso penalty zero’d out all but 41 of the original coefficients (Of which there were originally around 100 dimensions). A few things of note from the Lasso feature reduction was that, all else being equal totals, types of property land uses (the total residential, commercial, etc), the number of owner occupied buildings in a location (as zipcode),  types and frequencies of PW violations (like improper trash storage, failures to obtain necessary inspections, and citations for not clearing snow from sidewalks, among others), and total loss due to fire incident were more likely to indicate location (as zipcode).
      - The model scored very well on standard classification metrics. Particularly on both precision and recall we can also see the accuracy in class prediciton in this multi class confusion matrix, which visualizes the correctness in classifying zipcode correctly across each class.

Given all of this, my thought as of now is to do further investigation into these features for future model improvements.


-------------------------------------------------
### <b>Data Dictionary of Included Variables in Model Feature Set</b>:

|Discrete Variables|Continuous Variables|
|---|---|
|`zipcode`, `lu_category`, `owner_occ`, `pw_viol_count`, `description_category`, `fire_incidents`, `requests_total`, `requests_open_status`| `land_value`-`bldg_value`-`total_value`-`gross_tax`, `pw_fine_value`, `fire_prop_loss`-`fire_content_loss`-`sum_fire_loss`|


-------------------------------------------------
### ***Conclusions***

- **Public safety data does seem to be a reliable predictor of location, but ultimately we have to ask, to what end is that important?**
- **From the outset of this project, I had wanted to include some mapping of findings, or generating heatmaps based on results from eda or modeling. Due to the conditions of the API, this wasn’t viable in this first iteration of the project. I think with enough time and patience, any number of informative maps could be created from the data contained here.**

- **Lastly, I think a more nuanced approach to this project would be to identify areas as at-risk on a smaller scale at a street or possibly at the property level. Certain computational constraints prevented me from exploring this in this first iteration, but I think this would make the overall utility of the modeling much greater. To get to this level of specificity, I believe more data would need to be collected as processed.**

- **Ultimately, this model may prove useful for further research.**
