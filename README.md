# ![](https://ga-dash.s3.amazonaws.com/production/assets/logo-9f88ae6c9c3871690e33280fcf557f33.png) Kaggle - West Nile Virus Prediction

## Introduction

We are a team of data scientist at the Disease And Treatment Agency, a division of Societal Cures In Epidemiology and New Creative Engineering. Due to the recent epidemic of West Nile Virus in the Windy City, we've had the Department of Public Health set up a surveillance and control system. We're hoping it will let us learn something from the mosquito population as we collect data over time.

West Nile virus (WNV) is the leading mosquito-borne disease in the United States ([CDC, 2009](https://www.cdc.gov/westnile/index.html)). It is a single-stranded RNA virus from the family Flaviviridae, which also contains the Zika, dengue, and yellow fever virus. It is primarily transmitted by mosquitoes (mainly *Culex* spp.), which become infected when they feed on birds, WNV's primary hosts. Infected mosquitos then spread WNV to people and another animals by biting them. Cases of WNV occur during mosquito season, which starts in the summer and continues through fall ([CDC, 2009](https://www.cdc.gov/westnile/index.html)).

While WNV is not extremely virulent and only about 1 in 5 people who are infected develop West Nile fever and other symptoms, about 1 out of every 150 infected develop a serious and sometimes fatal illness ([CDC, 2009](https://www.cdc.gov/westnile/index.html)). There is currently no vaccine against WNV.

In view of the recent outbreak of WNV in Chicago, the Chicago Department of Public Health (CDPH) has set up a surveillance and control system to trap mosquitos and test for the presence of WNV. The goal of this project is to use these surveillance data to predict the occurrence of WNV given time, location, and mosquito species. Findings from this project will guide and inform decisions on where and when to deploy pesticides throughout the city, to maximise pesticide effectiveness and minimise spending. 

## Dataset

The dataset, along with description, can be found here: [https://www.kaggle.com/c/predict-west-nile-virus/](https://www.kaggle.com/c/predict-west-nile-virus/).

## 1. Problem

In this competition, you will be analyzing weather data and GIS data and predicting whether or not West Nile virus is present, for a given time, location, and species.

Every year from late-May to early-October, public health workers in Chicago setup mosquito traps scattered across the city. Every week from Monday through Wednesday, these traps collect mosquitos, and the mosquitos are tested for the presence of West Nile virus before the end of the week. The test results include the number of mosquitos, the mosquitos species, and whether or not West Nile virus is present in the cohort.

We will generate a prediction model on the presence of the Virus for a given location, date and species, taking into account key factors (features) and weather [temperature, location] or spray patterns. More importantly, the effectiveness of spray on curbing the number of mosquitos. 

## 2. Data Description

## train.csv, test.csv

The training set consists of data from 2007, 2009, 2011, and 2013, while in the test set you are requested to predict the test results for 2008, 2010, 2012, and 2014.

- Id: the id of the record
- Date: date that the WNV test is performed
- Address: approximate address of the location of trap. This is used to send to the GeoCoder.
- Species: the species of mosquitos
- Block: block number of address
- Street: street name
- Trap: Id of the trap
- AddressNumberAndStreet: approximate address returned from GeoCoder
- Latitude, Longitude: Latitude and Longitude returned from GeoCoder
- AddressAccuracy: accuracy returned from GeoCoder
- NumMosquitos: number of mosquitoes caught in this trap
- WnvPresent: whether West Nile Virus was present in these mosquitos. 1 means WNV is present, and 0 means not present.

## spray.csv

GIS data of spraying efforts in 2011 and 2013

- Date, Time: the date and time of the spray
- Latitude, Longitude: the Latitude and Longitude of the spray

## weather.csv

weather data from 2007 to 2014. Column descriptions in noaa_weather_qclcd_documentation.pdf.

![img](https://lh6.googleusercontent.com/KX-BIHkHIyMSjgEQiB_YWV3ruxRiKskG1tJyXQ1mOKgmic8JgJmAY_P3Lk0JdJHbdcRpCw_i9eYiFAn70CoHW1JfukMVwH-CicOtk_JxZm8uaFDu6eTD7UNNyCxUmA-8zhK9-z-djVg)

## 3. Pipeline

**EDA** 

1. Drop any Null Values (if necessary)
2. Drop any duplicates (if necessary)

Train

- Dropped Address, Block, Street as it would be better represented with geographical coordinates (Lon/Lat)

Weather

- Replace Trace Values represented by T with 0
- Replace Missing Values -> Impute from Station 1, Forward Fill, Tavg 

Spray

- Eliminate the column 'Time' as it has the most null values (584) and it would be irrelavant to our analysis
- Merge Date of 2013-08-16 with 2013-08-15
- Drop Lon/Lat above 42.1 
- Drop 5 entries in the 10:49 timestamp

**FEATURE ENGINEER** 

Train 

- Create 'Year' 'Time' 'Date' columns
- Link Train via shortest spatial distance to either Station 1 or 2
- Using Kmeans to create spray cluster
- Initialise midpoint for each cluster (midpoint Lat and midpoint Lon)
- Map trap record to spray cluster by shortest distance of midpoints to clusters using a specific threshold [haversine function]
- One-hot encode Species
- Design a feature for 'Days between Spray and Test'
- Impute a binary variable column 'Has been Sprayed' if the respective observation has been sprayed

![img](https://lh5.googleusercontent.com/JthIW3oXutYveR_3Bau_xBnbv_HT-FGAWO48iNNjCfo9AHgxb_d8LT5CKKPppejTZGNzJphM6hBCOUOrd8KrIgg7Y9jd8v4zsdvJEn1FIVnmmsCsRY9qqZP37eim-i1jXKHKMfieGIo)

![img](https://lh4.googleusercontent.com/gDd5VCUe8xf6wMulUVNTmenrcHj-w1NDsBgu8nH9nbtrAhNmUqVn0VraDb1gPhMYrXPXhyMbMrSNSSHFypzKRcWWrjtLOUBi9bto6Ftou8Iafxq1rjuZovZ5Z4vKCeurRmvSD3zrQl0)

Weather 

- 7-day rolling average for certain elements 
- Generate a 'Daylight Hours' feature by computing the difference in the time of Sunrise and Sunset 

**MODELING** 

List of Models

1. Decision Tree
2. RandomForest
3. Logistic Regression
4. K-Nearest-Neighbour
5. Support Vector 
6. AdaBoost
7. BaggingClass
8. GradientBoosting
9. XGBoost

| Model               | Score |
| ------------------- | ----- |
| Decision Tree       | 0.5   |
| RandomForest        | 0.512 |
| Logistic Regression | 0.5   |
| K-Nearest-Neighbour | 0.5   |
| Support Vector      | 0.5   |
| AdaBoost            | 0.5   |
| BaggingClass        |       |
| GradientBoosting    | 0.518 |
| XGBoost             | 0.837 |

Evaluate on the ROC-AUC score - XGBoost

**COST AND BENEFIT** 

## Spray characteristics

__Spray type__

Zenivex™ has been used effectively to control disease-carrying __adult__ mosquitoes and is non-persistent, decomposing rapidly in the environment.

__Spray Contractor__: Vector Disease Control 

__Spray application__

- In hotspots where the West Nile Virus is more serious/prevalent
- 1.5 fluid ounces per acre
- Apply to wind speeds >= 1mph <= 10 mph
- Use a vehicle-mounted cold aerosol ULV sprayer to apply the product

![img](https://lh3.googleusercontent.com/-g-bZUPxVQ02FBmg5MRtTBkiIpExZbSEmeE_mbndoxyn-QhIS8G_OaC-RT1oXSgUXLGzjbEZuBYvpIOP6HON1H0uZQQ7xyHT3kdxrOasJNixuxyhJtJ3rLvfQj9nnvxeAkOSGbsV19g)

Benefits

- Lowers the number of mosquitos which are carriers of not only West Nile virus but also other vector bourne diseases such as Zika 
- Lower the high costs of possible medical and hospitalisation fees
- Reduce costs of the state to handle such outbreaks and emergencies

Costs

## Costs

We know what the benefits are and obviously lowering the occurence of virus will be beneficial for the population of Chicago. 
Now let's look at how cost effective it is spraying really is. 

Yearly estimated costs can be segregated into two categories:
- Medical fees 
- Vector control measures - Spraying insecticide

__Incorporating the cost of inaccuracy__ 

Since this is a classification problem, we would ideally want to work towards minimizing the number of false positive and false negative classifications. Especially when such misclassification would result in an increased probability of epidemic or unnecessarily spending money on spraying locations where there is low likelihood of WNV being detected. 

In our classification:

False Positive refers to the scenario where we predicted that the area has the WNV virus but it does not actually have the virus. This could result in an increase in spraying when it is not required, as well as creating fear and loss in productivity if anyone is quarantined. 

False Negative on the other hand refers to the scenario where we predicted that an area is free of WNV but in actual fact it's plagued with the WNV. This may potentially lead to an increased chance of human epidemic and eventually resulting in a huge negative economic impact downstream.

__Medical fees__ 

It was researched that medical and hospitalisation fees of WNV related cases amounted to an estimated \\$673 million to \\$1\.01 billion in 1999 to 2012 with a 95% confidence level. This amounts to about \\$168 million to \\$250 million per year.

https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3945683/

When should we spray?

![img](https://lh4.googleusercontent.com/VobBIyOXLAjrjOdwFt28mJo19DJZrBiV_tdrO9rgbb0rH2BoAlGc2sGWFjskhsVvZEujHNatkvcNSU73VMzqSE75h5j3wwfD8RbjuLwCUQkR-gNd4RBvYL1Gj9BH0hL2M-aRhmz1Ok0)

By plotting the heatmap on prevalence of WNV in 2013 by month, we can see that prevalence of the WNV indeed peaks between July and September. This suggests that if we start spraying in early-July (before the expected spike in mid July), we may be able to reduce the prevalence of WNV in subsequent months.

Next, we will look at where we can target the spraying to increase the effectiveness of sprays.

Where should we spray?

![img](https://lh5.googleusercontent.com/f0JE80YkP51BEFPVq4XfyfEuO9a7Q4BTyswnuw4lGr7FFcvRF6AqR_koCzCtPkR_BRYEmnc7qztoz2Wu3sNBIkb0YshFNjd3fESOLu9JzWqLXnMZZ4s8eUbGR_3Muc7G3nKHNGsTcnE)

This heatmap shows the predicted area where there will be WNV based on our model. We can hence target the sprays to these area to increase the effectiveness of the spray, and thereby reducing the prevalence of WNV.

**CONCLUSION AND RECOMMENDATION****

| Type     | Cost                             |
| -------- | -------------------------------- |
| Medical  | \\$168 million to \\$250 million |
| Spraying | \\$701,790                       |


Assuming there will be an outbreak, the medical fees are estimated to be \\$168 million to \\$250 million and conversely if the state will have to spray, assuming they spray 477 square kilometres, it will be only just \\$701,790.

This seems to be an obvious choice to choose one of the vector control methods, spraying of insecticide to save future costs impacted by the existence of the West Nile Virus. 

However, combating the West Nile Virus and curb mosquito populations ought to be recognized as everyone's responsibility so that we can eradicate the West Nile Virus and other vector bourne diseases together. We can educate the public to remove stagnant water and other favourable conditions of mosquito breeding starting from their own homes and workplaces. This should be the best way forward to prevent the laying eggs of mosquitos. 

When a substantial density of WNV was detected in mosquito pools for 2 consecutive weeks, CDPH sprayed implicated geographic areas to control adult mosquitos

Since the savings from spraying is greater then the cost of spraying, we would recommend to always spray in order to control the spread of WNV. We could also consider spraying slightly earlier, near early July before the expected peak of WNV to further reduce the spread of west nile virus.

Furthermore, we could consider larviciding in late June to control the presence of mosquitose when they are still in a larva or pupa form. This could further prevent the spread of WNV in subsequent months.

One limitation to our model is that there are overlapping clusters so if a point falls in between or part of clusters that overlaps, that point will be assigned the closest cluster between the two and it is not very definitive if that particular point has been sprayed. 

Increasing the range of parameters in the gridsearch with the objective to tune the model with a higher ROC-AUC score should also be explored.

Since there is a missing component in the data, we should include the bird migratory data for affected birds that carry the West Nile Virus. 

Even though the number of mosquitos are not directly related to the existence of the virus, we should still aim to reduce the number of mosquitos as they are afterall a carrier of the virus. So we could model to predict the number of mosquitos so that we can further aim at cluster areas to reduce the population of mosquitos. 