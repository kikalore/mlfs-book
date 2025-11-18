# Predict air quality data for Berlin

### Overview

This project aims to predict daily air quality (specifically PM2.5 levels) for the city of Berlin. It leverages historical weather data and air quality readings from 10 different sensors located across the city of Berlin, which can be retrieved from https://waqi.info/.

The system is designed as an end-to-end MLOps pipeline, which includes:

* Backfilling a feature store with historical data.

* A daily feature pipeline to ingest new data.

* A training pipeline to create and update a predictive model.

* A batch inference pipeline to generate daily forecasts.
  
### Folder organisation
The files for grades A are the ones provided in the main folder, moreover we provided also the files for grade C and E in their respective folders, for completeness.
### Technology used

* Feature Store: Hopsworks

* Data Sources:

* Air Quality: AQICN API (https://waqi.info/)

* Weather: Open-Meteo API

* ML Model: XGBoost (XGBRegressor)

+ GitHub Actions (for daily pipeline scheduling)


### Project Workflow & Components

The project is broken down into four main pipelines, represented by the notebooks:

1. **Lab A.1**: Feature Backfill `labA.1_air_quality_feature_backfill.ipynb`: To populate the Hopsworks feature store with historical data. 
Firstly we read historical PM2.5 data from local CSV files (for 10 sensors in Berlin) and we create lagged PM2.5 features for 1, 2, and 3 days, then the corresponding historical weather data from the Open-Meteo API are fetched.
Finally, after having defined data validation rules using `GreatExpectations`, data are ingested into two feature groups in Hopsworks: `air_quality_berlin and weather_berlin`.

2. **Lab A.2**: Feature Pipeline `labA.2_air_quality_feature_pipeline.ipynb`: A daily scheduled pipeline to fetch new data and keep the feature store up-to-date.
It fetches the today's air quality data for all 10 sensors from the AQICN API and calculates new lagged features based on the most recent data, then fetches the today's weather data and the upcoming 7-day forecast from the Open-Meteo API.
At the end, the new daily data and forecast data are inserted into the `air_quality_berlin` and `weather_berlin feature groups`.

3. **Lab A.3**: Training Pipeline `labA.3_air_quality_training_pipeline.ipynb`: To train the air quality prediction model using the available features.
A feature view is created in Hopsworks to join the `air_quality_berlin` and `weather_berlin` feature groups and a training dataset is generated from this feature view.
This feature view is used to train an XGBoost regression model to predict the next day's pm25 value.
The model is trained on all data coming from the 10 sensors but, since we manintainn the street as a feature, the model can distinguish and make predictions also based on sensor's name, as they will retrieved in the plotting section, where 10 plots are reported, showing reliable predictions for every sensor.
The trained model is saved to the Hopsworks model registry.

4. **Lab A.4**: Batch Inference Pipeline `labA.4_air_quality_batch_inference.ipynb`: A daily pipeline to generate batch predictions for the upcoming days.

The latest trained model from the Hopsworks Model Registry is loaded and a batch inference feature view is created, which gathers the 7-day weather forecast and the most recent lagged air quality data.
After that predictions for each of the sensors for the next 7 days are generated and saved back to a new feature group (`aq_predictions_berlin`) for monitoring and application use.

### Setup
To run the 4 notebooks in this repository: 
Clone the repository.

Install the required Python dependencies `pip install -r requirements.txt`.

Set up your .env file with the following:

* Hopsworks Project Name and API Key

* AQICN API Key
