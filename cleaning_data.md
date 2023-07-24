# What issues will you address by cleaning the data?

This process is aimed at cleaning the data in the 5 tables and rendering it into an efficient format for querying. The ultimate objective is to create a reliable and coherent dataset that can be readily used to extract valuable insights and support decision-making processes.

To be addressed:
- Duplicates, Missing Values & Empty Columns
- Assigning keys to tables
- Renaming columns to convention cases
- Creation of columns based on present data
- Data type conversion

*New tables were created from the imported tables to enable direct updating and cleaning of data.*



## Queries:
The SQL queries used to clean the data.

### all_sessions table

```
-- View and group data to check for duplicates
SELECT * FROM all_sessions_new;

SELECT DISTINCT * FROM all_sessions_new;

SELECT *, COUNT(*) 
FROM all_sessions_new
GROUP BY
		fullVisitorId,
		channelGrouping,
		time,
		country,
		city,
		totalTransactionRevenue,
		transactions,
		timeOnSite,
		pageviews,
		sessionQualityDim,
		date,
		visitId,
		type,
		productRefundAmount,
		productQuantity,
		productPrice,
		productRevenue,
		productSKU,
		v2ProductName,
		v2ProductCategory,
		productVariant,
		currencyCode,
		itemQuantity,
		itemRevenue,
		transactionRevenue,
		transactionId,
		pageTitle,
		searchKeyword,
		pagePathLevel1,
		eCommerceAction_type,
		eCommerceAction_step,
		eCommerceAction_option
HAVING COUNT(*) > 1;
```

```

/*Rename attributes to snake_case for ease of understanding*/ (This was done for all 5 tables)

From the analytics table:

```
```
BEGIN;
ALTER TABLE analytics_new
RENAME COLUMN visitnumber TO visit_number;

ALTER TABLE analytics_new
RENAME COLUMN visitid TO visit_id;

ALTER TABLE analytics_new
RENAME COLUMN visitstarttime TO visit_start_time;

ALTER TABLE analytics_new
RENAME COLUMN date TO visit_date;

ALTER TABLE analytics_new
RENAME COLUMN fullvisitorid TO full_visitor_id;

ALTER TABLE analytics_new
RENAME COLUMN userid TO user_id;

ALTER TABLE analytics_new
RENAME COLUMN channelgrouping TO channel_groups;

ALTER TABLE analytics_new
RENAME COLUMN socialengagementtype TO social_engagement_type;

ALTER TABLE analytics_new
RENAME COLUMN pageviews TO page_views;

ALTER TABLE analytics_new
RENAME COLUMN timeonsite TO time_on_site;

COMMIT;
```

```
/*Prior to assigning primary key, check for duplicates using full_visitor_id
and visit_id; both did not meet the unique constraint requirements when importing the data.*/
```
```
SELECT * 
FROM all_sessions_new
WHERE full_visitor_id IN (
	SELECT full_visitor_id FROM all_sessions_new
	GROUP BY full_visitor_id
	HAVING COUNT(*) > 1 --794 rows
	)
ORDER BY full_visitor_id; 
-- Observation: Records appearing to have duplicate information have different visit_ids.

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

```
/*Check for uniqueness before creating primary key constraint*/
SELECT * 
FROM all_sessions_new
WHERE visit_id IN (
	SELECT visit_id FROM all_sessions_new
	GROUP BY visit_id
	HAVING COUNT(*) > 1
	)
ORDER BY full_visitor_id; 


/*Assign primary key*/
ALTER TABLE all_sessions_new
ADD CONSTRAINT pk_visitor_visit
PRIMARY KEY (full_visitor_id, visit_id);
```

```
/*Clean column: channel_groups
One category has parenthesis; remove and update table*/
SELECT COUNT(*), channel_groups
FROM all_sessions_new
GROUP BY channel_groups;

SELECT 	CASE 
		WHEN channel_groups = '(Other)' THEN 'Other'
		ELSE channel_groups 
		END AS channel_groups
FROM all_sessions_new
GROUP BY channel_groups;

UPDATE all_sessions_new
SET channel_groups = CASE 
					WHEN channel_groups = '(Other)' THEN 'Other'
					ELSE channel_groups 
					END
;
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
/*Drop empty columns or columns with less than 100 values;
columns considered redundant without additional information:

total_transaction_revenue < 100
transactions < 100
product_refund_amount NULL
product_quantity < 100
product_revenue < 100
product_variant < 100
item_quantity NULL
item_revenue NULL
transaction_revenue < 100
transaction_id < 100
search_keyword < 100
ecommerce_action_option < 100
page_title
page_path
*/

