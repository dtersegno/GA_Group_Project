Health Outcomes Modeled Over Multiple CalEnviroScreen Reporting Periods
---
We aggregated data from `4` <a href = "https://oehha.ca.gov/calenviroscreen/about-calenviroscreen">CA OEHHA CalEnviro Screen</a> reports, appended warehouse counts extracted from <a href = "link"> US Census Business Data</a> and trained numerous estimators, however none reliably modelled health outcomes using the aggregated data.

---

## Background 
California Office of Environmental Health Hazard Assessment (<a href = "https://oehha.ca.gov/calenviroscreen/about-calenviroscreen">CA OEHHA</a>) has compiled data from various government agencies to create a mapping tool used to identifying communities most affected by various pollution sources, producing four reports total in years 2013, 2014, 2018 and 2021 with scores to for each identified disadvantaged community that was disproportionally facing enviornmental impacts. Because trasportation of goods and people continues to be the primary source of emissions, and because increased commerce requires increased warehousing and associated transportation, we sought to create models to answer the following questoins: 

* Is effect of increased warehouse presence on health outcomes quantifiable ?
* What are primary mitigating factors the State can address in response to increased warehouse density?
* How well do the CalEnviroScreen scores reflect emergency healthcare counts?
* What indicators from the CalEnviroScreen dataset best determine the number of emergency healthcare visits?

---
## Data Acquisition & Cleaning 
---
### <b>Data:</b> CalEnviroScreen 
**Description:** CA OEHHA compiles data specfically regarding pollutants and the communities affected by them. 


 Source  | File  | Report Date  | Shape | 
 ---     | ---   |  ---         |  ---  |
 <a href = "https://oehha.ca.gov/calenviroscreen/report-general-info/calenviroscreen-10"> CalEnviroScreen 1 </a> |  <a href = "https://github.com/gigi-codes/CO2_modelling/blob/g_branch/raw_data/calenviroscreendatav11.xls"> Data (xlsx) </a> | April 2013 | (1500,49) |
 <a href = "https://data.ca.gov/dataset/calenviroscreen-2-0"> CalEnviroScreen 2 </a> |  <a href = "https://oehha.ca.gov/media/downloads/calenviroscreen/report/ces20updateoct2014.xlsx"> Data (xlsx) </a> | Oct 2014 | (8035, 51) |
 <a href = "https://oehha.ca.gov/calenviroscreen/report/calenviroscreen-30"> CalEnviroScreen 3 </a>  | <a href =  "https://oehha.ca.gov/media/downloads/calenviroscreen/document/ces3results.xlsx"> Data (xlsx) </a>     | June 2018 |  (8035, 51) |
 <a href = "https://calenviroscreen-oehha.hub.arcgis.com"> CalEnviroScreen 4 </a> | <a href = "https://calenviroscreen-oehha.hub.arcgis.com"> Data, Dictionarty ZIP </a>| Oct 2021 |  (8035, 51) |
 |   | |  Combined Data   | (25444, 62) |
 |   | |   **Used Data**   | (14912, 59) |
 
<br>

### <b>Data:</b> <a href = "https://www2.census.gov/programs-surveys/cbp/datasets/"> US Census Business Survey: Warehouse Counts, Density </a>
**Description:**  The US Census data is business counts by county, zip code and business type. 

<!-- 
year(s) | name/link     | description                  | size 
---     | ---           | ---                          | ---
2012    |   xxx         |  business by type in county  | row x col 
2012    |   xxx         |  business counts by zip      | row x col  -->


name        | description 
---         | --- 
est total   | total number of warehouses in class 
est ag      | total number of warehouse for agricultural 
est cold    | total number of warehouses for cold storage
est gen     | total number of warehouses for general storage

### Data: _Model Features_

<br >

variable name   | type      | description 
---             | ---       | ---   
diesel pm       | numeric   | particulate matter, spatially modelled
ozone           | numeric   | concentration
traffic         | numeric   | volume: vehicles per length of time over fixed distance

---

## _Target_

We trained multiple estimators to model the outcomes reported in the _CalEnviroScreen_ reports to build upon that work: 

