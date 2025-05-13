**📊 Sales Performance Dashboard**
Sales Performance & Customer Retention Through Data-Driven Insights

Domain: Sales Analytics & CRM
Tools Used: SQL, Power BI
🔗https://app.powerbi.com/groups/me/reports/e7f5fb18-63eb-449b-b609-83e811b93710/9a47df7ebe3ed0a8bb37?experience=power-bi

🧩 Problem Statement
Sales teams often struggle with:

Identifying key factors influencing deal success

Understanding reasons for lost deals

Optimizing pipeline efficiency

Predicting customer churn

Uncovering seasonal trends

This project addresses these challenges using SQL for backend data processing and Power BI for data storytelling.

🎯 Objectives
Analyze Sales Performance
→ Identify top-performing agents, best-selling products, and high-value deals

Understand Deal Success & Failure
→ Explore reasons behind deal loss and conversion rates

Predict Churn Risk
→ Detect accounts at risk using engagement and loss trends

Detect Seasonal Sales Trends
→ Analyze revenue fluctuations across time periods

Enhance Decision-Making
→ Drive strategic decisions using insights from SQL and Power BI


🧪 Dataset Tables & Structure
1. accounts
sql
Copy
Edit
CREATE TABLE accounts (
    account VARCHAR(255) PRIMARY KEY,
    sector VARCHAR(255),
    year_established INT,
    revenue DECIMAL(15, 2),
    employees INT,
    office_location VARCHAR(255),
    subsidiary_of VARCHAR(255)
);
2. products
sql
Copy
Edit
CREATE TABLE products (
    product VARCHAR(255) PRIMARY KEY,
    series VARCHAR(255),
    sales_price DECIMAL(10, 2)
);
3. sales_teams
sql
Copy
Edit
CREATE TABLE sales_teams (
    sales_agent VARCHAR(255) PRIMARY KEY,
    manager VARCHAR(255),
    regional_office VARCHAR(255)
);
4. sales_pipeline
sql
Copy
Edit
CREATE TABLE sales_pipeline (
    opportunity_id VARCHAR(20) PRIMARY KEY,
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
Data Import: Data loaded via BULK INSERT from CSV files with proper delimiters and error handling.

🧠 SQL Analysis Highlights
🎯 1) **Predicting High-Value Deals Using Revenue Percentile Analysis **
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

📌 Conclusion
This project offers a comprehensive look into sales performance, deal success patterns, and churn prediction. The combination of SQL-based insights and Power BI visualization enables business leaders to make strategic, data-backed decisions.

If you’d like, I can help you:

Generate a README.md file from this format

Upload this structure into your GitHub repo

Create Power BI screenshots for your visuals/ folder


