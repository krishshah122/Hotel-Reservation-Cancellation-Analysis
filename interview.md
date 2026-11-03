# 🎯 Interview Questions – Hotel Booking Cancellation Analysis Project

> A comprehensive interview preparation guide covering all aspects of this data analytics project. Includes questions on **Python, Pandas, EDA, Power BI, SQL, Data Cleaning, Business Insights, KPIs**, and **project-specific scenarios**.

---

## 📌 Section 1: Project Overview Questions

### Q1. Can you explain the objective of your Hotel Booking Cancellation Analysis project?
**Answer:**  
The objective was to analyze hotel booking data to uncover patterns and insights behind booking cancellations. The project aimed to help hotels reduce cancellations, improve revenue, and make data-driven business decisions by identifying key factors like pricing (ADR), lead time, hotel type, customer segments, and seasonal trends that influence cancellation behavior.

### Q2. What dataset did you use? Describe its key features.
**Answer:**  
I used a hotel booking dataset containing **119,390 records** and **32 columns**. Key features include:
- `hotel` — Hotel type (City Hotel / Resort Hotel)
- `is_canceled` — Booking cancellation status (0 = Not Cancelled, 1 = Cancelled)
- `lead_time` — Days between booking and arrival
- `adr` — Average Daily Rate (price per room per night)
- `arrival_date_year`, `arrival_date_month` — Arrival date info
- `market_segment` — How the booking was made (Online TA, Offline TA/TO, Direct, etc.)
- `customer_type` — Type of customer (Transient, Contract, Group, etc.)
- `deposit_type` — No Deposit, Non Refund, Refundable
- `country` — Guest's country of origin (177 countries)
- `reservation_status` — Check-Out, Canceled, No-Show
- `total_of_special_requests` — Number of special requests made

### Q3. What was the overall cancellation rate in the dataset?
**Answer:**  
The overall cancellation rate was approximately **37.04%** — meaning roughly 37 out of every 100 bookings were canceled. This is a significant proportion, indicating cancellations are a major operational challenge for hotels.

### Q4. What tech stack did you use for this project?
**Answer:**  
- **Python** (Pandas, NumPy, Matplotlib, Seaborn) — For data cleaning, analysis, and visualization
- **Jupyter Notebook** — For interactive analysis
- **Power BI** — For building interactive dashboards with KPI cards, slicers, and drill-through pages
- **SQL (PostgreSQL)** — For business-level queries and aggregations
- **Git/GitHub** — For version control

---

## 📊 Section 2: Data Cleaning & Preprocessing Questions

### Q5. What data cleaning steps did you perform?
**Answer:**
1. **Checked for missing values** — `children` had 4 nulls, `country` had 488 nulls, `agent` had 16,340 nulls, `company` had 112,593 nulls
2. **Dropped `company` and `agent` columns** — Due to extreme null counts (94%+ and 13.7% respectively)
3. **Handled remaining nulls** — Filled `children` nulls with 0, `country` nulls with "Unknown"
4. **Removed ADR outliers** — Filtered out rows where `adr >= 5000` (extreme outlier detected via boxplot)
5. **Converted date types** — Converted `reservation_status_date` to datetime using `pd.to_datetime()` with `dayfirst=True`
6. **Created derived features** — Added `month` column from reservation_status_date for monthly analysis

### Q6. Why did you drop `company` and `agent` columns instead of imputing?
**Answer:**  
The `company` column had **112,593 out of 119,390** values missing (94.3%), making any imputation unreliable and meaningless. The `agent` column had **16,340 nulls** (13.7%). Since both columns were IDs (not descriptive features) and wouldn't contribute meaningful patterns to our cancellation analysis, dropping was the cleanest approach.

### Q7. How did you handle outliers in the ADR column?
**Answer:**  
I first plotted a **boxplot** of the `adr` column to visualize the distribution. It revealed extreme outliers — one value was as high as **5400**. I filtered the dataframe to only include records where `adr < 5000`. This removed extreme anomalies while preserving valid high-rate bookings.

### Q8. What does the `dayfirst=True` parameter do in `pd.to_datetime()`?
**Answer:**  
It tells pandas that the date format is in **DD/MM/YYYY** format rather than the American **MM/DD/YYYY** format. The dataset had dates like `1/7/2015` which should be interpreted as **July 1, 2015** (day first), not **January 7, 2015**.

---

## 📈 Section 3: EDA & Visualization Questions

