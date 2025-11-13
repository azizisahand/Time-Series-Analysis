## **My Journey Through a Time Series Maze: Forecasting London's Air Quality**

Trying to predict the future with time series data can feel like navigating a maze. You face challenges like missing information, seasonal cycles, picking the right model, and the constant fear of *overfitting* (creating a model that looks good on old data but can't predict new data).

Here, I'll guide you through my process of creating a forecast from the beginning, using data on London's monthly NO₂ levels. I won't just cover the technical parts, but also the little frustrations that happen along the way.

### **1\. Getting and Cleaning the Data**

The data I used tracked the average monthly NO₂ levels from London roadsides. I ran into my first issues almost immediately:

* **Duplicate Data:** There was one repeated entry in the 'Month' column. This created errors when I tried to set that column as the index, but a simple `.drop_duplicates()` fixed it.  
* **Missing Values:** Data for two different months was missing. Rather than deleting those rows, I decided to use **interpolation** (estimating the missing values) to maintain a continuous timeline.  
* **Date Formatting:** The 'Month' column was stored as text (e.g., "Jan-20"). I needed to convert this into a proper datetime format using `pd.to_datetime` and then set a correct monthly frequency for the dataset.

This just proved an early lesson: time series data almost never arrives in a perfectly clean format.

### **2\. A First Look: Visualizing the Data**

Once the data was cleaned up, my first step was to create a visual plot. The line plot showed two obvious patterns:

1. A clear **downward trend** over time (likely due to London's work on air quality).  
2. A **strong seasonal pattern**, with predictable ups and downs throughout the year.

Making this chart wasn't just for appearances; it gave me a better intuition for which models might be successful.

### **3\. Breaking It Down (Decomposition)**

After that, I used a **classical decomposition** technique. This method splits the time series into its three core components:

* **Trend:** The long-term downward movement.  
* **Seasonality:** The repeating yearly patterns.  
* **Residuals:** The random, unpredictable part of the data.

![][image1]

This step was a good "sanity check." The seasonal component was clearly very strong, which suggested that basic models would likely have trouble. This plot shows the strong yearly pattern:

![][image2]

### **4\. Splitting the Data (Carefully\!)**

Forecasting isn't only about building a model; it's about seeing if it can *really* predict new data. I divided my dataset:

* **Train set:** All data except for the final 82 weeks.  
* **Test set:** The last \~2 years, which I used only for testing.

The main challenge is that you **must respect the chronological order**. Shuffling the data, as you might in other projects, is not allowed\!

![][image3]

### **5\. Is the Data "Stable"? (Stationarity)**

Many traditional models, like ARIMA, require the data to be **stationary** (meaning its average and variance don't change over time). My data had a clear trend, so it was **non-stationary**. I used the Augmented Dickey-Fuller test to prove this statistically.

**The solution was differencing.** This technique subtracts a previous value from the current value, which removed most of the trend and made the data stable enough for ARIMA.

### **6\. The Fun Part: Testing the Models**

Here came the fun (and frustrating) part: modeling.

#### **Baseline Models**

First, I tried simple baseline models. They are often bad at predicting, but they give you a starting score to beat.

Here is how they performed:

![][image4]

Out of these simple models, the **Drifted Forecast** performed the best, achieving a solid 11% mean absolute percentage error (MAPE).

#### **"Real" Models**

Now it was time for the more advanced models.

**ARIMA**

* This model works well for trends and autocorrelation.  
* It **had difficulty capturing the strong seasonal pattern**.  
* Choosing the right parameters (p, d, q) required careful tuning and felt like a process of trial-and-error.

To get the parameters right, you have to look at an Autocorrelation Function (ACF) plot. This shows how current values relate to past values (lags).

plot\_acf(time\_series\['London Mean Roadside:Nitrogen Dioxide (ug/m3)'\])![][image5]

* **What this plot told me:**  
  * Strong correlation at lags 1-2 means the series has "memory."  
  * Clear spikes around lag 10-12 proved the **strong yearly seasonality**.  
  * This pattern clearly showed that a simple ARIMA model wasn't enough.

**SARIMA**

* This model is an extension of ARIMA that includes **seasonal components**.  
* Unsurprisingly, it provided a **much better fit** for the yearly patterns in the data.

**AR (AutoRegressor) Model**

* Because the ACF plot was available, I also tested an AutoRegressor (AR) model.  
* This type of model forecasts by using a weighted average of past values. It's a straightforward model, easy to interpret, and is effective for stationary data.

**Prophet**

* Using Facebook's Prophet library was a relief because it was so **easy to use**.  
* It automatically detects trends and seasonality.  
* Its main drawback was that it sometimes **over-smoothed** the data, flattening out the highest and lowest spikes.

![][image6]

![][image7]

**XGBoost**

* This machine learning model isn't a typical choice for time series.  
* It demanded more preparation, like engineering "lag features" (using past data as inputs).  
* The results were **surprisingly competitive**, but the model is much harder to interpret (it's a "black box").

XGBoost MAE: 7.52  
XGBoost MAPE: 0.18  
XGBoost RMSE: 8.72  
![][image8]

### **7\. Evaluating the Models**

To judge the models, I used a few key metrics:

* **MAE (Mean Absolute Error):** Shows the average size of the errors.  
* **RMSE (Root Mean Squared Error):** This metric penalizes large errors more.  
* **MAPE (Mean Absolute Percentage Error):** This shows the error as a percentage, which is easy to understand.

Here is how all the metrics stacked up against each other:

![][image9]

Not one of the models was perfect, and they all had tradeoffs:

* **ARIMA** was easy to explain but failed to capture the seasonal patterns.  
* **Prophet** was reliable and simple but often smoothed the data too much.  
* **XGBoost** handled the complex patterns but had a high risk of overfitting.

My biggest takeaway was this: **there is no single best method**. The right model truly depends on your specific goals. Do you need to explain your forecast (interpretability) or do you just need the most accurate number? This journey showed me that forecasting is as much about understanding and cleaning your data as it is about complex modeling.
