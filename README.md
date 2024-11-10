# ID2223 Lab 1: Air Quality Prediction in Reutlingen

## 🖥️ [Click me to see the dashboard](https://arraying.github.io/Reutlingen/) ✨

The purpose of this README is to explain the prediction service at a conceptual level.
For more information on how to run locally, please scroll down.

## Way of Working

We both did the whole (base) assignment.
We wanted to do this such that both of us learn as much as possible, and we had the time.
Jonas used `main` and Paul used `ph` to prototype the model.

Jonas was responsible for fine-tuning the expectations, hindcast and CI.
Paul was responsible for the lagged air quality and dashboard frontend.
Overall, the division of labor was equal.

## Data

We decided to investigate [Zaisentalstraße](https://aqicn.org/station/@54451/) in Reutlingen.
In order to facilitate prediction, data has been provided for noncommercial purposes by AQICN and Open-Meteo.
This project uses the data for educational purposes, in-line with their respective licenses.

## Tasks

Below is an explanation of all the tasks.
Primarily, the purpose of these sections is to provide insights on both the "how" and "what".
The descriptions have been obtained from the course website.
The majority of the code is adapted from the "Building Machine Learning Systems with a Feature Store" book.

### Backfill

> Write a backfill feature pipeline that downloads historical weather data (ideally >1
year of data), loads a csv file with historical air quality data (downloaded from
https://aqicn.org) and registers them as 2 Feature Groups with Hopsworks.

We decided on the Zaisentalstraße sensor as it contained a lot of historical data with little gaps.
Additionally, the sensor did not seem to produce invalid readings.

We additionally impose expectations on our data.

Air Quality Data:
- The PM2.5 value may not be below -1 or above 500.
- The location may not change.

Weather Data:
- The temperature may not be below -60 or above 60.
- Precipitation and wind speed may not be below 0 or above 1000.

### Daily Updates

> Schedule a daily feature pipeline notebook that downloads yesterday’s weather
data and air quality data, and also the weather prediction for the next 7-10 days
and update the Feature Groups in Hopsworks. Use GH Actions or Modal.

To perform a daily update, we scheduled GitHub Actions in a cronjob. 
Our code was already in a repository and this does not require a credit card, therefore it was an easy decision.
We utilize as much forecast data as Open-Meteo provides for the coordinates.

### Training

> Write a training pipeline that (1) selects the features for use in a feature view, (2)
reads training data with the Feature View, trains a regression or classifier model to
predict air quality (pm25). Register the model with Hopsworks.

To train the model, we use XGBoost. 
We do not perform hyperparameter tuning.

### Batch Inference

> Write a batch inference pipeline that creates a dashboard. The program should
download your model from Hopsworks and plot a dashboard that predicts the air
quality for the next 7-10 days for your chosen air quality sensor.

We perform batch inference rather than online inference. 
This means that we update forecasts once a day, rather than whenever a query is made.
Consequently, our update notebook also had to be scheduled with a cronjob.

Our data is visualized on the dashboard linked above.
We primarily present the data in German, in order to convey the information to the local community.
English text is added as a fallback language.

### Monitoring

> Monitor the accuracy of your predictions by plotting a hindcast graph showing
your predictions vs outcomes (measured air quality).

The monitoring is transparently placed on the dashboard.
It shows that the predictions are just that -- predictions.
We decided to only show a few days as this helps the reader "get a feel" of the accuracy.
Ideally, our model would re-train often anyway and therefore long-term innacuracy is irrelevant.

### Lagged Air Quality

> Update your Model by adding a new feature, lagged air quality for the previous 1-3
days.

The training data was augmented with Pandas' `df.rolling()`.
Each datapoint received a `lagging` column which represented the mean of the past 3 observations (excl. itself).
With this, it was possible to train the model on an additional feature.
Our change in performance (Paul's model) can be seen below.

| Model  | MSE | R Squared |
|--------| --- | --- |
| No lag | 22.54 | 0.21 |
| Lag | 5.42 | 0.28 |

As expected, the addition of the lagged air quality improves the model.
While the MSE sinks drastically, the R squared still remains quite low.
Nonetheless, this is probably "good enough" for the purpose of air quality prediction.
It is also unsurprising that according to XGBoost, the lag became the most important feature.

### Future Steps

Some future steps that could be performed to improve the model are:
- Perform hyperparameter tuning.
- Use an RNN to utilize predictions in the forecasts.
- Use more data points, such as by taking into account (with a weight) other sensors in close proximity.
- Improve efficiency by using views with e.g. aggregate functions.
- Improve efficiency by only downloading parts of the data.
- Re-train the model periodically.
- Utilize versions in Hopsworks.

## Setup 

Before the code can be set up, you need an API key for Hopsworks and AQICN.
It should be noted that the Hopsworks API key must have _all_ scopes selected, including `USER`.

**TODO: Explain the clone, dependency install and running process.**