### Q9. What visualizations did you create and why?
**Answer:**  
| # | Visualization | Type | Purpose |
|---|---------------|------|---------|
| 1 | ADR Boxplot | Box Plot | Detect outliers and understand pricing distribution |
| 2 | Reservation Status Count | Bar Chart | Show overall cancellation vs. non-cancellation ratio |
| 3 | Cancellation by Hotel Type | Grouped Bar Chart (countplot) | Compare cancellations between City and Resort hotels |
| 4 | ADR Over Time (City vs Resort) | Line Chart | Compare pricing trends between hotel types |
| 5 | Monthly Reservation Status | Grouped Bar Chart | Identify seasonal cancellation patterns |
| 6 | ADR per Month (Cancelled) | Bar Chart | Link pricing with cancellation by month |
| 7 | Top 10 Countries (Cancellations) | Pie Chart | Identify geographic cancellation hotspots |
| 8 | ADR Over Time (Cancelled vs Not) | Line Chart | Compare pricing patterns between cancelled and confirmed bookings |
| 9 | ADR Over Time (Filtered 2016-2017) | Line Chart | Zoom into key period for clearer trend analysis |

### Q10. What library did you use for visualizations and why?
**Answer:**  
I used **Matplotlib** for basic plots (bar charts, pie charts, line charts, boxplots) and **Seaborn** for statistical visualizations (countplots, barplots with aggregation). Seaborn was chosen because it provides aesthetically pleasing defaults and has built-in support for hue-based grouping, which was essential for comparing cancelled vs. non-cancelled bookings.

### Q11. What did the ADR boxplot reveal?
**Answer:**  
The boxplot revealed that ADR values have a **wide spread** with several extreme outliers. The median ADR was around **94.57**, the IQR ranged from **69.29 to 126.00**, but there were values as high as **5400** — clearly erroneous or luxury outliers. This indicated significant variation in room pricing and that higher-priced bookings might be more prone to cancellations.

### Q12. What insights did the hotel-type comparison reveal?
**Answer:**  
- **City Hotels** had a significantly higher cancellation rate compared to **Resort Hotels**
- Resort Hotels showed a higher proportion of confirmed bookings
- City Hotels: ~41.7% cancellation rate
- Resort Hotels: ~27.8% cancellation rate
- This suggests city hotels receive more short-term/business bookings that are easier to cancel

---

## 🧠 Section 4: Business Insight Questions

### Q13. What were the key insights and conclusions from your analysis?
**Answer:**

1. **City Hotels have higher cancellation rates** — ~41.7% vs ~27.8% for Resort Hotels. Business/short-term bookings are more volatile.

2. **Longer lead times increase cancellations** — The more days between booking and arrival, the higher the probability of cancellation.

3. **Higher ADR correlates with higher cancellations** — Months with higher average daily rates show more cancellations. Price sensitivity drives customer behavior.

4. **Online Travel Agents generate the most bookings** — Online TA is the dominant market segment (~47%), but also contributes significantly to cancellations.

5. **Seasonal patterns exist** — Certain months show peak cancellations. Hotels can apply stricter policies during these periods.

6. **Repeat guests cancel less** — Only ~3.2% of guests are repeat visitors, and they have lower cancellation rates.

7. **Portugal (PRT) leads in cancellations** — A small number of countries contribute to a large proportion of cancellations. Region-specific strategies can help.

8. **Price instability drives cancellations** — Cancelled bookings show higher and more fluctuating ADR values compared to confirmed bookings.

### Q14. What business recommendations would you give based on your findings?
**Answer:**
1. **Implement dynamic yet stable pricing** — Avoid extreme price fluctuations that trigger cancellations
2. **Stricter cancellation policies for long lead-time bookings** — Require deposits or non-refundable rates for bookings made far in advance
3. **Focus retention strategies on City Hotels** — Since they have higher cancellation rates
4. **Region-specific booking policies** — Target high-cancellation countries (PRT, GBR, etc.) with confirmation reminders
5. **Incentivize repeat bookings** — Loyalty programs can reduce cancellation rates
6. **Seasonal planning** — Apply promotional strategies during high-cancellation months
7. **Reduce reliance on OTAs** — Encourage direct bookings which may have lower cancellation rates

### Q15. How does lead time affect cancellations?
**Answer:**  
Lead time (the number of days between booking date and arrival date) has a **strong positive correlation** with cancellations. Average lead time in the dataset is **104 days**, with a maximum of **737 days**. Bookings made very far in advance (high lead time) are significantly more likely to be cancelled because customer plans change, competitive offers emerge, or commitments weaken over time.

---

## 🔑 Section 5: KPI Questions

### Q16. What KPIs did you track in this project?
**Answer:**

