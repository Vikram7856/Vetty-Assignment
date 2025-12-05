# SQL Questions with Answers (MySQL)

This repository contains SQL interview-style questions and their solutions using MySQL.

---

## 1. Create Tables and Insert Data

```sql
USE interview_questions;

-- Drop & Create transactions table
DROP TABLE IF EXISTS transactions;
CREATE TABLE transactions (
    buyer_id INT,
    purchase_time TIMESTAMP,
    refund_time TIMESTAMP,
    store_id VARCHAR(5),
    item_id VARCHAR(5),
    gross_transaction_value DECIMAL(10, 2)
);

-- Drop & Create items table
DROP TABLE IF EXISTS items;
CREATE TABLE items (
    store_id VARCHAR(5),
    item_id VARCHAR(5),
    item_category VARCHAR(50),
    item_name VARCHAR(50)
);

-- Insert transactions data
INSERT INTO transactions VALUES
(3,  '2019-09-19 21:19:06.544', NULL,                     'a', 'a1', 58.00),
(12, '2019-12-10 20:10:14.324', '2019-12-15 23:19:06.544','b', 'b2', 475.00),
(3,  '2020-09-01 23:59:46.561', '2020-09-02 21:22:06.331','f', 'f9', 33.00),
(2,  '2020-04-30 21:19:06.544', NULL,                     'd', 'd3', 250.00),
(1,  '2020-10-22 22:20:06.531', NULL,                     'f', 'f2', 91.00),
(8,  '2020-04-16 21:10:22.214', NULL,                     'e', 'e7', 24.00),
(5,  '2019-09-23 12:09:35.542', '2019-09-27 02:55:02.114','g', 'g6', 61.00);

-- Insert items data
INSERT INTO items VALUES
('a', 'a1', 'pants',    'denim pants'),
('a', 'a2', 'tops',     'blouse'),
('f', 'f1', 'table',    'coffee table'),
('f', 'f5', 'chair',    'lounge chair'),
('f', 'f6', 'chair',    'armchair'),
('d', 'd2', 'jewelry',  'bracelet'),
('b', 'b4', 'earphone', 'airpods');

SELECT * FROM transactions;
SELECT * FROM items;


# SQL Questions and Answers (Q1–Q8)

Below are the full questions and SQL solutions based on the `transactions` and `items` tables.

---

## **Q1. What is the count of purchases per month (excluding refunded purchases)?**

```sql
SELECT
    DATE_FORMAT(purchase_time, '%Y-%m') AS months,
    COUNT(*) AS purchase_count
FROM transactions
WHERE refund_time IS NULL
GROUP BY months
ORDER BY months;

