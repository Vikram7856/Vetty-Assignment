# SQL Questions with Answers (MySQL)
---

## 1. Create Tables and Insert Data

USE interview_questions;

# Drop & Create transactions table
DROP TABLE IF EXISTS transactions;

CREATE TABLE transactions (
    buyer_id INT,
    purchase_time TIMESTAMP,
    refund_time TIMESTAMP,
    store_id VARCHAR(5),
    item_id VARCHAR(5),
    gross_transaction_value DECIMAL(10, 2)
);

# Drop & Create items table
DROP TABLE IF EXISTS items;

CREATE TABLE items (
    store_id VARCHAR(5),
    item_id VARCHAR(5),
    item_category VARCHAR(50),
    item_name VARCHAR(50)
);

# Insert transactions data
INSERT INTO transactions VALUES

(3,  '2019-09-19 21:19:06.544', NULL,                     'a', 'a1', 58.00),

(12, '2019-12-10 20:10:14.324', '2019-12-15 23:19:06.544','b', 'b2', 475.00),

(3,  '2020-09-01 23:59:46.561', '2020-09-02 21:22:06.331','f', 'f9', 33.00),

(2,  '2020-04-30 21:19:06.544', NULL,                     'd', 'd3', 250.00),

(1,  '2020-10-22 22:20:06.531', NULL,                     'f', 'f2', 91.00),

(8,  '2020-04-16 21:10:22.214', NULL,                     'e', 'e7', 24.00),

(5,  '2019-09-23 12:09:35.542', '2019-09-27 02:55:02.114','g', 'g6', 61.00);

# Insert items data
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
---

## **Q1. What is the count of purchases per month (excluding refunded purchases)?**

SELECT
    DATE_FORMAT(purchase_time, '%Y-%m') AS months,
    COUNT(*) AS purchase_count
FROM transactions
WHERE refund_time IS NULL
GROUP BY months
ORDER BY months;

![Solution_Q1](https://github.com/Vikram7856/Vetty-Assignment/blob/main/Solution_Q1.png)

## **Q2.How many stores receive at least 5 orders/transactions in October 2020?**

SELECT
    store_id,
    COUNT(buyer_id) AS order_count
FROM transactions
WHERE DATE_FORMAT(purchase_time, '%Y-%m') = '2020-10'
GROUP BY store_id
HAVING COUNT(buyer_id) >= 5;

![Solution_Q2](https://github.com/Vikram7856/Vetty-Assignment/blob/main/Solution_Q2.png)

## **Q3.For each store, what is the shortest interval (in minutes) from purchase to refund time?**

SELECT
    store_id,
    MIN(TIMESTAMPDIFF(MINUTE, purchase_time, refund_time)) AS shortest_refund_minutes
FROM transactions
WHERE refund_time IS NOT NULL
GROUP_BY store_id;

![Solution_Q3](https://github.com/Vikram7856/Vetty-Assignment/blob/main/Solution_Q3.png)

## **Q4.What is the gross_transaction_value of every store’s first order?**

SELECT
    store_id,
    gross_transaction_value
FROM (
    SELECT
        store_id,
        purchase_time,
        gross_transaction_value,
        ROW_NUMBER() OVER (
            PARTITION BY store_id ORDER BY purchase_time
        ) AS rn
    FROM transactions
) x
WHERE rn = 1;

![Solution_Q4](https://github.com/Vikram7856/Vetty-Assignment/blob/main/Solution_Q4.png)


## **Q5.What is the most popular item name that buyers order on their first purchase?**

WITH first_purchases AS (
    SELECT
        buyer_id,
        item_id,
        ROW_NUMBER() OVER (
            PARTITION BY buyer_id ORDER BY purchase_time
        ) AS rn
    FROM transactions
)
SELECT x.item_name
FROM (
    SELECT
        i.item_name,
        COUNT(*) AS first_purchase_count
    FROM first_purchases fp
    JOIN items i ON fp.item_id = i.item_id
    WHERE fp.rn = 1
    GROUP BY i.item_name
    ORDER BY first_purchase_count DESC
    LIMIT 1
) AS x;

![Solution_Q5](https://github.com/Vikram7856/Vetty-Assignment/blob/main/Solution_Q5.png)


## **Q6. Create a flag in the transaction items table indicating whether the refund can be processed or not.

The refund can only be processed if it happens within 72 hours of purchase time.
Expected Output: Only 1 of the 3 refunds should be processed.**

SELECT
    buyer_id,
    purchase_time,
    refund_time,
    store_id,
    item_id,
    gross_transaction_value,
    CASE
        WHEN refund_time IS NOT NULL 
             AND TIMESTAMPDIFF(HOUR, purchase_time, refund_time) <= 72
        THEN 'processed'
        ELSE 'not_processed'
    END AS refund_status
FROM transactions;

![Solution_Q6](https://github.com/Vikram7856/Vetty-Assignment/blob/main/Solution_Q6.png)


## **Q7. Create a rank by buyer_id in the transaction items table and filter for only the second purchase per buyer.

Ignore refunds.
Expected Output: Only the second purchase of buyer_id = 3 should appear.**

WITH ranked_purchases AS (
    SELECT
        buyer_id,
        purchase_time,
        store_id,
        item_id,
        gross_transaction_value,
        ROW_NUMBER() OVER (
            PARTITION BY buyer_id ORDER BY purchase_time
        ) AS rn
    FROM transactions
)
SELECT *
FROM ranked_purchases
WHERE rn = 2;

![Solution_Q7](https://github.com/Vikram7856/Vetty-Assignment/blob/main/Solution_Q7.png)

## **Q8. How will you find the second transaction time per buyer (don’t use MIN/MAX; assume multiple transactions exist for each buyer)?

Expected Output: Each buyer’s second transaction timestamp.**

WITH second_txn AS (
    SELECT
        buyer_id,
        purchase_time,
        ROW_NUMBER() OVER (
            PARTITION BY buyer_id ORDER BY purchase_time
        ) AS rn
    FROM transactions
)
SELECT
    buyer_id,
    purchase_time AS second_transaction_time
FROM second_txn
WHERE rn = 2;

![Solution_Q8](https://github.com/Vikram7856/Vetty-Assignment/blob/main/Solution_Q8.png)

