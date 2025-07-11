# Step-by-Step Guide to Data Cleaning and Analysis in Python

This guide provides a comprehensive workflow for cleaning and analyzing a dataset using Python, tailored for data analysts without machine learning experience. It assumes a CSV dataset (e.g., sales data) with columns like `order_id`, `date`, `region`, `product`, `category`, `sales`, `quantity`, `customer_id`. Adjust column names to match your dataset.

## Step 1: Load the Dataset
- **Purpose**: Load the CSV file into a pandas DataFrame for analysis.
- **Action**: Use `pd.read_csv()` with error handling for common issues (e.g., file not found, parsing errors).
- **Code**:
  ```python
  import pandas as pd
  import numpy as np
  from datetime import datetime
  import warnings
  warnings.filterwarnings('ignore')

  try:
      df = pd.read_csv('sales_data.csv')  # Replace with your CSV file path
      print("Dataset loaded successfully!")
  except FileNotFoundError:
      print("Error: File not found. Please check the file path.")
      exit()
  except pd.errors.ParserError:
      print("Error: File format issue. Ensure it's a valid CSV.")
      exit()
  ```
- **Notes**:
  - Replace `'sales_data.csv'` with your file path.
  - For Excel files, use `pd.read_excel('file.xlsx')`.
  - **Pitfall**: Ensure the file path is correct (relative or absolute).

## Step 2: Initial Exploration
- **Purpose**: Understand the dataset’s structure, columns, and data types.
- **Actions**:
  - Display the first 5 rows (`df.head()`).
  - Show dataset shape (`df.shape`).
  - List column names and data types (`df.dtypes`, `df.info()`).
  - Count unique values per column to identify potential issues (e.g., too many unique values in a categorical column).
- **Code**:
  ```python
  print("\n=== Dataset Overview ===")
  print("First 5 rows:")
  print(df.head())
  print("\nShape (rows, columns):", df.shape)
  print("\nColumns and data types:")
  print(df.dtypes)
  print("\nBasic info:")
  df.info()
  print("\nUnique values in each column:")
  for col in df.columns:
      print(f"{col}: {df[col].nunique()} unique values")
  ```
- **Notes**:
  - Helps detect incorrect data types (e.g., `sales` as object instead of float).
  - **Pitfall**: For large datasets, `df.info()` may be slow; use `df.head(100)` to sample.

## Step 3: Data Quality Assessment
- **Purpose**: Identify issues like missing values, duplicates, and inconsistencies.
- **Actions**:
  - **Missing Values**: Calculate counts and percentages of missing values per column.
  - **Duplicates**: Count and report duplicate rows.
  - **Consistency Checks**: Detect negative values in numerical columns (e.g., `sales`, `quantity`) and invalid categorical values (e.g., empty strings).
- **Code**:
  ```python
  print("\n=== Missing Values ===")
  missing_data = df.isnull().sum()
  missing_percentage = (missing_data / len(df) * 100).round(2)
  print("Missing values per column:")
  print(pd.DataFrame({'Count': missing_data, 'Percentage (%)': missing_percentage}))

  print("\n=== Duplicates ===")
  duplicate_count = df.duplicated().sum()
  print("Number of duplicate rows:", duplicate_count)
  if duplicate_count > 0:
      df = df.drop_duplicates()
      print("Duplicates removed. New shape:", df.shape)

  print("\n=== Data Consistency ===")
  numerical_cols = df.select_dtypes(include=[np.number]).columns
  for col in numerical_cols:
      if (df[col] < 0).sum() > 0:
          print(f"Negative values in '{col}': {(df[col] < 0).sum()}")

  categorical_cols = df.select_dtypes(include=['object']).columns
  for col in categorical_cols:
      unique_values = df[col].dropna().unique()
      if '' in unique_values or ' ' in unique_values:
          print(f"Empty or whitespace values found in '{col}'")
      if len(unique_values) < 10:
          print(f"Unique values in '{col}': {unique_values}")
  ```
