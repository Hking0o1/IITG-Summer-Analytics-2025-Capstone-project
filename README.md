# IITG-Summer-Analytics-2025-Capstone-project

# Dynamic Pricing for Urban Parking

## 🧾 Project Overview

This project aims to develop a dynamic pricing system for urban parking lots using real-time occupancy and environmental data. The primary objective is to maximize resource utilization and optimize pricing using demand-driven and competitive strategies.

---

## 📊 Dataset Overview

**Source**: Provided `dataset.csv` includes the following columns:

* `SystemCodeNumber`: Unique parking lot ID
* `Capacity`, `Occupancy`: Lot size and current usage
* `Latitude`, `Longitude`: Geo-coordinates
* `QueueLength`: Vehicles waiting
* `TrafficConditionNearby`: Low, Medium, High, etc.
* `IsSpecialDay`: Binary flag (e.g., public holiday)
* `Timestamp`: Combined from date and time

---

## 🔧 Data Preprocessing

* Combined `LastUpdatedDate` and `LastUpdatedTime` into a `Timestamp`
* Removed null values
* Encoded categorical variables:

  * `VehicleType`: car=0, bike=1, truck=2, cycle=3
  * `TrafficConditionNearby`: low=0, average/medium=1, high=2

```python
# Code snippet
vehicle_map = {'car': 0, 'bike': 1, 'truck': 2, 'cycle': 3}
traffic_map = {'low': 0, 'average': 1, 'medium': 1, 'high': 2}
df['VehicleTypeEncoded'] = df['VehicleType'].map(vehicle_map)
df['TrafficEncoded'] = df['TrafficConditionNearby'].map(traffic_map)
```

---

## 📈 Model 1: Baseline Linear Pricing

### Formula:

$Price_{t+1} = Price_t + \alpha \cdot \left( \frac{Occupancy}{Capacity} \right)$

* $\alpha = 2$, Base Price = 10
* Price bounded between \$5 and \$20

### Logic:

* Loop through each parking lot by time
* Update price based on occupancy ratio

```python
price += alpha * (occupancy / capacity)
```

---

## 📈 Model 2: Demand-Based Pricing

### Formula:

$Demand = \alpha\cdot\left(\frac{Occupancy}{Capacity}\right) + \beta\cdot QueueLength - \gamma\cdot Traffic + \delta\cdot IsSpecialDay + \varepsilon\cdot VehicleTypeWeight$

* Normalize demand per parking lot
* Price computed as:
  $Price = BasePrice \cdot (1 + \lambda \cdot NormalizedDemand)$
* $\lambda = 0.5$

```python
# MinMax normalization
scaler = MinMaxScaler()
df['NormalizedDemand'] = df.groupby('SystemCodeNumber')['RawDemand'].transform(lambda x: scaler.fit_transform(x.values.reshape(-1, 1)).flatten())
```

---

## 📈 Model 3: Competitive Pricing (Planned/Optional)

* Incorporates price signals from nearby parking lots
* Adjusts pricing based on local supply/demand density
* Uses Haversine distance to compute proximity

---

## 🔄 Real-Time Simulation with Pathway

### Components:

* Input: Streamed from `input_stream.csv`
* Processing: UDF applies demand model row-by-row
* Output: Written to `streamed_prices.jsonl`

```python
input_tbl = pw.demo.replay_csv("input_stream.csv", schema=..., input_rate=5.0)

@pw.udf
def price_udf(...):
    # apply demand_pricing_model(row)

pw.io.jsonlines.write(priced, filename="streamed_prices.jsonl")
pw.run()
```

---

## 📊 Visualization (Bokeh Dashboard)

* Interactive dropdown to explore price trends per lot
* Time-series plot of dynamic pricing
* Reads from `streamed_prices_clean.jsonl`

```python
from bokeh.plotting import figure
p.line('Timestamp', 'Price', source=ColumnDataSource(df))
```

---

## ✅ Results & Insights

* Baseline model shows increasing prices with occupancy
* Demand-based model better captures variability
* Real-time simulation using Pathway is stable and scalable

---

## 🔚 Conclusion

* A full dynamic pricing pipeline was implemented
* Models adjust prices in real time using demand features
* Pathway provides seamless real-time ingestion and computation

---

## 📌 Next Steps (if extended)

* Integrate competitive model fully
* Add prediction for expected queue length
* Enable frontend live updates

---

## 👨‍💻 Contributors

* Himanshu Kulshrestha (Author)
* Guided by: Summer Analytics 2025 Capstone Team
