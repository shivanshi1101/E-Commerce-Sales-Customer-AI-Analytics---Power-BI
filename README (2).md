# E-Commerce Sales, Customer & AI Analytics — Power BI

## Project Overview

This project is a portfolio-ready **E-Commerce Analytics solution** built around the Olist Brazilian E-Commerce dataset.

The goal is not simply to create a dashboard, but to build an end-to-end analytics workflow that demonstrates:

- Python / pandas for data profiling and analytical exploration
- SQL for business analysis and advanced queries
- Power Query for ETL and data preparation
- Power BI star-schema modeling
- DAX for business KPIs and analytical measures
- Customer analytics and RFM segmentation
- K-Means customer segmentation
- Seller and delivery-performance analysis
- Statistical analysis
- Power BI AI / analytics features
- Interactive dashboard design
- Business recommendations and storytelling

The project follows a business-question-driven approach:

**What happened → Where → Why → Who → What next → What should we do?**

---

## Dataset

### Olist Brazilian E-Commerce Public Dataset

Source:

https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce

The dataset contains approximately 100,000 orders from the Brazilian e-commerce marketplace Olist, covering transactions primarily from 2016–2018.

### Source Tables

The project uses these original CSV files:

1. `olist_customers_dataset.csv`
2. `olist_geolocation_dataset.csv`
3. `olist_orders_dataset.csv`
4. `olist_order_items_dataset.csv`
5. `olist_order_payments_dataset.csv`
6. `olist_order_reviews_dataset.csv`
7. `olist_products_dataset.csv`
8. `olist_sellers_dataset.csv`
9. `product_category_name_translation.csv`

A derived `geo_lookup` table was also created for Power BI modeling.

---

# Project Architecture

```text
Raw CSV Data
     ↓
Python / pandas
     ↓
Data Profiling + EDA
     ↓
SQLite Database
     ↓
SQL Business Analysis
     ↓
Power Query
     ↓
Star Schema
     ↓
DAX Measures
     ↓
Customer / Seller / Delivery Analytics
     ↓
Python ML / Statistical Analysis
     ↓
Power BI AI & Visualization
     ↓
Interactive Executive Dashboard
     ↓
Business Recommendations
```

---

# Data Model

The Power BI model uses a star-schema-oriented structure.

## Fact Tables

### FactOrders

Grain:

**One row per order**

Important fields:

- `OrderID`
- `CustomerID`
- `CustomerUniqueID`
- `OrderStatus`
- `OrderPurchaseDateTime`
- `OrderPurchaseDate`
- `ApprovedDateTime`
- `CarrierDateTime`
- `DeliveryDateTime`
- `EstimatedDeliveryDate`
- `DeliveryDelayDays`
- `DeliveryStatus`

### FactOrderItems

Grain:

**One row per order item line**

Important fields:

- `OrderID`
- `OrderItemID`
- `ProductID`
- `SellerID`
- `Price`
- `FreightValue`
- `ItemTotalValue`
- `CustomerUniqueID`
- `OrderPurchaseDate`

`ItemTotalValue` is defined as:

```text
Price + FreightValue
```

This project deliberately defines this as **Total Order Value / item-level order value**, rather than automatically calling it accounting revenue.

### FactPayments

Grain:

**One row per payment record**

Important fields:

- `OrderID`
- `PaymentSequential`
- `PaymentType`
- `PaymentInstallments`
- `PaymentValue`
- `CustomerUniqueID`

Some orders have multiple payment records, so payment rows must not be treated as one-to-one with orders.

### FactReviews

Grain:

**One row per review**

Important fields:

- `ReviewID`
- `OrderID`
- `ReviewScore`
- `ReviewCommentTitle`
- `ReviewCommentMessage`
- `ReviewCreationDate`
- `ReviewAnswerDateTime`
- `CustomerUniqueID`

Multiple reviews can exist for an order, so review grain is preserved.

---

# Dimension Tables

## DimCustomerUnique

Grain:

**One row per real/unique customer**

Uses:

`CustomerUniqueID`

This is preferred for customer-level analytics because the original dataset contains multiple operational `CustomerID` records for some real customers.

## DimProduct

Grain:

**One row per product**

Important fields:

- `ProductID`
- `ProductCategoryFinal`
- `ProductNameLength`
- `ProductDescriptionLength`
- `ProductPhotosQty`
- `ProductWeightG`
- `ProductLengthCm`
- `ProductHeightCm`
- `ProductWidthCm`

Product categories were translated into English where possible.

Two untranslated categories were mapped:

- `pc_gamer` → `gaming_pc`
- `portateis_cozinha_e_preparadores_de_alimentos` → `kitchen_portable_appliances`

Missing categories are represented as:

`Unknown / Unclassified`

## DimSeller

Grain:

**One row per seller**

Important fields:

- `SellerID`
- `SellerZipCode`
- `SellerCity`
- `SellerState`

## DimGeography

Grain:

**One row per ZIP code**

The original geolocation dataset contains over one million rows and many duplicate/variant observations for the same ZIP code.

A ZIP-level lookup was therefore created using aggregated coordinates.

## DimDate

A dedicated calendar table covering:

**2016-01-01 → 2018-12-31**

Columns include:

- Date
- Year
- MonthNumber
- MonthName
- Quarter
- YearMonth

The table is marked as the Power BI Date Table.

---

# Key Relationships

The model intentionally avoids unnecessary fact-to-fact relationships.

Main relationships:

```text
DimCustomerUnique
    ↓
FactOrders
FactOrderItems
FactPayments
FactReviews

DimProduct
    ↓
FactOrderItems

DimSeller
    ↓
FactOrderItems

DimGeography
    ↓
DimCustomerUnique

DimDate
    ↓
FactOrders
FactOrderItems
```

This structure reduces ambiguity and makes filter propagation more predictable.

---

# Data Quality Findings

Several important data-quality issues were identified during profiling.

## Orders

- 99,441 orders
- 160 missing approval timestamps
- 1,783 missing carrier dates
- 2,965 missing customer delivery dates
- 8 delivered orders lack a customer delivery date
- Some records contain inconsistent timestamp sequences

The project **preserves these records rather than fabricating missing dates**.

A `date_quality_issue` concept was also explored for inconsistent date sequences.

## Order Items

- 112,650 item lines
- 98,666 unique orders represented
- `(OrderID, OrderItemID)` identifies an item line

## Payments

- 103,886 payment records
- Some orders contain multiple payment records

## Reviews

- 99,224 review records
- Review text contains substantial missingness
- Multiple reviews can exist for an order

## Products

- 32,951 products
- 610 products have missing product categories
- Missing category values are handled explicitly rather than silently discarded

## Geolocation

- 1,000,163 raw records
- 261,831 exact duplicate rows
- 19,015 unique ZIP prefixes after aggregation

The raw geolocation table was not blindly deduplicated. Instead, a ZIP-level lookup was created.

---

# Major Analytical Findings

These findings were produced during Python/SQL exploration and should be treated as analytical observations from this dataset, not universal business truths.

## Order Status

Delivered orders dominate the dataset:

- Delivered: 96,478
- Shipped: 1,107
- Canceled: 625
- Unavailable: 609
- Invoiced: 314
- Processing: 301
- Created: 5
- Approved: 2

## Order Value

Using:

```text
Product Price + Freight Value
```

delivered-order value is approximately:

**R$15.42 million**

Total order-item value across all order statuses is approximately:

**R$15.84 million**

These are project-defined analytical metrics and should not automatically be interpreted as recognized accounting revenue.

---

# Customer Analytics

There are:

- 99,441 operational customer records
- 96,096 unique customers

Most customers purchase only once.

Among delivered-order customers:

- Average frequency ≈ 1.03 orders
- Maximum observed frequency = 15 orders
- Approximately 97% have only one delivered order

This is an important business finding because it limits how confidently we can describe customers as genuinely "loyal."