- **Notes**:
  - Missing value percentages help prioritize cleaning efforts.
  - Negative values in `sales` or `quantity` may indicate errors (e.g., data entry issues).
  - **Tip**: Log inconsistencies (e.g., `df[df['sales'] < 0].to_csv('inconsistencies.csv')`) for review.

## Step 4: Data Cleaning
- **Purpose**: Address missing values, standardize formats, handle outliers, and correct inconsistencies.
- **Actions**:
  - **Missing Values**: Fill numerical columns with median (robust to outliers) and categorical columns with mode.
  - **Standardization**: Convert text columns to consistent formats (e.g., lowercase `region`, title case `product`/`category`).
  - **Date Conversion**: Convert `date` to datetime, handling invalid formats.
  - **Outliers**: Cap numerical outliers using the IQR method.
  - **Negative Values**: Clip negative `sales` or `quantity` to 0 (assuming they’re invalid).
- **Code**:
  ```python
  print("\n=== Handling Missing Values ===")
  for col in df.columns:
      if missing_data[col] > 0:
          if df[col].dtype in ['int64', 'float64']:
              median_value = df[col].median()
              df[col] = df[col].fillna(median_value)
              print(f"Filled missing values in '{col}' with median: {median_value}")
          else:
              mode_value = df[col].mode()[0]
              df[col] = df[col].fillna(mode_value)
              print(f"Filled missing values in '{col}' with mode: {mode_value}")

  print("\n=== Standardizing Data ===")
  if 'region' in df.columns:
      df['region'] = df['region'].str.lower().str.strip()
      print("Standardized 'region' to lowercase and removed whitespace.")
  if 'product' in df.columns:
      df['product'] = df['product'].str.title().str.strip()
      print("Standardized 'product' to title case and removed whitespace.")
  if 'category' in df.columns:
      df['category'] = df['category'].str.title().str.strip()
      print("Standardized 'category' to title case and removed whitespace.")

  if 'date' in df.columns:
      try:
          df['date'] = pd.to_datetime(df['date'], errors='coerce')
          if df['date'].isnull().sum() > 0:
              print(f"Warning: {df['date'].isnull().sum()} invalid dates found after conversion. Filled with most frequent date.")
              df['date'] = df['date'].fillna(df['date'].mode()[0])
          print("Converted 'date' column to datetime.")
      except Exception as e:
          print(f"Error converting 'date' column: {e}. Check format.")

  print("\n=== Handling Outliers ===")
  for col in numerical_cols:
      Q1 = df[col].quantile(0.25)
      Q3 = df[col].quantile(0.75)
      IQR = Q3 - Q1
      lower_bound = Q1 - 1.5 * IQR
      upper_bound = Q3 + 1.5 * IQR
      outliers = df[(df[col] < lower_bound) | (df[col] > upper_bound)][col]
      if len(outliers) > 0:
          print(f"Outliers in '{col}': {len(outliers)}")
          df[col] = df[col].clip(lower=lower_bound, upper=upper_bound)
          print(f"Capped outliers in '{col}' to IQR bounds.")

  if 'sales' in df.columns:
      negative_sales = (df['sales'] < 0).sum()
      if negative_sales > 0:
          df['sales'] = df['sales'].clip(lower=0)
          print(f"Corrected {negative_sales} negative values in 'sales' to 0.")
  if 'quantity' in df.columns:
      negative_quantity = (df['quantity'] < 0).sum()
      if negative_quantity > 0:
          df['quantity'] = df['quantity'].clip(lower=0)
          print(f"Corrected {negative_quantity} negative values in 'quantity' to 0.")
  ```
- **Notes**:
  - Median is preferred for numerical missing values to avoid outlier influence.
  - **Pitfall**: Imputing missing values can introduce bias; verify with domain knowledge (e.g., missing `sales` might mean 0).
  - Check date formats with `df['date'].head()` if parsing fails.

