# CUSTOMER-SEGMENTATION-REVENUE-OPTIMIZATION-USING-RFM-ANALYSIS-Excel-SQL

**Executive Summary**

In today’s competitive market, understanding customer behavior is critical for driving revenue and retention. This project leverages RFM (Recency, Frequency, Monetary) analysis on transactional data to segment customers and uncover actionable insights.

Using SQL, customers were classified into meaningful segments such as Loyal, At Risk, Lost, and Best Customers, enabling a deeper understanding of purchasing behavior across different countries.

**Business Problems**

Stakeholders often hold the following challenges:

- Which customers are likely to churn?
- Who hasn’t purchased recently?
- Which customers and countries drive the most revenue?
- Who are most valuable customers?
- Which customers should we prioritize?
- Which segment should receive offers or campaigns?

**Tools used**

- SQL- Data cleaning & transformation, Aggregations and joins, Window functions
- Excel- Data preparation & validation
- RFM Analysis Framework

**Methodology** 

Step 1 : Data Cleaning

- Converted invoice date into proper date format
- Filtered only valid transactions.

Step 2 : RFM Metric Calculation

- Recency - Days since last purchase
- Frequency - Number of transactions
- Monetary - Total spend

```sql
WITH RFM_BASE AS 
(
SELECT `CUSTOMER ID`,
datediff(CURDATE(), MAX(`INVOICE_DATE_CLEAN`)) AS RECENCY,
COUNT(*) AS FREQUENCY,
SUM(QUANTITY * PRICE) AS MONETARY
FROM product_table
WHERE STATUS = 'PAID'
GROUP BY `Customer Id`
),
RFM_SCORES AS (
SELECT `CUSTOMER ID`,
		NTILE(5) OVER (ORDER BY RECENCY DESC) AS R_SCORE,
        NTILE(5) OVER (ORDER BY FREQUENCY ASC) AS F_SCORE,
        NTILE(5) OVER (ORDER BY MONETARY DESC) AS M_SCORE
        FROM RFM_BASE )
SELECT * FROM RFM_SCORES;

```

Step 3 : Customers were ranked into 5 groups:

- Score 5 = Best
- Score 1 = Worst

Using percentile-based ranking logic.

Step 5: Customer Segmentation 

Customers were grouped into segments based on RFM scores:

- BEST CUSTOMERS - High value, frequent, recent
- LOYAL CUSTOMERS - Frequent Buyers
- AVERAGE CUSTOMERS - Moderate engagement
- AT Risk - Declining activity
- LOST CUSTOMERS - Inactive customers

Step 6- Advanced Analysis

Additional insights were generated:

- Revenue by country & segment
- Top-performing countries using:

```sql
        WITH RFM_BASE AS 
(
SELECT p.`CUSTOMER ID`as customer_id, c.country as country,
datediff(CURDATE(), MAX(`INVOICE_DATE_CLEAN`)) AS RECENCY,
COUNT(*) AS FREQUENCY,
SUM(QUANTITY * PRICE) AS MONETARY
FROM product_table p
join customer_data c
on p.`customer id`= c.ï»¿Customer_Id
WHERE STATUS = 'PAID'
GROUP BY `Customer_Id`, country
),
RFM_SCORES AS (
SELECT *,
		NTILE(5) OVER (ORDER BY RECENCY DESC) AS R_SCORE,
        NTILE(5) OVER (ORDER BY FREQUENCY ASC) AS F_SCORE,
        NTILE(5) OVER (ORDER BY MONETARY DESC) AS M_SCORE
        FROM RFM_BASE ),
rfm_segment as (
SELECT *,
CASE WHEN R_SCORE = 5 AND F_SCORE=5 AND M_SCORE=5 THEN 'BEST_CUSTOMERS'
     WHEN R_SCORE >=4 AND F_SCORE>=4 THEN 'LOYAL_CUSTOMERS'
     WHEN R_SCORE = 5 AND F_SCORE<=2  THEN 'NEW_CUSTOMERS'
     WHEN R_SCORE <=2 AND F_SCORE>=4 THEN 'AT_RISK'
     WHEN R_SCORE <=2 AND F_SCORE<=2 THEN 'LOST_CUSTOMERS'
     ELSE 'AVERAGE_CUSTOMERS'
     END AS SEGMENT
     FROM RFM_SCORES
     )
select * 
from( 
  select country, segment, sum(monetary) as revenue,
  rank() over(partition by segment order by sum(monetary) desc) as rnk
  from rfm_segment
  group by country, segment  )t
  where rnk = 1;

```

- Customer distribution by segment
- Customer Lifetime Value (CLV)

**Results & Insights**

After applying RFM analysis on the transactional dataset, clear pattern began to emerge about customer behavior, revenue concentration, and geographic performance. These insights reveal not just what is happening, but where the business should focus next.

**Revenue is Driven by a Small Group of Customers**
The analysis shows that Loyal Customers contribute the highest share of total revenue, despite not being the largest group. 

What this means:

- The business is highly dependent on repeat purchases, losing these customers would have a direct and significant revenue impact.
- Highlights the importance of customer retention over acquisition.

**China Emerges as the Dominant Market**
Across multiple queries, China consistently appears as the top-performing country in terms of:
Revenue contribution 
Number of loyal customers
Presence across multiple segments.

What this means:

- China is not just a large market, but also a high-value market.
- Customers in this region show strong purchasing behavior and engagement.

**Lost Customers Represent a Hidden Opportunity**
One of the most surprising findings was that Lost customers still account for a noticeable share of revenue, especially in countries like:

- Philippines
- Sweden

What this means:

- These customers are not “low value” - they are disengaged high-value users.
- The business is currently losing revenue it once had

**At-Risk Customers Signal Future Revenue Decline
The At Risk segment includes customers who:**

- Previously purchased frequently
- Have recently reduced activity
- Today’s at-risk customers are tomorrow’s lost customers

What this means:

- Without intervention, revenue from this group will decline
- The business has a narrow window to act.

**Customer Lifetime Value is Concentrated in High-Frequency Buyers** 

Customers with high frequency and high monetary value show significantly higher Customer Lifetime Value (CLV)

What this means:

- Frequency is a strong driver of long-term revenue
- Encouraging repeat purchases is more valuable th one-time sales

**Recommendations**

- Retention Strategies (At Risk Customers) - Run targeted email campaigns. Offer time-based discounts. Personalized recommendations.
- Loyalty Program (Loyal Customers) - Introduce reward points or VIP tiers. Early access to products. Provide exclusive benefits.
- Win-Back Campaign (Lost Customers) - “We miss you” campaigns. Give special comeback discounts. Re-engagement emails.
- Geographic Focus - Invest more in high revenue area like China ang high loyalty based area like Indonesia.
- Revenue Optimization - Upsell high-value customers. Cross-sell based on purchase history. Focus marketing spend on high ROI segments.
