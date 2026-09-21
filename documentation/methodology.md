# Methodology

## 1. Analytical Objective

The objective of this project was to investigate shipping delays and understand which observable factors were associated with SLA non-compliance.

The analysis was exploratory. Rather than starting with a predetermined explanation for late orders, different hypotheses were tested against the available data and discarded when the evidence was not sufficiently consistent.

---

## 2. Understanding the Data Grain

The original dataset contains **180,519 rows**.

However, the original table operates at **order-line level**, meaning that a single order may appear multiple times when it contains multiple products or line items.

Shipping performance, on the other hand, needs to be evaluated at **order level**.

Using the original line-level table directly for order-level KPIs could therefore distort results by giving orders with more line items greater weight.

For this reason, an order-level analytical table was created in Power Query.

### Result

* Original dataset: **180,519 order-line records**
* Analytical table: **65,752 unique orders**
* Analytical grain: **one row per Order ID**

The original line-level data was retained separately, while the order-level table became the primary fact table for the shipping analysis.

---

## 3. Data Preparation

Power Query was used to clean, transform and consolidate the source data.

The main preparation steps included:

* Consolidating multiple order lines into one record per order.
* Retaining shipping, order, geographic and product-related attributes required for analysis.
* Calculating total order quantity at order level.
* Preparing order date and time fields for temporal analysis.
* Extracting **Order Hour** from the order timestamp.
* Creating variables required to compare scheduled and actual shipping performance.
* Validating the resulting order-level dataset before analysis.

This transformation allowed shipping KPIs to be calculated consistently at the same analytical grain.

---

## 4. SLA Logic

Shipping performance was evaluated by comparing:

**Actual Shipping Days**

with:

**Scheduled Shipping Days**

An SLA deviation was defined conceptually as:

`SLA Deviation = Actual Shipping Days - Scheduled Shipping Days`

Therefore:

* **SLA Deviation > 0** → shipment exceeded its scheduled target
* **SLA Deviation = 0** → shipment matched its scheduled target
* **SLA Deviation < 0** → shipment was completed earlier than scheduled

Cancelled orders were excluded from SLA compliance calculations.

The final Power BI model uses explicit measures for the principal KPIs:

* Total Orders
* Evaluable Orders
* Late Orders
* On-Time Orders
* Late %
* On-Time %
* Average SLA Deviation

---

## 5. Exploratory Analysis

The investigation tested multiple potential explanations for shipping delays.

### Order Quantity

One hypothesis was that orders containing more units could require additional preparation time and therefore experience higher Late rates.

Orders were segmented by quantity and compared across shipping performance.

The results did not show a sufficiently strong or consistent relationship to consider order quantity a primary driver of SLA non-compliance.

### Product Category

Product categories were analyzed to determine whether certain types of products were systematically associated with shipping delays.

Differences existed between categories, but the results were not sufficiently consistent to explain the overall Late rate.

Product Category was therefore retained as an exploratory dimension rather than identified as a principal driver.

### Geography

Shipping performance was analyzed across regions and geographic destinations.

Some differences in Late rates were observed and initially appeared potentially relevant.

However, further comparison did not reveal a geographic pattern sufficiently consistent to explain the overall behavior of the shipping process.

Geography was therefore retained in the Operational Deep Dive for segmentation rather than treated as an established root cause.

### Order Volume

The possibility that higher order volumes were associated with poorer shipping performance was also investigated.

The observed results did not provide sufficient evidence that volume alone explained the overall pattern of SLA non-compliance.

---

## 6. Shipping Mode Analysis

The strongest patterns emerged when performance was analyzed by Shipping Mode.

### First Class

First Class has a scheduled shipping target of **1 day**.

Observed actual shipping time was consistently **2 days**, producing a **100% Late rate**.

This indicates a systematic mismatch between the defined service target and observed performance.

### Same Day

Same Day initially showed approximately half of orders meeting the SLA and half being Late.

Several dimensions were explored without identifying a sufficiently consistent explanation.

Order Hour produced a substantially stronger result:

* Orders placed between **00:00 and 11:59 → 100% On Time**
* Orders placed **from 12:00 onward → 100% Late**

This revealed an effective operational threshold around noon.

The available data does not contain the intermediate process timestamps required to determine the exact operational reason for this threshold.

### Second Class vs Standard Class

Second Class and Standard Class initially appeared to have substantially different performance because their Late rates were very different.

However, their distributions of **actual shipping days were remarkably similar**, with observed durations distributed across approximately 2–6 days.

Their scheduled targets differ:

* Second Class: **2 days**
* Standard Class: **4 days**

Consequently, similar actual operational performance generates very different SLA outcomes.

This shifted the analytical question from:

*"Why is Second Class operationally slower?"*

to:

*"Are the SLA definitions aligned with the actual process performance?"*

---

## 7. Data Model

The final Power BI model uses the transformed **Orders** table as the primary fact table for shipping analysis.

A dedicated date dimension (`DimDate`) was created and related to the order date using a one-to-many relationship.

The original order-line table was retained separately because it operates at a different grain.

It was intentionally not connected directly to the order-level analytical model where doing so could introduce ambiguous filtering or double counting.

This distinction between **order-line grain** and **order grain** was important for maintaining consistent KPI calculations.

---

## 8. Validation

Results produced in Power BI were compared against the earlier exploratory analysis to verify that measures and filtering behavior reproduced the expected patterns.

Particular attention was given to:

* Total unique order counts
* SLA compliance calculations
* Shipping Mode behavior
* Same Day performance by Order Hour
* Second Class vs Standard Class shipping distributions
* Filter context when using fields from different tables

This validation also highlighted the importance of using dimensions from the correct analytical table when measures are evaluated under filter context.

---

## 9. Analytical Limitations

The dataset does not contain sufficient process-level timestamps to identify the exact operational stage responsible for shipping delays.

It is not possible to determine conclusively whether the measured shipping interval represents:

* order entry to warehouse completion,
* order preparation to carrier collection,
* warehouse dispatch to customer delivery,
* or another operational milestone.

The dataset also lacks timestamps for intermediate activities such as:

* picking,
* packing,
* warehouse release,
* carrier handoff.

Therefore, the analysis distinguishes between **observed patterns** and **causal explanations**.

The findings identify where further operational investigation should be concentrated, but they should not be interpreted as proof of a specific underlying root cause.

---

## 10. Analytical Outcome

The investigation did not identify product mix, order quantity, geography or order volume as sufficiently consistent explanations for overall SLA non-compliance.

The strongest findings instead emerged from the relationship between **Shipping Mode, scheduled service targets and observed shipping performance**.

This led to three principal areas for further investigation:

1. The feasibility of the **First Class one-day SLA**.
2. The apparent **12:00 operational cut-off for Same Day**.
3. The alignment of **Second Class and Standard Class SLA definitions** with their similar observed shipping-time distributions.

The final Power BI dashboard was designed to communicate these findings while retaining an interactive Operational Deep Dive for further segmentation.
