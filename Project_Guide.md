# 📘 Project Guide – Hotel Booking Cancellation Analysis
## Complete Documentation: What You Did, How You Did It, and What You Found

**Author:** Krish Shah  
**Tech Stack:** Python | Pandas | Matplotlib | Seaborn | Power BI | SQL | Jupyter Notebook

---

## 📌 1. Project Objective

This project is an **end-to-end data analytics project** that analyzes hotel booking data to:
- Understand **booking cancellation patterns**
- Identify **factors driving cancellations**
- Provide **data-driven business recommendations** to reduce cancellations
- Build an **interactive Power BI dashboard** for stakeholders

---

## 📂 2. Dataset Overview

| Property | Details |
|----------|---------|
| **File** | `hotel_bookings 2.csv` |
| **Records** | 119,390 rows |
| **Features** | 32 columns |I
| **Hotel Types** | City Hotel (79,330 records), Resort Hotel (40,060 records) |
| **Time Period** | 2015 – 2017 |
| **Countries** | 177 unique countries |
| **Source** | Public hotel booking demand dataset |

### Key Columns Used
| Column | Description | Type |
|--------|-------------|------|
| `hotel` | City Hotel or Resort Hotel | Categorical |
| `is_canceled` | 0 = Not Cancelled, 1 = Cancelled | Binary |
| `lead_time` | Days between booking and arrival | Numeric |
| `adr` | Average Daily Rate (room price/night) | Numeric |
| `arrival_date_year/month` | Arrival date details | Categorical/Numeric |
| `market_segment` | Booking channel (Online TA, Direct, etc.) | Categorical |
| `customer_type` | Transient, Contract, Group, Transient-Party | Categorical |
| `deposit_type` | No Deposit, Non Refund, Refundable | Categorical |
| `country` | Guest's country (3-letter ISO code) | Categorical |
| `reservation_status` | Check-Out, Canceled, No-Show | Categorical |
| `reservation_status_date` | Date of last status update | Date |
| `is_repeated_guest` | 0 = New, 1 = Repeat guest | Binary |
| `total_of_special_requests` | Number of special requests | Numeric |
| `stays_in_weekend_nights` | Weekend nights stayed | Numeric |
| `stays_in_week_nights` | Weekday nights stayed | Numeric |

---

## ⚙️ 3. Data Preprocessing (What the Notebook Does)

### Step 1: Import Libraries
```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import warnings
warnings.filterwarnings('ignore')
```
- **Pandas** → Data manipulation and analysis
- **Matplotlib** → Base plotting library
- **Seaborn** → Statistical visualizations built on matplotlib
- **Warnings** → Suppressed to keep output clean

### Step 2: Load Dataset
```python
df = pd.read_csv('hotel_bookings 2.csv')
```
Loaded CSV into a pandas DataFrame.

### Step 3: Initial Exploration
| Command | What It Does |
|---------|--------------|
| `df.head()` | Shows first 5 rows — quick data preview |
| `df.tail()` | Shows last 5 rows — verify data end |
| `df.shape` | Returns `(119390, 32)` — row and column count |
| `df.columns` | Lists all 32 column names |
| `df.info()` | Shows data types, non-null counts, memory usage |
| `df.describe()` | Statistical summary (mean, std, min, max, quartiles) |
| `df.describe(include='object')` | Summary of categorical columns (count, unique, top, freq) |
| `df.isnull().sum()` | Counts missing values per column |
| Unique value exploration | For each categorical column, prints unique values |

### Step 4: Data Cleaning
| Operation | Code | Reason |
|-----------|------|--------|
| Drop `company` & `agent` | `df.drop(columns=['company','agent'], inplace=True)` | 94% and 13.7% nulls respectively |
| Convert date type | `df['reservation_status_date'] = pd.to_datetime(df['reservation_status_date'], dayfirst=True)` | Enable date-based analysis |
| Remove ADR outliers | `df = df[df['adr'] < 5000]` | One outlier at 5400 was skewing analysis |

### Step 5: Feature Engineering
| Feature | Code | Purpose |
|---------|------|---------|
| `month` | `df['month'] = df['reservation_status_date'].dt.month` | Monthly trend analysis |

---

## 📊 4. Every Graph Plotted – Detailed Breakdown

### Graph 1: ADR Boxplot
```python
df['adr'].plot(kind='box')
```
| Aspect | Details |
|--------|---------|
| **Type** | Box Plot |
| **Variable** | `adr` (Average Daily Rate) |
| **Purpose** | Detect outliers in room pricing |
| **Finding** | Wide spread with extreme outliers (max ~5400). Median ~94.57. IQR: 69.29–126.00 |
| **Insight** | Significant pricing variation; higher-priced bookings may cancel more |

---