ALTER TABLE all_sessions_new
	DROP COLUMN total_transaction_revenue,
	DROP COLUMN	transactions,
	DROP COLUMN	product_refund_amount,
	DROP COLUMN	product_quantity,
	DROP COLUMN	product_revenue,
	DROP COLUMN	product_variant,
	DROP COLUMN	item_quantity,
	DROP COLUMN	item_revenue,
	DROP COLUMN	transaction_revenue,
	DROP COLUMN	transaction_id,
	DROP COLUMN	search_keyword,
	DROP COLUMN	ecommerce_action_option,
	DROP COLUMN page_path,
	DROP COLUMN page_title
;
-- Check update:
SELECT * FROM all_sessions_new;
```

```
/*Session Quality & Page Views*/
SELECT * 
FROM all_sessions_new
WHERE session_quality_dim IS NOT NULL;

SELECT COUNT(*), session_quality_dim
FROM all_sessions_new
GROUP BY session_quality_dim
ORDER BY session_quality_dim
;
/* null - 12,835, values - 1168, could explore the quality of a web session
assuming a rating from 0 to 100, among those who left ratings.
Or percentage of those who left ratings and how they differ by other attributes.
*/
```

```
/*Time & Date
Convert time on site to time interval and convert to time type
Convert date to date data type and update in table.
*/

SELECT COUNT(*), time_on_site
FROM all_sessions_new
GROUP BY time_on_site
ORDER BY time_on_site
; -- 3199 NULLS

SELECT 	time_on_site, 
		MAKE_INTERVAL(secs => time_on_site)::TIME AS time_interval
FROM all_sessions_new;

SELECT
  visit_date,
  TO_DATE(visit_date::TEXT, 'YYYYMMDD') AS converted_date
FROM all_sessions_new;

/*Rename and update columns*/

ALTER TABLE all_sessions_new
	ADD COLUMN time_interval TIME,
	ADD COLUMN converted_date DATE
;

UPDATE all_sessions_new
	SET time_interval = MAKE_INTERVAL(secs => time_on_site)::TIME,
		converted_date = TO_DATE(visit_date::TEXT, 'YYYYMMDD')
;

ALTER TABLE all_sessions_new
	DROP COLUMN time_on_site,
	DROP COLUMN visit_date
;

ALTER TABLE all_sessions_new
	RENAME COLUMN time_interval TO time_on_site;

ALTER TABLE all_sessions_new
	RENAME COLUMN converted_date TO visit_date;

SELECT time_on_site, visit_date
FROM all_sessions_new
;
```

```
/*Interaction_Type: change case
*/

SELECT COUNT(*), interaction_type
FROM all_sessions_new
GROUP BY interaction_type
ORDER BY interaction_type
; -- no NULLS

SELECT 	interaction_type,
		INITCAP(LOWER(interaction_type)) AS int_type
FROM all_sessions_new
GROUP BY interaction_type
;

UPDATE all_sessions_new
	SET interaction_type = INITCAP(LOWER(interaction_type))
;
```

```
/*Product Category
Clean up categories of products using CASE WHEN
*/
```
```
SELECT COUNT(*), product_category
FROM all_sessions_new
GROUP BY product_category;
```
```
SELECT 	
		product_category,
			CASE
				WHEN POSITION('/' IN product_category) > 0 THEN
							CASE
								WHEN product_category = 'Bottles/' THEN 'Drinkware'
								WHEN product_category = 'Lifestyle/' THEN 'Lifestyle'
								WHEN product_category = 'Wearables/Men''s T-Shirts/' THEN 'Apparel'
								WHEN SPLIT_PART(product_category, '/', 2) = 'Shop by Brand' THEN 'Brands'
								WHEN SPLIT_PART(product_category, '/', 2) = 'Fun' THEN 'Unknown'
								ELSE SPLIT_PART(product_category, '/', 2)
							END
						WHEN product_category = 'Nest-USA' THEN 'Nest'
						WHEN product_category IN ('Waze', 'YouTube') THEN 'Brands'
						WHEN product_category = 'Headgear' THEN 'Apparel'
						WHEN product_category IN ('(not set)', '${escCatTitle}')
							THEN 'Unknown'
						ELSE product_category
						END AS category,
			CASE
				WHEN POSITION('/' IN product_category) > 0 THEN
							CASE
								WHEN product_category = 'Wearables/Men''s T-Shirts/' THEN 'Men''s'
								WHEN SPLIT_PART(product_category, '/', 2) = 'Fun' THEN 'Fun'
								WHEN SPLIT_PART(product_category, '/', 3) = '' THEN 'Unknown'
								ELSE SPLIT_PART(product_category, '/', 3)
							END	
						WHEN product_category = 'Nest-USA' THEN 'Nest-USA'
						WHEN product_category = 'Waze' THEN 'Waze'
						WHEN product_category = 'YouTube' THEN 'YouTube'
						WHEN product_category = 'Headgear' THEN 'Headgear'
						ELSE 'Unknown'
						END AS sub_category