| KPI | Value | Purpose |
|-----|-------|---------|
| **Total Bookings** | 119,390 | Volume metric |
| **Cancellation Rate** | ~37.04% | Primary health metric |
| **Average ADR** | $101.83 | Pricing indicator |
| **Average Lead Time** | 104 days | Booking behavior metric |
| **Revenue Lost** | Sum of (ADR × nights) for cancelled bookings | Financial impact |
| **Repeat Guest %** | ~3.19% | Customer loyalty indicator |
| **Top Cancellation Country** | Portugal (PRT) | Geographic insight |
| **Peak Cancellation Month** | Varies by year, but summer months are peak | Seasonal trend |
| **City Hotel Cancel Rate** | ~41.7% | Hotel-type comparison |
| **Resort Hotel Cancel Rate** | ~27.8% | Hotel-type comparison |

### Q17. Why is cancellation rate an important KPI for hotels?
**Answer:**  
Cancellation rate directly impacts:
- **Revenue forecasting** — High cancellations make revenue unpredictable
- **Room inventory management** — Cancelled rooms may remain unsold
- **Operational planning** — Staffing, food preparation, and amenities are planned based on bookings
- **Pricing strategy** — Understanding cancellation patterns helps optimize dynamic pricing
- **Overbooking decisions** — Hotels overbook based on expected cancellation rates

---

## 💻 Section 6: Technical/Tool Questions

### Q18. What Python libraries did you use and for what purpose?
**Answer:**
| Library | Purpose |
|---------|---------|
| `pandas` | Data loading, cleaning, manipulation, aggregation |
| `matplotlib` | Creating bar charts, line charts, pie charts, boxplots |
| `seaborn` | Statistical visualizations with grouping (countplot, barplot) |
| `warnings` | Suppressing deprecation warnings for clean output |

### Q19. How would you write a SQL query to find the hotel type with the highest cancellation rate?
**Answer:**
```sql
SELECT 
    hotel,
    COUNT(*) AS total_bookings,
    SUM(is_canceled) AS total_cancellations,
    ROUND(SUM(is_canceled) * 100.0 / COUNT(*), 2) AS cancellation_rate
FROM hotel_bookings
GROUP BY hotel
ORDER BY cancellation_rate DESC
LIMIT 1;
```

### Q20. Write a SQL query to find the top 5 countries with the most cancellations.
**Answer:**
```sql
SELECT 
    country,
    COUNT(*) AS total_cancellations
FROM hotel_bookings
WHERE is_canceled = 1
GROUP BY country
ORDER BY total_cancellations DESC
LIMIT 5;
```

### Q21. How would you find the average ADR per month for cancelled bookings in SQL?
**Answer:**
```sql
SELECT 
    EXTRACT(MONTH FROM reservation_status_date) AS month,
    ROUND(AVG(adr), 2) AS avg_adr
FROM hotel_bookings
WHERE is_canceled = 1
GROUP BY month
ORDER BY month;
```

### Q22. What is the difference between `value_counts()` and `value_counts(normalize=True)` in pandas?
**Answer:**
- `value_counts()` returns the **absolute count** of each unique value
- `value_counts(normalize=True)` returns the **relative proportion** (percentage as decimal) of each unique value
- Example: If there are 75,166 not-cancelled and 44,224 cancelled:
  - `value_counts()` → `0: 75166, 1: 44224`
  - `value_counts(normalize=True)` → `0: 0.6296, 1: 0.3704`

---

## 📊 Section 7: Power BI Questions

### Q23. What visuals did you use in your Power BI dashboard?
**Answer:**
- **KPI Cards** — For Total Bookings, Cancellation Rate, Revenue Lost, Avg ADR
- **Clustered/Stacked Bar Charts** — For hotel-wise cancellations, monthly trends
- **Donut/Pie Charts** — For market segment distribution, hotel-wise cancellation split
- **Line Charts** — For ADR trends over time
- **Treemap** — For top 10 countries with cancellations
- **Scatter Plot** — For lead time vs cancellation analysis
- **Slicers** — For filtering by hotel type, year, month, customer type

### Q24. What DAX measures did you create?
**Answer:**
```dax
Cancellation Rate = DIVIDE(
    CALCULATE(COUNTROWS(hotel_bookings), hotel_bookings[is_canceled] = 1),
    COUNTROWS(hotel_bookings),
    0
) * 100

Revenue Lost = CALCULATE(
    SUM(hotel_bookings[revenue]),
    hotel_bookings[is_canceled] = 1
)

Repeat Guest Pct = DIVIDE(
    CALCULATE(COUNTROWS(hotel_bookings), hotel_bookings[is_repeated_guest] = 1),
    COUNTROWS(hotel_bookings),
    0
) * 100
```

