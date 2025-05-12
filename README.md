# Sales_Performace_Dashboard
Sales Performance &amp;  Customer Retention Through Data-Driven Insights

 DOMAIN- Sales Analytics & CRM domain

 **Problem Statement**
 Sales teams often struggle with identifying key factors influencing deal success, understanding 
why deals are lost, and optimizing their sales pipeline efficiency. Additionally, predicting 
customer churn risk and uncovering seasonal sales trends is crucial for improving revenue and 
retention. This project aims to analyze sales performance, deal success patterns, and customer 
churn risks using SQL and Power BI to drive strategic decision-making.
 
** Objective:**

 1) Analyze Sales Performance – Identify top-performing sales agents, best-selling products, 
and high-value deals.

2) Understand Deal Success & Failure – Determine factors leading to lost deals and measure 
conversion rates.

 3) Predict Churn Risk – Identify at-risk accounts based on lost deals and engagement trends.
 
 4) Detect Seasonal Sales Trends – Analyze revenue fluctuations over time to optimize 
marketing and resource allocation.

 5) Enhance Decision-Making – Provide data-driven insights for sales strategy optimization using 
SQL and Power BI or Tableau


**SQL QUERY ANALYSIS**

 **Predicting High-Value Deals Using Revenue Percentile Analysis **
- classifies deals into high, mid, and low-value segments and identifies which sales agents 
close the most high-value deals.

** Sales Pipeline Velocity Analysis**
 - identify which accounts are moving faster or slower through different deal stages and can 
highlight bottlenecks.

 **Average Deal Closing Time by Industry**
-  Calculate the average number of days taken to close a deal for each industry.
  
 **Churn Risk Analysis Based on Deal Loss Patterns**
 - This query identifies accounts with a high risk of churn by calculating the lost deal 
percentage and the time gap since their last won deal.

** Sales Seasonality & Revenue Impact Analysis**
 - This query identifies seasonal trends in sales performance by analyzing revenue fluctuations 
across months and years.

**Table Creation**

Use HNI;

--Create Accounts Table

CREATE TABLE accounts (
    account VARCHAR(255) PRIMARY KEY,
    sector VARCHAR(255),
    year_established INT,
    revenue DECIMAL(15, 2),
    employees INT,
    office_location VARCHAR(255),
    subsidiary_of VARCHAR(255)
);

---Insert Data


Bulk Insert accounts

from 'C:\Users\AKHIL\Downloads\accounts.csv' --path

with -- need to providee the parameter
	(fieldterminator = ',' , rowterminator = '\n' , firstrow = 2, maxerrors = 20);

	Select * from accounts;

---Create a Products Table

CREATE TABLE products (
    product VARCHAR(255) PRIMARY KEY,
    series VARCHAR(255),
    sales_price DECIMAL(10, 2)
);

-- Insert Data for Products Table


Bulk Insert products

from 'C:\Users\AKHIL\Downloads\products_Data.csv' --path

with -- need to providee the parameter
	(fieldterminator = ',' , rowterminator = '\n' , firstrow = 2, maxerrors = 20);

	Select * from products;

---Create a Table for Sales_teams

CREATE TABLE sales_teams (
    sales_agent VARCHAR(255) PRIMARY KEY,
    manager VARCHAR(255),
    regional_office VARCHAR(255)
);

--Insert a Data of Sales_Table

Bulk Insert sales_teams

from 'C:\Users\AKHIL\Downloads\sales_teams.csv' --path

with -- need to providee the parameter
	(fieldterminator = ',' , rowterminator = '\n' , firstrow = 2, maxerrors = 20);

	Select * from sales_teams;

-- Create Tale for Sales_ pipeline

CREATE TABLE sales_pipeline (
    opportunity_id INT PRIMARY KEY,
    sales_agent VARCHAR(255),
    product VARCHAR(255),
    account VARCHAR(255),
    deal_stage VARCHAR(50),
    engage_date DATE,
    close_date DATE,
    close_value DECIMAL(15, 2),
    FOREIGN KEY (sales_agent) REFERENCES sales_teams(sales_agent),
    FOREIGN KEY (product) REFERENCES products(product),
    FOREIGN KEY (account) REFERENCES accounts(account)
);