---

# RFM Analysis

RFM was calculated using delivered orders.

Dimensions:

### Recency

Days since the customer's most recent delivered purchase.

### Frequency

Number of delivered orders.

### Monetary

Total delivered-order value.

The analysis date was set to:

**2018-08-30**

based on the latest delivered purchase plus one day.

Traditional RFM segments were created:

- Champions
- Loyal / High Value
- Recent Potential
- Needs Development
- At Risk
- Inactive

### Important Caveat

The dataset is heavily concentrated around one-time purchases.

Therefore, traditional RFM labels such as "Champions" should not automatically be interpreted as long-term loyalty.

---

# K-Means Segmentation

K-Means clustering was tested using standardized/log-transformed customer metrics.

Silhouette scores were evaluated for multiple values of K.

The strongest separation was observed at:

**K = 2**

with a silhouette score of approximately:

**0.709**

The resulting clusters largely separated:

### Cluster 0

A smaller group of repeat/higher-value customers.

Approximately:

- 2,801 customers
- 3.0% of customers
- 5.6% of spend
- Average frequency ≈ 2.11
- Average monetary value ≈ R$308.53

### Cluster 1

The dominant one-time customer population.

Approximately:

- 90,557 customers
- Average frequency ≈ 1.00
- Average monetary value ≈ R$160.73

The segmentation is useful because it exposes a fundamental characteristic of the dataset:

**The marketplace has a very large one-time customer base and a much smaller repeat-purchase segment.**

---

# Churn Analysis Decision

A churn-modeling approach was investigated.

Repeat-purchase intervals showed:

- Average repeat interval ≈ 79 days
- 68.8% of repeat purchases occurred within 90 days
- 83.5% occurred within 180 days

A 180-day inactivity threshold was considered.

However, when a churn target was constructed, the resulting eligible population was almost entirely classified as churned.

Therefore:

**Churn ML was intentionally not included as a final model.**

This is an important modeling decision.

A model should not be built simply because machine learning is expected in a portfolio project. The target must contain enough meaningful variation to support a useful prediction problem.

---

# Seller Analytics

Seller performance was analyzed using:

- Delivered orders
- Sales value
- Average review score
- Late-delivery rate

Sellers with at least 100 delivered orders were evaluated.

A seller-risk prioritization score was also explored:

```text
40% Late Delivery Risk
40% Review Risk
20% Sales Exposure
```

This is a **constructed prioritization metric**, not an objectively validated industry-standard score.

It should therefore be presented as a business-prioritization framework rather than as ground truth.

---

# Delivery Analysis

Delivery performance is represented through:

`DeliveryDelayDays`

Calculated as:

```text
Actual Delivery Date
-
Estimated Delivery Date
```

Therefore:

- Negative = delivered early
- Zero = delivered on estimated date
- Positive = delivered late

A categorical `DeliveryStatus` was created:

- Early / On Time
- Late
- Not Delivered

---

# Delivery vs Customer Satisfaction

A strong relationship was observed between delivery delay and review score.

Average review score by delivery performance:

| Delivery Performance | Average Review |
|---|---:|
| Early / On Time | 4.29 |
| 1–3 days late | 3.77 |
| 4–7 days late | 2.32 |
| 8+ days late | 1.73 |

The relationship was statistically tested using Spearman correlation.

Result:

```text
Correlation ≈ -0.176
Observations ≈ 96,353
p-value < 0.001
```

Interpretation:

There is a statistically significant but modest negative association between delivery delay and customer satisfaction.

**This does not establish causality.**

---

# SQL Analysis

SQLite was used as the analytical SQL environment.

The database contains:

```text
customers
geolocation
orders
order_items
payments
reviews
products
sellers
category_translation
geo_lookup
```

SQL analysis included:

- Aggregation
- GROUP BY
- JOINs
- CTEs
- Window functions
- Ranking
- Customer analysis
- Pareto analysis
- Category performance
- Delivery analysis
- Review analysis

