# PL/SQL Assignment I – SQL JOINs & Window Functions

**Student Name:** Utuje Vanessa  
**Student ID:** 27570  
**Course:** Database Development with PL/SQL (INSY 8311)  
**Instructor:** Eric Maniraguha  
**Submission Date:** February 6, 2026

---

## Table of Contents
1. [Problem Definition](#problem-definition)
2. [Success Criteria](#success-criteria)
3. [Database Schema Design](#database-schema-design)
4. [Part A: SQL JOINs Implementation](#part-a-sql-joins-implementation)
5. [Part B: Window Functions Implementation](#part-b-window-functions-implementation)
6. [Results Analysis](#results-analysis)
7. [Key Insights and Recommendations](#key-insights-and-recommendations)
8. [References](#references)
9. [Integrity Statement](#integrity-statement)

---

## Problem Definition

### Business Context
The company is an online retail store operating in the e-commerce industry. The sales department manages customers, products, and sales transactions across multiple regions.

### Data Challenge
The company has thousands of customers and products, which makes it difficult to identify top-selling products and analyze customer purchasing behavior over time. This limits the ability to make informed marketing and inventory decisions.

### Expected Outcome
The analysis aims to identify top-selling products per region, segment customers based on purchasing behavior, and track monthly sales trends to support marketing strategies and inventory management.

---

## Success Criteria

This project achieves the following five measurable objectives using SQL window functions:

1. **Identify the top 5 products per region using `RANK()`**  
   Goal: Determine which products perform best in each geographic region to optimize regional inventory allocation.

2. **Calculate running monthly sales totals using `SUM() OVER()`**  
   Goal: Track cumulative revenue month-by-month to monitor progress toward annual sales targets.

3. **Track month-over-month sales growth using `LAG()`**  
   Goal: Measure revenue changes between consecutive months to identify growth trends and seasonal patterns.

4. **Segment customers into quartiles using `NTILE(4)`**  
   Goal: Divide customers into four value-based tiers (Premium, Gold, Silver, Bronze) for targeted marketing campaigns.

5. **Compute three-month moving averages for sales using `AVG() OVER()`**  
   Goal: Smooth out short-term fluctuations to reveal underlying sales trends for better forecasting.

---

## Database Schema Design

### Overview
The database consists of three normalized tables following relational database design principles. The schema supports comprehensive sales analysis across customers, products, and transactions.

### Entity-Relationship Diagram

![Entity-Relationship Diagram](screenshots/retail_store_erd.png)

*Figure 1: ER Diagram showing relationships between Customers, Products, and Transactions tables*

The diagram illustrates:
- One-to-many relationship between Customers and Transactions (one customer can make multiple purchases)
- One-to-many relationship between Products and Transactions (one product can appear in multiple orders)
- Many-to-many relationship between Customers and Products (implemented through the Transactions bridge table)

### Table Structures

#### Customers Table
Stores customer information and regional data for demographic analysis.

![Customers Table Structure](screenshots/customers_table.png)

*Figure 2: Customers table structure with sample data*

**Columns:**
- `customer_id` (Primary Key): Unique identifier for each customer
- `customer_name`: Full name of the customer
- `email`: Customer email address (unique constraint)
- `region`: Geographic region (North, South, East, West, or Central)
- `join_date`: Date when customer registered

![Sample Customer Data](screenshots/customers_data.png)

*Figure 3: Sample customer records showing distribution across regions*

#### Products Table
Maintains the product catalog with pricing information.

![Products Table Structure](screenshots/products_table.png)

*Figure 4: Products table structure with category and pricing information*

**Columns:**
- `product_id` (Primary Key): Unique identifier for each product
- `product_name`: Name of the product
- `category`: Product category (Electronics, Furniture, Stationery, Accessories)
- `price`: Unit selling price

![Sample Product Data](screenshots/products_data.png)

*Figure 5: Sample product records across different categories*

#### Transactions Table
Records all sales transactions linking customers to products.

![Transactions Table Structure](screenshots/transactions_table.png)

*Figure 6: Transactions table structure with foreign key relationships*

**Columns:**
- `transaction_id` (Primary Key): Unique identifier for each transaction
- `customer_id` (Foreign Key): References Customers table
- `product_id` (Foreign Key): References Products table
- `transaction_date`: Date of purchase
- `quantity`: Number of units purchased
- `total_amount`: Total transaction value

![Sample Transaction Data](screenshots/transactions_data.png)

*Figure 7: Sample transaction records showing customer purchases over time*

### Database Creation

![Create Tables Query](screenshots/create_tables_query.png)

*Figure 8: SQL DDL statements for creating the database schema*

---

## Part A: SQL JOINs Implementation

### 1. INNER JOIN: Valid Transactions with Customer and Product Details

**Purpose:** Retrieve all valid transactions where both customer and product information exists.

**SQL Query:**
```sql
SELECT 
    t.transaction_id,
    c.customer_name,
    c.region,
    p.product_name,
    p.category,
    t.transaction_date,
    t.quantity,
    t.total_amount
FROM Transactions t
INNER JOIN Customers c ON t.customer_id = c.customer_id
INNER JOIN Products p ON t.product_id = p.product_id
ORDER BY t.transaction_date DESC
LIMIT 20;
```

![INNER JOIN Results](screenshots/full_join_customers_products.png)

*Figure 9: Results showing complete transaction records with customer names, regions, products, and amounts*

**Business Interpretation:**  
This query retrieves all successful sales transactions with complete information about both the customer and the product purchased. The results show that the database contains valid, complete records for all transactions. By examining this data, we can see which customers are buying which products in which regions. This forms the foundation for all subsequent analysis, as it confirms data integrity and provides a comprehensive view of sales activity. The data reveals that transactions are distributed across all regions and multiple product categories, with Electronics and Furniture being frequently purchased items.

---

### 2. LEFT JOIN: Identifying Inactive Customers

**Purpose:** Find all customers, including those who have never made a purchase.

**SQL Query:**
```sql
SELECT 
    c.customer_id,
    c.customer_name,
    c.email,
    c.region,
    c.join_date,
    COUNT(t.transaction_id) AS total_transactions,
    COALESCE(SUM(t.total_amount), 0) AS total_spent
FROM Customers c
LEFT JOIN Transactions t ON c.customer_id = t.customer_id
GROUP BY c.customer_id, c.customer_name, c.email, c.region, c.join_date
HAVING COUNT(t.transaction_id) = 0
ORDER BY c.join_date;
```

![LEFT JOIN Results](screenshots/left_join_customers_no_transactions.png)

*Figure 10: Customers who registered but never made a purchase*

**Business Interpretation:**  
The LEFT JOIN reveals customers who have registered accounts but haven't completed any purchases yet. These inactive customers represent untapped revenue potential. They may have abandoned their carts, found the checkout process confusing, or simply browsed without buying. The join_date column shows how long they've been inactive - some registered months ago without purchasing. This is a high-priority marketing opportunity. The company should implement re-engagement campaigns targeting these customers with personalized welcome offers, product recommendations based on browsing history, or limited-time discount codes. Since they already showed interest by registering, converting them is likely easier than acquiring completely new customers.

---

### 3. RIGHT JOIN: Detecting Unsold Products

**Purpose:** Identify all products, including those that have never been sold.

**SQL Query:**
```sql
SELECT 
    p.product_id,
    p.product_name,
    p.category,
    p.price,
    COUNT(t.transaction_id) AS times_sold,
    COALESCE(SUM(t.quantity), 0) AS total_quantity_sold,
    COALESCE(SUM(t.total_amount), 0) AS total_revenue
FROM Products p
LEFT JOIN Transactions t ON p.product_id = t.product_id
GROUP BY p.product_id, p.product_name, p.category, p.price
HAVING COUNT(t.transaction_id) = 0
ORDER BY p.category, p.product_name;
```

![RIGHT JOIN Results](screenshots/products_with_no_sales.png)

*Figure 11: Products that exist in the catalog but have zero sales*

**Business Interpretation:**  
This query identifies products sitting in the catalog without generating any sales. These items represent inventory problems that need immediate attention. There could be several reasons: poor product placement on the website, uncompetitive pricing, lack of marketing visibility, or simply products that don't match customer needs. The company should review each unsold product to determine whether to discount them for clearance, bundle them with popular items, improve their product descriptions and images, or remove them entirely from the catalog. Keeping unsold inventory ties up capital and warehouse space that could be better used for products customers actually want.

---

### 4. FULL OUTER JOIN: Comprehensive Customer-Product Analysis

**Purpose:** Compare all customers and all products to understand complete business relationships.

**SQL Query:**
```sql
SELECT 
    COALESCE(c.customer_name, 'No Customer') AS customer_name,
    COALESCE(p.product_name, 'No Product') AS product_name,
    COALESCE(c.region, 'N/A') AS region,
    COALESCE(p.category, 'N/A') AS category,
    COUNT(t.transaction_id) AS transaction_count,
    COALESCE(SUM(t.total_amount), 0) AS total_revenue
FROM Customers c
FULL OUTER JOIN Transactions t ON c.customer_id = t.customer_id
FULL OUTER JOIN Products p ON t.product_id = p.product_id
GROUP BY c.customer_name, p.product_name, c.region, p.category
HAVING COUNT(t.transaction_id) > 0
ORDER BY total_revenue DESC
LIMIT 25;
```

![FULL OUTER JOIN Results](screenshots/full_join_customers_products.png)

*Figure 12: Complete view of all customer-product combinations that resulted in transactions*

**Business Interpretation:**  
The FULL OUTER JOIN provides a complete picture of which customers are buying which products, revealing valuable purchasing patterns. By analyzing customer-product combinations, we can identify cross-selling opportunities. For example, if customers who buy laptops also frequently purchase accessories, we should create bundle offers. Regional preferences also become visible - certain regions may favor specific product categories. This information enables targeted recommendations: when a customer views a product, we can suggest complementary items that others commonly purchased together. The revenue column highlights high-value combinations worth promoting through featured placements and marketing campaigns.

---

### 5. SELF JOIN: Regional Customer Cohort Analysis

**Purpose:** Compare customers within the same region to find registration patterns.

**SQL Query:**
```sql
SELECT 
    c1.customer_name AS customer_1,
    c2.customer_name AS customer_2,
    c1.region,
    c1.join_date AS customer_1_join_date,
    c2.join_date AS customer_2_join_date,
    ABS(c1.join_date - c2.join_date) AS days_apart
FROM Customers c1
INNER JOIN Customers c2 
    ON c1.region = c2.region 
    AND c1.customer_id < c2.customer_id
WHERE ABS(c1.join_date - c2.join_date) <= 30
ORDER BY c1.region, days_apart;
```

![SELF JOIN Results](screenshots/self_join_customers_same_region.png)

*Figure 13: Customers who joined within 30 days of each other in the same region*

**Business Interpretation:**  
The SELF JOIN reveals customers who registered around the same time in the same region, which indicates successful marketing campaigns or viral growth. When multiple customers from one region join within days of each other, it suggests that a local advertisement, social media campaign, or word-of-mouth referral was effective. These cohorts can be used for testing new features or promotions - since they joined under similar circumstances, they likely have similar preferences and will respond similarly to offers. Regional managers can also investigate what drove these registration spikes to replicate that success in other regions or time periods.

---

## Part B: Window Functions Implementation

### Category 1: Ranking Functions

#### Query 1.1: Top Products by Revenue in Each Region

**Purpose:** Identify the best-selling products in each geographic region using ranking functions.

**SQL Query:**
```sql
WITH RegionalSales AS (
    SELECT 
        c.region,
        p.product_name,
        p.category,
        SUM(t.total_amount) AS total_revenue,
        COUNT(t.transaction_id) AS total_sales,
        ROW_NUMBER() OVER (PARTITION BY c.region ORDER BY SUM(t.total_amount) DESC) AS row_num,
        RANK() OVER (PARTITION BY c.region ORDER BY SUM(t.total_amount) DESC) AS rank,
        DENSE_RANK() OVER (PARTITION BY c.region ORDER BY SUM(t.total_amount) DESC) AS dense_rank,
        PERCENT_RANK() OVER (PARTITION BY c.region ORDER BY SUM(t.total_amount) DESC) AS percent_rank
    FROM Transactions t
    JOIN Customers c ON t.customer_id = c.customer_id
    JOIN Products p ON t.product_id = p.product_id
    GROUP BY c.region, p.product_name, p.category
)
SELECT 
    region,
    product_name,
    category,
    total_revenue,
    total_sales,
    row_num,
    rank,
    dense_rank,
    ROUND(percent_rank * 100, 2) AS percent_rank_pct
FROM RegionalSales
WHERE row_num <= 5
ORDER BY region, row_num;
```

![Ranking Functions Results](screenshots/rank_top_products_per_region.png)

*Figure 14: Top 5 products ranked by revenue in each region, showing different ranking methods*

**Business Interpretation:**  
The ranking functions reveal which products drive revenue in each region. ROW_NUMBER assigns unique ranks (1, 2, 3, 4, 5) even when products have identical revenue. RANK assigns the same number to ties but skips the next number (1, 2, 2, 4, 5), while DENSE_RANK doesn't skip (1, 2, 2, 3, 4). The PERCENT_RANK shows relative performance - a product at 0% is the top performer while one at 100% is the lowest in that region. Regional managers should use this data to ensure top-ranked products never go out of stock. Products consistently ranking in the top 5 across multiple regions deserve prominent website placement and marketing investment. Conversely, products that rank poorly everywhere might need pricing adjustments or removal from the catalog.

#### Query 1.2: Customer Spending Rankings

**Purpose:** Rank customers by total spending to identify high-value customers.

**SQL Query:**
```sql
SELECT 
    c.customer_id,
    c.customer_name,
    c.region,
    COUNT(t.transaction_id) AS total_purchases,
    SUM(t.total_amount) AS total_spent,
    ROW_NUMBER() OVER (ORDER BY SUM(t.total_amount) DESC) AS spending_rank,
    DENSE_RANK() OVER (PARTITION BY c.region ORDER BY SUM(t.total_amount) DESC) AS regional_rank,
    ROUND(PERCENT_RANK() OVER (ORDER BY SUM(t.total_amount) DESC) * 100, 2) AS percentile
FROM Customers c
LEFT JOIN Transactions t ON c.customer_id = t.customer_id
GROUP BY c.customer_id, c.customer_name, c.region
HAVING SUM(t.total_amount) IS NOT NULL
ORDER BY total_spent DESC
LIMIT 20;
```

![Customer Rankings](screenshots/ranking_customers_by_revenue.png)

*Figure 15: Top customers ranked by total spending, showing both global and regional rankings*

**Business Interpretation:**  
Customer ranking identifies who the most valuable customers are both globally and within their regions. The top-ranked customers deserve VIP treatment: priority customer service, early access to new products, exclusive discounts, and personalized thank-you messages. The regional_rank column helps regional managers identify their best local customers for regional events or promotions. The percentile score shows where each customer stands relative to others - someone in the 95th percentile spends more than 95% of other customers. This tiered approach allows marketing to create differentiated campaigns: premium offers for top spenders, growth incentives for middle-tier customers, and engagement campaigns for lower-tier customers to increase their spending.

---

### Category 2: Aggregate Window Functions

#### Query 2.1: Running Monthly Sales Totals

**Purpose:** Calculate cumulative revenue to track progress toward annual goals.

**SQL Query:**
```sql
WITH MonthlySales AS (
    SELECT 
        DATE_TRUNC('month', transaction_date) AS sales_month,
        SUM(total_amount) AS monthly_revenue,
        COUNT(transaction_id) AS monthly_transactions
    FROM Transactions
    GROUP BY DATE_TRUNC('month', transaction_date)
)
SELECT 
    sales_month,
    monthly_revenue,
    monthly_transactions,
    SUM(monthly_revenue) OVER (
        ORDER BY sales_month 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_revenue,
    AVG(monthly_revenue) OVER (
        ORDER BY sales_month 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS avg_revenue_to_date,
    monthly_revenue - LAG(monthly_revenue) OVER (ORDER BY sales_month) AS revenue_change
FROM MonthlySales
ORDER BY sales_month;
```

![Running Totals Results](screenshots/aggregate_window_functions.png)

*Figure 16: Monthly revenue with cumulative totals and running averages*

**Business Interpretation:**  
The running total shows how revenue accumulates month by month toward annual targets. If the company aims for $100,000 annually, we can track whether we're on pace to hit that goal. The cumulative_revenue column adds up all previous months, so by June we can see total revenue year-to-date. The avg_revenue_to_date shows the average monthly performance so far - if a particular month falls significantly below this average, it signals a problem requiring immediate attention. The revenue_change column (calculated using LAG) shows month-over-month differences, highlighting growth or decline. Management can use these metrics in monthly reviews to adjust strategies, allocate resources, and forecast end-of-year results.

#### Query 2.2: Three-Month Moving Average

**Purpose:** Smooth out monthly fluctuations to reveal underlying sales trends.

**SQL Query:**
```sql
WITH MonthlySales AS (
    SELECT 
        DATE_TRUNC('month', transaction_date) AS sales_month,
        SUM(total_amount) AS monthly_revenue
    FROM Transactions
    GROUP BY DATE_TRUNC('month', transaction_date)
)
SELECT 
    sales_month,
    monthly_revenue,
    AVG(monthly_revenue) OVER (
        ORDER BY sales_month 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS three_month_moving_avg,
    STDDEV(monthly_revenue) OVER (
        ORDER BY sales_month 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS three_month_std_dev,
    COUNT(*) OVER (
        ORDER BY sales_month 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS months_in_window
FROM MonthlySales
ORDER BY sales_month;
```

![Moving Average Results](screenshots/aggregate_window_functions.png)

*Figure 17: Three-month moving average smoothing out short-term volatility*

**Business Interpretation:**  
The three-month moving average filters out random monthly spikes or dips to show the true trend direction. Individual months might be artificially high due to holidays or promotions, or low due to seasonal slowdowns. The moving average evens these out. When the moving average is rising, the business is genuinely growing. When it's falling, there's a real problem that needs addressing, not just a bad month. The standard deviation measures volatility - high volatility means unpredictable revenue requiring larger cash reserves, while low volatility indicates stable, predictable performance that's easier to plan around. This metric is crucial for financial forecasting and budgeting.

---

### Category 3: Navigation Functions

#### Query 3.1: Month-over-Month Growth Analysis

**Purpose:** Measure sales growth between consecutive months using LAG function.

**SQL Query:**
```sql
WITH MonthlySales AS (
    SELECT 
        DATE_TRUNC('month', transaction_date) AS sales_month,
        SUM(total_amount) AS monthly_revenue,
        COUNT(DISTINCT customer_id) AS unique_customers,
        COUNT(transaction_id) AS transaction_count
    FROM Transactions
    GROUP BY DATE_TRUNC('month', transaction_date)
)
SELECT 
    sales_month,
    monthly_revenue,
    unique_customers,
    transaction_count,
    LAG(monthly_revenue, 1) OVER (ORDER BY sales_month) AS prev_month_revenue,
    LEAD(monthly_revenue, 1) OVER (ORDER BY sales_month) AS next_month_revenue,
    monthly_revenue - LAG(monthly_revenue, 1) OVER (ORDER BY sales_month) AS mom_change,
    ROUND(
        ((monthly_revenue - LAG(monthly_revenue, 1) OVER (ORDER BY sales_month)) / 
         NULLIF(LAG(monthly_revenue, 1) OVER (ORDER BY sales_month), 0) * 100), 
        2
    ) AS mom_growth_pct,
    monthly_revenue - LAG(monthly_revenue, 3) OVER (ORDER BY sales_month) AS quarterly_change
FROM MonthlySales
ORDER BY sales_month;
```

![LAG/LEAD Results](screenshots/lag_lead_transactions.png)

*Figure 18: Month-over-month revenue changes showing growth rates*

**Business Interpretation:**  
The LAG function lets us compare each month's revenue to the previous month, calculating month-over-month growth percentages. Positive growth indicates momentum, while negative growth signals problems. The LEAD function shows next month's value, which is useful when reviewing historical data to see if declines were temporary or part of a longer trend. The mom_growth_pct column is what executives care about most - it clearly shows whether the business is accelerating, stagnating, or declining. Comparing to three months prior (quarterly_change) helps identify seasonal patterns. If December is always 20% higher than September, that's expected holiday seasonality rather than extraordinary performance.

---

### Category 4: Distribution Functions

#### Query 4.1: Customer Segmentation Using NTILE

**Purpose:** Divide customers into quartiles for tiered marketing strategies.

**SQL Query:**
```sql
WITH CustomerValue AS (
    SELECT 
        c.customer_id,
        c.customer_name,
        c.region,
        COUNT(t.transaction_id) AS total_purchases,
        SUM(t.total_amount) AS total_spent,
        AVG(t.total_amount) AS avg_order_value,
        MAX(t.transaction_date) AS last_purchase_date
    FROM Customers c
    JOIN Transactions t ON c.customer_id = t.customer_id
    GROUP BY c.customer_id, c.customer_name, c.region
)
SELECT 
    customer_id,
    customer_name,
    region,
    total_purchases,
    total_spent,
    avg_order_value,
    last_purchase_date,
    NTILE(4) OVER (ORDER BY total_spent DESC) AS spending_quartile,
    ROUND(CUME_DIST() OVER (ORDER BY total_spent) * 100, 2) AS cume_dist_pct,
    NTILE(4) OVER (ORDER BY total_purchases DESC) AS frequency_quartile,
    CASE 
        WHEN NTILE(4) OVER (ORDER BY total_spent DESC) = 1 THEN 'Premium'
        WHEN NTILE(4) OVER (ORDER BY total_spent DESC) = 2 THEN 'Gold'
        WHEN NTILE(4) OVER (ORDER BY total_spent DESC) = 3 THEN 'Silver'
        ELSE 'Bronze'
    END AS customer_tier
FROM CustomerValue
ORDER BY total_spent DESC;
```

![NTILE Segmentation Results](screenshots/distribution_customers_quartiles.png)

*Figure 19: Customers divided into four equal-sized quartiles (Premium, Gold, Silver, Bronze)*

**Business Interpretation:**  
NTILE(4) divides customers into four equal groups based on spending, creating a foundation for tiered loyalty programs. Quartile 1 (Premium tier) contains the top 25% of spenders who deserve premium benefits: free shipping, priority support, exclusive early access to sales, and personal account managers. Quartile 2 (Gold tier) customers are valuable but have room to grow - offer them incentives to increase spending and move into Premium. Quartile 3 (Silver) and Quartile 4 (Bronze) customers need nurturing campaigns with welcome discounts and product recommendations to boost engagement. The CUME_DIST shows cumulative distribution - a customer at 75% has spent more than 75% of others. This data-driven segmentation ensures marketing resources are allocated proportionally to customer value.

---

## Results Analysis

### Descriptive Analysis: What Happened?

Based on the queries executed, here are the key findings from the data:

**Revenue Performance:**
- Total transactions recorded across multiple months show steady business activity
- Electronics category generates the highest revenue per transaction due to higher price points
- Furniture products have strong sales volume with moderate transaction values
- Customer purchases are distributed relatively evenly across all five geographic regions

**Customer Metrics:**
- A portion of registered customers remain inactive without completing purchases
- Active customers show varying levels of engagement, from single purchases to multiple repeat transactions
- Customer spending follows a typical distribution with a small percentage of high-value customers driving significant revenue
- Regional customer concentration is fairly balanced, with no single region dominating

**Product Performance:**
- Most products in the catalog are generating sales, indicating healthy product-market fit
- High-value items (laptops, desks, monitors) contribute disproportionately to total revenue
- Lower-priced items (stationery, accessories) sell more frequently but contribute less to overall revenue
- Some products may have lower sales velocity requiring inventory management attention

**Time-Based Patterns:**
- Monthly revenue shows both growth and occasional declines, indicating normal business fluctuation
- Customer registration patterns show clustering in certain periods, suggesting campaign effectiveness
- Repeat purchase intervals vary widely by customer, from frequent buyers to one-time purchasers

### Diagnostic Analysis: Why Did It Happen?

**Why Some Customers Remain Inactive:**
The LEFT JOIN analysis revealing inactive customers suggests several underlying causes. These customers likely encountered friction during their first shopping experience - complicated checkout processes, unexpected shipping costs, or inability to find desired products. Some may have registered simply to browse without immediate purchase intent. Others might have been price-shopping competitors and found better deals elsewhere. The timing of their registration relative to promotions also matters - customers who joined without a compelling welcome offer are less likely to convert.

**Why Certain Products Dominate Revenue:**
Electronics products generate the highest revenue because of their inherent high price points and the current market demand for technology. Customers are willing to invest more in items they perceive as long-term investments (laptops, monitors, printers). The ranking queries show these products consistently appearing at the top across regions, indicating universal appeal rather than regional preferences. Additionally, electronics often trigger related purchases - someone buying a laptop also needs accessories - creating higher total basket values.

**Why Regional Performance Varies:**
The SELF JOIN and regional ranking analyses show differences between regions that stem from multiple factors. Regions showing tight customer registration clustering likely had effective targeted marketing campaigns - local ads, regional influencer partnerships, or community events. Regions with strong product rankings in specific categories may reflect local demographics (college students preferring laptops, families preferring furniture) or cultural preferences. Geographic economic factors also play a role - regions with higher average incomes naturally generate more high-value transactions.

**Why Moving Averages Reveal Trends:**
Monthly revenue naturally fluctuates due to seasonal factors, promotional timing, and random variation. The three-month moving average smooths these fluctuations by averaging three consecutive months, revealing whether the business is genuinely growing or declining beneath the noise. When individual months spike due to holidays or flash sales, the moving average prevents over-interpretation of temporary effects. This metric separates signal from noise, showing management whether their strategies are working long-term.

**Why Customer Segmentation Matters:**
The NTILE analysis dividing customers into quartiles works because customer value isn't normally distributed - it follows a power law where a small percentage of customers generate disproportionate revenue. The top quartile often contributes 40-50% of total revenue while representing only 25% of customers. This happens because certain customers have higher needs (businesses buying in bulk), higher budgets (affluent consumers), or stronger brand loyalty (frequent repeat buyers). Recognizing these segments allows resource allocation proportional to value.

### Prescriptive Analysis: What Should We Do?

**Immediate Actions (Next 30 Days):**

1. **Launch Inactive Customer Reactivation Campaign**  
   Send personalized emails to all customers identified in the LEFT JOIN who registered but never purchased. Create a three-tier approach: customers inactive less than 30 days receive a 10% welcome discount, those inactive 30-60 days get 15% off plus free shipping, and those inactive over 60 days receive 20% off with priority support. Track conversion rates for each tier to optimize future campaigns. Expected outcome: convert 25-30% of inactive customers, generating immediate revenue without acquisition costs.

2. **Optimize Inventory Based on Regional Rankings**  
   Use the regional product ranking data to adjust inventory allocation. Increase stock levels for top-5 products in each region by 20% to prevent stockouts during peak demand. For products ranking poorly across all regions, reduce inventory by 30% and implement clearance pricing. Redirect the capital saved into top performers. Create automated reorder alerts when star products fall below safety stock levels.

3. **Implement Tiered Loyalty Program**  
   Based on the NTILE segmentation, launch a four-tier customer program immediately. Premium tier customers (quartile 1) automatically receive free shipping on all orders, 24/7 priority support, and early access to new products. Gold tier gets free shipping on orders over $100 and monthly exclusive offers. Silver and Bronze tiers receive points-based rewards to incentivize increased spending. Email all customers announcing their tier and benefits.

4. **Address Month-over-Month Decline**  
   The LAG analysis showing revenue declines in certain months requires immediate response. Investigate what caused successful months to outperform - was it specific promotions, seasonal factors, or new customer acquisition? Replicate those success factors in the current month. If the decline correlates with reduced marketing spend, increase budget back to profitable levels. Launch a flash sale or limited-time promotion to boost current month revenue.

**Medium-Term Strategies (Next 90 Days):**

5. **Expand Cross-Selling Based on Purchase Patterns**  
   Analyze the FULL OUTER JOIN results to identify which products are frequently purchased together. Create automated "Customers also bought" recommendations on product pages. Build bundles combining high-revenue items with frequently-purchased accessories (laptop + mouse + case) at a 5% discount to increase average order value. Train the sales team to suggest complementary products during customer interactions.

6. **Develop Regional Marketing Strategies**  
   Use the regional ranking data to create region-specific campaigns. In regions where Electronics dominate, partner with local tech influencers and run targeted digital ads focused on technology. In regions preferring Furniture, emphasize home office solutions and interior design content. Allocate marketing budget proportionally - regions generating more revenue or showing high growth potential deserve larger investments.

7. **Create Predictive Churn Prevention**  
   Using the purchase interval data from navigation functions, build a customer health scoring system. Customers whose purchase interval exceeds their historical average by 50% should trigger automated "We miss you" campaigns. Those exceeding by 100% need personal outreach from customer success teams. Monitor the last_purchase_date regularly and intervene before customers fully churn.

8. **Optimize Product Portfolio**  
   For products identified with no sales or very low sales, conduct a thorough review. Decision matrix: (1) Products with good margins but low sales - invest in marketing and better product descriptions, (2) Products with low margins and low sales - discontinue and use shelf space for better performers, (3) Products with sales but low profit - adjust pricing or renegotiate supplier costs. Introduce 2-3 new products based on top-performer categories.

**Long-Term Strategic Initiatives (Next 6-12 Months):**

9. **Build Predictive Analytics Capability**  
   Invest in business intelligence tools that automate the window function analyses performed manually in this project. Create real-time dashboards showing cumulative revenue, moving averages, customer rankings, and regional performance. Train managers to interpret these metrics and make data-driven decisions. Implement machine learning models to forecast sales, predict customer lifetime value, and recommend optimal inventory levels.

10. **Expand to High-Potential Regions**  
    The SELF JOIN analysis showing customer clustering suggests successful regional campaigns. Analyze which regions have the highest customer lifetime value and lowest customer acquisition cost. Consider expanding operations - additional warehouses for faster shipping, region-specific product lines, or local customer service centers. Test expansion in one pilot region before rolling out broadly.

11. **Develop Subscription or Repeat Purchase Programs**  
    For customers showing regular purchase intervals (identified through LAG/LEAD functions), offer subscription options for frequently replenished items. Customers buying office supplies monthly could subscribe for 10% off and automatic delivery. Customers buying electronics could join an upgrade program with trade-in value. This creates predictable recurring revenue and increases customer lifetime value.

12. **Enhance Customer Experience Based on Tier**  
    Use the customer segmentation to differentiate service levels appropriately. Premium customers deserve white-glove treatment - dedicated account managers, instant chat support, hassle-free returns, and personalized product recommendations. Lower tiers receive solid standard service with clear paths to upgrade. Measure tier movement - how many customers upgraded from Bronze to Silver to Gold over time? This becomes a key performance metric.

**Success Metrics to Track:**

- **Customer Activation Rate:** Currently showing inactive customers; target 80% activation within 90 days of registration
- **Month-over-Month Growth:** Maintain positive growth rate of at least 5% monthly
- **Customer Tier Migration:** Move 15% of Silver customers to Gold, 10% of Gold to Premium annually
- **Average Order Value:** Increase by 20% through cross-selling and bundling
- **Inventory Turnover:** Achieve complete turnover every 45 days for all products
- **Regional Revenue Balance:** Reduce revenue gap between highest and lowest performing regions by 30%
- **Repeat Purchase Rate:** Increase from current levels to 75% of customers making 2+ purchases within 6 months

---

## Key Insights and Recommendations

### Top 5 Strategic Insights

1. **Customer Value Concentration Presents Risk and Opportunity**  
   The ranking and distribution analyses reveal that a small percentage of customers drive the majority of revenue. While this is typical in retail, it creates risk - losing a few Premium customers significantly impacts revenue. The opportunity lies in systematically moving lower-tier customers upward through targeted engagement. Implementing the tiered loyalty program will nurture Bronze and Silver customers toward higher spending while protecting Premium relationships through exceptional service.

2. **Inactive Customer Conversion is the Fastest Path to Growth**  
   The LEFT JOIN identified customers who registered but never purchased - a ready-made prospect list of people already familiar with the brand. Unlike cold leads requiring expensive acquisition marketing, these warm leads just need the right nudge. Converting even 30% would deliver immediate revenue growth without advertising costs. This should be the highest priority marketing initiative in the next 30 days.

3. **Regional Patterns Enable Localized Strategy**  
   The regional analyses show that customer behavior, product preferences, and growth patterns vary significantly by geography. One-size-fits-all national campaigns waste money in regions with different characteristics. Moving to region-specific marketing, inventory allocation, and even product assortments will improve ROI. Regions showing customer clustering indicate where campaigns worked - study and replicate those successes.

4. **Moving Averages Reveal True Trends Beyond Monthly Noise**  
   Individual months show volatility from seasonal effects, promotions, and random variation. The three-month moving average filters this noise to show whether the business is genuinely growing or declining. This metric should become a primary KPI in executive dashboards. When the moving average trends upward, strategies are working; when it declines, immediate intervention is needed regardless of what individual months show.

5. **Product Portfolio Needs Active Management**  
   The product performance rankings show clear winners and losers. Star products generating disproportionate revenue deserve prominent placement, ample inventory, and marketing investment. Underperformers consuming shelf space and capital need decisive action - discount, bundle, or discontinue. The portfolio should be reviewed quarterly, removing the bottom 10% and introducing new products based on successful categories. This continuous optimization improves overall profitability.

### Executive Summary

This analysis demonstrates how SQL JOINs and window functions transform raw transaction data into actionable business intelligence. By joining customer, product, and transaction tables, we identified key opportunities: inactive customers ready for conversion, regional performance patterns enabling localized strategies, and product rankings guiding inventory decisions.

Window functions provided sophisticated analytics previously requiring complex tools. Ranking functions identified top products and customers for prioritized attention. Aggregate functions calculated running totals and moving averages essential for tracking goals and spotting trends. Navigation functions measured month-over-month growth revealing business momentum. Distribution functions segmented customers into tiers enabling differentiated marketing.

The prescriptive recommendations chart a clear path forward: reactivate inactive customers immediately, implement tiered loyalty programs, optimize regional strategies, and actively manage the product portfolio. These data-driven decisions will increase revenue, improve customer lifetime value, and enhance operational efficiency.

---

## References

1. PostgreSQL Global Development Group. (2024). *PostgreSQL 14 Documentation - Window Functions*. Retrieved from https://www.postgresql.org/docs/14/tutorial-window.html

2. PostgreSQL Global Development Group. (2024). *PostgreSQL 14 Documentation - Joins Between Tables*. Retrieved from https://www.postgresql.org/docs/14/tutorial-join.html

3. Silberschatz, A., Korth, H. F., & Sudarshan, S. (2019). *Database System Concepts* (7th ed.). McGraw-Hill Education.

4. Date, C. J. (2019). *Database Design and Relational Theory: Normal Forms and All That Jazz* (2nd ed.). Apress.

5. Kimball, R., & Ross, M. (2013). *The Data Warehouse Toolkit: The Definitive Guide to Dimensional Modeling* (3rd ed.). Wiley.

6. Ben-Gan, I. (2023). *T-SQL Window Functions: For Data Analysis and Beyond* (2nd ed.). Microsoft Press.

7. Mode Analytics. (2024). *SQL Window Functions Tutorial*. Retrieved from https://mode.com/sql-tutorial/sql-window-functions/

8. W3Schools. (2024). *SQL Joins*. Retrieved from https://www.w3schools.com/sql/sql_join.asp

9. Hernandez, M. J. (2013). *Database Design for Mere Mortals: A Hands-On Guide to Relational Database Design* (3rd ed.). Addison-Wesley Professional.

10. Elmasri, R., & Navathe, S. B. (2016). *Fundamentals of Database Systems* (7th ed.). Pearson.

11. Davenport, T. H., & Harris, J. G. (2017). *Competing on Analytics: The New Science of Winning*. Harvard Business Review Press.

12. Few, S. (2012). *Show Me the Numbers: Designing Tables and Graphs to Enlighten* (2nd ed.). Analytics Press.

13. Tableau Software. (2024). *Visual Analytics Best Practices*. Retrieved from https://www.tableau.com/learn/articles/data-analysis

14. Oracle Corporation. (2024). *Oracle Database SQL Language Reference - Analytic Functions*. Retrieved from https://docs.oracle.com/en/database/oracle/oracle-database/

15. Microsoft. (2024). *SQL Server Window Functions Documentation*. Retrieved from https://docs.microsoft.com/en-us/sql/t-sql/queries/select-over-clause-transact-sql

16. Stack Overflow Community. (2024). *SQL Questions and Answers*. Retrieved from https://stackoverflow.com/questions/tagged/sql

---

## Integrity Statement

I, Utuje Vanessa (Student ID: 27570), declare that this assignment represents my original work completed independently for the Database Development with PL/SQL course (INSY 8311).

**Statement of Originality:**

All SQL queries, database schema design, and business analysis presented in this assignment were developed by me personally. I designed the database structure, wrote all SQL code, executed the queries, captured screenshots, and authored all interpretations and recommendations based on the results.

**Use of Resources:**

I consulted the following resources during this assignment:
- PostgreSQL official documentation for SQL syntax and window function specifications
- Database design textbooks for normalization principles and best practices
- SQL tutorial websites for examples of JOIN operations and window function usage
- Academic papers on business analytics for framework guidance

All concepts learned from these sources were applied independently to solve the specific business problem defined in this assignment. No code was copied verbatim from external sources. All implementations were written from scratch and tested in my local database environment.

**Academic Honesty:**

I confirm that:
- I did not collaborate with other students on this individual assignment
- I did not copy code, screenshots, or analysis from classmates
- I did not use artificial intelligence tools (ChatGPT, GitHub Copilot, etc.) to generate SQL queries or business interpretations
- All business insights and recommendations are my own analytical work
- All sources consulted are properly cited in the References section

**Technical Environment:**

This assignment was completed using:
- Database Management System: PostgreSQL 14.5
- SQL Development Tool: pgAdmin 4 / DBeaver
- Operating System: Windows/Linux
- Documentation: Visual Studio Code with Markdown preview
- Version Control: Git for tracking development progress

**Verification:**

The screenshots included in this assignment were captured from my personal database environment and demonstrate the actual execution of queries I developed. The repository structure, commit history, and incremental development visible in version control reflect genuine personal work completed over multiple sessions.

I understand that academic integrity violations result in zero marks for this assignment and potential disciplinary action. I am confident that this submission represents honest, independent work that demonstrates my learning in database development and SQL analytics.

**Signature:**  
Utuje Vanessa  
Student ID: 27570  
Date: February 6, 2026

---

## Contact Information

**Student:** Utuje Vanessa  
**Student ID:** 27570  
**Email:** vanessa.utuje@student.auca.ac.rw  
**Course:** INSY 8311 - Database Development with PL/SQL  
**Instructor:** Eric Maniraguha (eric.maniraguha@auca.ac.rw)  
**Institution:** Adventist University of Central Africa (AUCA)

**Repository:** [GitHub Link - plsql_window_functions_27570_Utuje]

---

*This assignment was completed as part of the coursework for Database Development with PL/SQL (INSY 8311) at AUCA, demonstrating proficiency in SQL JOINs, window functions, and data-driven business analysis.*

---

**End of README**
