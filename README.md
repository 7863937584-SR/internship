# SwiftKart E-Commerce Customer Behavior & Delivery Analytics

## Project Overview

This project is a foundational data analytics practicum for **SwiftKart**, a multi-category e-commerce platform operating across Tier 1, Tier 2, and Tier 3 Indian markets.

The project analyzes end-to-end order data to understand three major business challenges:

1. **Unit Economics** — increasing product returns and their impact on net realization.
2. **Fulfillment Friction** — delivery delays and logistics bottlenecks that can affect customer satisfaction.
3. **Margin Dilution** — heavy discounting and whether deep discounts are associated with higher-risk returned transactions.

The analysis is performed using **Python, NumPy, Pandas, and Matplotlib**. No machine-learning algorithms are used. The project relies on data cleaning, vectorized numerical operations, aggregation, business rules, and visualization.

---

## Business Objectives

The analysis aims to answer:

- Which product categories generate the most net revenue?
- How much do discounts reduce realized revenue?
- Which city tier has the highest severe-delay rate?
- How does fulfillment performance relate to customer ratings?
- Are heavily discounted returned orders a high-risk segment?
- Which transactions can be classified as loyal and satisfied?

---

## Dataset

### Input file

```text
ecommerce_orders.csv
```

The raw dataset contains these 11 operational attributes:

| Column | Description |
|---|---|
| `Order_ID` | Unique platform transaction identifier |
| `Customer_ID` | Unique consumer account identifier |
| `Category` | Product category |
| `Payment_Method` | Checkout/payment method |
| `Order_Value` | Gross order value before discounts, in INR |
| `Discount_Percent` | Discount applied at checkout |
| `Est_Delivery_Days` | Promised delivery turnaround |
| `Actual_Delivery_Days` | Actual delivery time |
| `Customer_Rating` | Customer rating from 1 to 5 |
| `Returned` | Whether the order was returned |
| `City_Tier` | Destination classification: Tier 1, Tier 2, or Tier 3 |

The raw dataset intentionally contains data-quality issues such as duplicate records and missing order values. These are handled during the cleaning stage.

---

# Analysis Workflow

## Section A — Data Inspection & Cleaning

### A1. Load and Validate

The CSV is loaded using Pandas.

The following checks are performed:

- Dataset dimensions using `df.shape`
- Column data types using `df.dtypes`
- Missing values using `df.isnull().sum()`

### A2. Primary Key Deduplication

`Order_ID` is treated as the primary transaction identifier.

Duplicate records are removed while retaining the first occurrence:

```python
df.drop_duplicates(subset="Order_ID", keep="first")
```

The expected cleaned transaction count is **1,000 records**.

### A3. Payment Method Standardization

The `Payment_Method` column is standardized by:

- Removing leading/trailing whitespace.
- Normalizing casing.
- Converting lowercase `upi` to `UPI`.
- Standardizing the other payment-method labels.

### A4. Category-Wise Median Imputation

Missing `Order_Value` values are imputed using the median value of the corresponding product category:

```python
df.groupby("Category")["Order_Value"].transform("median")
```

Category-level median imputation is preferred because order values are skewed and may contain high-value transactions. The median is less affected by extreme values than the mean and preserves category-specific spending behavior.

### A5. Customer Rating Handling

Missing `Customer_Rating` values are filled using the overall mode.

This is appropriate because customer ratings are discrete 1–5 star values. The mode represents the most frequently observed rating without introducing an artificial fractional rating.

---

# Section B — Numerical Operations & Vectorized Logic

NumPy is used for vectorized calculations.

## B1. Delivery Delay

Delivery delay is calculated as:

```text
Delivery_Delay = Actual_Delivery_Days - Est_Delivery_Days
```

Interpretation:

- Negative → Early
- Zero → On-Time
- Positive → Delayed

## B2. Fulfillment Status

`np.select()` is used to classify transactions:

| Delivery Delay | Fulfillment Status |
|---:|---|
| `< 0` | Early |
| `= 0` | On-Time |
| `> 0 and <= 2` | Minor Delay |
| `> 2` | Severe Delay |

No iterative Python loops or row-wise `apply()` statements are used for this classification.

## B3. High-Value Order Detection

The exact 95th percentile of `Order_Value` is calculated using:

```python
np.percentile(df["Order_Value"], 95)
```

Orders strictly exceeding this threshold are flagged as:

```text
High-Value Order
```

## B4. Net Realized Revenue

Net realized revenue is calculated after applying the discount:

```text
Net_Revenue =
Order_Value × (1 - Discount_Percent / 100)
```

This represents commercial revenue realized after the promotional discount.

---

# Section C — Business Analytics & Aggregations

## C1. Category Performance Matrix

Transactions are grouped by `Category` and evaluated using:

- Total Net Revenue
- Average Discount Offered
- Return Rate

Return rate is calculated as:

```text
(Returned == "Yes").mean() × 100
```

Results are sorted by total net revenue in descending order.

## C2. Regional Fulfillment Friction

Orders are grouped by `City_Tier`.

For each tier, the percentage of orders classified as `Severe Delay` is calculated.

The city tier with the highest severe-delay rate is identified as the priority area for logistics investigation.

## C3. Customer Sentiment Cross-Tabulation

A Pandas pivot table is created with:

- `Category` as index rows.
- `Fulfillment_Status` as columns.
- Average `Customer_Rating` as values.