---

# Pareto Analysis

Delivered customers were ranked by total delivered-order value.

Cumulative contribution:

| Customer Segment | Share of Delivered-Order Value |
|---|---:|
| Top 1% | 10.34% |
| Top 5% | 26.79% |
| Top 10% | 38.25% |
| Top 20% | 53.52% |

The key observation is:

**The top 20% of customers account for approximately 53.5% of delivered-order value.**

This can support customer-prioritization analysis, although it should not automatically be interpreted as a profitability analysis.

---

# Category Analysis

Category-level performance was evaluated using:

- Number of delivered orders
- Average review score
- Delivery performance

The analysis explicitly avoids duplicating review scores when an order contains multiple item lines.

This is an important modeling/SQL lesson:

> Always respect the grain of the metric being calculated.

For example, review score belongs to the review/order level, while product sales belong to the order-item level.

---

# Power BI Development

## Power Query

Power Query was used for:

- Renaming fields
- Data type conversion
- Creating derived columns
- Merging order/customer information
- Translating categories
- Creating customer-level dimensions
- Aggregating geolocation
- Creating delivery metrics
- Cleaning and standardizing the model

Power Query is responsible for **data preparation**, while DAX is being used for **analytical calculations**.

---

# Current DAX Measures

The first base measure created is:

```DAX
Total Order Value =
SUM(FactOrderItems[ItemTotalValue])
```

Planned core measures include:

```DAX
Total Orders =
DISTINCTCOUNT(FactOrders[OrderID])
```

```DAX
Total Customers =
DISTINCTCOUNT(DimCustomerUnique[CustomerUniqueID])
```

```DAX
Average Order Value =
DIVIDE([Total Order Value], [Total Orders], 0)
```

```DAX
Total Orders Delivered =
CALCULATE(
    [Total Orders],
    FactOrders[OrderStatus] = "delivered"
)
```

Additional measures will be added progressively.

---

# Important Modeling Decision: Delivered Order Value

A simple measure such as:

```DAX
Delivered Order Value =
CALCULATE(
    [Total Order Value],
    FactOrders[OrderStatus] = "delivered"
)
```

should **not** be used blindly in the current model.

Why?

`FactOrders` and `FactOrderItems` are separate fact tables.

Filtering `FactOrders` does not automatically filter the rows of `FactOrderItems`.

A model-safe solution will be implemented before this KPI is finalized, most likely by bringing the order status into `FactOrderItems` during Power Query transformation.

---

# Planned Power BI Report

The final report is expected to contain several analytical sections.

## Page 1 — Executive Overview

KPIs:

- Total Order Value
- Total Orders
- Total Customers
- Average Order Value
- Delivered Orders
- Delivery Performance

Visuals:

- Sales/order trend
- Order-status breakdown
- Category performance
- Geographic distribution
- Key business insights

## Page 2 — Customer Analytics

- Customer count
- Repeat vs one-time customers
- RFM segments
- Customer value distribution
- Pareto analysis
- Customer geography

## Page 3 — Product & Category Performance

- Category sales
- Order volume
- Review score
- Delivery performance
- Top products
- Category comparison

## Page 4 — Seller Performance

- Seller sales
- Delivered orders
- Review score
- Late-delivery rate
- Seller risk/prioritization matrix

## Page 5 — Delivery & Customer Experience

- Early/on-time/late distribution
- Delay distribution
- Delivery trend
- Review score vs delivery performance
- Problem categories/sellers

## Page 6 — AI / Advanced Analytics

Potential features:

- Key Influencers
- Decomposition Tree
- Forecasting
- Anomaly Detection
- Q&A
- Smart Narrative
- What-if parameters

The final selection will depend on what adds genuine analytical value.

---

# AI / Machine Learning Components

The project is intended to demonstrate AI without forcing AI into every part of the dashboard.

Potential components:

### Customer Segmentation

- RFM
- K-Means

### Statistical Analysis

- Correlation
- Significance testing
- Delivery vs satisfaction analysis

