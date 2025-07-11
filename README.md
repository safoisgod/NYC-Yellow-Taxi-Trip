# 🗽 NYC Yellow Taxi Trip Data - Analytics Project Guide

## 📁 Dataset Info
- **Source:** [NYC TLC Trip Records](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)
- **Format:** Parquet
- **Dataset:** `yellow_tripdata_YYYY-MM.parquet`
- **Optional Lookup:** `taxi+_zone_lookup.csv` (maps LocationID to Zone/Borough)

---

## 🛠 Tools
- Python (Pandas, Seaborn, Matplotlib)
- SQL (DuckDB or SQLite)
- Jupyter Notebook / VS Code

---

## 📌 Project Objectives

### 1. 📆 Time-Based Trip Analysis
- Trip count by:
  - Day of week
  - Hour of day
- Peak vs. off-peak trend analysis
- Heatmap: Hour vs. Weekday trip volume

### 2. 💰 Revenue Analysis
- Total fare & tip per day
- Average fare per mile
- Revenue by hour

### 3. 👥 Passenger Behavior
- Most common passenger counts
- Avg. trip distance per passenger group
- Compare short vs long trip frequencies

### 4. 📍 Pickup & Dropoff Zones
- Join with `taxi+_zone_lookup.csv`
- Top 10 pickup zones by volume
- Dropoff hotspots
- Borough-level pickup distribution

### 5. ⏱ Trip Duration & Speed
- Calculate trip time from timestamps
- Estimate average speed: `distance / duration`
- Identify outliers (very long or very short trips)

### 6. 💳 Payment Types
- Payment method distribution
- Tip amounts by payment method
- Avg. total per type

---

## 🔍 Bonus Ideas (Optional)
- Combine multiple months for trend analysis
- Compare Yellow vs Green taxi patterns
- Visualize zone-to-zone trip matrix
- Integrate weather or event data for deeper insight

---

## 📊 Suggested Visuals
- Countplots (hour, day)
- Line plots (revenue trends)
- Heatmaps (day/hour, pickup frequency)
- Boxplots (fare, tips by category)
- Geo heatmaps (if using location zones)

---

## 📝 Reporting Suggestions
- Clean layout with sectioned insights
- Visuals with brief interpretations
- Wrap-up with findings & business implications
- Export to PDF or share notebook for portfolio

---

## 📦 Starter Code Snippets

```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

# Load dataset
df = pd.read_parquet('yellow_tripdata_2024-05.parquet')

# Basic summary
df.info()
df.describe()
