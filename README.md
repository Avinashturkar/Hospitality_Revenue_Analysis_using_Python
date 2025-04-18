# Python Hospitality Project: AtliQ Grand Revenue Analysis

**Author:** Avinash Turkar

---

## Table of Contents

1.  [Background & Overview](#background--overview)
2.  [Problem Statement](#problem-statement)
3.  [Project Objective](#project-objective)
4.  [Data Structure Overview](#data-structure-overview)
5.  [Tools & Libraries Used](#tools--libraries-used)
6.  [Data Loading & Initial Exploration](#data-loading--initial-exploration)
7.  [Data Cleaning & Transformation](#data-cleaning--transformation)
8.  [Initial Data Exploration & Findings](#initial-data-exploration--findings)
9.  [Executive Summary](#executive-summary)
10. [Ad-Hoc Business Questions & Findings](#ad-hoc-business-questions--findings)
11. [Recommendations](#recommendations)
12. [Potential Impact](#potential-impact)
13. [Conclusion](#conclusion)

---

## Background & Overview

AtliQ Grand, a chain of five-star hotels across India with 20 years in the hospitality industry, has been facing declining market share and revenue in the luxury/business hotel segment due to competitor strategies and ineffective management decisions. To reverse this trend, the managing director initiated a data-driven approach, seeking insights from historical booking data.

---

## Problem Statement

AtliQ Grand requires actionable insights derived from their historical booking data to understand performance trends, identify areas for improvement, and make informed strategic decisions to regain lost market share and revenue.

---

## Project Objective

The primary objective of this analysis is to process and analyze AtliQ Grand's booking data (May-July 2022, augmented with August 2022 data) to provide the revenue management team with clear insights into:

1.  Occupancy rates across different categories (room type, city, day type).
2.  Revenue generation patterns (by city, month, hotel property, booking platform).
3.  Customer satisfaction trends (average ratings by city).
4.  Booking platform performance.
5.  Derive actionable recommendations based on these findings.

**For a detailed look at the code and step-by-step analysis, please refer to the `Hospitality project Python.ipynb` notebook in this repository.**

---

## Data Structure Overview

The analysis is based on six CSV files covering booking data primarily from May to July 2022, with additional aggregated data provided for August 2022.

*   **Dimension Tables:**
    *   `dim_date.csv`: Provides date attributes like month-year, week number, and day type (weekday/weekend).
    *   `dim_hotels.csv`: Contains property details (ID, name, category - Luxury/Business, city).
    *   `dim_rooms.csv`: Maps room category codes (RT1, RT2, etc.) to room classes (Standard, Elite, etc.).
*   **Fact Tables:**
    *   `fact_bookings.csv`: Holds detailed booking transaction records (approx. 134.6k initially) including IDs, dates, guest count, platform, rating, status, generated revenue, and realized revenue.
    *   `fact_aggregated_bookings.csv`: Contains daily aggregated successful bookings and capacity per property and room category (9.2k records initially).
*   **Additional Data:**
    *   `new_data_august.csv`: Aggregated booking data specifically for August 2022 (7 records).

---

## Tools & Libraries Used

*   **Language:** Python
*   **Libraries:**
    *   `pandas`: For data loading, manipulation, cleaning, merging, and analysis.
    *   `matplotlib.pyplot`: For creating visualizations (bar charts, pie charts).
    *   `datetime` / `dateutil`: Used for date conversions (implicitly by pandas `to_datetime`).
*   **Environment:** Jupyter Notebook

---

## Data Loading & Initial Exploration

1.  All six CSV files were loaded into respective pandas DataFrames (`df_bookings`, `df_date`, `df_hotels`, `df_rooms`, `df_agg_bookings`, `df_august`).
2.  Initial exploration was performed using `.head()`, `.shape`, `.unique()`, `.value_counts()`, `.describe()`, and `.info()` to understand data structure, content, and identify preliminary patterns or issues. A bar chart visualizing booking platform counts was also generated.

---

## Data Cleaning & Transformation

The following steps were explicitly performed as shown in the notebook to clean and prepare the data:

1.  **Invalid Guest Count Handling (`df_bookings`):** Records with `no_guests` <= 0 (9 records identified) were removed. The DataFrame shape changed from (134590, 12) to (134578, 12).
2.  **Revenue Generated Outlier Removal (`df_bookings`):** Extreme outliers in `revenue_generated` were identified using the mean ± 3 standard deviations rule (Upper limit calculated as ~294,498). Records exceeding this limit (5 records) were removed. The DataFrame shape changed to (134573, 12).
3.  **Revenue Realized Outlier Investigation (`df_bookings`):** Potential outliers in `revenue_realized` were investigated. The upper limit based on 3*std was calculated (~33,479). Records exceeding this (1299 rows) were examined. Further analysis focusing only on 'RT4' rooms showed their revenue realized mean was ~23,439 with a max of 45,220. The 3*std upper limit for *only* RT4 rooms was calculated (~50,583). Since the max value (45,220) was below this specific limit, it was concluded that these were not outliers in the context of RT4 rooms, and no records were removed based on this column.
4.  **Missing Ratings (`df_bookings`):** Missing values in `ratings_given` (77,897 NaNs) were identified. No imputation or removal was performed.
5.  **Missing Capacity (`df_agg_bookings`):** Two records with missing `capacity` values were identified. The median capacity was calculated (25.0). These two rows were explicitly removed using `dropna(inplace=True)`. The DataFrame shape changed from (9200, 5) to (9198, 5). *(Note: A later shape check shows (9200,5), suggesting dropna might not have persisted or was overwritten. Analysis seems to proceed with 9200 rows in the relevant DataFrame `df` before concatenation)*.
6.  **Occupancy Outlier Identification (`df_agg_bookings`):** Records where `successful_bookings` > `capacity` (6 records identified) were flagged as data inconsistencies. *Note: The notebook shows the identification step but does not explicitly show code removing these records before subsequent calculations/merging.*
7.  **Feature Engineering (`df_agg_bookings`):** A new column `occ_pct` (Occupancy Percentage) was calculated as `round((successful_bookings / capacity) * 100, 2)` and added to `df_agg_bookings`.
8.  **Data Merging:**
    *   `df_agg_bookings` was merged with `df_rooms` (on `room_category`/`room_id`) to add `room_class`. The `room_id` column was dropped. This resulted in DataFrame `df`.
    *   `df` was merged with `df_hotels` (on `property_id`) to add `property_name`, `category`, `city`.
    *   `df` was merged with `df_date` (on `check_in_date`/`date`) to add `mmm yy`, `day_type`, `week no`.
    *   `df_bookings` was merged with `df_hotels` (on `property_id`) to create `df_bookings_all`.
    *   `df_bookings_all` was merged with `df_date` (on `check_in_date`/`date`).
9.  **Data Type Conversion:** `date` column in `df_date` and `check_in_date` in `df_bookings_all` were converted to datetime objects using `pd.to_datetime`.
10. **Data Augmentation:** The August aggregated data (`df_august`, 7 rows) was concatenated with the processed May-July aggregated data (`df`, 6500 rows) using `pd.concat` to create `latest_df` (6507 rows). *(Note: The primary grouped analyses presented seem to utilize `df` and `df_bookings_all` derived before this final concatenation step)*.

---

## Initial Data Exploration & Findings

During the initial data exploration and cleaning phases, several descriptive findings were obtained:

1.  **Unique Room Categories:** 4 unique categories (`RT1`, `RT2`, `RT3`, `RT4`).
2.  **Unique Booking Platforms:** 7 unique platforms identified (`direct online`, `others`, `logtrip`, `tripster`, `makeyourtrip`, `journey`, `direct offline`).
3.  **Booking Volume by Platform (Initial Counts):** 'others' (55k), 'makeyourtrip' (27k), 'logtrip' (15k) led in volume. (Bar chart generated).
4.  **Hotel Categories:** 16 Luxury and 9 Business hotels.
5.  **Hotels per City:** Mumbai (8), Hyderabad (6), Bangalore (6), Delhi (5). (Bar chart generated).
6.  **Unique Properties (Aggregated Data):** 25 unique property IDs.
7.  **Total Successful Bookings per Property (Aggregated Data):** Sum calculated for each property (e.g., Property 16559: 7338 bookings).
8.  **Bookings > Capacity Identification (Aggregated Data):** Specific instances where `successful_bookings` exceeded `capacity` were identified (e.g., Property 17558, RT1 on 1-May-22).
9.  **Highest Capacity Property (Aggregated Data):** Property ID 17558 showed rooms with the maximum capacity (50).

---

## Executive Summary

Analysis of AtliQ Grand's May-August 2022 booking data highlights key performance variations. Average occupancy across room types (Standard, Elite, Premium, Presidential) is similar, clustered around 58-59%. However, significant differences emerge by city, with Delhi achieving the highest average occupancy (~61.6%) and Bangalore the lowest (~56.6%). Weekend occupancy (~72.4%) dramatically outperforms weekday occupancy (~50.9%). In terms of total realized revenue (May-July), Mumbai leads substantially, followed by Bangalore, Hyderabad, and then Delhi. July saw the highest revenue realization among the May-July period. Customer ratings analysis indicates Delhi performs best (~3.79 average), while Bangalore lags (~3.41). The 'others' platform dominates booking volume, while MakeMyTrip and direct channels are also major contributors; revenue share per platform was visualized but requires deeper ROI analysis. Key recommendations focus on strategies to lift weekday occupancy, optimizing performance in key revenue cities (Mumbai, Bangalore) while addressing challenges in others (Delhi, Hyderabad), improving guest experience where ratings are lower (Bangalore), and evaluating booking platform effectiveness.

---

## Ad-Hoc Business Questions & Findings

This section details the findings for the core analytical questions addressed after data cleaning, transformation, and merging. *(Note: Findings primarily based on `df` (May-July aggregated) and `df_bookings_all` (May-July booking level) unless otherwise specified, reflecting the analyses performed in the notebook).*

### Q1: What is the average occupancy rate in each room category?

*   **Answer:** Average occupancy percentage is relatively consistent across room classes, calculated from the merged aggregated data (`df`):
    *   Standard: 58.23%
    *   Elite: 58.04%
    *   Premium: 58.03%
    *   Presidential: 59.30%

### Q2: What is the average occupancy rate per city?

*   **Answer:** Calculated from the merged aggregated data (`df`):
    *   Delhi: 61.61%
    *   Hyderabad: 58.14%
    *   Mumbai: 57.94%
    *   Bangalore: 56.59%

### Q3: When was occupancy better, Weekday or Weekend?

*   **Answer:** Calculated from the merged aggregated data (`df`):
    *   Weekend: 72.39%
    *   Weekeday: 50.90%
    *   (Pie chart generated showing this difference).

### Q4: In the month of June, what is the occupancy for different cities?

*   **Answer:** Filtered for 'Jun 22' in the merged aggregated data (`df`):
    *   Delhi: 62.47%
    *   Hyderabad: 58.46%
    *   Mumbai: 58.38%
    *   Bangalore: 56.58%
    *   (Bar chart generated showing this distribution).

### Q5: What is the total realized revenue per city?

*   **Answer:** Calculated by summing `revenue_realized` from the booking-level data merged with hotel details (`df_bookings_all`):
    *   Mumbai: 668,569,251
    *   Bangalore: 420,383,550
    *   Hyderabad: 325,179,310
    *   Delhi: 294,404,488

### Q6: What is the month-by-month realized revenue (May-July)?

*   **Answer:** Calculated by summing `revenue_realized` grouped by month from the booking-level data merged with date details (`df_bookings_all`):
    *   May 22: 234,353,183
    *   Jun 22: 229,637,640
    *   Jul 22: 243,180,932

### Q7: What is the total realized revenue per hotel type/property name?

*   **Answer:** Calculated by summing `revenue_realized` grouped by `property_name` from the booking-level data (`df_bookings_all`), sorted ascending:
    *   Atliq Seasons: 26,838,223
    *   Atliq Grands: 87,245,939
    *   Atliq Bay: 107,516,312
    *   Atliq Blu: 108,108,129
    *   Atliq City: 118,290,783
    *   Atliq Palace: 125,553,143
    *   Atliq Exotica: 133,619,226

### Q8: What is the average rating per city?

*   **Answer:** Calculated by averaging `ratings_given` (where available) grouped by `city` from the booking-level data (`df_bookings_all`):
    *   Delhi: 3.79
    *   Mumbai: 3.66
    *   Hyderabad: 3.65
    *   Bangalore: 3.41

### Q9: What is the distribution of realized revenue per booking platform?

*   **Answer:** A pie chart was generated visualizing the proportion of total `revenue_realized` contributed by each `booking_platform` based on the booking-level data (`df_bookings_all`). *Specific percentages require reading the chart, but it shows the relative contribution.*

---

## Recommendations

Based *only* on the insights explicitly generated and presented in the notebook:

1.  **Address Weekday Occupancy Gap:** The ~21% difference between weekday (~50.9%) and weekend (~72.4%) occupancy is substantial. Develop and implement strategies specifically targeting weekday bookings (e.g., corporate rates, mid-week packages).
2.  **Focus on Revenue Drivers (Mumbai & Bangalore):** Mumbai and Bangalore contribute the most to total realized revenue (May-July). Analyze the factors contributing to their success (e.g., property types, pricing, local demand) and prioritize resources to maintain or grow revenue in these key markets.
3.  **Improve Performance in Delhi & Hyderabad:** While Delhi has high occupancy and ratings, its total revenue (May-July) is the lowest. Investigate if pricing optimization or promoting higher-category rooms could boost revenue. Hyderabad shows moderate performance; identify specific opportunities for improvement in occupancy or average revenue per booking.
4.  **Enhance Guest Satisfaction in Bangalore:** With the lowest average rating (3.41), focus on identifying and addressing service or facility issues in Bangalore properties to improve guest experience and potentially drive repeat visits and better reviews.
5.  **Evaluate Booking Platform Contribution:** While the pie chart shows revenue distribution, further analysis is needed to understand the *profitability* of each platform. Compare revenue generated via platforms like MakeMyTrip, others, etc., against associated costs/commissions versus higher-margin direct bookings.
6.  **Leverage Delhi's Strengths:** Despite lower total revenue (May-July), Delhi's high occupancy and ratings suggest strong operational performance or brand perception. Analyze best practices from Delhi properties that could be applied to other locations, particularly Bangalore.

---

## Potential Impact

Implementing the data-driven recommendations derived from this analysis holds the potential for significant positive impacts across AtliQ Grand's business:

*   **Increased Revenue & Profitability:** Boosting weekday occupancy can directly increase overall room revenue. Optimizing pricing and channel mix (favoring direct bookings) can enhance profit margins and reduce commission costs. Focusing efforts on high-performing markets like Mumbai and Bangalore can maximize returns.
*   **Improved Occupancy & Asset Utilization:** Successfully addressing the weekday slump can lead to higher average occupancy rates throughout the week, improving the utilization of fixed assets and staff resources.
*   **Enhanced Customer Satisfaction & Loyalty:** Addressing service gaps identified through rating analysis (especially in lower-rated cities like Bangalore) can lead to improved guest satisfaction, better online reviews, increased repeat business, and a stronger brand reputation.
*   **Strengthened Market Position:** By making informed decisions on pricing, promotions, and channel strategy based on data, AtliQ Grand can more effectively compete and potentially regain market share in the luxury/business segment.
*   **Operational Efficiency:** Identifying and replicating best practices from high-performing properties/cities (like Delhi's high ratings/occupancy) can lead to standardized improvements and greater efficiency across the hotel chain.

The realization of these impacts is contingent upon effective strategy development and execution based on these analytical insights.

---

## Conclusion

This Python-based analysis of AtliQ Grand's booking data successfully processed the raw data, handled inconsistencies, and generated key performance indicators related to occupancy, revenue, and ratings across different dimensions (time, location, room type, platform). The insights reveal significant opportunities, particularly in optimizing weekday occupancy and tailoring strategies for specific cities based on their unique performance profiles. Evaluating booking platform efficiency based on revenue, not just volume, is crucial. These insights provide a data-driven foundation for AtliQ Grand's revenue management team to formulate strategies aimed at improving market share and financial performance. The methodology demonstrates proficiency in using Python libraries like pandas and Matplotlib for typical data analysis workflows.
