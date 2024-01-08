# Dining-Satisfaction-Analysis

## Description
![Uploading 2-Page Dashboard.jpg…]()
This repository contains descriptive analysis using *Looker Studio* of dining satisfaction based on various factors as depicted in the provided dashboard, as well as predictive analysis using *BigQuery ML* to predict whether a customer will be satisfied or not based on their information and survey ratings. The dashboard is divided into two pages: Survey Ratings and Statistical Analysis.

### Page 1: Survey Ratings
/path/to/1.jpg

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

We can limit the range of the score or slide bar to single value and see the result of exact record with unique combination, an whether they are satisfied or not.

### Page 2: Statistical Analysis
/path/to/2.jpg

Includes some charts breaking down each survey metric and customer information by satisfaction level.

There are 4 charts:

* Satisfaction Percentage by Age Group, Gender, Customer Type, Type of Dining, and Ambiance.
* Food and Dining Experience Ratings.
* Ease of Access and Entertainment Services Rating.
* How Waiting Time Changes with Ambiance, Customer Type, Age Group, and how it Affects Customer Satisfaction.

Each chart has multiple Drill Down options to navigate between, and the ability to sort axes values by each axis label asc. or desc.

