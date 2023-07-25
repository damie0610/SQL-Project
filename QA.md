# What are your risk areas? Identify and describe them.

### Risk/Potential Areas
Some problems areas I identified and corrected to ensure data accuracy and consistency are stated below:

### QA Process:
- **Identifying missing values and duplicates**

```
/*Corrective action was to check for nulls, empty columns, remove nulls or assign appropriate
default values*/
SELECT *
FROM products_new
WHERE product_sku IS NULL OR product_name IS NULL OR ordered_quantity IS NULL
;

UPDATE products_new
	SET sentiment_score = COALESCE(sentiment_score, 0),
		sentiment_magnitude = COALESCE(sentiment_magnitude, 0)
;

/*City and Country Cleanup
After grouping, rename missing values and update columns
*/
SELECT COUNT(*), country
FROM all_sessions_new
GROUP BY country
ORDER BY country;
--'(not set)' - replace

SELECT COUNT(*), city
FROM all_sessions_new
GROUP BY city
ORDER BY city;
-- '(not set)', 'not available in demo dataset' - replace

SELECT 	CASE
		WHEN country = '(not set)' THEN 'Unknown'
		ELSE country
		END AS country,
			CASE
				WHEN city IN ('(not set)', 'not available in demo dataset')
				THEN 'Unknown'
				ELSE city
				END AS city
FROM all_sessions_new
GROUP BY city, country;

UPDATE all_sessions_new
SET	country =	CASE
				WHEN country = '(not set)' THEN 'Unknown'
				ELSE country
				END,
	city =		CASE
				WHEN city IN ('(not set)', 'not available in demo dataset')
				THEN 'Unknown'
				ELSE city
				END
;
-- Re-run grouping queries to check the update.
```

```
/* Select and drop duplicate rows from the table: all_sessions_new
*/
WITH cte_visit AS (
	SELECT 	visit_id, 
			ROW_NUMBER() 
			OVER(PARTITION BY visit_id
			ORDER BY visit_id) AS duplicate_count
    FROM all_sessions_new
)
SELECT *
FROM cte_visit ct
	JOIN all_sessions_new sub
	ON ct.visit_id = sub.visit_id
WHERE duplicate_count > 1
; 

/*Delete with below query and run the above to check that the rows are dropped*/
WITH cte_visit AS (
	SELECT 	visit_id, 
			ROW_NUMBER() 
			OVER(PARTITION BY visit_id
			ORDER BY visit_id) AS duplicate_count
    FROM all_sessions_new
)
DELETE FROM all_sessions_new
WHERE visit_id IN (
	SELECT visit_id
	FROM cte_visit
	WHERE duplicate_count > 1
);
```

- **Checking for Inaccurate data such as Negative Values**
```
SELECT *
FROM products_new
WHERE ordered_quantity < 0 OR stock_level < 0 OR restocking_lead_time < 0 
;

SELECT *
FROM products_new
WHERE sentiment_score < 0 OR sentiment_magnitude < 0
```

- **Data Integrity Issues and Inconsistencies**

```
/*Check for uniqueness before creating primary key constraint*/
SELECT product_sku, COUNT(*)
FROM sales_by_sku_new
GROUP BY 1
HAVING COUNT(*)>1; -- no duplicates

/*Add Foreign key constraint*/
SELECT * FROM all_sessions_new
WHERE product_sku NOT IN (SELECT product_sku
						 FROM products_new);

ALTER TABLE all_sessions_new
ADD CONSTRAINT fk_product_sessions
FOREIGN KEY (product_sku)
REFERENCES products_new (product_sku);
```

```
/*Identify inconsistencies across datasets*/
SELECT product_sku, total_ordered
FROM sales_by_sku_new
WHERE product_sku NOT IN(
	SELECT sr.product_sku
	FROM sales_report_new sr
	JOIN sales_by_sku_new ss
	ON sr.product_sku = ss.product_sku
	ORDER BY sr.total_ordered)
-- 8 items not in sales report
```

- **Standardize data, remove preceding and trailing spaces and non-sense characters.**

```
/*Removing spaces*/
SELECT DISTINCT(TRIM(product_name)) FROM sales_report_new;

UPDATE sales_report_new
	SET product_name = TRIM(product_name);

SELECT DISTINCT(TRIM(product_name)) FROM products_new;

UPDATE products_new
	SET product_name = TRIM(product_name);
	
SELECT * FROM sales_report_new;
```

```
/*Clean column: visit_time
Convert the column values to time data type
Since it was imported as integer, I created a new column with the time values
and renamed it.*/

SELECT * 
FROM all_sessions_new;

SELECT visit_time 
FROM all_sessions_new;

SELECT 	TIME '00:00:00' + MAKE_INTERVAL(secs => visit_time) 
		AS visit_time
FROM all_sessions_new;
```
```
-- Conversion and renaming
ALTER TABLE all_sessions_new
ADD COLUMN converted_time TIME;

UPDATE all_sessions_new
SET converted_time = TIME '00:00:00' + MAKE_INTERVAL(secs => visit_time);

ALTER TABLE all_sessions_new
DROP COLUMN visit_time;

ALTER TABLE all_sessions_new
RENAME COLUMN converted_time TO visit_time;

SELECT visit_time, TO_CHAR(visit_time, 'HH:MI:SS AM') as time_ampm 
FROM all_sessions_new;
```

```
/*Correct product_name*/
SELECT product_name
FROM all_sessions_new
WHERE product_name ~ '[^a-zA-Z]'
;

SELECT * 
FROM all_sessions_new
WHERE product_name LIKE '%;%'
;

UPDATE all_sessions_new
	SET product_name = 'Dog Frisbee'
	WHERE product_name LIKE '%;%'
;

```

- **Verify that numeric columns have valid integer values.**

```
SELECT 	
	city, 
	country, 
	SUM(CASE WHEN revenue NOT BETWEEN 0 AND 999999 THEN NULL ELSE revenue END) as transaction_revenue,
	RANK() OVER (ORDER BY SUM(revenue) DESC) AS rank
FROM 
	all_sessions_new asn
JOIN 
	analytics_clean ac ON asn.visit_id = ac.visit_id
WHERE 
	city != 'Unknown' AND country != 'Unknown'
GROUP BY 1, 2
HAVING 
	SUM(revenue) > 0;
```
