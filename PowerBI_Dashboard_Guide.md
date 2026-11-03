# 🏨 Power BI Dashboard – Complete Build Guide
## Hotel Booking Cancellation Analysis

---

## 📌 Overview

This guide walks you through building a **professional, interactive Power BI dashboard** from the `hotel_bookings 2.csv` dataset. The dashboard will replicate and extend the analysis done in your `DataAnalysis.ipynb` notebook.

---

## 🔧 STEP 1: Data Import & Preparation

### 1.1 Import the Dataset
1. Open **Power BI Desktop**
2. Click **Home → Get Data → Text/CSV**
3. Browse to `hotel_bookings 2.csv` and click **Open**
4. In the preview window, check the column types → click **Transform Data** (to open Power Query Editor)

### 1.2 Data Cleaning — Using Power Query M Code (Advanced Editor)

Since the table is very wide (32 columns) and you may not be able to scroll to see all columns in the visual editor, **use the Advanced Editor instead**:

#### How to Open Advanced Editor:
1. In **Power Query Editor**, click **Home** tab in the ribbon
2. Click **Advanced Editor** button (top-left area of the ribbon)
3. A code window will open — **select all the existing code and delete it**
4. **Paste the code below** (change the file path on line 3 to match your actual CSV location)
5. Click **Done**
6. Check the preview — you should see the cleaned data with new columns
7. Click **Close & Apply** (top-left) to load the data into Power BI

#### Power Query M Code — Copy & Paste This:

