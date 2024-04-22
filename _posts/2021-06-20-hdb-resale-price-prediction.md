---
layout: post
title: Predicting HDB Resale Price Data Using Regression
subtitle: Analysis of HDB Resale Prices
cover-img: /assets/img/HDB-Resale/HDB-flats-in-Singapore.jpeg
thumbnail-img: /assets/img/HDB-Resale/hdb_thumbnail_image.png
share-img: /assets/img/HDB-Resale/HDB-flats-in-Singapore.jpeg
tags: [Resale, HDB, Data Analytics, Machine Learning, Housing]
readtime: True
---

# HDB Resale Price Analysis
A large portion of flats built in Singapore is by the Housing Development Board (HDB) of up to 80% with a lease of 99 years. It's goal was to provide affordable housing to Singaporeans as a way to increase home ownership among Singaporeans.

As an aspiring home-owner that is considering a resale flat. Things brings about the question, what is the fair value of a HDB flat?

***Objectives***
- What are the key factors affecting HDB Resale prices?
- Is there a model that can accurately predict the resale price of a HDB flat?

# Data / Feature Engineering
The dataset was sourced from [Resale Flat Prices](https://data.gov.sg/dataset/resale-flat-prices). This provides the data beginning from 1990.

Notable Features of the dataset
- town: Associated location of the flat
- flat_type: What type of flat, ranging from (1-room HDB to Executive Maisonettes)
- floor_area_sqm: Floor area in square meters
- lease_commencement_date: When was the flat built
- remaining_lease: Remaining lease of when the flat was sold
- resale_price: Price of the flat sold in Singapore Dollars(SGD)

Apart from these factors provided, some other factors that we see on property websites list the time taken to amenities such as schools, shopping malls and public transport (rail network). Unfortunately these information are not provided along with the HDB dataset and have to be obtained elsewhere.

## Distance Computation
Considering that Singapore is a small country, for the sake of simplicity, the distance between amenities will be computed using euclidean distance in a 2D plane (Pythagoras theorem) using the latitude and longitude of each place. In order to get the latitude and longitude of each location, I decided to give different services a try.

### Latitude and Longitude
The latitude and longitude of the HDB flats were obtained using HERE Geocoding API using the blocks and street addresses provided.

### List of Schools
The list of schools were obtained from [School Directory](https://data.gov.sg/dataset/school-directory-and-information) and latitudes and longitudes computed using HERE Geocoding API

![Schools DataFrame](/assets/img/HDB-Resale/school_dataframe.png)

### List of Shopping Malls
The list of shopping malls were obtained from Wikipedia [Singapore Shopping Malls](https://en.wikipedia.org/wiki/List_of_shopping_malls_in_Singapore). Noting that we only have the names of the shopping malls and not the addresses provided, I decided to give [Onemap](https://www.onemap.gov.sg/home/) a try.

![Malls DataFrame](/assets/img/HDB-Resale/mall_dataframe.png)

### List of Train Stations
List of train stations were obtained from [landtransportguru](https://landtransportguru.net/). Selenium with BeautifulSoup was used to extract the opened date of all currently opened train stations till 2021.

As Singapore builds up its rail infrastructure, not all stations are open at the point of transaction. Distance is calculated to the nearest train station at the point of transaction.

![Train DataFrame](/assets/img/HDB-Resale/train_dataframe.png)

# Exploratory Data Analysis
A correlation matrix is used to understand each predictors correlation with each other

![EDA_heatmap](/assets/img/HDB-Resale/heatmap.png)

Looking at the heatmap its obvious that some factors show a correlation with the resale price. They are:
- floor_area_sqm
- year(yr)
- train_station_distance

Plot of Resale Price vs Floor Area(square meters)
![Resale Price vs Floor Area](/assets/img/HDB-Resale/scatterplot_resale_vs_floor_area.png)

Taking a look at the trend of the resale price vs floor area (square meters). In conjunction with the heatmap its no surprise that the larger the floor area the higher the resale price (correlation of 0.63).

## Trend of Resale Flat prices
For this analysis, 4-room flats will be used as they consist of the largest percentage of HDB flat type in Singapore

![4-Room HDB Flat Prices](/assets/img/HDB-Resale/4_room_resale_vs_years.png)

## Trend of Resale in Towns
Knowing this, the next step is to find out if there are any trends that differ between each town.

![4-room HDB Flat Price Group By Town](/assets/img/HDB-Resale/4_room_groupby_town.png)

## Does Height Matter?
Does staying at a higher floor give a higher flat price? Depending on where you stay, having a higher floor can get you a view of the city skyline or a front row seat whenever fireworks are on display.

![Price vs Storey Height](/assets/img/HDB-Resale/resale_price_vs_storey.png)

## What Kind of Flat?

A look into the price vs flat_type is performed. Taking a look its clear that the larger the flat, the higher the price. This is no surprise as property is sold on a per area basis. Its no surprise that a larger flat would command a higher price.

![Resale Price vs Flat Type](/assets/img/HDB-Resale/boxplot_price_vs_flattype.png)

# Evaluation of Results
The criteria used to evaluate the models are:
- R-Squared
- Mean Absolute Error(MAE)
- Root Mean Squared Error(RMSE)

The following machine learning models were used to generate the results:
- Random Forest Regressor
- Decision Tree Regressor
- Linear Regressor
- XGBoost Regressor
- Gradient Boosting Regressor

![Evaluation Results](/assets/img/HDB-Resale/evaluation_results_new.png)

# Conclusion

From the results its clear that Random Forest Regression gives the best results among all three metrics.

Even though this model incoporates features such as distance to the nearest train station, schools and malls, it is possible that the model can be improved by using a more accurate distance calculation or the walking time computed. There are other unaccounted features such as time taken to the CBD (Raffles Place) or shopping district (Orchard) which can be implemented in an attempt to achieve a better model.

It is also noted that in this model, a reduction in dimensions using PCA can be performed to have a working model that can be deployed on a web application where potential buyers/sellers can obtain an estimate of an expected fair value of a flat.