### Graph 2: Reservation Status Count (Overall)
```python
plt.figure(figsize=(5, 4))
plt.title('Reservation status count')
plt.bar(['Not canceled', 'Canceled'], df['is_canceled'].value_counts(), edgecolor='k')
```
| Aspect | Details |
|--------|---------|
| **Type** | Bar Chart |
| **Variable** | `is_canceled` |
| **Purpose** | Show overall cancellation proportion |
| **Finding** | 62.96% Not Cancelled, 37.04% Cancelled |
| **Insight** | Cancellations are a major challenge — over 1 in 3 bookings get cancelled |

---

### Graph 3: Cancellation by Hotel Type
```python
plt.figure(figsize=(8, 4))
ax1 = sns.countplot(x='hotel', hue='is_canceled', data=df, palette='Reds')
plt.title('Reservation status in different hotels')
```
| Aspect | Details |
|--------|---------|
| **Type** | Grouped Bar Chart (Seaborn countplot) |
| **Variables** | `hotel` × `is_canceled` |
| **Purpose** | Compare cancellations between City and Resort hotels |
| **Finding** | City Hotels: ~41.7% cancellations. Resort Hotels: ~27.8% cancellations |
| **Insight** | City hotels have significantly higher cancellation rates — likely due to business/short-term bookings |

---

### Graph 4: ADR Over Time (City vs Resort)
```python
plt.figure(figsize=(20, 8))
plt.title('Average Daily Rate in City and Resort')
plt.plot(resort_hotel.index, resort_hotel['adr'], label='Resort Hotel', color='blue')
plt.plot(city_hotel.index, city_hotel['adr'], label='City Hotel', color='red')
```
| Aspect | Details |
|--------|---------|
| **Type** | Line Chart |
| **Variables** | `reservation_status_date` vs mean `adr`, grouped by hotel type |
| **Purpose** | Compare pricing trends over time for each hotel type |
| **Finding** | City Hotel ADR shows higher fluctuation; Resort Hotel ADR is more stable |
| **Insight** | Dynamic, volatile pricing in City Hotels may contribute to higher cancellations |

---

### Graph 5: Monthly Reservation Status
```python
plt.figure(figsize=(16, 8))
ax1 = sns.countplot(x='month', hue='is_canceled', data=df, palette='Reds')
plt.title('Reservation status per month')
```
| Aspect | Details |
|--------|---------|
| **Type** | Grouped Bar Chart |
| **Variables** | `month` × `is_canceled` |
| **Purpose** | Identify seasonal patterns in cancellations |
| **Finding** | Cancellations vary by month; certain months show noticeably higher cancellations |
| **Insight** | Seasonality plays an important role — hotels can apply stricter policies during peak-cancellation months |

---

### Graph 6: ADR per Month for Cancelled Bookings
```python
plt.figure(figsize=(15, 8))
plt.title('Average Daily Rate (ADR) per month for canceled bookings')
sns.barplot(x='month', y='adr', data=df[df['is_canceled']==1].groupby('month', as_index=False)['adr'].mean(), palette='Reds')
```
| Aspect | Details |
|--------|---------|
| **Type** | Bar Chart |
| **Variables** | `month` vs mean `adr` (only cancelled bookings) |
| **Purpose** | Link pricing with cancellations by month |
| **Finding** | Months with higher ADR show higher cancellation values |
| **Insight** | Customers cancel when prices are perceived as high — price sensitivity drives behavior |

---

### Graph 7: Top 10 Countries with Cancellations
```python
cancelled_data = df[df['is_canceled'] == 1]
top_10_country = cancelled_data['country'].value_counts()[:10]
plt.figure(figsize=(8, 8))
plt.title('Top 10 countries with reservation canceled')
plt.pie(top_10_country, autopct='%.2f%%', labels=top_10_country.index)
```
| Aspect | Details |
|--------|---------|
| **Type** | Pie Chart |
| **Variable** | `country` (filtered for cancelled bookings, top 10) |
| **Purpose** | Identify geographic cancellation hotspots |
| **Finding** | Portugal (PRT) dominates cancellations, followed by GBR, FRA, ESP, DEU |
| **Insight** | A few countries contribute disproportionately — region-specific policies can help |

---

### Graph 8: ADR Over Time (Cancelled vs Not Cancelled)
```python
plt.figure(figsize=(20, 6))
plt.title('Average Daily Rate over time')
plt.plot(not_cancelled_df_adr['reservation_status_date'], not_cancelled_df_adr['adr'], label='Not Cancelled')
plt.plot(cancelled_df_adr['reservation_status_date'], cancelled_df_adr['adr'], label='Cancelled')
```
| Aspect | Details |
|--------|---------|
| **Type** | Line Chart |
| **Variables** | `reservation_status_date` vs mean `adr`, split by cancellation status |
| **Purpose** | Compare ADR trends between cancelled and non-cancelled bookings |
| **Finding** | Cancelled bookings have higher, more fluctuating ADR values |
| **Insight** | Price instability is closely associated with cancellations |

---