## Step 5: Exploratory Data Analysis
- **Purpose**: Extract insights through summary statistics, grouping, time-based analysis, and correlations.
- **Actions**:
  - **Summary Statistics**: Compute stats for numerical and categorical columns.
  - **Grouped Analysis**: Aggregate `sales` by `region`, `category`, and `product`.
  - **Time-Based Analysis**: Summarize `sales` by year, quarter, and month (if `date` exists).
  - **Customer Analysis**: Analyze total sales and purchase frequency by `customer_id`.
  - **Correlation Analysis**: Compute correlations between numerical columns.
- **Code**:
  ```python
  print("\n=== Summary Statistics ===")
  print("\nNumerical columns summary:")
  numerical_summary = df[numerical_cols].describe().round(2)
  print(numerical_summary)
  print("\nCategorical columns summary:")
  for col in categorical_cols:
      print(f"\nValue counts for '{col}':")
      print(df[col].value_counts().head(10))

  print("\n=== Grouped Analysis ===")
  if 'region' in df.columns and 'sales' in df.columns:
      print("\nSales by region (mean, sum, count):")
      region_sales = df.groupby('region')['sales'].agg(['mean', 'sum', 'count']).round(2)
      print(region_sales)

  if 'category' in df.columns and 'sales' in df.columns:
      print("\nSales by category (mean, sum, count):")
      category_sales = df.groupby('category')['sales'].agg(['mean', 'sum', 'count']).round(2)
      print(category_sales)

  if 'product' in df.columns and 'sales' in df.columns:
      print("\nTop 10 products by total sales:")
      product_sales = df.groupby('product')['sales'].sum().sort_values(ascending=False).head(10).round(2)
      print(product_sales)

  if 'date' in df.columns and df['date'].dtype == 'datetime64[ns]':
      print("\n=== Time-Based Analysis ===")
      df['year'] = df['date'].dt.year
      df['month'] = df['date'].dt.to_period('M')
      df['quarter'] = df['date'].dt.to_period('Q')
      print("\nTotal sales by year:")
      yearly_sales = df.groupby('year')['sales'].sum().round(2)
      print(yearly_sales)
      print("\nTotal sales by quarter:")
      quarterly_sales = df.groupby('quarter')['sales'].sum().round(2)
      print(quarterly_sales)
      print("\nTotal sales by month (last 12 months):")
      monthly_sales = df.groupby('month')['sales'].sum().tail(12).round(2)
      print(monthly_sales)

  if 'customer_id' in df.columns and 'sales' in df.columns:
      print("\n=== Customer Analysis ===")
      customer_sales = df.groupby('customer_id')['sales'].agg(['sum', 'count']).round(2)
      print("\nTop 10 customers by total sales:")
      print(customer_sales.sort_values('sum', ascending=False).head(10))
      print("\nCustomer purchase frequency (number of orders):")
      print(customer_sales['count'].describe().round(2))

  if len(numerical_cols) > 1:
      print("\n=== Correlation Analysis ===")
      correlation_matrix = df[numerical_cols].corr().round(2)
      print("Correlation matrix:")
      print(correlation_matrix)
  ```
- **Notes**:
  - Use `median` instead of `mean` for skewed data in grouped analysis.
  - Limit value counts to top 10 for brevity in large categorical columns.
  - **Tip**: Add custom metrics (e.g., `df['sales_per_unit'] = df['sales'] / df['quantity']`).

## Step 6: Save Results
- **Purpose**: Save the cleaned dataset and analysis results for reporting.
- **Actions**:
  - Save cleaned dataset as CSV.
  - Export summaries (numerical, categorical, grouped, time-based, customer analyses) to Excel with multiple sheets.
