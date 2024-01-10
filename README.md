# Dining-Satisfaction-Analysis

## Description
![2-Page Dashboard](https://github.com/Ibrahim-Izz/Dining-Satisfaction-Analysis/assets/104682497/cc1aaf6b-cb45-4faf-bc9e-883191863bc3)

![Learning - Copy](https://github.com/Ibrahim-Izz/Dining-Satisfaction-Analysis/assets/104682497/496be947-9911-441a-91a0-04dbe5ca709f)

This repository contains descriptive analysis using *Looker Studio* of dining satisfaction based on various factors as depicted in the provided dashboard, as well as predictive analysis using *BigQuery ML* to predict whether a customer will be satisfied or not based on their information and survey ratings. The dashboard is divided into two pages: Survey Ratings and Statistical Analysis.

<br>

Link to Dashboard: https://lookerstudio.google.com/reporting/5d9fd4fa-455c-4d3c-a817-c5b43826fe06

 <br>
  <br>

# 1. Looker Dashboard
### Page 1: Survey Ratings
![Page 1](https://github.com/Ibrahim-Izz/Dining-Satisfaction-Analysis/assets/104682497/5473cf65-41f8-4baf-8d3f-ea79992c23f7)

Contains customer and dining information, as well as visual representation and scores (out of 5) of different survey metrics and their total average for the selected filters and ranges like:

* Baggage Handling
* Check-in Service
* Cleanliness
* Ease of Booking
* Entertainment Options
* Food and Drink Quality
* Online Booking Ease
* Online Menu Access
* Online Support
* Reservation Convenience
* Seating Location
* Space Between Tables
* Table Service
* Wait Time for a Table
* WiFi Service

We see how changing customer information and the range of these scores affect the number of total satisfied and dissatisfied people.
We can limit the range of the score or slide bar to single value and see the result of an exact record with unique combination, an whether they are satisfied or not.
*Note that neutral and dissatisfied both are in the same class and not distinguishable, which might cause some reduction in our results but still OK*

### Page 2: Statistical Analysis
![Page 2](https://github.com/Ibrahim-Izz/Dining-Satisfaction-Analysis/assets/104682497/55e0ca61-2a21-4700-8ba5-b51fa964c654)

Includes some charts breaking down each survey metric and customer information by satisfaction level.

There are 4 charts:

* Satisfaction Percentage by Age Group, Gender, Customer Type, Type of Dining, and Ambiance.
* Food and Dining Experience Ratings.
* Ease of Access and Entertainment Services Rating.
* How Waiting Time Changes with Ambiance, Customer Type, Age Group, and how it Affects Customer Satisfaction.

Each chart has multiple Drill Down options to navigate between, and the ability to sort axes values by each axis label asc. or desc.
You can interact with the dashboard at: https://lookerstudio.google.com/reporting/5d9fd4fa-455c-4d3c-a817-c5b43826fe06

Note that the dashboard in dynamic and cross filtered. It can also take some time to render at first, just warming up, and heavily depends on your machine capabilities.


 <br>
  <br>

# 2. BigQuery ML Logistic Regression Binary Classification Model
First things first: we derived some new columns in the ETL stage earlier in Power Query to help us for both analysis and prediction. These columns are:
* Age Group: Assigning an age group to the customer (21-25, 26-30, etc)
* Satisfaction Level Boolean: Representing "Satisfied" as *True* and "Neutral or Dissatisfied" as *False*
* Satisfaction Level Numeric: Representing "Satisfied" as *1* and "Neutral or Dissatisfied" as *0*
* Avg Rating: Total running average of all 14 survey metrics

Steps to create the model:
1. Upload dataset to BigQuery.
2. Create a new table with a partitioning field of three classes (trainig, evaluation, prediction) to split our dataset for modeling:

```sql
CREATE OR REPLACE TABLE <Table1> AS
(
SELECT 
    Ambiance, Type_of_Dining, Customer_Type, Gender, Wait_Time_for_a_Table, Online_Menu_Access, Cleanliness,
    Baggage_Handling, Space_Between_Tables, Table_Service, Ease_of_Online_Booking, Online_Support,
    Entertainment_Options, WiFi_Service, Seating_Location, Food_and_Drink_Quality, Reservation_Convenience,
    Table_Comfort, target,
    CASE
        WHEN split_field < 0.8 THEN 'training'
        WHEN split_field = 0.8 THEN 'evaluation'
        WHEN split_field > 0.8 THEN 'prediction'
    END as dataframe
FROM (
    SELECT 
        Ambiance, Type_of_Dining, Customer_Type, Gender, Wait_Time_for_a_Table, Online_Menu_Access, Cleanliness,
        Baggage_Handling, Space_Between_Tables, Table_Service, Ease_of_Online_Booking, Online_Support,
        Entertainment_Options, WiFi_Service, Seating_Location, Food_and_Drink_Quality, Reservation_Convenience,
        Table_Comfort, Satisfaction_Level_Numeric as target,
        ROUND(ABS(RAND()),1) as split_field
    FROM <Table>
))
```
3. Create our classification model using logistic regression function:

```sql
CREATE OR REPLACE MODEL <Model>

TRANSFORM(Ambiance, Type_of_Dining, Customer_Type, Gender, Wait_Time_for_a_Table, Online_Menu_Access, Cleanliness,
Baggage_Handling, Space_Between_Tables, Table_Service, Online_Support, Ease_of_Online_Booking, Entertainment_Options,
WiFi_Service, Seating_Location, Food_and_Drink_Quality, Reservation_Convenience, Table_Comfort, target)

OPTIONS(
    model_type='LOGISTIC_REG',
    auto_class_weights=TRUE,
    input_label_cols=['target']
) AS
SELECT 
    * EXCEPT(dataframe)
FROM 
    natural-axiom-410319.NEW.satisfaction3
WHERE 
    dataframe = 'training'
```
4. See prediction results compared to target labels:

```sql
SELECT *
FROM ML.PREDICT(MODEL<Model>,
(
    SELECT 
    *
    FROM <Table1>
    WHERE dataframe = 'prediction'
))
```

5. See parameters and curves of our model:

![Learning](https://github.com/Ibrahim-Izz/Dining-Satisfaction-Analysis/assets/104682497/73ddf31b-8154-4055-b21e-029cc3066259)

![Loss](https://github.com/Ibrahim-Izz/Dining-Satisfaction-Analysis/assets/104682497/c4c786e2-2fe7-420e-97f0-e35b6df22661)

![Prameters](https://github.com/Ibrahim-Izz/Dining-Satisfaction-Analysis/assets/104682497/a634c4bf-fad6-46b2-b076-0bc7a7c2814b)

![Curves](https://github.com/Ibrahim-Izz/Dining-Satisfaction-Analysis/assets/104682497/62cffde3-8d9a-4fb7-a8f6-87157f185d7d)

![Confusion Matrix](https://github.com/Ibrahim-Izz/Dining-Satisfaction-Analysis/assets/104682497/abd50af7-1efa-4000-af4b-0a4502af95d1)

 <br>
  <br>

# 3. Some Thoughts
* Doing some ETL before loading our data into the data warehouse in benefial. Often times we want to derive new columns or change types of some fields that can help us analyse and predict better. In this project, we used Power BI Power Query for the job.
* Clustering ages by age groups is often a best practice to ease analytics.
* Deriving a boolean field out of 2-value string categorical field is often a best practice to ease analytics.
* Some metrics like Table Comfort and Food Quality have the lowest average ratings, therefore must be taken care of.
* For this project, we removed all records having one or more 0 or non applicable rating to ease analytics and deliver more accurate models and assure data integrity. Since they werer only 7.8% of total dataset, the difference it will make is slight. However, if this percentage increased significantly, we must reconsider having them into our OLAP model, amd do necessary manuevers around nulls and 0s to ensure data integrity and accuracy.
* For the our classification model to run on user input data, it must be deployed using some APIs and pipelines. MLOps engineers know it better.
* Changing classification function from logistic regression to other complex and advanced functions like random forests or SVM, and tuning its hyperparameters, might slightly improve our model accuracy.

Example of a possible Deployment Pipeline:
![2024-01-08 08_34_29-Alessandro Marrandino - Machine Learning with BigQuery ML_ Create, execute, and ](https://github.com/Ibrahim-Izz/Dining-Satisfaction-Analysis/assets/104682497/aab41936-c7ed-45e9-acf0-d0873686cbed)

<br>
<br>



# Major Update
We've retrained our model using XGBoost Boosted Trees. Believe it or not, accuracy jumped to actual 1.00 (100%)!

```sql
CREATE OR REPLACE MODEL <Model>
OPTIONS(
MODEL_TYPE='BOOSTED_TREE_CLASSIFIER', BOOSTER_TYPE = 'GBTREE', NUM_PARALLEL_TREE = 1, MAX_ITERATIONS = 50, TREE_METHOD = 'HIST', EARLY_STOP = FALSE, AUTO_CLASS_WEIGHTS=TRUE) AS
SELECT 
    * EXCEPT(dataframe), target as label
FROM 
    <Table1>
WHERE 
    dataframe = 'training'
```

![2024-01-10 06_51_29-BigQuery – My First Project – Google Cloud console](https://github.com/Ibrahim-Izz/Dining-Satisfaction-Analysis/assets/104682497/cf686505-df1b-472d-991b-2a221a8e7581)

<br>

![2024-01-10 06_55_30-BigQuery – My First Project – Google Cloud console](https://github.com/Ibrahim-Izz/Dining-Satisfaction-Analysis/assets/104682497/c2c41cdb-28b1-4f65-9bf7-dcdaa21ee81f)
