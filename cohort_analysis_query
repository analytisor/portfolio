-- Step 1: Create or replace the cohorts table
CREATE OR REPLACE TABLE ECOM.cohorts AS

-- Step 2: Aggregate raw sales data
WITH customer_purchases AS (
    SELECT 
        CustomerID,
        InvoiceDate,
        DATE_TRUNC(CAST(InvoiceDate AS DATE), MONTH) as InvoiceMonth,  -- Truncate InvoiceDate to the month level
        Quantity * UnitPrice as TotalPrice  -- Calculate total price for each invoice
    FROM 
        `ECOM.sales`
),

-- Step 3: Identify the first purchase month for each customer
customer_cohorts AS (
    SELECT 
        CustomerID,
        MIN(InvoiceMonth) as FirstPurchaseMonth  -- Find the first purchase month for each customer
    FROM 
        customer_purchases
    GROUP BY 
        CustomerID
),

-- Step 4: Calculate months after the first purchase and total price
cohort_analysis AS (
    SELECT 
        p.CustomerID,
        c.FirstPurchaseMonth,
        DATE_DIFF(CAST(p.InvoiceDate AS DATE), CAST(c.FirstPurchaseMonth AS DATE), MONTH) as MonthsAfterFirstPurchase,  -- Calculate the difference in months from the first purchase
        p.TotalPrice
    FROM 
        customer_purchases p
    JOIN 
        customer_cohorts c ON p.CustomerID = c.CustomerID
),

-- Step 5: Aggregate cohort data
aggregated_cohorts AS (
    SELECT 
        FirstPurchaseMonth,
        MonthsAfterFirstPurchase,
        COUNT(DISTINCT CustomerID) as Customers,  -- Count the number of unique customers
        SUM(TotalPrice) as TotalRevenue  -- Sum the total revenue
    FROM 
        cohort_analysis
    GROUP BY 
        FirstPurchaseMonth, MonthsAfterFirstPurchase
),

-- Step 6: Calculate cumulative metrics for each cohort
cumulative_data AS (
    SELECT 
        FirstPurchaseMonth,
        MonthsAfterFirstPurchase,
        MAX(CASE WHEN MonthsAfterFirstPurchase BETWEEN 1 AND 3 THEN Customers END) 
            OVER(PARTITION BY FirstPurchaseMonth ORDER BY MonthsAfterFirstPurchase) as CumulativeCustomers1To3,  -- Cumulative customers retained between 1 to 3 months
        SUM(TotalRevenue) OVER(PARTITION BY FirstPurchaseMonth ORDER BY MonthsAfterFirstPurchase) as CumulativeRevenue  -- Cumulative revenue for each cohort
    FROM 
        aggregated_cohorts
)

-- Step 7: Select final cohort analysis data
SELECT 
    a.FirstPurchaseMonth,
    a.MonthsAfterFirstPurchase,
    a.Customers,
    FIRST_VALUE(a.Customers) OVER(PARTITION BY a.FirstPurchaseMonth ORDER BY a.MonthsAfterFirstPurchase) as CohortSize,  -- Size of the initial cohort
    SAFE_DIVIDE(a.Customers, FIRST_VALUE(a.Customers) OVER(PARTITION BY a.FirstPurchaseMonth ORDER BY a.MonthsAfterFirstPurchase)) as RetentionRate,  -- Calculate retention rate
    c.CumulativeCustomers1To3,
    c.CumulativeRevenue,
    SAFE_DIVIDE(c.CumulativeRevenue, FIRST_VALUE(a.Customers) OVER(PARTITION BY a.FirstPurchaseMonth ORDER BY a.MonthsAfterFirstPurchase)) as LTV  -- Calculate lifetime value (LTV)
FROM 
    aggregated_cohorts a
JOIN 
    cumulative_data c ON a.FirstPurchaseMonth = c.FirstPurchaseMonth AND a.MonthsAfterFirstPurchase = c.MonthsAfterFirstPurchase
ORDER BY 
    a.FirstPurchaseMonth, a.MonthsAfterFirstPurchase;