### Explainability

Where ML models are used, feature importance or SHAP-style explanations may be incorporated where practical.

### Power BI Native AI

Potentially:

- Key Influencers
- Decomposition Tree
- Q&A
- Smart Narrative
- Forecasting
- Anomaly Detection

The guiding principle is:

**Use AI where it answers a business question better than a simple chart.**

---

# Features Intended for the Final Report

The project aims to demonstrate:

- Star schema
- DAX measures
- Time intelligence
- Dynamic Top-N
- Pareto analysis
- Customer segmentation
- Drill-down
- Drill-through
- Tooltips
- Bookmarks
- Buttons
- Dynamic titles
- Conditional formatting
- Maps
- What-if parameters
- Key Influencers
- Decomposition Tree
- Forecasting
- Anomaly Detection
- Q&A
- Smart Narrative
- Row-Level Security
- Power BI Service deployment

Not every feature will be added merely for checklist purposes. Features should contribute to the business story.

---

# Data Integrity Principles

This project follows several important principles.

### 1. Never invent missing information

Missing values are preserved or explicitly categorized.

### 2. Respect data grain

Order, order-item, payment, review, customer, seller, and product data have different grains.

### 3. Avoid fact-to-fact relationships

Shared dimensions are preferred.

### 4. Do not claim causality from correlation

Statistical relationships are described as associations unless a causal design supports stronger conclusions.

### 5. Do not build meaningless ML models

The attempted churn model was dropped because the target was not sufficiently informative.

### 6. Define business metrics precisely

For example:

`ItemTotalValue = Price + FreightValue`

is treated as an analytical order-value metric, not automatically as accounting revenue.

---

# Current Project Status

Completed:

- Dataset acquisition
- Python setup
- Data extraction
- Raw backup
- Data profiling
- Data-quality assessment
- Key/grain analysis
- SQLite database
- SQL business analysis
- Customer RFM analysis
- K-Means exploration
- Seller analysis
- Delivery analysis
- Statistical analysis
- Power Query transformations
- Dimension/fact structure
- Customer unique dimension
- Geography lookup
- Date table
- Power BI relationships
- Initial DAX measure

Current stage:

**DAX KPI development**

Next immediate measures:

1. Total Orders
2. Total Customers
3. Average Order Value
4. Total Orders Delivered

Then:

- Delivery KPIs
- Customer KPIs
- Category KPIs
- Seller KPIs
- Advanced DAX
- Report pages
- AI features
- Final business story

---

# Portfolio Value

The project demonstrates more than dashboard creation.

It shows the complete analytics lifecycle:

```text
Business Problem
      ↓
Data Understanding
      ↓
Data Quality
      ↓
ETL
      ↓
SQL
      ↓
Data Modeling
      ↓
DAX
      ↓
EDA
      ↓
Statistics
      ↓
Machine Learning
      ↓
Visualization
      ↓
AI
      ↓
Business Recommendations
```

This makes the project suitable as a portfolio case study for roles involving:

- Data Analytics
- Business Intelligence
- Power BI
- SQL
- Business Analysis
- Customer Analytics
- E-Commerce Analytics
- Junior Data Science / Analytics roles

---

# Key Lessons From the Project

1. **The hardest part of BI is often modeling, not visualization.**
2. **Grain matters more than the number of tables.**
3. **A technically possible metric can still be analytically wrong.**
4. **Missing data should be handled explicitly.**
5. **One-time customers require caution when interpreting loyalty.**
6. **Statistical significance does not imply business causality.**
7. **Machine learning should solve a real analytical problem.**
8. **Power BI is strongest when Python, SQL, Power Query, DAX, and visualization each do the job they are best suited for.**

---

# Project Status

**Status:** In Progress

**Current phase:** DAX and analytical KPI development

**Primary tool:** Microsoft Power BI

**Supporting tools:**

- Python
- pandas
- SQLite
- SQL
- Power Query
- scikit-learn
- SciPy

