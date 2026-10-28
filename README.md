# E-commerce Customer Intelligence

## Business Problem

A global online retailer lacks visibility into how customers behave. As a result, it cannot tell who its high-value customers are, which products sell together, or where it is losing customers. This leads to inefficient marketing, poor targeting and missed cross-selling opportunities.

## Objectives

1. **Customer Segmentation:** group customers by Recency, Frequency and Monetary (RFM) behaviour and map their lifecycle.
2. **Customer Value Prediction:** predict whether a customer will become high-value using only their first-day purchase behaviour.
3. **Market Basket Analysis:** find products that are frequently bought together to support bundling and cross-selling.

## Dataset

- **541,909** transaction rows, 8 columns (InvoiceNo, StockCode, Description, Quantity, InvoiceDate, UnitPrice, CustomerID, Country)
- **4,338** unique customers
- Real-world retail data, with prices in £

## Approach

### 1. Data Cleaning
- Dropped **135,080** rows with missing `CustomerID`, since customer-level analysis needs an identifier
- Removed cancellation invoices (prefixed with `C`) after validating that the negative quantities (-274,808) belonged only to those invoices
- Removed 40 zero-price records
- Created a `Revenue` feature (Quantity × UnitPrice) and found a highly right-skewed distribution (median £12.45, max £168,469), consistent with a mix of retail and wholesale buyers
- Final cleaned dataset: **392,692** rows

### 2. Phase 1: Customer Segmentation (RFM)
- Calculated Recency, Frequency and Monetary values per customer and converted them to ranked scores
- Defined rule-based segments, then refined them to six: **Champions, Loyalist, New Customer, At-Risk, Hibernating, Lost**
- Mapped the customer lifecycle, including the migration paths between segments
- Built KPI cards and a segment treemap with Plotly

### 3. Phase 2: Customer Value Prediction
- **Target:** high-value customer (Champions + Loyalist) vs regular customer
- **Features (first-day behaviour only):** first-day spend, first-day quantity, number of unique items, shopping hour and weekend shopper flag
- **Models compared:** Logistic Regression, Decision Tree, Random Forest, Gradient Boosting, XGBoost
- **Class imbalance:** 1,430 high-value vs 2,908 regular customers. Tested class weights and **SMOTE** (applied to the training set only, so the test set stayed untouched and there was no data leakage)

### 4. Phase 3: Market Basket Analysis
- Removed `POSTAGE` rows and built an invoice × product basket matrix
- Mined frequent itemsets with **Apriori** (min support 0.02) and generated association rules with lift ≥ 1.0

## Key Findings

### Customer Segments

| Segment | Share of customers |
|---|---|
| Champions | 12.1% |
| Loyalist | 20.9% |
| At-Risk | ~13% |
| Hibernating | 26.1% |
| Lost | 21.8% |
| New Customer | ~6.2% |

- **47.9%** of customers are Hibernating or Lost, so nearly half of everyone who ever shopped is now inactive.
- Champions are only ~12% of customers but spend the most, averaging about **£8,472** each.
- **At-Risk** customers are not low spenders. They average about £1,764 and are valuable customers the business is losing, making them a priority for re-engagement.
- New customers are only ~6% of the base, so acquisition is not keeping up with churn.
- **The 20-day golden window:** new customers who do not reach a third order within about 20 days are at much higher risk of churning.

### Customer Value Prediction

| Metric (high-value class) | Random Forest + SMOTE (final model) |
|---|---|
| Recall | **0.73** |
| F1-score | **0.50** |

- The baseline Logistic Regression model almost never detected high-value customers (2% recall) because of class imbalance.
- The **SMOTE-trained Random Forest** achieved the highest recall (**73%**) and F1 (**0.50**) of all models, so it was selected as the final model.
- **Trade-off:** overall accuracy and precision are lower (accuracy 0.52, precision 0.38), because the model is tuned to catch as many future high-value customers as possible. This suits a marketing use case, where missing a valuable customer costs more than contacting an extra one.
- **Top predictors:** first-day spend (~40%), first-day quantity (~25%) and unique items purchased (~21%). Together, first-basket characteristics drive over 86% of the model's decisions, while shopping hour and weekend purchasing add very little.

### Product Affinity
- Analysed **18,480 baskets** across **3,876 products** and found **242** frequent itemsets and **76** association rules
- The strongest rules involve variants of the **Regency Teacup and Saucer** (pink, green, roses), with confidence of 0.69 to 0.89 and lift of about 18 to 24
- Customers who buy the Pink and Roses variants together buy the Green variant about 89% of the time
- Other strong pairs include Gardeners Kneeling Pad variants and the Dolly Girl / Spaceboy Lunch Boxes

---

## Business Recommendations

1. **Activate new customers within the golden window.** Trigger a third-order incentive (discount or loyalty points) around day 15, before the 20-day decay point.
2. **Run a VIP recovery programme for At-Risk high spenders.** Offer personalised outreach, such as early access to new collections, instead of generic discounts.
3. **Use Champions for referrals.** Reward referrals to bring in higher-quality new customers at a lower acquisition cost.
4. **Prioritise high-spending first-time buyers** for loyalty programmes, and encourage larger and more varied first orders through bundles and complementary product suggestions.
5. **Bundle co-purchased products**, such as the Regency Teacup and Saucer variants and the Gardeners Kneeling Pad variants, and use them for cross-sell recommendations.
6. **Focus on purchase behaviour rather than timing.** Shopping hour and weekend purchasing added little predictive value.

### Proposed Success Metrics
- **Activation rate:** share of new customers reaching a third order within 20 days. Target: reduce the Lost segment by 10 to 15% over a quarter.
- **Whale retention:** recover 5% of At-Risk high spenders into the Loyalist segment.
- **Long-term:** test whether referral-acquired customers show about 20% higher six-month LTV than customers from paid ads.

## Limitations
- Class imbalance limits precision.
- The model is trained on historical purchasing behaviour, so it would need periodic retraining as seasonality and trends change.
- It uses a limited set of purchase features. Demographic or other behavioural data could improve performance.
- The "high-value" label is derived from the RFM segments, so the model predicts that segment definition rather than directly observed lifetime value.