--Insert the Sales_Pipeline Data

Bulk Insert sales_pipeline

from 'C:\Users\AKHIL\Downloads\sales_pipeline.csv' --path

with -- need to providee the parameter
	(fieldterminator = ',' , rowterminator = '\n' , firstrow = 2, maxerrors = 20);

	Select * from sales_pipeline;

-- Alter the Column

	ALTER TABLE sales_pipeline
ALTER COLUMN opportunity_id VARCHAR(20);

-- Drop Primary Key

ALTER TABLE sales_pipeline
DROP CONSTRAINT PK__sales_pi__B975AC8A8736868C;

--ALTER TABLE sales_pipeline
ALTER TABLE sales_pipeline
ALTER COLUMN opportunity_id VARCHAR(20) NOT NULL;

-- Recreate the Primary key
ALTER TABLE sales_pipeline
ADD CONSTRAINT PK_sales_pipeline_opportunity_id
PRIMARY KEY (opportunity_id);

**Solutions Query**

1) **Predicting High-Value Deals Using Revenue Percentile Analysis **
- classifies deals into high, mid, and low-value segments and identifies which sales agents 
close the most high-value deals.

  ** Query Solutions -**
  
  WITH CleanedDates AS (
    SELECT 
        sp.account,
        a.sector,
        TRY_CAST(sp.engage_date AS DATE) AS engage_dt,
        TRY_CAST(sp.close_date AS DATE) AS close_dt
    FROM sales_pipeline sp
    JOIN accounts a ON sp.account = a.account
    WHERE sp.engage_date IS NOT NULL AND sp.close_date IS NOT NULL
),
Durations AS (
    SELECT 
        sector,
        DATEDIFF(DAY, engage_dt, close_dt) AS duration_days
    FROM CleanedDates
    WHERE engage_dt IS NOT NULL AND close_dt IS NOT NULL
)
SELECT 
    sector,
    COUNT(*) AS deal_count,
    AVG(duration_days) AS avg_days_to_close,
    MAX(duration_days) AS max_days,
    MIN(duration_days) AS min_days
FROM Durations
GROUP BY sector
ORDER BY avg_days_to_close DESC;


2) ** Sales Pipeline Velocity Analysis**
 - identify which accounts are moving faster or slower through different deal stages and can 
highlight bottlenecks.

**Query Solutions -**

WITH CleanedDates AS (
    SELECT 
        account,
        deal_stage,
        TRY_CAST(engage_date AS DATE) AS engage_dt,
        TRY_CAST(close_date AS DATE) AS close_dt
    FROM sales_pipeline
    WHERE engage_date IS NOT NULL AND close_date IS NOT NULL
),
Durations AS (
    SELECT 
        account,
        deal_stage,
        DATEDIFF(DAY, engage_dt, close_dt) AS duration_days
    FROM CleanedDates
    WHERE engage_dt IS NOT NULL AND close_dt IS NOT NULL
),
Deal_StageWiseVelocity AS (
    SELECT 
        account,
        deal_stage,
        COUNT(*) AS deal_count,
        AVG(duration_days) AS avg_days_to_close,
        MAX(duration_days) AS long_duration_days,
		MIN(duration_days) AS short_duration_days

    FROM Durations
    GROUP BY account, deal_stage
)
SELECT *
FROM Deal_StageWiseVelocity
ORDER BY avg_days_to_close DESC;

3) **Average Deal Closing Time by Industry**
-  Calculate the average number of days taken to close a deal for each industry.

  **Query Solution -**

  WITH CleanedDates AS (
    SELECT 
        sp.account,
        a.sector,
        TRY_CAST(sp.engage_date AS DATE) AS engage_dt,
        TRY_CAST(sp.close_date AS DATE) AS close_dt
    FROM sales_pipeline sp
    JOIN accounts a ON sp.account = a.account
    WHERE sp.engage_date IS NOT NULL AND sp.close_date IS NOT NULL
),
Durations AS (
    SELECT 
        sector,
        DATEDIFF(DAY, engage_dt, close_dt) AS duration_days
    FROM CleanedDates
    WHERE engage_dt IS NOT NULL AND close_dt IS NOT NULL
)
SELECT 
    sector,
    COUNT(*) AS deal_count,
    AVG(duration_days) AS avg_days_to_close,
    MAX(duration_days) AS max_days,
    MIN(duration_days) AS min_days