```m
let
    // STEP 1: Load the CSV file (Points to your actual file location)
    Source = Csv.Document(
        File.Contents("C:\Users\kriss\OneDrive\Documents\Desktop\hotelbooking analysis\Hotel-Reservation-Cancellation-Analysis\hotel_bookings 2.csv"),
        [Delimiter=",", Columns=32, Encoding=65001, QuoteStyle=QuoteStyle.Csv]
    ),

    // STEP 2: Promote first row as column headers
    PromotedHeaders = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),

    // STEP 3: Remove 'company' (94% null) and 'agent' (13.7% null) columns upfront
    RemovedUnneededCols = Table.RemoveColumns(PromotedHeaders, {"company", "agent"}, MissingField.Ignore),

    // STEP 4: Clean text placeholders ("NA", "NULL", blanks) BEFORE type conversion to prevent errors
    // Fix 'children' text placeholders
    CleanChildren1 = Table.ReplaceValue(RemovedUnneededCols, "NA", "0", Replacer.ReplaceValue, {"children"}),
    CleanChildren2 = Table.ReplaceValue(CleanChildren1, "NULL", "0", Replacer.ReplaceValue, {"children"}),
    CleanChildren3 = Table.ReplaceValue(CleanChildren2, "", "0", Replacer.ReplaceValue, {"children"}),

    // Fix 'country' text placeholders ("NULL" / blanks -> "Unknown")
    CleanCountry1 = Table.ReplaceValue(CleanChildren3, "NULL", "Unknown", Replacer.ReplaceValue, {"country"}),
    CleanCountry2 = Table.ReplaceValue(CleanCountry1, "", "Unknown", Replacer.ReplaceValue, {"country"}),

    // STEP 5: Set data types with culture "en-GB" (ensures Day/Month/Year dates parse with ZERO errors)
    ChangedTypes = Table.TransformColumnTypes(CleanCountry2, {
        {"hotel", type text},
        {"is_canceled", Int64.Type},
        {"lead_time", Int64.Type},
        {"arrival_date_year", Int64.Type},
        {"arrival_date_month", type text},
        {"arrival_date_week_number", Int64.Type},
        {"arrival_date_day_of_month", Int64.Type},
        {"stays_in_weekend_nights", Int64.Type},
        {"stays_in_week_nights", Int64.Type},
        {"adults", Int64.Type},
        {"children", Int64.Type},
        {"babies", Int64.Type},
        {"meal", type text},
        {"country", type text},
        {"market_segment", type text},
        {"distribution_channel", type text},
        {"is_repeated_guest", Int64.Type},
        {"previous_cancellations", Int64.Type},
        {"previous_bookings_not_canceled", Int64.Type},
        {"reserved_room_type", type text},
        {"assigned_room_type", type text},
        {"booking_changes", Int64.Type},
        {"deposit_type", type text},
        {"days_in_waiting_list", Int64.Type},
        {"customer_type", type text},
        {"adr", type number},
        {"required_car_parking_spaces", Int64.Type},
        {"total_of_special_requests", Int64.Type},
        {"reservation_status", type text},
        {"reservation_status_date", type date}
    }, "en-GB"),

    // STEP 6: Replace any stray errors or nulls in 'children' with 0
    SafetyChildren = Table.ReplaceErrorValues(ChangedTypes, {{"children", 0}}),
    FilledChildren = Table.ReplaceValue(SafetyChildren, null, 0, Replacer.ReplaceValue, {"children"}),

    // STEP 7: Filter out ADR outliers (keep adr < 5000 as done in Python)
    FilteredADR = Table.SelectRows(FilledChildren, each [adr] < 5000),

    // STEP 8: Add 'total_guests' column (adults + children + babies)
    AddedTotalGuests = Table.AddColumn(FilteredADR, "total_guests", each [adults] + [children] + [babies], Int64.Type),

    // STEP 9: Add 'total_nights' column (weekend nights + week nights)
    AddedTotalNights = Table.AddColumn(AddedTotalGuests, "total_nights", each [stays_in_weekend_nights] + [stays_in_week_nights], Int64.Type),

    // STEP 10: Add 'revenue' column (adr × total_nights)
    AddedRevenue = Table.AddColumn(AddedTotalNights, "revenue", each [adr] * [total_nights], type number),

    // STEP 11: Add 'month' column (month number 1-12 needed for Monthly Trend charts)
    AddedMonth = Table.AddColumn(AddedRevenue, "month", each Date.Month([reservation_status_date]), Int64.Type),

    // STEP 12: Add 'month_name' column (e.g. "January", "July")
    AddedMonthName = Table.AddColumn(AddedMonth, "month_name", each Date.MonthName([reservation_status_date]), type text),

    // STEP 13: Remove duplicate rows
    RemovedDuplicates = Table.Distinct(AddedMonthName)

in
    RemovedDuplicates
```

> **⚠️ Important:** On line 4, update the file path to match where your CSV is saved. The path above should already be correct for your setup.

#### What This Code Does (Summary):
| Step | What It Does |
|------|--------------|
| 1-2 | Loads CSV and promotes headers |
| 3 | Removes `company` and `agent` columns upfront |
| 4 | Cleans text null placeholders (`NA`, `NULL`, blanks) in `children` and `country` BEFORE type conversion |
| 5 | Sets correct data types using `"en-GB"` culture (fixes date format `d/M/yyyy` with 0 errors) |
| 6 | Safety checks and fills any remaining nulls/errors in `children` with 0 |
| 7 | Filters out ADR outliers (removes rows with adr ≥ 5000) |
| 8 | Creates `total_guests` = adults + children + babies |
| 9 | Creates `total_nights` = weekend nights + week nights |
| 10 | Creates `revenue` = adr × total_nights |
| 11-12 | Adds `month` (1-12) and `month_name` for monthly trend visuals |
| 13 | Removes duplicate rows |

Click **Close & Apply** when done.

---

## 📐 STEP 2: Create DAX Measures

Go to **Modeling → New Measure** and create these calculated measures:

### KPI Measures

