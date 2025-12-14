# Citi-Bike-Large-Scale-Mobility-Analysis
Large-scale analysis of millions of Citi Bike trips to identify station-level demand imbalances and temporal mobility patterns using Python.


This project analyzes **Citi Bike trip data at scale** to:
1) Engineer behavioral + temporal features for **unsupervised segmentation**, and  
2) Quantify **station–hour demand imbalance** using departures vs arrivals (net outflow).

The repository includes an exported HTML report (`CITI_Analysis.html`) so that anyone can view the results without running the code.

---

## What's Inside
- **Unsupervised Segmentation** (baseline K-Means vs improved Gaussian Mixture Model)
- **Feature engineering** for behavior/time + approximate trip distance
- **Station–Hour Imbalance Heatmap** using net outflow (Departures − Arrivals) for the busiest stations
- Commentary on suitability + limitations for segmentation (behavioral + geographic, no demographics)

---

## Dataset
- Source: Citi Bike public trip data
- Link: https://s3.amazonaws.com/tripdata/index.html
- In my workflow, multiple CSVs from the same release were **combined into one master file** before analysis.

---

## How to View
### Option A — Open the HTML report
Download and open:
- `CITI_Analysis.html`

### Option B — (Optional) Run the notebook
If you include the notebook, run it in Jupyter after installing typical DS packages (pandas, numpy, scikit-learn, plotly).

---

## Methodology

### 1) Data Ingestion (Multi-CSV → Master)
- Read multiple CSV files from a dataset folder and concatenate them into a single **master CSV** for analysis. 
- Note: reading large CSVs can raise dtype warnings due to mixed column types; the pipeline still proceeds after ingest. 

### 2) Basic Cleaning + Time Parsing
- Drop missing rows (to ensure station/time fields are available for downstream analysis).
- Convert timestamps (`started_at`, `ended_at`) to datetime objects for time-based features. 

### 3) Feature Engineering (Behavior + Time + Distance)
- **Ride duration (minutes)** computed from `ended_at - started_at`. 
- Filter out invalid trips: remove zero/negative durations and trips longer than 24 hours. 
- **Approximate trip distance (km)** using the Haversine formula between start/end coordinates (approximation; routes may vary).
- **Time features**: start hour, day-of-week, month + flags:
  - `is_weekend`
  - `is_peak_hour` (commute peaks)  

### 4) Sampling + Scaling for Segmentation
- Because the dataset is very large, clustering is performed on a **sample (e.g., 100k rows)** to keep runtime reasonable on a laptop.  
- Standardize features using **StandardScaler** before clustering.

### 5) Unsupervised Segmentation Models
**Baseline: K-Means**
- Fit K-Means across multiple `k` values (e.g., 3–6).
- Evaluate using:
  - **Silhouette Score**
  - **Davies–Bouldin Index**
- Select the best `k` by silhouette score, then assign final cluster labels.

**Improved: Gaussian Mixture Model (GMM)**
- Fit GMM across multiple component counts (e.g., 3–6).
- Evaluate using the same metrics (silhouette + Davies–Bouldin).
- Select best component count by silhouette score, then assign final GMM cluster labels.

**Cluster Profiling**
- For each cluster, compute summary stats (means of duration, distance, hour/day features, weekend share, peak-hour share, and trip count).

### 6) Station–Hour Imbalance (Net Outflow) + Heatmap
To quantify shortages/surpluses by station and hour:

- Compute **departures** per `(start_station_name, start_hour)` and **arrivals** per `(end_station_name, end_hour)`.
- Rename columns so both datasets align on `(station_name, hour)`, then **outer join** and fill missing with 0.
- Compute:
  - `net_outflow = departures − arrivals` 
- Identify the **busiest stations** by total volume (departures + arrivals) and select the **top N (e.g., 25)** for visualization. 
- Pivot to a station×hour matrix and plot an imbalance heatmap (“Station–Hour Imbalance Heatmap (Shortage Risk)”). 
- Business interpretation: red hours indicate shortage risk (high net outflow); blue indicates surplus periods (net inflow). 
---

## Key Outputs
- **Cluster profiles** describing distinct usage patterns (duration, distance, peak-hour share, weekend share).
- **Station-hour imbalance heatmap** highlighting predictable commuter-driven shortages and surpluses at high-volume stations.

---

## Limitations
- Trip distance is approximated (straight-line Haversine).
- Segmentation is based on behavioral + geographic signals; the dataset lacks demographics, limiting “who the user is” segmentation.

---

## Author
Shree Aditya Gujju 
