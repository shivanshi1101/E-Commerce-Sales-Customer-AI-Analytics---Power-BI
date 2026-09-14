# E Commerce Analytics and Delivery Risk Prediction

## Project Overview

This is an end to end e commerce analytics project built from a
Brazilian e commerce dataset.

The project combines data profiling, data quality analysis, Python
exploration, SQL business analysis, Power BI dashboard development, DAX
modeling, customer segmentation, and machine learning.

The main goal is to understand sales performance, customers, products,
sellers, payments, reviews, delivery performance, and delivery delay
risk.

## Business Questions

-   How much revenue is generated and how does it change over time?
-   Which product categories generate the most revenue?
-   Which customer states contribute the most revenue and orders?
-   Which sellers contribute the most revenue?
-   Which payment methods are used most frequently?
-   What does the review distribution tell us about customer
    satisfaction?
-   How does delivery performance relate to customer satisfaction?
-   Which orders have the highest predicted delivery delay risk?
-   Which customer and seller locations show higher delivery risk?
-   How can machine learning support delivery risk prioritization?

## Dataset

The project uses nine source datasets covering customers, geolocation,
orders, order items, payments, reviews, products, sellers, and category
translation.

  Dataset                       Rows
  ---------------------- -----------
  Customers                   99,441
  Geolocation              1,000,163
  Orders                      99,441
  Order Items                112,650
  Payments                   103,886
  Reviews                     99,224
  Products                    32,951
  Sellers                      3,095
  Category Translation            71

## Data Profiling and Quality

The Python analysis performed:

-   Row and column profiling
-   Data type checks
-   Missing value analysis
-   Unique value analysis
-   Duplicate detection
-   Relationship validation
-   Date range checks
-   Order status analysis
-   Payment consistency checks
-   Review analysis
-   Product category validation
-   Customer identifier validation
-   Seller identifier validation

Important findings included missing delivery dates for some non
delivered orders, substantial missing review comments, missing product
categories, duplicate geolocation records, multiple payment records for
some orders, and repeated customer identities across customer records.

The analysis also found that two product categories were not present in
the translation table. These were mapped to practical English labels.

## Data Preparation

Power Query was used for:

-   Data type conversion
-   Date extraction
-   Column renaming
-   Customer enrichment
-   Product category enrichment
-   Delivery status creation
-   Delivery delay calculation
-   Model preparation

Delivery delay was calculated as actual customer delivery date minus
estimated delivery date.

A positive value means the order was delivered late.

A zero value means the order arrived on the estimated date.

A negative value means the order arrived early.

## Power BI Data Model

The model uses fact and dimension tables.

### Fact Tables

-   Fact Orders
-   Fact Order Items
-   Fact Payments
-   Fact Reviews

### Dimension Tables

-   Dim Product
-   Dim Seller
-   Dim Customer Unique
-   Dim Geography
-   Dim Date

The model supports analysis across products, customers, sellers,
geography, payments, reviews, order status, and time.

## Key DAX Measures

### Total Order Value

``` text
Total Order Value = SUM(FactOrderItems[ItemTotalValue])
```

### Total Orders

``` text
Total Orders = DISTINCTCOUNT(FactOrders[OrderID])
```

### Average Order Value

``` text
Average Order Value = DIVIDE([Total Order Value], [Total Orders])
```

Additional measures were created for item counts, category order counts,
payment value, review scores, customer state analysis, and seller
analysis.

## Power BI Dashboard

The report contains multiple analytical pages.

### Sales and Executive Analysis

The dashboard includes:

-   Total Order Value
-   Average Order Value
-   Total Items Sold
-   Total Orders
-   Total Customers
-   Total Sellers
-   Revenue by product category
-   Revenue by customer state
-   Revenue by seller
-   Revenue over time
-   Orders by order status
-   Orders by product category

### Payment Analysis

The dashboard includes payment value and order count by payment type.

