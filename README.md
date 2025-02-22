# Reseller Segmentation using RFM Analysis and Clustering

---

## Overview

This project applies RFM (Recency, Frequency, Monetary) Analysis combined with K-Means clustering to segment resellers based on their purchasing behavior. The goal is to identify valuable resellers, optimize marketing strategies, and enhance customer engagement through data-driven insights.

## Dataset

The analysis is performed using the `FactResellerSales` table, which contains transactional data related to reseller purchases.

**Source:** `AdventureWorksDW2022` which you can download [here](https://learn.microsoft.com/en-us/sql/samples/adventureworks-install-configure?view=sql-server-ver16&tabs=ssms)

## Tools Used

- **Python** for data extraction, processing, and visualization.
- **PyODBC** for SQL database connection.
- **Matplotlib** & **Seaborn** for data visualization.
- **Scikit-learn** for clustering and analysis.

  ## Data Cleaning and Preparation

- Removing duplicate transactions.
- Handling missing values in OrderDate, SalesAmount, or SalesOrderNumber.
- Aggregating transactions per reseller.
- Detecting and handling outliers using boxplots and statistical methods.

## Exploratory Data Analysis (EDA)

- Analyzing the distribution of `SalesAmount`, `OrderDate`, and `SalesOrderNumber`.
- Identifying purchasing behavior trends among resellers.
- Detecting outliers that may impact segmentation.

## Data Analysis

The segmentation process is based on RFM metrics, I use `pd.qcut()`:
```python
current_date = datetime(2013, 11, 29)
df_recency = df_FactResellerSales.groupby('ResellerKey')['OrderDate'].max().reset_index()
df_recency['GapDay'] = (current_date - df_recency['OrderDate']).dt.days
df_recency['Recency'] = pd.qcut(df_recency['GapDay'], 4, labels=False, duplicates='drop')
```

And tables are merged:
```python
df_rfm = df_monetary[['ResellerKey','SalesAmount', 'Monetary']]
df_rfm = df_rfm.merge(df_frequency[['ResellerKey','SalesOrderNumber', 'Frequency']], on='ResellerKey')
df_rfm = df_rfm.merge(df_recency[['ResellerKey', 'GapDay', 'Recency']], on='ResellerKey')
```

The histogram below represents the distribution of Recency, Frequency, and Monetary values.

![Figure_1](https://github.com/user-attachments/assets/f6515885-c6d8-42eb-88ce-653e6a9a95bb)

Similarly, the box plot below also represents the same distribution.

![Figure_1](https://github.com/user-attachments/assets/e9a83983-3b64-42c6-bdd1-6c78b9027fc9)