FROM all_sessions_new
GROUP BY 1, 2, 3
;
```
```
ALTER TABLE all_sessions_new
	ADD COLUMN category VARCHAR(50),
	ADD COLUMN product_subcategory VARCHAR(50)
;
```
```
UPDATE all_sessions_new
	SET category =	CASE
						WHEN POSITION('/' IN product_category) > 0 THEN
							CASE
								WHEN product_category = 'Bottles/' THEN 'Drinkware'
								WHEN product_category = 'Lifestyle/' THEN 'Lifestyle'
								WHEN product_category = 'Wearables/Men''s T-Shirts/' THEN 'Apparel'
								WHEN SPLIT_PART(product_category, '/', 2) = 'Shop by Brand' THEN 'Brands'
								WHEN SPLIT_PART(product_category, '/', 2) = 'Fun' THEN 'Unknown'
								ELSE SPLIT_PART(product_category, '/', 2)
							END
						WHEN product_category = 'Nest-USA' THEN 'Nest'
						WHEN product_category IN ('Waze', 'YouTube') THEN 'Brands'
						WHEN product_category = 'Headgear' THEN 'Apparel'
						WHEN product_category IN ('(not set)', '${escCatTitle}')
							THEN 'Unknown'
						ELSE product_category
						END,	
		product_subcategory = 	CASE
						WHEN POSITION('/' IN product_category) > 0 THEN
							CASE
								WHEN product_category = 'Wearables/Men''s T-Shirts/' THEN 'Men''s'
								WHEN SPLIT_PART(product_category, '/', 2) = 'Fun' THEN 'Fun'
								WHEN SPLIT_PART(product_category, '/', 3) = '' THEN 'Unknown'
								ELSE SPLIT_PART(product_category, '/', 3)
							END	
						WHEN product_category = 'Nest-USA' THEN 'Nest-USA'
						WHEN product_category = 'Waze' THEN 'Waze'
						WHEN product_category = 'YouTube' THEN 'YouTube'
						WHEN product_category = 'Headgear' THEN 'Headgear'
						ELSE 'Unknown'
						END
;

--- Check and group by categories created
SELECT product_category, category, product_subcategory
FROM all_sessions_new
GROUP BY 1,2,3;

SELECT DISTINCT(product_subcategory), COUNT(*)
FROM all_sessions_new
GROUP BY product_subcategory;
```

```
/*Update category table and drop intermediate column*/
UPDATE all_sessions_new
	SET product_category = category
;

ALTER TABLE all_sessions_new
DROP COLUMN category;

/*Clean and update product_price*/
SELECT 	product_price,
		CAST(product_price AS FLOAT)/1000000 as product_unit_price
FROM all_sessions_new;

ALTER TABLE all_sessions_new
ALTER COLUMN product_price TYPE FLOAT 
		USING CAST(product_price AS FLOAT) / 1000000;
```

```
/*Correct product_name*/
SELECT product_name
FROM all_sessions_new
WHERE product_name ~ '[^a-zA-Z]';

SELECT * 
FROM all_sessions_new
WHERE product_name LIKE '%;%'
;

UPDATE all_sessions_new
	SET product_name = 'Dog Frisbee'
	WHERE product_name LIKE '%;%'
;
```


### For the analytics table:
Most of the queries above were repeated for each table, other queries which are table-specific are included below.

```
SELECT 	visit_Number,
		visit_Id,
		visit_Start_Time,
		visit_date,
		full_visitor_Id,
		user_id,
		channel_Groups,
		social_Engagement_Type,
		units_sold,
		page_views,
		time_on_site,
		bounces,
		revenue,
		unit_price, COUNT(*) FROM analytics_new
GROUP BY
		visit_Number,
		visit_Id,
		visit_Start_Time,
		visit_date,
		full_visitor_Id,
		user_id,
		channel_Groups,
		social_Engagement_Type,
		units_sold,
		page_views,
		time_on_site,
		bounces,
		revenue,
		unit_price