```dax
-- Total Bookings
Total Bookings = COUNTROWS('hotel_bookings 2')

-- Total Cancellations
Total Cancellations = CALCULATE(COUNTROWS('hotel_bookings 2'), 'hotel_bookings 2'[is_canceled] = 1)

-- Cancellation Rate (%)
Cancellation Rate = 
    DIVIDE([Total Cancellations], [Total Bookings], 0) * 100

-- Total Revenue (Estimated)
Total Revenue = SUM('hotel_bookings 2'[revenue])

-- Revenue Lost (from cancellations)
Revenue Lost = 
    CALCULATE(SUM('hotel_bookings 2'[revenue]), 'hotel_bookings 2'[is_canceled] = 1)

-- Average Daily Rate
Avg ADR = AVERAGE('hotel_bookings 2'[adr])

-- Average Lead Time
Avg Lead Time = AVERAGE('hotel_bookings 2'[lead_time])

-- Repeat Guest %
Repeat Guest Pct = 
    DIVIDE(
        CALCULATE(COUNTROWS('hotel_bookings 2'), 'hotel_bookings 2'[is_repeated_guest] = 1),
        [Total Bookings],
        0
    ) * 100

-- Confirmed Bookings
Confirmed Bookings = CALCULATE(COUNTROWS('hotel_bookings 2'), 'hotel_bookings 2'[is_canceled] = 0)

-- Average Nights Stayed
Avg Nights = AVERAGE('hotel_bookings 2'[total_nights])
```

---

## 🎨 STEP 3: Dashboard Layout & Visuals

### Recommended Layout: **3 Pages**

---

### 📄 PAGE 1: Executive Overview

**Title:** `Hotel Booking Cancellation – Executive Overview`

| Visual | Type | Fields | Position |
|--------|------|--------|----------|
| **Total Bookings** | KPI Card | `[Total Bookings]` | Top-left |
| **Cancellation Rate** | KPI Card | `[Cancellation Rate]` | Top-center-left |
| **Total Revenue** | KPI Card | `[Total Revenue]` | Top-center-right |
| **Revenue Lost** | KPI Card | `[Revenue Lost]` | Top-right |
| **Cancellation by Hotel Type** | Clustered Bar Chart | Axis: `hotel`, Values: `[Total Cancellations]`, `[Confirmed Bookings]` | Middle-left |
| **Monthly Booking Trend** | Stacked Bar Chart | X-axis: `month` (from reservation_status_date), Y-axis: Count of rows, Legend: `is_canceled` | Middle-right |
| **Cancellation % by Hotel** | Donut Chart | Legend: `hotel`, Values: `[Total Cancellations]` | Bottom-left |
| **Reservation Status Breakdown** | Pie Chart | Legend: `reservation_status`, Values: Count | Bottom-right |

**Slicers (Sidebar):**
- `hotel` (Dropdown)
- `arrival_date_year` (Dropdown)
- `arrival_date_month` (List/Dropdown)
- `customer_type` (Dropdown)

---

### 📄 PAGE 2: Revenue & Pricing Analysis

**Title:** `Revenue & ADR Analysis`

| Visual | Type | Fields | Position |
|--------|------|--------|----------|
| **Avg ADR** | KPI Card | `[Avg ADR]` | Top-left |
| **Avg Lead Time** | KPI Card | `[Avg Lead Time]` | Top-center |
| **Avg Nights** | KPI Card | `[Avg Nights]` | Top-right |
| **ADR Over Time (Cancelled vs Not Cancelled)** | Line Chart | X-axis: `reservation_status_date`, Y-axis: Average of `adr`, Legend: `is_canceled` | Middle (full width) |
| **ADR by Month (Cancelled)** | Clustered Bar Chart | X-axis: `month`, Y-axis: Average of `adr`, Filter: `is_canceled = 1` | Bottom-left |
| **Revenue by Market Segment** | Horizontal Bar Chart | Y-axis: `market_segment`, X-axis: `[Total Revenue]` | Bottom-right |

---

### 📄 PAGE 3: Customer & Geographic Insights

**Title:** `Customer Segmentation & Geography`