Credit card payments are the dominant payment method in the source data.

### Customer Satisfaction

The dashboard includes:

-   Review score distribution
-   Average review score by product category
-   Review based category analysis

The source data contains a strong concentration of high review scores.

### Delivery Analysis

The dashboard includes:

-   Average delivery delay by delivery status
-   Revenue by customer state
-   Orders by customer state
-   Product category performance
-   Delivery related analysis

## Python Exploratory Analysis

Python was used for deeper analysis of:

-   Customer behavior
-   Order behavior
-   Product categories
-   Seller performance
-   Payment methods
-   Review scores
-   Delivery delays
-   Repeat purchase behavior
-   Customer segmentation
-   Churn analysis
-   Delivery and review relationships

## Customer Segmentation

RFM analysis was performed using:

-   Recency
-   Frequency
-   Monetary value

The analysis applied logarithmic transformation, scaling, and K Means
clustering.

Different cluster counts were evaluated using silhouette scores before
selecting the clustering approach.

Traditional RFM scoring was also used to create customer segments.

## Churn Analysis

The notebook contains exploratory churn analysis based on historical
purchasing behavior.

It examines:

-   Repeat purchase intervals
-   Repurchase behavior across several time windows
-   A historical churn cutoff
-   Customer eligibility
-   Churn label distribution

Churn was not selected as the main predictive model because the
historical cutoff introduces limitations for a production style
prediction problem.

## Delivery Delay Machine Learning

The primary machine learning problem is predicting whether a delivered
order will arrive later than its estimated delivery date.

### Target

The target is `late_delivery`.

-   `1` means actual delivery was later than estimated.
-   `0` means actual delivery was on time or early.

The machine learning dataset contains 96,470 eligible delivered orders.

  Outcome      Orders           Share
  ---------- -------- ---------------
  Late          7,826    8.11 percent
  Not Late     88,644   91.89 percent

Because late deliveries are a minority class, accuracy alone is not
sufficient for evaluation.

## Feature Engineering

Order level features were created from orders, order items, sellers, and
customers.

The model uses:

-   Total items
-   Total price
-   Total freight
-   Average item price
-   Unique products
-   Unique sellers
-   Seller state
-   Customer state
-   Purchase year
-   Purchase month
-   Purchase day of week
-   Purchase hour
-   Estimated delivery duration

## Machine Learning Models

Two classification models were evaluated.

### Logistic Regression

  Metric               Result
  ----------- ---------------
  Accuracy      62.98 percent
  Precision     13.23 percent
  Recall        64.09 percent
  F1 Score      21.93 percent
  ROC AUC              0.6932

### Random Forest

At the default probability threshold:

  Metric               Result
  ----------- ---------------
  Accuracy      82.95 percent
  Precision     25.24 percent
  Recall        56.17 percent
  F1 Score      34.83 percent
  ROC AUC              0.7813

Random Forest produced stronger overall performance and was selected for
the final delivery risk analysis.

## Threshold Optimization

Probability thresholds from 0.20 through 0.80 were evaluated.

The threshold of 0.60 produced the highest F1 score among the tested
thresholds.

At 0.60:

-   Accuracy: 88.92 percent
-   Precision: 33.45 percent
-   Recall: 37.00 percent
-   F1 Score: 35.13 percent

The threshold was selected to improve precision while retaining
meaningful recall.

## Confusion Matrix at the Final Threshold

                      Predicted Not Late   Predicted Late
  ----------------- -------------------- ----------------
  Actual Not Late                 16,577            1,152
  Actual Late                        986              579

The model therefore produced:

-   16,577 true negatives
-   1,152 false positives
-   986 false negatives
-   579 true positives

The model should be treated as a risk prioritization tool rather than a
certainty mechanism.

## Risk Categories

The final test predictions were grouped into:

-   Low Risk
-   Medium Risk
-   High Risk

  Risk Category     Orders
  --------------- --------
  Low Risk           5,826
  Medium Risk       11,737
  High Risk          1,731