- **Code**:
  ```python
  df.to_csv('cleaned_sales_data.csv', index=False)
  print("\nCleaned dataset saved as 'cleaned_sales_data.csv'")

  with pd.ExcelWriter('detailed_analysis_summary.xlsx') as writer:
      numerical_summary.to_excel(writer, sheet_name='Numerical_Summary')
      for col in categorical_cols:
          df[col].value_counts().head(10).to_excel(writer, sheet_name=f'{col}_Value_Counts')
      if 'region' in df.columns:
          region_sales.to_excel(writer, sheet_name='Region_Analysis')
      if 'category' in df.columns:
          category_sales.to_excel(writer, sheet_name='Category_Analysis')
      if 'product' in df.columns:
          product_sales.to_excel(writer, sheet_name='Top_Products')
      if 'date' in df.columns and df['date'].dtype == 'datetime64[ns]':
          yearly_sales.to_excel(writer, sheet_name='Yearly_Sales')
          quarterly_sales.to_excel(writer, sheet_name='Quarterly_Sales')
          monthly_sales.to_excel(writer, sheet_name='Monthly_Sales')
      if 'customer_id' in df.columns:
          customer_sales.sort_values('sum', ascending=False).head(10).to_excel(writer, sheet_name='Top_Customers')
  print("Detailed analysis saved as 'detailed_analysis_summary.xlsx'")
  ```
- **Notes**:
  - Excel output is stakeholder-friendly.
  - **Tip**: Add timestamps to filenames (e.g., `f'cleaned_sales_data_{datetime.now().strftime('%Y%m%d')}.csv'`) for versioning.

## How to Use This Guide
1. **Setup**:
   - Copy the code into a `.py` file (e.g., `data_cleaning_analysis.py`) or Jupyter Notebook.
   - Replace `'sales_data.csv'` with your file path.
   - Ensure libraries are installed.
2. **Customize**:
   - Update column names (e.g., `sales`, `region`) to match your dataset.
   - Skip steps if columns are missing (e.g., `date` for time-based analysis).
   - For non-CSV files, use `pd.read_excel()` or other pandas readers.
3. **Run and Interpret**:
   - Run the script and review console outputs (e.g., missing values, summaries, grouped results).
   - Use tables to identify insights (e.g., top products, high-value customers, trends).
   - Check `detailed_analysis_summary.xlsx` for a report-ready summary.
4. **Handle Errors**:
   - **Column Not Found**: Verify with `print(df.columns)`.
   - **Type Errors**: Cast columns (e.g., `df['sales'] = pd.to_numeric(df['sales'], errors='coerce')`).
   - **Date Issues**: Check `df['date'].head()` and specify format in `pd.to_datetime()` (e.g., `format='%Y-%m-%d'`).
   - **Memory Issues**: Sample data (`df.sample(frac=0.1)`) or use chunking (`pd.read_csv(..., chunksize=10000)`).

## Additional Tips
- **Scalability**: For datasets >1M rows, use `dask` or chunking.
- **Domain Knowledge**: Validate cleaning decisions (e.g., are negative `sales` valid refunds?).
- **Custom Analysis**: Add metrics like sales per unit or filter specific periods.
- **Audit Trail**: Log cleaning steps to a file (e.g., `with open('cleaning_log.txt', 'w') as f: f.write(...)`).
- **Error Handling**: Add try-except blocks for variable datasets.

## Example Output
For a 10,000-row dataset:
- **Console**: Missing value counts/percentages, duplicate count, negative value counts, numerical/categorical summaries, grouped analyses (region, category, product), time-based summaries (yearly, quarterly, monthly), top customers, correlations.
- **Files**: `cleaned_sales_data.csv` (cleaned dataset), `detailed_analysis_summary.xlsx` (tables for summaries, grouped analyses, etc.).

## Troubleshooting
- **Missing Columns**: Check `df.columns` and update code.
- **Invalid Types**: Use `df.dtypes` and cast (e.g., `df['sales'].astype(float)`).
- **Memory Issues**: Sample or chunk large datasets.
- **Date Parsing**: Specify format or use `errors='coerce'`.