HAVING COUNT(*) > 1;
-- 870577 rows
```

Given the number of duplicates, instead of dropping duplicates, I decided to filter out the non-duplicates.

```
/*Isolate unique rows based on full_visitor_id and visit_id
and use as analysis table*/

CREATE TABLE analytics_clean AS
SELECT DISTINCT ON (full_visitor_id, visit_id)
    full_visitor_id,
    visit_id,
    visit_number,
    visit_Start_Time,
    visit_date,
    channel_Groups,
    units_sold,
    page_views,
    time_on_site,
    bounces,
    revenue,
    unit_price
FROM analytics_new
ORDER BY full_visitor_id, visit_id;

SELECT *
FROM analytics_clean;
```

```
/*Check for uniqueness before creating primary key constraint*/
SELECT full_visitor_id, visit_id, COUNT(*)
FROM analytics_clean
GROUP BY 1, 2
HAVING COUNT(*)>1

/*Assign primary key*/
ALTER TABLE analytics_clean
ADD CONSTRAINT pk_analytics
PRIMARY KEY (full_visitor_id, visit_id);
```

```
/*Converting date and time values*/

SELECT 	visit_start_time,
		TO_TIMESTAMP(visit_start_time) AS converted_time,
		visit_date,
		TO_DATE(visit_date::TEXT, 'YYYYMMDD') AS converted_date
FROM analytics_clean;

ALTER TABLE analytics_clean
	ADD COLUMN converted_time TIME,
	ADD COLUMN converted_date DATE
;

UPDATE analytics_clean
	SET converted_time =	TO_TIMESTAMP(visit_start_time),
		converted_date = 	TO_DATE(visit_date::TEXT, 'YYYYMMDD')
;

ALTER TABLE analytics_clean
	DROP COLUMN visit_start_time,
	DROP COLUMN visit_date
;

ALTER TABLE analytics_clean
	RENAME COLUMN converted_time TO visit_start_time;

ALTER TABLE analytics_clean
	RENAME COLUMN converted_date TO visit_date;

SELECT visit_start_time, visit_date
FROM analytics_clean;
```

```
/*Clean and update units_sold, unit_price & revenue*/
SELECT units_sold, revenue
FROM analytics_clean
GROUP BY 1,2
;

SELECT 	units_sold, 
		COALESCE(units_sold, 0) AS adjusted_units_sold,
		unit_price, 
		CAST(unit_price AS FLOAT)/1000000 AS adjusted_unit_price,
		revenue,
		COALESCE((CAST(revenue AS FLOAT)/1000000), 0) as adjusted_revenue
FROM analytics_clean
WHERE units_sold IS NOT NULL
;

SELECT units_sold
FROM analytics_clean
WHERE units_sold < 0;

ALTER TABLE analytics_clean
ALTER COLUMN unit_price TYPE FLOAT 
		USING CAST(unit_price AS FLOAT) / 1000000;

UPDATE analytics_clean
	SET units_sold = 	CASE 	WHEN units_sold < 0 THEN 0
								ELSE COALESCE(units_sold, 0)
						END
;

ALTER TABLE analytics_clean
ALTER COLUMN revenue TYPE FLOAT 
		USING units_sold*unit_price;
		
SELECT units_sold, unit_price, revenue
FROM analytics_clean
GROUP BY 1, 2, 3;
```

```
/*Clean page_views and time_on_site*/
SELECT * 
FROM analytics_clean
WHERE page_views IS NULL;

UPDATE analytics_clean
	SET page_views = COALESCE(page_views, 0)
;

SELECT COUNT(*), time_on_site
FROM analytics_clean
GROUP BY time_on_site
ORDER BY time_on_site
;

SELECT 	time_on_site, 
		(TIME '00:00:00' + MAKE_INTERVAL(secs => COALESCE(time_on_site, 0)))::TIME AS time_interval
FROM analytics_clean;

/*Rename and update columns*/

ALTER TABLE analytics_clean
	ADD COLUMN time_interval TIME
;

UPDATE analytics_clean
	SET time_interval = (TIME '00:00:00' + MAKE_INTERVAL(secs => COALESCE(time_on_site, 0)))::TIME
;

ALTER TABLE analytics_clean
	DROP COLUMN time_on_site
;

ALTER TABLE analytics_clean
	RENAME COLUMN time_interval TO time_on_site;