High risk represents approximately 9 percent of the test set.

Among high risk orders:

-   579 were actually late.
-   High risk precision was 33.5 percent.
-   High risk recall was 37.0 percent.

This means high risk classification is useful for prioritization but
does not guarantee a late delivery.

## Feature Importance

Permutation importance showed that the strongest predictive variables
included:

-   Purchase month
-   Estimated delivery duration
-   Customer state
-   Purchase year
-   Seller state
-   Total freight

Timing and geography were more predictive than many order economic
variables.

Feature importance indicates predictive usefulness and should not be
interpreted as proof of causation.

## Business Recommendations

### Prioritize High Risk Orders

Use predicted delivery risk to identify orders that may require
additional operational attention.

### Focus on Geographic Risk

Customer and seller geography are important predictive signals and can
help identify areas for logistics investigation.

### Review Delivery Promises

Estimated delivery duration is a strong predictive feature, so promised
delivery windows should be reviewed for realism.

### Monitor Seasonal Patterns

Purchase month is an important predictive feature, suggesting that
seasonal changes in demand and logistics pressure should be monitored.

### Connect Delivery and Customer Experience

Delivery performance should be evaluated together with review scores
because poor delivery experiences can affect customer satisfaction.

### Use Machine Learning as Decision Support

The model should support operational prioritization rather than
automatically determine actions.

## Tools and Technologies

-   Python
-   Pandas
-   NumPy
-   Scikit Learn
-   SQL
-   Power Query
-   Power BI
-   DAX
-   K Means
-   Random Forest
-   Logistic Regression

## Project Workflow

1.  Business question definition
2.  Raw data collection
3.  Data profiling
4.  Data quality validation
5.  Python exploratory analysis
6.  SQL business analysis
7.  Power Query transformation
8.  Power BI data modeling
9.  DAX measure development
10. Interactive dashboard development
11. Customer segmentation
12. Delivery delay machine learning
13. Model evaluation
14. Threshold tuning
15. Machine learning integration into Power BI
16. Business recommendations

## Limitations

-   The dataset represents historical e commerce activity.
-   The delivery model is evaluated on a historical test set.
-   Current machine learning predictions in Power BI represent the test
    set rather than a live scoring system.
-   The model is designed for delivered orders with known actual and
    estimated delivery dates during model construction.
-   High risk precision is 33.5 percent, so high risk classifications
    require operational validation.
-   Feature importance describes predictive contribution and not
    causation.
-   The historical churn analysis should not be treated as a production
    churn system without additional validation.

## Conclusion

This project demonstrates a complete analytics workflow from raw data to
business intelligence and predictive decision support.

The Power BI dashboard provides visibility into revenue, orders,
customers, products, sellers, payments, reviews, and delivery
performance.

The machine learning component extends the analysis from understanding
what happened to identifying orders that may require attention because
of elevated delivery delay risk.

The project demonstrates practical skills in data cleaning, data
modeling, SQL, Python, DAX, visualization, customer segmentation,
classification modeling, model evaluation, and business recommendation
development.


<img width="1530" height="742" alt="image" src="https://github.com/user-attachments/assets/888ddaed-7251-4826-a289-58c1ab616a31" />


<img width="1502" height="736" alt="image" src="https://github.com/user-attachments/assets/ed8495f5-a40a-41fd-93d5-175a147ea52c" />


<img width="1505" height="692" alt="image" src="https://github.com/user-attachments/assets/97e8dbf9-3b54-4e34-911c-9cef424d42a1" />


<img width="1213" height="535" alt="image" src="https://github.com/user-attachments/assets/ef84c903-8d0e-454c-aa05-edd7bba0027e" />


<img width="1191" height="505" alt="image" src="https://github.com/user-attachments/assets/dbd50812-8a17-43a1-83df-aedc03a26ae8" />



