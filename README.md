SQL CODE -

-- 1. Create the TABLES
USE interview_questions;
DROP TABLE IF EXISTS transactions;
CREATE TABLE transactions (
    buyer_id INT,
    purchase_time TIMESTAMP,
    refund_time TIMESTAMP,
    store_id VARCHAR(5),
    item_id VARCHAR(5),
    gross_transaction_value DECIMAL(10, 2)
);

DROP TABLE IF EXISTS items;
CREATE TABLE items (
    store_id VARCHAR(5),
    item_id VARCHAR(5),
    item_category VARCHAR(50),
    item_name VARCHAR(50)
);

-- 2. Insert the DATA from your screenshot
INSERT INTO transactions VALUES 
(3, '2019-09-19 21:19:06.544', NULL, 'a', 'a1', 58.00),
(12, '2019-12-10 20:10:14.324', '2019-12-15 23:19:06.544', 'b', 'b2', 475.00),
(3, '2020-09-01 23:59:46.561', '2020-09-02 21:22:06.331', 'f', 'f9', 33.00),
(2, '2020-04-30 21:19:06.544', NULL, 'd', 'd3', 250.00),
(1, '2020-10-22 22:20:06.531', NULL, 'f', 'f2', 91.00),
(8, '2020-04-16 21:10:22.214', NULL, 'e', 'e7', 24.00),
(5, '2019-09-23 12:09:35.542', '2019-09-27 02:55:02.114', 'g', 'g6', 61.00);

INSERT INTO items VALUES 
('a', 'a1', 'pants', 'denim pants'),
('a', 'a2', 'tops', 'blouse'),
('f', 'f1', 'table', 'coffee table'),
('f', 'f5', 'chair', 'lounge chair'),
('f', 'f6', 'chair', 'armchair'),
('d', 'd2', 'jewelry', 'bracelet'),
('b', 'b4', 'earphone', 'airpods');

SELECT * FROM transactions;
SELECT * FROM items;

-- Questions: 
-- 1. What is the count of purchases per month (excluding refunded purchases)? 
SELECT 
    DATE_FORMAT(purchase_time, '%Y-%m') AS months,
    COUNT(*) AS purchase_count
FROM transactions
WHERE refund_time IS NULL
GROUP BY months
ORDER BY months;

<img width="661" height="209" alt="Solution Q1" src="https://github.com/user-attachments/assets/b8ef8148-2253-4beb-ad00-a357590cd892" />

-- 2. How many stores receive at least 5 orders/transactions in October 2020? 

SELECT 
    store_id,
    COUNT(buyer_id) AS order_count
FROM transactions
WHERE DATE_FORMAT(purchase_time, '%Y-%m') = '2020-10'
GROUP BY store_id
HAVING COUNT(buyer_id) >=5;


<img width="762" height="244" alt="Solution Q2" src="https://github.com/user-attachments/assets/b868a5b8-eb2e-4497-8719-96b2cb646a46" />


-- 3. For each store, what is the shortest interval (in min) from purchase to refund time? 
SELECT
    store_id,
    MIN(TIMESTAMPDIFF(MINUTE, purchase_time, refund_time)) AS shortest_refund_minutes
FROM transactions
WHERE refund_time IS NOT NULL
GROUP BY store_id;

<img width="696" height="224" alt="Solution Q3" src="https://github.com/user-attachments/assets/964e49a6-a2e7-4172-a8a6-d5c50444f2fc" />


-- 4. What is the gross_transaction_value of every store’s first order? 
SELECT
    store_id,
    gross_transaction_value
FROM (
    SELECT
        store_id,
        purchase_time,
        gross_transaction_value,
        ROW_NUMBER() OVER (PARTITION BY store_id ORDER BY purchase_time) AS rn
    FROM transactions
) x
WHERE rn = 1;

<img width="711" height="225" alt="Solution Q4" src="https://github.com/user-attachments/assets/8c88a418-50ca-4a94-8c6f-bc00258da28f" />



-- 5. What is the most popular item name that buyers order on their first purchase? 

WITH first_purchases AS (
    SELECT
        buyer_id,
        item_id,
        ROW_NUMBER() OVER (PARTITION BY buyer_id ORDER BY purchase_time) AS rn
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

<img width="967" height="220" alt="Solution Q5" src="https://github.com/user-attachments/assets/7d78d57f-09cf-4ec0-bd10-2ac98a3353ac" />


/*
6. Create a flag in the transaction items table indicating whether the refund can be processed or not. 
The condition for a refund to be processed is that it has to happen within 72 of Purchase time. Expected Output: 
Only 1 of the three refunds would be processed in this case
*/

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

<img width="1049" height="314" alt="Solution Q6" src="https://github.com/user-attachments/assets/8aaeaedb-ef11-46c1-a51c-ff592b960c8a" />


/*
7. Create a rank by buyer_id column in the transaction items table and filter for only the second purchase per buyer. 
(Ignore refunds here) Expected Output: 
Only the second purchase of buyer_id 3 should the output
*/

WITH ranked_purchases AS (
    SELECT
        buyer_id,
        purchase_time,
        store_id,
        item_id,
        gross_transaction_value,
        ROW_NUMBER() OVER (
            PARTITION BY buyer_id 
            ORDER BY purchase_time
        ) AS rn
    FROM transactions
)
SELECT *
FROM ranked_purchases
WHERE rn = 2;

<img width="944" height="224" alt="Solution Q7" src="https://github.com/user-attachments/assets/463385f2-058c-4834-a007-7ef33ee8f937" />


/*
8. How will you find the second transaction time per buyer (don’t use min/max; assume there were more transactions per buyer in the table) 
Expected Output: Only the second purchase of buyer_id along with a timestamp 
*/

WITH second_txn AS (
    SELECT
        buyer_id,
        purchase_time,
        ROW_NUMBER() OVER (
            PARTITION BY buyer_id
            ORDER BY purchase_time
        ) AS rn
    FROM transactions
)
SELECT 
    buyer_id,
    purchase_time AS second_transaction_time
FROM second_txn
WHERE rn = 2;
<img width="1204" height="225" alt="Solution Q8" src="https://github.com/user-attachments/assets/6dee68e6-6ff4-4a34-ad0b-59f2baca851c" />


