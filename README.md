# DATS6101_Project1
This is where the code for the DATS6101 Project 1 will go
Using listing prices across all metro areas from Realtor.com, which sources its information from MLS-listed for-sale homes in the USA, questions we want to explore are:
1. How much does the listing price vary based on the number of days it's on the market? If it’s listed for longer will the listing price fluctuate more?
2. How much does the listing prices vary based on its listed metro area? How much does it vary over the average?
3. Does the metro area relate to the number of price increases/reductions made by a realtor? Are there more increases in more densely populated areas?
4. Does the total amount of listings in a metro area fluctuate during the year? Is there an increase during important times (Ex. start of school/summer)?
---
## Claude's suggestions and changes to the questions 
**(If we use AI suggestions, we just have to understand them, put it in our own words in the project)**

Although we are using technically the same questions we came up, with Claude here its:
1. Suggesting changes needed to make them SMART.
2. What Featured Columns would have to be made to answer.
3. What graphs and tests would be best for them.


All four use your **Jun 2023–Aug 2026** window.

### 1. Days on market vs. price variation

**Fix needed:** the data can't follow a single listing over time, so ask this at the metro level.

**SMART version:** From Jun 2023 to Aug 2026, do metros with a longer average `median_days_on_market` have more month-to-month variation in their median listing price?

