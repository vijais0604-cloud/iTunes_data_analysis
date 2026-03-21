# E-Commerce Analytics Query Documentation

This document provides an overview of the key SQL queries used to analyze patterns and insights from the e-commerce data. Each query is accompanied by a description, usage instructions, and sample interpretations.

## Customer Analysis Queries

### 1. Top Customers by Spending
```sql
SELECT 
    c.customer_id,
    c.first_name,
    c.last_name,
    t.total_spent
FROM customer c
JOIN (
    SELECT customer_id,
           SUM(total) AS total_spent
    FROM invoice
    GROUP BY customer_id
) t
ON c.customer_id = t.customer_id
ORDER BY total_spent DESC LIMIT 20;
```
*Interpretation:* Shows the top 20 customers by total spending, useful for identifying high-value customers.

### 2. Average Customer Lifetime Value
```sql
SELECT 
    AVG(total_spent) AS avg_customer_lifetime_value
FROM customer
JOIN (SELECT customer_id, SUM(total) AS total_spent FROM invoice GROUP BY customer_id) t
ON customer.customer_id = t.customer_id;
```
*Interpretation:* Provides the average revenue a customer generates over their lifetime.

## Purchase Patterns Queries

### 3. Most Purchased Tracks
```sql
SELECT 
    t.track_id,
    t.name AS track_name,
    SUM(i.total) AS total_earned,
    COUNT(il.track_id) AS purchase_count
FROM invoice i
JOIN invoice_line il ON i.invoice_line_id = il.invoice_line_id
JOIN track t ON il.track_id = t.track_id
GROUP BY t.track_id, t.name
ORDER BY total_earned DESC LIMIT 20;
```
*Interpretation:* Identifies the most profitable music tracks based on total earnings.

### 4. Tracks by Genre
```sql
SELECT 
    g.name AS genre_name,
    COUNT(t.track_id) AS track_count
FROM genre g
JOIN track t ON g.genre_id = t.genre_id
GROUP BY g.genre_id, g.name
ORDER BY track_count DESC;
```
*Interpretation:* Shows which genres have the most tracks in the library.

## Customer Segmentation Queries

### 5. Repeat vs One-Time Customers
```sql
SELECT 
    COUNT(*) AS total_customers,
    SUM(CASE WHEN purchase_count > 1 THEN 1 ELSE 0 END) AS repeat_customers,
    SUM(CASE WHEN purchase_count = 1 THEN 1 ELSE 0 END) AS one_time_customers
FROM (
    SELECT customer_id, COUNT(*) AS purchase_count FROM invoice GROUP BY customer_id
) t;
```
*Interpretation:* Helps understand the distribution of first-time vs returning customers.

### 6. Employees vs Customers Relationships
```sql
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    c.customer_id AS customer_id,
    SUM(i.total) AS total_spent
FROM employee e
JOIN customer c ON e.employee_id = c.support_rep_id
JOIN invoice i ON c.customer_id = i.customer_id
GROUP BY e.employee_id, e.first_name, e.last_name, c.customer_id
ORDER BY total_spent DESC LIMIT 20;
```
*Interpretation:* Identifies top employees by the revenue they generate through customer spending.

### 7. Support Rep Performance
```sql
SELECT 
    c.customer_id AS customer_id,
    c.first_name AS customer_first_name,
    c.last_name AS customer_last_name,
    c.support_rep_id AS employee_id,
    e.first_name AS employee_first_name,
    e.last_name AS employee_last_name,
    SUM(i.total) AS total_spent
FROM customer c
JOIN invoice i ON c.customer_id = i.customer_id
JOIN employee e ON c.support_rep_id = e.employee_id
GROUP BY c.customer_id, c.first_name, c.last_name, c.support_rep_id, e.first_name, e.last_name
ORDER BY total_spent DESC LIMIT 20;
```
*Interpretation:* Highlights top-performing support representatives by customer spending.

## Usage Instructions

1. Fork the repository and clone it locally
2. Navigate to the directory and edit the credentials.txt if needed
3. Run SQL queries using your database credentials:
```bash
sqlite3 database.db "SELECT Statement Here"
```

## Query Limitations

- The LIMIT 20 clause is applied to all results for easier viewing
- Queries assume a SQLite database structure
- Column names may vary slightly depending on your database schema

## Licensing

MIT License (permissive, ideal for open source projects)
