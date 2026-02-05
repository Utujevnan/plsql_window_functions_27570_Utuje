# PL/SQL Assignment I – SQL JOINs & Window Functions

**Student Name:** Utuje Vanessa  
**Student ID:** 27570  
**Course:** Database Development with PL/SQL (INSY 8311)

---

## Step 1: Problem Definition

### Business Context
The company is an online retail store operating in the e-commerce industry. The sales department manages customers, products, and sales transactions across multiple regions.

### Data Challenge
The company has thousands of customers and products, which makes it difficult to identify top-selling products and analyze customer purchasing behavior over time. This limits the ability to make informed marketing and inventory decisions.

### Expected Outcome
The analysis aims to identify top-selling products per region, segment customers based on purchasing behavior, and track monthly sales trends to support marketing strategies and inventory management.

---

## Step 2: Success Criteria

1. Identify the top 5 products per region using `RANK()`
2. Calculate running monthly sales totals using `SUM() OVER()`
3. Track month-over-month sales growth using `LAG()`
4. Segment customers into quartiles using `NTILE(4)`
5. Compute three-month moving averages for sales using `AVG() OVER()`

---

## Step 3: Database Schema Design

The database schema consists of three related tables designed to support sales analysis.

### Customers
Stores customer information and regional data.
- Primary Key: `customer_id`

### Products
Stores product details and pricing information.
- Primary Key: `product_id`

### Transactions
Stores purchase records linking customers and products.
- Primary Key: `transaction_id`
- Foreign Keys:
  - `customer_id` → Customers(customer_id)
  - `product_id` → Products(product_id)

An ER diagram is included in the repository to illustrate table relationships.

---