This enables comparison of customer sentiment across fulfillment-performance categories.

## C4. Rule-Based Customer Segmentation

Each transaction is assigned to one of three segments.

### High-Risk Transaction

```text
Discount_Percent >= 30%
AND
Returned == "Yes"
```

### Loyal & Satisfied

```text
Order_Value > median(Order_Value)
AND
Customer_Rating >= 4
AND
Returned == "No"
```

### Standard Order

All remaining transactions.

No machine-learning algorithm is used.

---

# Section D — Visual Analytics

Four Matplotlib visualizations are produced.

## D1. Commercial Contribution

A bar chart displays:

```text
Total Net Revenue by Product Category
```

Categories are sorted by revenue contribution.

## D2. Fulfillment Distribution

A histogram displays the distribution of:

```text
Actual_Delivery_Days
```

A dashed vertical line represents the average estimated delivery benchmark.

## D3. Discount Elasticity vs Return Propensity

A scatter plot maps:

- X-axis → `Discount_Percent`
- Y-axis → `Order_Value`

Points are differentiated by return status.

## D4. Logistics Impact on Sentiment

A bar chart compares average customer ratings across:

```text
Early
On-Time
Minor Delay
Severe Delay
```

This helps evaluate the relationship between delivery performance and customer sentiment.

---

# Technologies Used

- **Python** — Core programming language
- **Pandas** — Data loading, cleaning, grouping, aggregation, and pivot tables
- **NumPy** — Vectorized numerical operations and conditional classification
- **Matplotlib** — Business visualization
- **Jupyter Notebook** — Interactive analysis and documentation

---

# Project Structure

```text
SwiftKart-Analytics/
│
├── ecommerce_orders.csv
├── ecommerce_orders_cleaned.csv
├── swiftkart_analytics.ipynb
└── README.md
```

### `ecommerce_orders.csv`

Raw transactional dataset used as the input.

### `ecommerce_orders_cleaned.csv`

Cleaned dataset containing the original fields and engineered analytical features such as:

- `Delivery_Delay`
- `Fulfillment_Status`
- `Net_Revenue`
- `Order_Segment`

The high-value flag created for the 95th-percentile analysis may also be retained.

### `swiftkart_analytics.ipynb`

The complete executable analysis containing Sections A–D, outputs, tables, charts, insights, and recommendations.

---

# How to Run

## 1. Install Dependencies

Open Terminal:

```bash
pip3 install pandas numpy matplotlib jupyter
```

## 2. Start Jupyter

```bash
jupyter notebook
```

## 3. Open the Notebook

Open:

```text
swiftkart_analytics.ipynb
```

## 4. Place the Dataset Correctly

Make sure the following files are in the same project directory:

```text
ecommerce_orders.csv
swiftkart_analytics.ipynb
```

## 5. Run Cells in Order

Run:

```text
A1 → A2 → A3 → A4 → A5
→ B1 → B2 → B3 → B4
→ C1 → C2 → C3 → C4
→ D1 → D2 → D3 → D4
```

The cells should be executed sequentially because later analyses depend on columns created during earlier stages.

---

# Expected Deliverables

## 1. Jupyter Notebook

```text
swiftkart_analytics.ipynb
```

The notebook should contain:

- Numbered task sections
- Executable code
- Markdown explanations
- Analytical outputs
- Tables
- Four rendered Matplotlib figures
- Executive insights
- Strategic recommendations

## 2. Cleaned Dataset

```text
ecommerce_orders_cleaned.csv
```

## 3. Executive Summary

A one-page business brief should answer:

1. Which city tier requires immediate logistics improvement?
2. Does deep discounting create high-risk returns?
3. What discount thresholds should be recommended?

It should also include 3–4 strategic recommendations addressing logistics, returns, and discount targeting.

---

# Key Business Metrics

| KPI | Purpose |
|---|---|
| Total Net Revenue | Measures realized revenue after discounts |
| Average Discount | Measures promotional intensity |
| Return Rate | Measures product-return behavior |
| Severe Delay Rate | Measures logistics failure |
| Average Customer Rating | Measures customer sentiment |
| High-Value Order Rate | Identifies top-value transactions |
| High-Risk Transaction Rate | Identifies discounted returned transactions |
| Loyal & Satisfied Rate | Identifies valuable, satisfied, non-returning transactions |

---

# Business Interpretation

The final recommendations should be based on the actual analytical results produced by the notebook.

### Logistics

Prioritize the city tier with the highest severe-delay rate.

### Discounts

Compare discount levels with return rates and net revenue to determine whether deeper discounts are producing profitable demand or higher-risk transactions.

### Customer Experience

Compare customer ratings across fulfillment statuses to understand whether delivery delays are associated with lower satisfaction.

### Category Strategy

Evaluate category-level revenue, discounting, and return rates together to identify profitable growth opportunities and operational risks.

---

# Conclusion

This project demonstrates a complete foundational data analytics workflow:

```text
Raw Data
   ↓
Data Inspection
   ↓
Data Cleaning
   ↓
NumPy Feature Engineering
   ↓
Business Aggregation
   ↓
Customer Segmentation
   ↓
Visualization
   ↓
Business Insights
   ↓
Strategic Recommendations
```

The objective is to transform raw e-commerce transaction data into clear, evidence-based operational and growth recommendations for SwiftKart.
