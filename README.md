# Yellow Taxi Trip Analysis (Jan 2025)

This project analyzes NYC Yellow Taxi Trip data for January 2025, sourced from the NYC Taxi and Limousine Commission (TLC). The analysis explores trip details such as pickup and dropoff times, distances, fares, passenger behavior, payment methods, and location zones.

## Dataset

The dataset is the Yellow Taxi Trip Records for January 2025, provided by the NYC TLC. For more information about the data and its columns, visit the [NYC TLC Trip Record Data page](http://www.nyc.gov/html/tlc/html/about/trip_record_data.shtml). A detailed description of the columns is available in the data dictionary PDF included in the repository (`data_dictionary_trip_records_yellow.pdf`).

## Notebook Overview

The Jupyter notebook `yellow_taxi_analysis.ipynb` includes the following steps:

- **Data Loading and Initial Preview:** Loads the dataset and displays initial rows.
- **Data Inspection:** Examines column types and counts.
- **Data Cleaning:** Addresses missing values, duplicates, negative values, zero values, and outliers.
- **Feature Engineering:** Adds features like trip duration, day of the week, hour of the day, and average speed.
- **Location Zone Analysis:** Maps pickup and dropoff IDs to boroughs/zones and identifies hotspots.
- **Exploratory Data Analysis (EDA):** Conducts statistical summaries, time-based analysis, revenue analysis, passenger behavior, payment type analysis, tip analysis, and trip characteristics.

## Requirements

To run the notebook, you need:

- Python 3.x
- Jupyter Notebook
- Libraries: `pandas`, `matplotlib`, `seaborn`, `numpy`, `pyarrow`

Install the required libraries with:

```bash
pip install pandas matplotlib seaborn numpy pyarrow
```

## Usage

1. Clone the repository:
   ```bash
   git clone https://github.com/safoisgod/NYC-Yellow-Taxi-Trip.git
   ```
2. Navigate to the project directory:
   ```bash
   cd NYC-Yellow-Taxi-Trip
   ```
3. Open the Jupyter notebook:
   ```bash
   jupyter notebook yellow_taxi_analysis.ipynb
   ```
4. Run the cells in the notebook to perform the analysis.

## Results and Findings

Key insights from the analysis include:

- Peak trip hours occur during evening rush hour.
- Manhattan dominates as the top pickup area, especially near tourist and business districts.
- Credit card payments are prevalent and correlate with higher average tips.

Refer to the notebook for detailed findings.

## Future Work

Potential enhancements:

- Summarize business implications (e.g., resource allocation, pricing strategies).
- Analyze multiple months for seasonal trends.
- Compare with Green Taxi data.
- Create geo heatmaps for trip density.
- Integrate weather or event data for demand insights.

## Contact

For questions or suggestions, contact me at iamnanasafo@gmail.com.