SELECT time_on_site, visit_date
FROM analytics_clean
;
```

```
/*Convert visit_id to integer to match when making joins*/
SELECT visit_id::integer
FROM analytics_clean;

ALTER TABLE analytics_clean
ALTER COLUMN visit_id TYPE INTEGER
		USING CAST(visit_id AS INTEGER);
	
SELECT visit_id
FROM analytics_clean;
```

### For products, sales_by_sku and sales_report tables
Queries specific to the above-named tables.
```
-- View data and check for duplicates
SELECT * FROM sales_by_sku_new;

SELECT product_sku, COUNT(*)
FROM sales_by_sku_new
GROUP BY product_sku
HAVING COUNT(*)>1; -- no duplicates

/*Assign primary key*/
ALTER TABLE sales_by_sku_new
ADD CONSTRAINT pk_sales_sku
PRIMARY KEY (product_sku);
```
```
-- Check for nulls and negative values
SELECT * 
FROM sales_by_sku_new
WHERE product_sku IS NULL OR total_ordered IS NULL;

SELECT * 
FROM sales_by_sku_new
WHERE total_ordered < 0;
```
```
-- View data and check for duplicates
SELECT * FROM sales_report_new;

SELECT product_sku, COUNT(*)
FROM sales_report_new
GROUP BY product_sku
HAVING COUNT(*)>1; -- no duplicates

/*Assign primary key*/
ALTER TABLE sales_report_new
ADD CONSTRAINT pk_sales_report
PRIMARY KEY (product_sku);
```

```
/* Understanding Sentiment Score & Magnitude:*/
SELECT MAX(sentiment_magnitude), MIN(sentiment_magnitude)
FROM sales_report_new;

SELECT MAX(sentiment_score), MIN(sentiment_score)
FROM sales_report_new;
```

/*
*A sentiment score typically ranges from -1 to +1:*

A positive sentiment score close to +1 indicates a positive sentiment, meaning the text expresses positive emotions or opinions.
A negative sentiment score close to -1 indicates a negative sentiment, meaning the text expresses negative emotions or opinions.
A sentiment score close to 0 indicates a neutral sentiment, meaning the text does not strongly express positive or negative emotions.

*Grouping:*
- Positive Sentiment: sentiment score >= 0.5
- Neutral Sentiment: -0.5 < sentiment score < 0.5
- Negative Sentiment: sentiment score <= -0.5

*Magnitude (0 - 2)*
- Low Sentiment: 0 <= sentiment magnitude < 1.0
- Moderate Sentiment: 1.0 <= sentiment magnitude < 2.0
- High Sentiment: sentiment magnitude >= 2.0
*/
```
SELECT	sentiment_score,
		CASE
			WHEN sentiment_score >= 0.5 THEN 'Positive'
			WHEN sentiment_score > -0.5 AND sentiment_score <0.5 THEN 'Neutral'
			ELSE 'Negative'
		END AS sentiment_category,
		sentiment_magnitude,
		CASE
			WHEN sentiment_magnitude < 1 THEN 'Low'
			WHEN sentiment_magnitude >= 1 AND sentiment_magnitude < 2 THEN 'Moderate'
			ELSE 'High'
		END AS sentiment_strength
FROM sales_report_new;

ALTER TABLE sales_report_new
	ADD COLUMN 	sentiment_category VARCHAR(20),
	ADD COLUMN	sentiment_strength VARCHAR(20)
;

UPDATE sales_report_new
	SET	sentiment_category	=	CASE
									WHEN sentiment_score >= 0.5 THEN 'Positive'
									WHEN sentiment_score > -0.5 AND sentiment_score <0.5 THEN 'Neutral'
									ELSE 'Negative'
								END,
		sentiment_strength	=	CASE
									WHEN sentiment_magnitude < 1 THEN 'Low'
									WHEN sentiment_magnitude >= 1 AND sentiment_magnitude < 2 THEN 'Moderate'
									ELSE 'High'
								END
;
```

```
/*Add Foreign key constraint*/
ALTER TABLE sales_by_sku_new
ADD CONSTRAINT fk_sales_products
FOREIGN KEY (product_sku)
REFERENCES products_new (product_sku);

SELECT * FROM sales_by_sku_new
WHERE product_sku = 'GGOEGALJ057912';

SELECT * FROM sales_by_sku_new
WHERE product_sku NOT IN (SELECT product_sku
						 FROM products_new);
						 
DELETE FROM sales_by_sku_new
WHERE product_sku NOT IN (SELECT product_sku
						 FROM products_new);
```





