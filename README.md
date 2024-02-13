USE Amazon_sales;
SELECT * FROM report;

-- DATA CLEANING
-- Rename column names for Order ID, Courier Status and Ship-city

ALTER TABLE report
RENAME COLUMN `Order ID` to Order_ID
SELECT Order_ID 
FROM report;

ALTER TABLE report 
RENAME COLUMN `Courier Status`to Courier_Status
SELECT Courier_Status
FROM report;

ALTER TABLE report 
RENAME COLUMN `ship-city`to City
SELECT City 
FROM report;

-- Check for duplicate records in columns that should not be blank

SELECT Order_ID,COUNT(*) as dup
FROM report
GROUP BY 
	Order_ID,`Status`,Style,Category,Size,Courier_Status,Qty,Amount,`ship-postal-code`,`ship-country`
HAVING dup >1;

-- EXPLORING VARIABLES

-- 1.Most orders
-- 1a. Which category has the most sales and percentage of category to total orders
SELECT `Status`, 
COUNT(`Status`) as orders, (COUNT(*)/(SELECT COUNT(*) FROM report)*100) as Percentage
FROM report
GROUP BY `Status`
ORDER BY orders DESC;

-- 1b. Which category has the most Shipped/Delivered to Buyer sales and percentage of category to Total orders
SELECT Category, SUM(Qty) as Total, (COUNT(*)/(SELECT COUNT(*) FROM report)*100) as Percentage
FROM report
WHERE `Status`= 'Shipped' OR `Status`= 'Shipped - Delivered to Buyer' 
GROUP BY Category
ORDER BY Total DESC;

-- 2.What items were cancelled?

-- 2a. Making all cancelled orders courier status is 'unshipped'
SET Courier_Status = 'Unshipped'
WHERE `Status`= 'Cancelled' AND Courier_Status IS NOT NULL;


-- 2b. Category with most cancelled items
SELECT Category, SUM(Qty) as Total
FROM report
WHERE `Status`= 'Cancelled' 
GROUP BY Category
ORDER BY Total DESC;

-- 3. Sets ordered vs non-sets ordered
SELECT
SUM(case when Category IN ('Set') then 1 else 0 end)/COUNT(*) *100 AS SETS,
SUM(case when Category NOT IN ('Set') then 1 else 0 end)/COUNT(*) *100 AS NON_SETS
FROM report;

-- 4. Orders in each city

-- 4a. Checking City names only contains letters and not numbers
SELECT City 
FROM report
WHERE City = '%[0-9]%';

-- Making data uniform by converting City column into UPPERCASE
UPDATE
	report
SET 
	City = UPPER(City);
    
-- 4b. Top 5 cities with highest value of orders
SELECT City ,ROUND(SUM(AMOUNT),0) as TOP
FROM report
GROUP BY City
ORDER BY TOP DESC
LIMIT 5 ;


-- 4c. Average Order Value
SELECT City, ROUND(AVG(AMOUNT),1) as Average
FROM report
GROUP BY City 
ORDER BY Average DESC;

-- 4d. Average order per customer per City
SELECT City, AVG(customer_order) as Average
FROM
	(SELECT Order_ID, City, COUNT(*) as customer_order
	FROM report
	GROUP BY Order_ID, City) sub
GROUP BY City
ORDER BY Average DESC
LIMIT 3; 