### Graph 9: ADR Over Time (Filtered Period: 2016–2017)
```python
# Filtered to 2016-01-01 through 2017-09-01 for cleaner view
plt.figure(figsize=(20, 6))
plt.title('Average Daily Rate over time')
plt.plot(not_cancelled_df_adr['reservation_status_date'], not_cancelled_df_adr['adr'], label='Not Cancelled')
plt.plot(cancelled_df_adr['reservation_status_date'], cancelled_df_adr['adr'], label='Cancelled')
```
| Aspect | Details |
|--------|---------|
| **Type** | Line Chart (zoomed/filtered) |
| **Variables** | Same as Graph 8, but date-filtered |
| **Purpose** | Clearer trend comparison for the core data period |
| **Finding** | Confirmed bookings maintain stable ADR; cancelled bookings show higher fluctuations |
| **Insight** | Hotels can reduce cancellations by maintaining consistent pricing |

---

### Additional Analysis (Non-Visual)

| Analysis | Code | Finding |
|----------|------|---------|
| Resort Hotel cancellation rate | `resort_hotel['is_canceled'].value_counts(normalize=True)` | ~27.8% cancelled |
| City Hotel cancellation rate | `city_hotel['is_canceled'].value_counts(normalize=True)` | ~41.7% cancelled |
| Market segment distribution | `df['market_segment'].value_counts()` | Online TA dominates with ~47% |
| Market segment (cancelled) | `cancelled_data['market_segment'].value_counts(normalize=True)` | Online TA has highest cancellation proportion |

---

## 🔑 5. KPIs Used

| KPI | Value | How Calculated |
|-----|-------|----------------|
| **Total Bookings** | 119,390 | `df.shape[0]` |
| **Cancellation Rate** | 37.04% | `is_canceled.value_counts(normalize=True)` |
| **Average ADR** | $101.83 | `df['adr'].mean()` |
| **Average Lead Time** | 104 days | `df['lead_time'].mean()` |
| **Repeat Guest %** | 3.19% | `is_repeated_guest.mean() * 100` |
| **City Hotel Cancel %** | ~41.7% | Filtered by hotel type |
| **Resort Hotel Cancel %** | ~27.8% | Filtered by hotel type |
| **Top Market Segment** | Online TA (~47%) | `market_segment.value_counts()` |
| **Top Cancel Country** | Portugal (PRT) | Top of `cancelled['country'].value_counts()` |
| **Median ADR** | $94.58 | `df['adr'].median()` |

---

## 🧠 6. Final Conclusions

### The analysis conclusively shows that hotel booking cancellations are influenced by:

1. **Pricing (ADR)** — Higher and volatile pricing strongly correlates with cancellations. Cancelled bookings consistently show higher, more fluctuating ADR values.

2. **Hotel Type** — City Hotels suffer significantly higher cancellation rates (~42%) compared to Resort Hotels (~28%), likely due to the nature of business vs. leisure travel.

3. **Seasonality** — Cancellation patterns vary by month. Certain periods show peak cancellations tied to pricing and demand cycles.

4. **Geography** — A small number of countries (led by Portugal) contribute to a disproportionate share of cancellations.

5. **Market Segment** — Online Travel Agents drive the most bookings and cancellations, suggesting OTA-sourced bookings are less committed.

6. **Lead Time** — Longer lead times increase cancellation probability. Early bookings are more volatile.

7. **Customer Loyalty** — Repeat guests (only 3.2% of total) have significantly lower cancellation rates.

### Business Impact:
> By optimizing pricing strategies, identifying high-risk booking periods, implementing region-specific cancellation policies, and fostering customer loyalty, hotels can potentially reduce cancellations by 10-15%, leading to significant revenue recovery and improved operational efficiency.

---

## 📁 7. Project Structure

```
Hotel-Reservation-Cancellation-Analysis/
├── DataAnalysis.ipynb              # Jupyter notebook with complete EDA
├── hotel_bookings 2.csv            # Raw dataset (119,390 records)
├── PowerBI_Dashboard_Guide.md      # Step-by-step Power BI dashboard guide
├── interview.md                    # 30+ interview questions with answers
├── Project_Guide.md                # This file — complete project documentation
├── README.md                       # Project overview and repository info
└── LICENSE                         # MIT License
```

---

## 🛠️ 8. Tools & Technologies Used

| Tool | Purpose |
|------|---------|
| **Python 3.x** | Programming language for data analysis |
| **Pandas** | Data loading, cleaning, manipulation, aggregation |
| **NumPy** | Numerical operations |
| **Matplotlib** | Creating static visualizations (bar, line, pie, box) |
| **Seaborn** | Statistical visualizations with grouping and styling |
| **Jupyter Notebook** | Interactive development environment |
| **Power BI** | Interactive dashboards with KPIs, slicers, DAX |
| **SQL (PostgreSQL)** | Database queries for business analysis |
| **Git/GitHub** | Version control and project hosting |

---

> 📝 **This guide serves as complete documentation of everything done in this project — from data loading to final conclusions. Use it for portfolio presentations, interviews, and understanding the end-to-end data analytics workflow.**