- **Existing columns:** `cbsa_code`, `month_date_yyyymm`, `median_days_on_market`, `median_listing_price_mm` (the month-over-month price change already in the file; check whether it's stored as a percent or a decimal)
- **New columns:**
  - average days on market for each metro
  - price volatility for each metro (the standard deviation of its monthly price changes)
  - optional: a days-on-market tier (short, medium or long, splitting metros into equal thirds)
- **Tests:**
  - Pearson or Spearman correlation, after a normality check
  - or ANOVA comparing volatility across the three tiers
  - Levene's test if you're comparing spread between tiers
- **Graphs:** a scatterplot of days on market vs. volatility with a fitted line; box plots of volatility by tier

### 2. Price differences across metros

**Fix needed:** comparing about 900 metros one by one will show "they differ", which isn't useful. Group them first.

**SMART version:** From Jun 2023 to Aug 2026, does the average median listing price differ across the four Census regions, and how far above or below the national average does each metro sit?

- **Existing columns:** `cbsa_title` (to get the state), `median_listing_price`, and optionally `median_listing_price_per_square_foot` (adjusts for home size)
- **New columns:**
  - region, from the state abbreviation (use the first state for multi-state metros)
  - average price for each metro
  - % difference from the national average
  - log price, because prices are right-skewed
- **Tests:**
  - ANOVA + Tukey on log price (or Kruskal-Wallis if normality fails) standard deviation and coefficient of variation for each region
  - Levene's test to check whether some regions vary more than others
- **Graphs:**  
  - box plots by region;
  - a histogram of % difference from the national average;
  - a bar chart of the 10 most and 10 least expensive metros

### 3. Price increases and reductions by metro

**Fix needed:** the dataset has no population-density column. `HouseholdRank` measures metro **size** (number of households), not density. Also:

- Use **shares**, not counts. Big metros will always have more price changes simply because they have more listings.
- The data counts listings whose price changed, not what individual realtors did.

**SMART version:** From Jun 2023 to Aug 2026, does the average share of listings with price reductions (or increases) differ by metro size?

- **Existing columns:** `price_reduced_share`, `price_increased_share`, `HouseholdRank`
- **New columns:**
  - size group (quartiles of `HouseholdRank`, or your Top 100 / Bottom 100 groups)
  - average shares for each metro
  - a yes/no flag for whether the metro's price-cut share is above the overall median
- **Tests:**
  - ANOVA + Tukey across size groups (or a t-test for Top vs. Bottom 100)
  - chi-square test of size group × the high-cut flag
  - Spearman correlation between rank and share
- **Graphs:**  
  - box plots of each share by size group;
  - a stacked proportion bar for the chi-square test
**Note:** price increases are rare, so `price_increased_share` will be small and skewed. Check normality before running tests on it.

### 4. Seasonal changes in listings

**Verdict:** the strongest of the four, and the dataset suits it well.

**SMART version:** From Jun 2023 to Aug 2026, do listings vary by season, and are they higher in summer (Jun–Aug) than in winter (Dec–Feb)?

- **Existing columns:** `month_date_yyyymm`, `cbsa_code`, `active_listing_count`, and `new_listing_count` (better for "homes coming onto the market")
- **New columns:**
  - calendar month (1–12)
  - season label, plus a "school start" (Aug–Sep) label if you want to test that period
  - market year (Jun–May), which gives three full years inside your window
  - **indexed listings:** each month's count divided by that metro's average for the same market year, so big metros don't dominate and the overall upward inventory trend doesn't hide the seasonal pattern
- **Tests:**
  - paired t-test of summer vs. winter within each metro (the cleanest option, because each metro is compared with itself)
  - or ANOVA + Tukey across seasons or months
- **Graphs:** ;
  - a line chart of monthly listings across the whole window (the three yearly cycles should show clearly)
  - box plots of indexed listings by month

**How the set covers the rubric:**

- Q1 → correlation
- Q2 → ANOVA and measures of variance
- Q3 → chi-square
- Q4 → t-test
---
## Column Defintions
(We will change Claude's definitions to our own which are the featured columns. Existing columns are Realtor's definition.)

**Question key:** 
* Q1 = days on market vs. price variation,
* Q2 = price differences by region,
* Q3 = price increases/reductions by metro size,
* Q4 = seasonal changes in listings.

**Existing columns definitions:**

* **month_date_yyyymm**: The year and month of the observation, written as YYYYMM (e.g., 202308 = Aug 2023). Used to limit the data to Jun 2023–Aug 2026. (Q1, Q2, Q3, Q4)
* **cbsa_code**: Unique ID number for each metro area (Core-Based Statistical Area). Used to group rows by metro. (Q1, Q2, Q3, Q4)
* **cbsa_title**: Name of the metro area plus its state abbreviation(s), e.g., "Austin-Round Rock, TX". (Q2, plus labels for all)
* **quality_flag**: Marks months where Realtor.com notes the metro's data may be unreliable. Used to remove those rows. (Q1, Q2, Q3, Q4)
* **HouseholdRank**: The metro's rank by number of households, where 1 = most households. Measures metro size, not density. (Q3)
* **median_days_on_market**: Median number of days listings in the metro spent on the market that month. (Q1)
* **median_listing_price_mm**: Month-over-month change in the metro's median listing price. Check whether it's stored as a decimal or a percent. (Q1)
* **median_listing_price**: Median asking price of homes listed for sale in the metro that month. (Q2)
* **median_listing_price_per_square_foot**: Median asking price per square foot. Adjusts for differences in home size. (Q2, optional)
* **price_reduced_share**: Share of the metro's active listings that had their price lowered that month. (Q3)
* **price_increased_share**: Share of the metro's active listings that had their price raised that month. (Q3)
* **active_listing_count**: Number of homes for sale in the metro that month, not counting homes under contract (pending). (Q4)
* **new_listing_count**: Number of homes newly listed for sale in the metro that month. (Q4)

**Featured columns definitions:**

*Q1 Featured Columns used:*
* **avg_days_on_market**: Each metro's average `median_days_on_market` from Jun 2023 to Aug 2026. (Q1)
* **price_volatility**: Standard deviation of each metro's monthly price changes (`median_listing_price_mm`) over the window. A higher value means prices swing more from month to month. (Q1)
* **days_on_market_tier**: Short, Medium or Long, made by splitting metros into three equal groups based on `avg_dom`. (Q1)

*Q2 Featured Columns used:*
* **region**: Census region (Northeast, Midwest, South or West), taken from the first state abbreviation in `cbsa_title`. (Q2)
* **avg_price**: Each metro's average `median_listing_price` from Jun 2023 to Aug 2026. (Q2)
* **log_avg_price**: Natural log of `avg_price`, used to reduce right skew before the ANOVA. (Q2)
* **pct_diff_national**: How far a metro's `avg_price` is above or below the national average, as a percent: (metro average − national average) ÷ national average × 100. (Q2)
* **avg_price_sqft**: Each metro's average `median_listing_price_per_square_foot` over the window. (Q2, optional)

*Q3 Featured Columns used:*
* **size_group**: Metro size group based on `HouseholdRank`, either quartiles or Top 100 / Bottom 100. (Q3)
* **avg_reduced_share**: Each metro's average `price_reduced_share` over the window. (Q3)
* **avg_increased_share**: Each metro's average `price_increased_share` over the window. (Q3)
* **high_cut**: Yes/No, whether the metro's `avg_reduced_share` is above the median of all metros. Used for the chi-square test. (Q3)

*Q4 Featured Columns used:*
* **month_num**: Calendar month (1–12), taken from `month_date_yyyymm`. (Q4)
* **season**: Winter (Dec–Feb), Spring (Mar–May), Summer (Jun–Aug) or Fall (Sep–Nov), based on `month_num`. (Q4)
* **school_start**: Yes/No, whether the month is August or September. (Q4, optional)
* **market_year**: 12-month period running June to May: MY1 = Jun 2023–May 2024, MY2 = Jun 2024–May 2025, MY3 = Jun 2025–May 2026. Leave Jun–Aug 2026 out of Q4, because a summer-only partial year would skew the index. (Q4)
* **indexed_listings**: A metro's `active_listing_count` for the month divided by its average for that `market_year`. For example, 1.10 means 10% above that metro's usual level. This lets big and small metros be compared fairly. Make the same column for `new_listing_count` if you use it. (Q4)

