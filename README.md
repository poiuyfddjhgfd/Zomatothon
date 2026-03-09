Zomatothon – Rider Dispatch Optimization
📌 Problem Statement
In food delivery, riders often wait at restaurants because meals are not ready when they arrive. This idle time reduces efficiency and increases delivery costs. The goal of this project is to optimize the dispatch time of riders so they arrive at the restaurant exactly when the meal is expected to be ready, minimizing rider wait time while keeping customer delays within acceptable limits.

📊 Dataset
We use a synthetic Zomato dataset (Zomato_Optimized_Synthetic_Dataset.csv) containing ~45k delivery records. Key columns include:

Order_Date, Time_Orderd, Time_Order_picked – timestamps

Restaurant_ and Delivery_location_ coordinates

Delivery_person_Age, Delivery_person_Ratings

Weather_conditions, Road_traffic_density

Type_of_order, Type_of_vehicle

multiple_deliveries, Festival, City

Time_taken (min) – total delivery time

actual_prep_time – time from order confirmation to pickup (engineered)

predicted_KPT – simulated kitchen preparation time prediction

The raw data contains messy time formats (e.g., Excel decimal times) and requires careful cleaning.

🧠 Approach
1. Data Cleaning & Feature Engineering
Converted Order_Date and time columns to proper datetime, handling Excel decimals and next‑day rollovers.

Computed actual_prep_time (order confirmation → pickup) and travel_time_min (from Time_taken (min)).

Created rider_arrival_time = pickup_time - travel_time_min (baseline).

Added derived features: hour, rush_hour, city_avg_prep, city_count, rating_load.

2. Machine Learning for Prep Time Prediction
Used RandomForestRegressor (500 trees, max depth 15) to predict actual_prep_time.

Features: rush, distance, travel_time, Delivery_person_Age, Delivery_person_Ratings, city_avg, city_count, rating_load.

Achieved MAE reduction from 5.8 (baseline) to 2.9 minutes (~50% improvement).

3. Operations Research – Optimal Dispatch
For each delivery, given predicted prep time 
p
p and travel time 
t
t, we solve:

min
⁡
d
≥
0
[
max
⁡
(
0
,
 
p
−
(
d
+
t
)
)
+
2
⋅
max
⁡
(
0
,
 
(
d
+
t
)
−
p
)
]
d≥0
min
​
 [max(0,p−(d+t))+2⋅max(0,(d+t)−p)]
where 
d
d is the delay from order confirmation to dispatching the rider.

This balances rider wait (first term) and customer delay (second term, weighted 2×).

Optimal 
d
d is found by a simple grid search over 0.5‑minute increments.

4. Evaluation
Baseline wait: rider arrives immediately (d=0), wait = 
max
⁡
(
0
,
 
p
−
t
)
max(0,p−t).

Optimized wait: using optimal 
d
d, wait = 
max
⁡
(
0
,
 
p
−
(
d
+
t
)
)
max(0,p−(d+t)).

Customer delay = 
max
⁡
(
0
,
 
(
d
+
t
)
−
p
)
max(0,(d+t)−p).

🏆 Results
Metric	Value
Baseline wait (mean)	4.11 min
Optimized wait (mean)	1.50 min
Customer delay (mean)	1.48 min
Wait reduction	63.7%
95% confidence interval for improvement: [63.1%, 64.2%] (bootstrap).

Only 0.23% of orders have wait > 5 min after optimization (vs. 30% before).

Hard constraints limit max customer delay to 8 minutes.

📁 Repository Contents
Zomatothon.ipynb – full notebook with all experiments and final solution.

transformed_data.csv – intermediate dataset after initial preprocessing.

ZOMATHON_FINAL_WINNING_OUTPUT.csv – final dataset with optimized dispatch times.

README.md – this file.

🚀 How to Run
Requirements
Install the following Python libraries:

bash
pip install pandas numpy matplotlib seaborn scikit-learn tqdm
Steps
Place Zomato_Optimized_Synthetic_Dataset.csv in the same directory as the notebook.

Run all cells in Zomatothon.ipynb (Colab or local Jupyter).

The final results will be printed, and two output CSV files will be generated.

⚠️ The notebook includes multiple trial versions. The final winning pipeline is at the end (cells with headings like "FINAL WINNING SOLUTION"). You can also run the entire notebook sequentially – it will overwrite intermediate variables but the final results remain consistent.

🔍 Key Insights
Merchant‑specific quantiles outperform global predictions – using the distribution of past prep times for each restaurant yields better dispatch decisions.

Including a congestion index (KCI) that counts recent orders per merchant further improves robustness.

Hard delay constraints are essential to prevent excessive customer wait even if rider wait is reduced.

Combining ML with OR gives the best of both worlds: accurate prep time forecasts + principled trade‑off optimization.

📈 Visualisation
The notebook includes histograms comparing baseline vs. optimized wait distributions, showing a dramatic reduction in long waits.

Built for Zomato’s internal hackathon – Zomatothon.