### Q25. How did you make your Power BI dashboard interactive?
**Answer:**
- Added **Slicers** for filtering (hotel type, year, month, customer type)
- Enabled **cross-filtering** between visuals
- Created **drill-through pages** for country-level analysis
- Used **bookmarks** for quick views (City Hotel vs Resort Hotel)
- Added **tooltips** showing additional metrics on hover
- Used **conditional formatting** (red/green) on cancellation rates

---

## 🧩 Section 8: Scenario-Based Questions

### Q26. If you had to predict cancellations, what ML model would you use?
**Answer:**  
I would use a **classification model** since cancellation is a binary outcome (0 or 1). Recommended approaches:
1. **Logistic Regression** — Good baseline, interpretable coefficients
2. **Random Forest** — Handles non-linear relationships, feature importance
3. **XGBoost/Gradient Boosting** — Best performance for tabular data
4. **Key features**: `lead_time`, `adr`, `hotel`, `deposit_type`, `market_segment`, `previous_cancellations`, `total_of_special_requests`

### Q27. How would you handle class imbalance? (63% vs 37%)
**Answer:**  
The imbalance here is moderate (not extreme), but still:
1. **SMOTE** (Synthetic Minority Over-sampling) — Generate synthetic samples for minority class
2. **Adjust class weights** — Set `class_weight='balanced'` in model parameters
3. **Use appropriate metrics** — F1-score, Precision-Recall curve instead of just accuracy
4. **Stratified sampling** — Ensure train/test splits maintain class proportions

### Q28. What would you do differently if you had to redo this project?
**Answer:**
1. Build a **predictive ML model** for cancellation probability
2. Add **customer segmentation** using clustering (K-Means)
3. Perform **correlation analysis** with a heatmap
4. Create **feature importance analysis** to rank cancellation factors
5. Add **time series forecasting** for future demand prediction
6. Deploy dashboard on **Power BI Service** for real-time access
7. Add **hypothesis testing** to validate statistical significance of findings

### Q29. How would you explain the business impact of this project to a non-technical stakeholder?
**Answer:**  
"Our analysis found that **37% of all hotel bookings get cancelled**, and **City Hotels face 42% cancellations** — meaning nearly half their rooms booked go empty. We identified that **higher room prices**, **longer booking lead times**, and **bookings through online travel agents** are the biggest drivers of cancellations. By implementing **stable pricing**, **deposit requirements for advance bookings**, and **loyalty programs**, hotels could potentially **reduce cancellations by 10-15%**, translating to millions in saved revenue."

### Q30. How does this project demonstrate your data analytics skills?
**Answer:**  
This project showcases end-to-end data analytics proficiency:
1. **Data Collection** — Working with real-world hotel booking data (119K records, 32 features)
2. **Data Cleaning** — Handling missing values, outliers, data type conversions
3. **EDA** — Generating 9+ visualizations to uncover patterns
4. **Statistical Analysis** — Analyzing distributions, correlations, proportions
5. **Business Intelligence** — Creating interactive Power BI dashboards with KPIs
6. **SQL** — Writing business queries for data aggregation
7. **Storytelling** — Converting data insights into actionable business recommendations
8. **Tools** — Python, Pandas, Matplotlib, Seaborn, Power BI, SQL, Git

---

## ⚡ Rapid-Fire Questions

| Question | Answer |
|----------|--------|
| What is EDA? | Exploratory Data Analysis — process of analyzing data to summarize main characteristics |
| What is ADR? | Average Daily Rate — average revenue per paid occupied room |
| Difference between City Hotel and Resort Hotel? | City hotels are urban, business-oriented; Resort hotels are leisure-oriented |
| What is lead time? | Number of days between booking date and arrival date |
| What does `groupby()` do? | Groups data by specified columns for aggregation operations |
| What is a KPI? | Key Performance Indicator — measurable value showing effectiveness |
| What is DAX in Power BI? | Data Analysis Expressions — formula language for calculations in Power BI |
| What is `normalize=True`? | Returns proportions instead of absolute counts |
| What is a slicer in Power BI? | Interactive filter control on the dashboard |
| What is cross-filtering? | When clicking one visual filters other visuals on the same page |

---

> 💡 **Tip**: Practice explaining each answer **verbally** as if in an interview. Focus on **what you did**, **why you did it**, and **what you found**. Interviewers love candidates who can tell a data story.