| Visual | Type | Fields | Position |
|--------|------|--------|----------|
| **Repeat Guest %** | KPI Card | `[Repeat Guest Pct]` | Top-left |
| **Top 10 Countries (Cancellations)** | Treemap / Pie | Group: `country`, Values: `[Total Cancellations]`, Top N filter = 10 | Top-right |
| **Market Segment Distribution** | Donut Chart | Legend: `market_segment`, Values: Count | Middle-left |
| **Customer Type Breakdown** | Stacked Bar | X-axis: `customer_type`, Y-axis: Count, Legend: `is_canceled` | Middle-right |
| **Deposit Type Impact** | Clustered Bar | X-axis: `deposit_type`, Y-axis: Count, Legend: `is_canceled` | Bottom-left |
| **Lead Time vs Cancellation** | Scatter Plot | X-axis: `lead_time`, Y-axis: `is_canceled` (avg), Details: `hotel` | Bottom-right |

---

## 🎯 STEP 4: Formatting & Theme

### Color Palette (Recommended)
| Element | Color |
|---------|-------|
| Confirmed Bookings | `#2ECC71` (Green) |
| Cancelled Bookings | `#E74C3C` (Red) |
| City Hotel | `#3498DB` (Blue) |
| Resort Hotel | `#F39C12` (Orange) |
| Background | `#F5F6FA` (Light Gray) |
| Title Font | `#2C3E50` (Dark Blue-Gray) |

### Formatting Tips
1. **Title**: Use consistent font size (20pt for page titles, 12pt for chart titles)
2. **KPI Cards**: Use large font for the value (28-32pt), smaller subtitle (10pt)
3. **Conditional Formatting**: Apply red/green color scales on cancellation rate
4. **Borders**: Remove all visual borders for a clean look
5. **Background**: Use rounded corners and subtle shadows
6. **Data Labels**: Turn on for bar/column charts

---

## 🔗 STEP 5: Interactivity

1. **Cross-Filtering**: Ensure all visuals on the same page cross-filter each other
2. **Drill-through**: Create a drill-through page for country-level details
3. **Bookmarks**: Create bookmarks for "City Hotel View" and "Resort Hotel View"
4. **Tooltips**: Add custom tooltips showing ADR, Lead Time, and Cancellation Rate
5. **Buttons**: Add navigation buttons between pages

---

## 📊 Key KPIs for the Dashboard

| KPI | Formula/Source | Purpose |
|-----|---------------|---------|
| Total Bookings | Count of all rows | Volume indicator |
| Cancellation Rate | Cancelled / Total × 100 | Primary health metric |
| Avg ADR | Mean of `adr` | Pricing insight |
| Revenue Lost | Sum of revenue for cancelled rows | Financial impact |
| Avg Lead Time | Mean of `lead_time` | Booking behavior |
| Repeat Guest % | Repeated guests / Total × 100 | Customer loyalty |
| Top Cancel Country | Mode of `country` where cancelled | Geographic insight |
| Peak Cancel Month | Month with max cancellations | Seasonal trend |

---

## 💾 STEP 6: Publish & Share

1. **Save** your `.pbix` file
2. Click **Home → Publish**
3. Select your **Power BI Workspace**
4. Open **Power BI Service** (app.powerbi.com)
5. Pin visuals to a **shared dashboard**
6. Set up **scheduled refresh** if needed

---

## 🚀 Pro Tips

- Use **What-If Parameters** to simulate "What if we reduce cancellations by X%?"
- Add a **Decomposition Tree** to explore cancellation drivers
- Use **Key Influencers** visual to auto-detect cancellation factors
- Add **bookmarks** for presentation mode with guided navigation
- Export dashboard screenshots for your README/portfolio

---

> **Note:** Since Power BI is a desktop application, I cannot create the `.pbix` file directly. Follow this guide step-by-step in Power BI Desktop to build your complete dashboard. The DAX measures, visual layouts, and formatting guidelines above give you everything you need.