variable name   | type      | description 
---             | ---       | ---   
asthma hospitalization         | numeric   | incidence rate, cases/ 10k population
heat attack hospitalizatoin   | numeric   | incidence rate, cases/ 10k population
low birth weight| numeric   | % newborns weighing `< 2.5 kg` (#/100 live births)

---

## _Exploratory Data Analyses_ 

* outliers in features were not dropped: as the data from the report is meticulously collected and averaged, eliminating observations outside of some central-tendency would eliminate communities that are most affected by selected factors. 

<img src = Giovanna/images/warehousehealth.png><br>
_Health outcomes on vertical axis suggest locations with high-warehouse density have higher rates of negative health outcomes._

<img src = Giovanna/images/healthpm.png><br>
_Health outcomes on vertical axis suggest locations with high $PM_{2.5}$ have higher rates of negative health outcomes._

<img src = Giovanna/images/traffichealth.png><br>
_Health outcomes on vertical axis suggest locations with traffic concentrations have higher rates of negative health outcomes._

---
## _Modeling_

> ### David: Linear Models
The linear models give us a sense of which of the features to focus on if we want to address these health issues.

Coefficients for these models were scaled and used to mark the most important features for addressing the health data. Ultimately, we found that the socioeconomic factors of unemployment, housing burden, linguistic isolation, and poverty were as an ensemble more effective at predicting the health data than the ensemble of pollution and warehouse data.

In fact, we found that warehouse counts have nearly no predictive value.


Model   | Features                              | Target | $R^2 train$  |
---     |---                                    | ---    | ---          | 
1       | selected: no time, space or CES scores| Asthma | 0.29      |
2       | selected: no time, space or CES scores| LBW    | 0.14     |
3       | selected: no time, space or CES scores| CVD    | 0.23      |
4       | warehouse counts only                 | Asthma | 0.017      |
5       | warehouse counts only                 | LBW   | 0.002     |
6       | warehouse counts only                 | CVD   | 0.013     |
7       | socio-economic only                   | Asthma | 0.263     |
8       | socio-economic only                   | LBW    | 0.1337      |
9       | socio-economic only                   | CVD    | 0.11      |

The models that yielded these metrics are developed in [Notebook 7: Linear Regression](./David/7_linear_regression.ipynb). The procedure for processing the raw data leading up to this notebook begins with [Notebook 1: EDA on CAES 4 Data](./David/1_Deda_commit_da4c1ce.ipynb).

<br> 

> ### Giovanna: Random Forest & SVR 
<br> 

`Random Forest Regression` over a Gridsearch to find optimal values returned the following importances and low-performance scores, indicating that the metrics we used to model health outcomes are insufficient, and highlighting that socioeconomic factors are much more predictive. Specifically, the number of warehouses in a given zip code does not reflect the outcomes in that zip code. 

> Model   | Estimator                | Features   | Target    |  $R^2$
> ---     |---                       | ---        | ---       | --- 
> 1       | SVR                      | all        | asthma    | 0.29

## Random Forest Regression Modell Tuning over GridSearch

> ### Using all Features: 
> Model  | Estimator  | Target    | $R^2$             | Max Error   | Scale 
>  ---   | ---        | ---       | ---               | ---         | -- 
> 1      | RF         | Asthma    | -0.35             | 10.7        | #/10,000
> 2      | RF         | Low Birth Weight  |  -0.25    | 7           | #/100
> 2      | RF         | Cardiovascular Disease    | 0.01 | 30       | #/10,000

<br>

> ### Dropping SocioEconomic Features: 
> Model  | Estimator  | Target    | $R^2$             | Max Error   | Scale 
>  ---   | ---        | ---       | ---               | ---         | -- 
> 1      | RF         | Asthma    | -0.01             | 180 	    | #/10,000         
> 2      | RF         | Low Birth Weight                |  -0.25    | 8     | #/100          
> 2      | RF         | Cardiovascular Disease    | -0.05  | 30    | #/10,000     
> 

 ### None of these models predictably return health outcomes. 

<br> 

<img src = Giovanna/images/asthmaa.png><br>
_Features importances with all features, model target is `asthma`._

<img src = Giovanna/images/caridoa.png><br>
_Features importances with all features, model target is `heart-attacke` hospitalization._ 

<img src = Giovanna/images/lbwa.png><br>
_Features importances with all features, model target is `low birth-weight`._

<img src = Giovanna/images/asthmanse.png><br>
_Features importances without socio-economic features, model target is `asthma`._

<img src = Giovanna/images/heartnse.png><br>
_Features importances without socio-economic features, model target is `heart-attacke` hospitalization._ 

<img src = Giovanna/images/lbwnse.png><br>
_Features importances without socio-economic features, model l target is low birth-weight`._

<br > 

> ### Marshall: XGBOOST

Features used: 
| total population, ozone, pm2.5, diesel pm, pesticides, traffic, cleanup sites, groundwater threats, haz. waste, imp. water bodies, solid waste, pollution burden, low birth weight, education, linguistic isolation, poverty, pop. char. , drinking water, tox. release, unemployment, ces_per, cardiovascular disease, housing burden, est total, est gen, est cold, est farm, est other | gradient boosting supervised regression |
| Model         | features used  type   | evaluation metric  | Train Accuracy  | Test Accuracy | RMSE score | MAE test score |
| ---        | ---  | --- | --- | ---| ---| ---
| XGBoost       | R2, RMSE, & MAE | 0.9139 | 0.7853 | 13.6915  | 9.3296       |
| Random Forest  meta estimator regression               | R2, RMSE, & MAE | 0.9634 | 0.7503 | 14.7307  | 9.9952       |


## _Conclusion_
We saw saw no meaningful relationship to create robust models for health outcomes using our warehouse-aggregated data for any models we tried to fit. _Cal EnviroScreen_ scores highly reflect `asthma` and `pollution burden` but not `hospitalization rates`. Socioeconomic factors aggregated in _Cal EnviroScreen_ built best predictive models for negative health outcomes, highlighting the need for the State to address root causes for pollution burden. 

---
## _Next Steps_
We will continue to aggregate more data with finer granularity, and explore the raw data from which the CalEnviroScreen was sourced and modeled. Furthemore, spatial and temporal analyses will provide more robust models for projecting and adddressing communites to support. 
<br> 

---
> ### Background: From the Press 
* <a href ="https://www.epi.org/publication/unfulfilled-promises-amazon-warehouses-do-not-generate-broad-based-employment-growth/"> EPI: Warehouses Do Not Generate Broad-Based Employment </a> (cites the source I have for fulfillment center locations)
* <a href = https://www.cbre.com/insights/local-response/2022-north-america-industrial-big-box-los-angeles-county> CBRE: 2022 North America Industrial Big Box Review & Outlook: Los Angeles County</a>
> 
