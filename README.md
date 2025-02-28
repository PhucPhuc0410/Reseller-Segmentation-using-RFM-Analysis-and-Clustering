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

However, due to the presence of outliers, we need to remove them to prevent distortion of the mean and variance and to avoid obscuring the main trends in the data.
```python
R_Q1 = df_rfm['GapDay'].quantile(0.25)
R_Q3 = df_rfm['GapDay'].quantile(0.75)
R_IQR = R_Q3 - R_Q1
df_R_outliers = df_rfm[(df_rfm['GapDay'] > (R_Q3 + 1.5 * R_IQR)) | (df_rfm['GapDay'] < (R_Q1 - 1.5 * R_IQR))].copy()
```

And tables are merged:
```python
df_non_outliers = df_rfm[(~df_rfm.index.isin(df_M_outliers.index))
                         & (~df_rfm.index.isin(df_F_outliers.index))
                         & (~df_rfm.index.isin(df_R_outliers.index))]
```

![Figure_1](https://github.com/user-attachments/assets/f4b92fa8-32f1-485c-8c80-06b6e826008c)

When features have different units or value ranges, some may dominate the model more than others. Standard Scaling ensures that all features have equal weight. Therefore, Standard Scaling (variance normalization) is essential in this case.
```python
scaler = StandardScaler()
scaled_data = scaler.fit_transform(df_non_outliers[['SalesAmount', 'SalesOrderNumber', 'GapDay']])
df_scaled_data = pd.DataFrame(scaled_data, index=df_non_outliers.index, columns=('SalesAmount','SalesOrderNumber','GapDay'))
print(df_scaled_data)
```

![Figure_1](https://github.com/user-attachments/assets/ed2c3780-0b4a-4ed8-b9e9-ee4e62f004f5)

After preprocessing, we apply K-Means clustering to categorize resellers based on RFM scores.
```python
k_values = list(range(2, 11))
inertia = []
silhouette_scores = []

for k in k_values:
    kmeans = KMeans(n_clusters=k, max_iter=1000, random_state=42)
    cluster_labels = kmeans.fit_predict(df_scaled_data)
    sil_score = silhouette_score(df_scaled_data, cluster_labels)

    silhouette_scores.append(sil_score)
    inertia.append(kmeans.inertia_)
```

![Figure_1](https://github.com/user-attachments/assets/30314f9d-8617-4147-a6b2-69eb9053e3c8)

Through K-Means Inertia and Silhouette Scores, the best selected 𝑘 is 4.
```python
kmeans = KMeans(n_clusters=4, max_iter=1000, random_state=42)
cluster_labels = kmeans.fit_predict(df_scaled_data)

df_non_outliers = df_non_outliers.copy()
df_non_outliers['Cluster'] = cluster_labels
```

![Figure_1](https://github.com/user-attachments/assets/87b25f56-e22d-4142-930a-6ae2c2291f59)

If you find this project useful, feel free to ⭐. Your support will be my super motivation ❤️.

---

## References

- [Hands On Data Science Project: Understand Customers with KMeans Clustering in Python](https://www.youtube.com/watch?v=afPJeQuVeuY)
- [Visualizing KMeans Clustering with Python](https://www.youtube.com/watch?v=-GoYI746kEY)

  ---

📌 **Author:** Nguyễn Hoàng Gia Phúc  

📧 **Contact:** nguyenhoanggiaphucwork@gmail.com

🔗 **LinkedIn:** [Nguyen Hoang Gia Phuc](https://www.linkedin.com/in/nguyenhoanggiaphuc)