FROM Durations
GROUP BY sector
ORDER BY avg_days_to_close DESC;

4)  **Churn Risk Analysis Based on Deal Loss Patterns**
 - This query identifies accounts with a high risk of churn by calculating the lost deal 
percentage and the time gap since their last won deal.

**Query Solutions **
-- Step 1: Count total, lost, and won deals for each account
WITH DealCounts AS (
    SELECT 
        account,
        COUNT(*) AS total_deals,
        SUM(CASE WHEN deal_stage = 'Lost' THEN 1 ELSE 0 END) AS lost_deals,
        SUM(CASE WHEN deal_stage = 'Won' THEN 1 ELSE 0 END) AS won_deals
    FROM sales_pipeline
    GROUP BY account
),

-- Step 2: Calculate percentage of deals that were lost
LostDealPercentage AS (
    SELECT 
        account,
        (CAST(lost_deals AS FLOAT) / total_deals) * 100 AS lost_deal_percentage
    FROM DealCounts
),

-- Step 3: Get the most recent 'Won' deal date for each account
LastWonDealDate AS (
    SELECT 
        account,
        MAX(TRY_CAST(close_date AS DATE)) AS last_won_date
    FROM sales_pipeline
    WHERE deal_stage = 'Won'
    GROUP BY account
),

-- Step 4: Combine data and calculate days since last 'Won' deal
ChurnRisk AS (
    SELECT 
        ldp.account,
        ldp.lost_deal_percentage,
        DATEDIFF(DAY, lwd.last_won_date, GETDATE()) AS Time_gap_since_last_won
    FROM LostDealPercentage ldp
    LEFT JOIN LastWonDealDate lwd 
        ON ldp.account = lwd.account
)

-- Step 5: Final output sorted by risk
SELECT 
    account,
    lost_deal_percentage,
    Time_gap_since_last_won
FROM ChurnRisk
ORDER BY 
    lost_deal_percentage DESC, 
    Time_gap_since_last_won DESC;
    
5) ** Sales Seasonality & Revenue Impact Analysis**
 - This query identifies seasonal trends in sales performance by analyzing revenue fluctuations 
across months and years.

**Query Solutions**

-- Step 1: Convert close_date into Year and Month
WITH MonthlyRevenue AS (
    SELECT 
        DATEPART(YEAR, TRY_CAST(close_date AS DATE)) AS year,
        DATEPART(MONTH, TRY_CAST(close_date AS DATE)) AS month,
        SUM(close_value) AS total_revenue
    FROM sales_pipeline
    WHERE deal_stage = 'Won'
    GROUP BY 
        DATEPART(YEAR, TRY_CAST(close_date AS DATE)),
        DATEPART(MONTH, TRY_CAST(close_date AS DATE))
)

-- Step 2: View revenue trends by year and month
SELECT 
    year,
    month,
    total_revenue
FROM MonthlyRevenue
ORDER BY year, month;



**Visual Analysis  (PowerBI/Tableau)**

 1. Sales Performance 
KPIs: Total revenue, number of won deals, average deal size
 Revenue breakdown by industry, company, and sales agent

 3. Sales Funnel Analysis
 Drop-off analysis at each stage (Prospecting → Engaging → Won/Lost)
 Conversion rates at each stage

 5. Monthly Revenue Trends
 Line chart showing monthly revenue trends
 Forecasting for future revenue growth

 7. Sales Agent Performance
 Ranking of agents based on closed deals & revenue
 Win-rate comparison for different agents

 9. Product Performance Analysis
 Top-selling products by revenue and deal count
 Comparison of sales across different product series

 11. Industry & Regional Insights
 Revenue contribution by sector
 Performance by regional office


Please Refer the Below Mentioned Link

https://app.powerbi.com/groups/me/reports/e7f5fb18-63eb-449b-b609-83e811b93710/9a47df7ebe3ed0a8bb37?experience=power-bi




