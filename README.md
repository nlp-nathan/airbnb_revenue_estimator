Airbnb Yearly Revenue Estimator
==============================


[![Build](https://github.com/nathann3/airbnb_revenue_estimator/actions/workflows/python-app.yml/badge.svg)](https://github.com/nathann3/airbnb_revenue_estimator/actions/workflows/python-app.yml)

web app : https://airbnb-revenue.herokuapp.com

![web_app_demo](reports/figures/web_app_demo.gif "web_app_demo")

# Synopsis
Airbnb has grown exponentially since its inception in 2008 and has proved
itself to be a promising income stream for many hosts. With limited funds as
well as competition from other hosts, prospective hosts must make data-driven
decisions to maximize their listing's earning potential. In this project, we
tackle this very problem by exploring thousands of listings in Seattle as well
as attempt to create a regression model to estimate a new listing's potential
yearly revenue.

# Outcome

The most expensive Airbnb's are located downtown. The most common Airbnb
accommodates 2 and has 1 bed and bath. The median estimated yearly revenue is
$26,171.

With a voting regressor consisting of stacked regressors of XGBoost, KNN, and
linear models, we were able to achieve an MAE of $11,512.01, an 8.4% increase
in performance from a multiple linear regression model. 

![model_comparison](reports/figures/model_comparison.png "model_comparison")

A model is only as good as the data it is trained on.\
We discuss ways to improve performance in Model Performance below.

Project Organization
------------

    ├── LICENSE
    ├── Makefile           <- Makefile with commands like `make data` or `make train`
    ├── README.md          <- The top-level README for developers using this project.
    ├── app                <- Flask app with HTML/CSS
    ├── data
    │   └── processed      <- The final, canonical data sets for modeling.
    │
    ├── docs               <- A default Sphinx project; see sphinx-doc.org for details
    │
    ├── models             <- Trained and serialized models, model predictions, or model summaries
    │
    ├── notebooks          <- Jupyter notebooks ordered by number. Contains EDA,
    │                          feature engineering, feature selection, and modeling
    │
    ├── reports            <- Generated analysis as HTML, PDF, LaTeX, etc.
    │   └── figures        <- Generated graphics and figures to be used in reporting
    │
    ├── requirements.txt   <- The requirements file for reproducing the analysis environment, e.g.
    │                         generated with `pip freeze > requirements.txt`
    │
    ├── setup.py           <- makes project pip installable (pip install -e .) so src can be imported
    ├── src                <- Source code for use in this project.
    │   ├── __init__.py    <- Makes src a Python module
    │   │
    │   ├── data           <- Scripts to download or generate data
    │   │   └── make_dataset.py
    │   │
    │   ├── features       <- Scripts to turn raw data into features for modeling
    │   │   └── build_features.py
    │   │
    │   ├── models         <- Scripts to train models and then use trained models to make
    │   │   │                 predictions
    │   │   ├── predict_model.py
    │   │   └── train_model.py
    │   │
    │   └── visualization  <- Scripts to create exploratory and results oriented visualizations
    │       └── visualize.py
    │
    └── tox.ini            <- tox file with settings for running tox; see tox.readthedocs.io


--------

<p><small>Project based on the <a target="_blank" href="https://drivendata.github.io/cookiecutter-data-science/">cookiecutter data science project template</a>. #cookiecutterdatascience</small></p>

# Purpose / The Airbnb Business

Airbnb has become a thriving business for many entrepreneurs and investors, but
with its exponential growth, the Airbnb market has become competitive. Like the
housing and rental market, prospective Airbnb hosts must make significant
decisions on their listing, whether they already have a place or looking to
buy/rent a place, to maximize their potential earnings. A poor listing will not
yield a good return and thus the investment money will have been wasted.

To maximize their potential earnings, a host's decisions must be data-driven.
Airbnb Yearly Revenue Estimator was created to help hosts find potential
candidate homes/apartments for Airbnb. Using data from thousands of listings,
the web app estimates the yearly revenue a listing could generate, given the
main attributes of a listing (property type, location, beds, etc...). The web
app also features an interactive map with other similar listings with their
attributes and estimated yearly revenue plotted.

The goal is to allow potential hosts to be able to make informed data-driven
decisions before committing to buy/rent a place for Airbnb. This project is
focused on the Seattle area but can be modified to be used in any other major
city.

# Airbnb Data

Airbnb listings were retrieved from Inside Airbnb, an independent organization that provides tools and data to explore
Airbnb's in cities around the world. In our project, we focus only on Seattle, WA. Scraped every month from October 2020
to July 2021, the data provides information about listings as well as their calendar availability throughout the year.

In total, 7166 listings were active within our dataset, but only a fraction of these listings were active every month.
This can be attributed to COVID because people would have to retire their listings after the loss in demand for the
travel industry. New hosts who wanted to test out the Airbnb platform also attribute to the varying number of
continuously active Airbnbs.

![seattle_airbnbs_over_time](reports/figures/seattle_airbnbs_over_time.gif "seattle_airbnbs_over_time")
<p><small>Airbnb listings in Seattle from 2010 to 2021</small></p>

Most features in the data set are not required, because our mission is to create a tool for prospective Airbnb hosts to
estimate a listing's yearly revenue. To do this, we removed features that inherently contain no information as well as
features that are not readily available to the prospective host. The features leftover consist of accommodates,
bedrooms, beds, bathrooms, latitude, longitude, and property type.

<div align='center'>
    <img src='reports/figures/correlation.png', width="420">
    <p><small>Correlation of features in our processed data</small></p>
</div>

We cleaned up property type by moving levels that only had a few listings to bigger and broader levels.

There were a couple of missing data points, but most of them were resolved by finding the listing in Airbnb and filling
in the missing values manually.

### Interesting Findings

* Listings are most expensive in June on 
  Friday and Saturday, while they are least expensive in January on Mondays and 
  Tuesdays. The most expensive listings are downtown.
* The most common listing is a 1-bedroom apartment.
* Reviews for listings are overwhelming positive. Something is off if there is a negative review
* The most common minimum number of nights to book is surprisingly 30, with 2 and then 1 following.
  This might signal a rise in short term rentals and sublets, which is inline with what Airbnb has been trying to push.


# yearly_revenue Feature

Airbnb does not release data on the yearly revenue their listings earn. This provides a challenge against estimating
yearly revenue. To work around this, we must create our own response variable from the data available.
The yearly_revenue feature was created by joining the availability calendars of the listings for every month. With every
month joined, we update the availability calendar to estimate the days booked throughout the year. Along with
availability, the calendar data set also provides the price for the day. Adding up all the days and their prices allows
us to estimate a listing's yearly revenue. 

This also provides a problem of really low revenue estimates. Since not all
Airbnb listings are full-time, we remove listings that are not active every
month.

# Model Building

Before anything, we split the data 80-20 into training and test sets.
We will then model and tune all our models on the training set, including
cross-validation on only the training set. In the end, we will evaluate the
best models on the never-before-seen test set.

For preliminary modeling we used :

* linear regression
* ridge regression
* lasso regression
* elastic net
* k nearest neighbors (KNN)
* random forests
* XGBoost
* stack ensemble (consisting of all previous models)

Out of these models, the stacked ensemble performed the best with a mean
cross-validation MAE of 11329.90, followed by random forests with an MAE of
11454.30.

After preliminary modeling, we tuned all the previous models using a grid
search. Out of these models, the tuned stack ensemble performed the best with a
mean cross-validation of 11133.31. Random forests performed second best
followed by KNN with MAEs of 11288.79 and 11312.56 respectively.

We then put those three best models into a voting regressor, which achieved a
mean cross-validation MAE of 11074.38, the best performing model so far.

With these three best models, we then fit them on the entire training set and
evaluated their performance on the never before test set.

# Model Performance

For our evaluations, we will focus on the mean absolute error (MAE) because it
is not highly sensitive to outliers like root mean squared error (RMSE) is. 
Like housing prices, yearly revenue for listings can have many outliers, where
for our model we want to create a tool that allows prospective hosts to
estimate what their yearly revenue will be. Outliers aren’t particularly bad in
for this type of model This makes MAE more appropriate than RMSE. MAE is also
much more interpretable and thus we will focus on MAE, but also provide other
metrics as well in the final evaluation.

![model_comparison](reports/figures/model_comparison.png "model_comparison")

With a voting regressor consisting of stacked regressors of XGBoost, KNN, and
linear models, we were able to achieve an MAE of $11,512.01, an 8.4% increase
in performance from a multiple linear regression model. 

Due to our limited data from Oct 2020 to July 2021, small dataset, and limited
features, our model does not perform as well as we hoped since $11k is a large
MAE compared to the median estimated yearly income. Model per performance can
definitely be improved by collecting data throughout the year and continuously
updating the model. The model's feature set is not sufficient and needs to be
supplemented with amenities and even pictures to give more information about
the quality of the listing. But as always, we risk overfitting by adding too
many features.

# Flask and Heroku Web App

To allow hosts to actually use the model, a FlaskAPI endpoint was
built and then deployed onto Heroku (https://airbnb-revenue.herokuapp.com). The
web app not only estimates the yearly revenue given feature values of a
listing, but it also displays an interactive map generated by Folium. This map
allows users to see other Airbnb listings filtered by how many people it
accommodates, the number of bedrooms, and finally property type.

![web_app_demo_2](reports/figures/web_app_demo_2.gif "web_app_demo_2")

This app will help Airbnb hosts make informed decisions about their
potential listings to maximize their earnings.

# Conclusion

We have successfully created a model and a web app that estimates a potential
listing's yearly revenue as well as plot similar Airbnbs on a map. This tool
will no doubt provide value to prospective Airbnb hosts, allowing them to make
more data-driven decisions, such as where they should buy a  house. Being
deployed on Heroku, any person at any time will be able to use this tool, and
the code was designed to be modified for use for any city, not just Seattle, WA.

Regarding model performance, it can definitely be improved in the future with
more data coming in every month, as well as the inclusion of more features that
allow the model to understand a listing's value.


