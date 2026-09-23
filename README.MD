Markdown
# PLSQL Assignment 1: Sunrise Supermarket Analysis

**Student Name:** Uwajeneza Divine  
**Student ID:** 20251SEN234  
**DBMS Used:** MySQL / MariaDB (via XAMPP & phpMyAdmin)  
**Repository Name:** assignment_1_Uwajeneza_Divine-20251SEN234  

---

## 1. Project Overview & Business Scenario

**Sunrise Supermarket** sells products to customers who place orders containing one or more items. Management wants to understand customer purchasing behaviors, popular product categories, revenue trends over time, and repeat purchase patterns.

This project sets up the database schema, populates it with realistic store records, and uses SQL JOINs, Common Table Expressions (CTEs), and Window Functions to generate detailed insights for business decisions.

---

## 2. Database Setup & Sample Data

```sql
-- 1. Create Tables
CREATE TABLE customers (
    customer_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_name VARCHAR(100),
    email VARCHAR(100),
    city VARCHAR(50)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    product_name VARCHAR(100),
    category VARCHAR(50),
    price DECIMAL(10,2)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT,
    order_date DATE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

CREATE TABLE order_items (
    order_item_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT,
    product_id INT,
    quantity INT,
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- 2. Insert Customers (Includes 1 customer without orders)
INSERT INTO customers (customer_name, email, city) VALUES
('Alice Mutoni', 'alice@gmail.com', 'Kigali'),
('Bob Mugisha', 'bob@yahoo.com', 'Huye'),
('Charlie Keza', 'charlie@gmail.com', 'Kigali'),
('Diana Uwase', 'diana@outlook.com', 'Musanze'),
('Eric Manzi', 'eric@gmail.com', 'Rubavu'),
('Fiona Irakoze', 'fiona@gmail.com', 'Kigali');

-- 3. Insert Products
INSERT INTO products (product_name, category, price) VALUES
('Fresh Milk 1L', 'Dairy', 1.50),
('Cheddar Cheese 250g', 'Dairy', 4.00),
('White Bread', 'Bakery', 2.00),
('Chocolate Cake', 'Bakery', 15.00),
('Apple 1kg', 'Produce', 3.00),
('Banana 1kg', 'Produce', 1.20),
('Orange Juice 1L', 'Beverages', 2.50),
('Basmati Rice 5kg', 'Pantry', 12.00);

-- 4. Insert 15 Orders
INSERT INTO orders (customer_id, order_date) VALUES
(1, '2026-09-01'), (2, '2026-09-02'), (1, '2026-09-03'), (3, '2026-09-04'),
(4, '2026-09-05'), (2, '2026-09-06'), (5, '2026-09-07'), (1, '2026-09-08'),
(3, '2026-09-09'), (4, '2026-09-10'), (2, '2026-09-11'), (5, '2026-09-12'),
(1, '2026-09-13'), (3, '2026-09-14'), (4, '2026-09-15');

-- 5. Insert 25 Order Items
INSERT INTO order_items (order_id, product_id, quantity) VALUES
(1, 1, 2), (1, 3, 1), (2, 4, 1), (2, 8, 2),
(3, 2, 3), (4, 5, 5), (4, 7, 2), (5, 8, 1),
(5, 1, 4), (6, 3, 2), (7, 4, 2), (7, 6, 10),
(8, 2, 1), (8, 7, 3), (9, 5, 2), (9, 3, 3),
(10, 1, 5), (10, 8, 1), (11, 4, 1), (11, 6, 5),
(12, 2, 2), (13, 2, 1), (14, 5, 4), (15, 7, 2), (15, 1, 2);
3. SQL Queries & Plain English Explanations
Question 1: Order List with Customer Details
What it answers: Which customer placed each order, where do they live, and when was the purchase made?

Explanation: Links orders to customers using an INNER JOIN on customer_id.

SQL
SELECT 
    o.order_id, 
    c.customer_name, 
    c.city, 
    o.order_date 
FROM orders o 
INNER JOIN customers c ON o.customer_id = c.customer_id;
Question 2: Item Breakdown per Order
What it answers: What specific products were bought in each order, their category, unit price, and quantity?

Explanation: Connects order_items with products using an INNER JOIN on product_id.

SQL
SELECT 
    oi.order_item_id, 
    oi.order_id, 
    p.product_name, 
    p.category, 
    p.price, 
    oi.quantity 
FROM order_items oi 
INNER JOIN products p ON oi.product_id = p.product_id;
Question 3: Complete Customer Order History
What it answers: All registered customers and their orders, including customers who haven't placed an order yet.

Explanation: Uses a LEFT JOIN starting from customers to keep inactive profiles (like Fiona Irakoze) in the output.

SQL
SELECT 
    c.customer_id, 
    c.customer_name, 
    o.order_id, 
    o.order_date 
FROM customers c 
LEFT JOIN orders o ON c.customer_id = o.customer_id;
Question 4: Customers Spending Above Average
What it answers: Who are the high-value customers whose total spending exceeds the store's average customer spend?

Explanation: Uses a CTE to calculate total spent per customer (SUM(quantity * price)), then filters for totals above the global average.

SQL
WITH customer_spending AS (
    SELECT 
        c.customer_id, 
        c.customer_name, 
        IFNULL(SUM(oi.quantity * p.price), 0) AS total_spent
    FROM customers c
    LEFT JOIN orders o ON c.customer_id = o.customer_id
    LEFT JOIN order_items oi ON o.order_id = oi.order_id
    LEFT JOIN products p ON oi.product_id = p.product_id
    GROUP BY c.customer_id, c.customer_name
)
SELECT 
    customer_id, 
    customer_name, 
    total_spent 
FROM customer_spending
WHERE total_spent > (SELECT AVG(total_spent) FROM customer_spending);
Question 5: Rank Customers by Total Amount Spent
What it answers: What is every customer's relative rank based on total spending, starting from highest?

Explanation: Computes spending per customer inside a CTE and applies DENSE_RANK() ordered by total_spent DESC.

SQL
WITH customer_totals AS (
    SELECT 
        c.customer_id, 
        c.customer_name, 
        IFNULL(SUM(oi.quantity * p.price), 0) AS total_spent
    FROM customers c
    LEFT JOIN orders o ON c.customer_id = o.customer_id
    LEFT JOIN order_items oi ON o.order_id = oi.order_id
    LEFT JOIN products p ON oi.product_id = p.product_id
    GROUP BY c.customer_id, c.customer_name
)
SELECT 
    customer_id, 
    customer_name, 
    total_spent, 
    DENSE_RANK() OVER (ORDER BY total_spent DESC) AS spend_rank
FROM customer_totals;
Question 6: Sequential Order Numbering Per Customer
What it answers: Which order was a customer's 1st, 2nd, or 3rd purchase chronologically?

Explanation: Applies ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date) to number orders per customer.

SQL
SELECT 
    o.customer_id, 
    o.order_id, 
    o.order_date, 
    ROW_NUMBER() OVER (
        PARTITION BY o.customer_id 
        ORDER BY o.order_date
    ) AS order_sequence
FROM orders o;
Question 7: Running Revenue Total over Time
What it answers: How is total store revenue accumulating day by day as orders come in?

Explanation: Calculates daily order totals and uses a running window function (SUM(...) OVER (ORDER BY order_date)) to keep a running sum.

SQL
WITH order_totals AS (
    SELECT 
        o.order_id, 
        o.order_date, 
        SUM(oi.quantity * p.price) AS order_revenue
    FROM orders o
    JOIN order_items oi ON o.order_id = oi.order_id
    JOIN products p ON oi.product_id = p.product_id
    GROUP BY o.order_id, o.order_date
)
SELECT 
    order_id, 
    order_date, 
    order_revenue, 
    SUM(order_revenue) OVER (ORDER BY order_date) AS running_total_revenue
FROM order_totals;
Question 8: Days Between Repeat Purchases
What it answers: For repeat buyers, how many days passed between their previous order and their current order?

Explanation: Uses LAG(order_date) to look back at the previous visit date per customer and subtracts it from order_date.

SQL
SELECT 
    o.customer_id, 
    o.order_id, 
    o.order_date, 
    LAG(o.order_date) OVER (
        PARTITION BY o.customer_id 
        ORDER BY o.order_date
    ) AS previous_order_date,
    DATEDIFF(
        o.order_date, 
        LAG(o.order_date) OVER (
            PARTITION BY o.customer_id 
            ORDER BY o.order_date
        )
    ) AS days_since_last_order
FROM orders o;
Markdown
---

## 4. Query Results & Screenshots

### Question 1: Order List with Customer Details
![Query 1 Result](01_orders_customer.png.jpg)

### Question 2: Order Items Breakdown
![Query 2 Result](02_orderItems_product.png.jpg)

### Question 3: Complete Customer Order History
![Query 3 Result](03_customer_order_date.png.jpg)

### Question 4: Spending Above Average
![Query 4 Result](04_cutomer_totalSpent.png.png)

### Question 6: Order Sequence per Customer
![Query 6 Result](06_customer_order_placed.png.jpg)

### Question 7: Running Revenue Total
![Query 7 Result](07_order_date_total_revenue.png.jpg)

### Question 8: Days Between Repeat Orders
![Query 8 Result](08_previous_last_ordes.png.jpg)

---

## 5. Business Interpretation

1. **VIP Spenders Drive Profitability:** Top spenders like Charlie Keza ($68.00) and Bob Mugisha ($65.50) generate a substantial share of total revenue. Creating a VIP loyalty discount program will help keep these valuable shoppers.
2. **Consistent Purchase Intervals:** Repeat customers return consistently every 4 to 5 days. Management can schedule marketing promotions every 3–4 days to encourage frequent shopping visits.
3. **Re-engaging Inactive Profiles:** Using `LEFT JOIN` exposed registered customers with zero orders (e.g., Fiona Irakoze). Sunrise Supermarket can send an introductory discount code to convert them into buyers.

---

## 6. Challenges Encountered & Resolutions

* **Challenge 1: Handling NULL values for inactive customers.**  
  * **Resolution:** Used MySQL's `IFNULL(..., 0)` function to ensure inactive customers show `$0.00` instead of blank values in spending reports.
* **Challenge 2: Calculating date differences in MySQL vs PL/SQL.**  
  * **Resolution:** Replaced standard arithmetic subtraction with MySQL's native `DATEDIFF(order_date, previous_order_date)` function inside the `LAG()` query.

---
⬆