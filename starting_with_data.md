> # **Question 1: Retrieve the average and total monthly orders. Group by month and year and observe any patterns.** 

**SQL Queries:**

```
/*Orders by Month*/
SELECT MAX(visit_date), MIN(visit_date)
FROM all_sessions_new; -- data spans a year

SELECT MAX(visit_date), MIN(visit_date)
FROM analytics_clean; -- data spans 3 months

SELECT *
FROM all_sessions_new
WHERE visit_date < '1990-01-01';

SELECT	
	EXTRACT(MONTH FROM visit_date) AS month_id,
	TO_CHAR(visit_date, 'Month') AS month_name,
	ROUND(AVG(ordered_quantity)) as Avg_Orders_By_Month,
	SUM(ordered_quantity) as Total_Monthly_Orders
FROM 
	all_sessions_new asn
JOIN 
	products_new pn ON asn.product_sku = pn.product_sku
GROUP BY 1, 2
ORDER BY 4 DESC
;
```
```
/*Orders by Month & Year*/
SELECT	
	EXTRACT(MONTH FROM asn.visit_date) AS month_id,
	TO_CHAR(asn.visit_date, 'Month') AS month_name,
	EXTRACT(YEAR FROM asn.visit_date) AS order_year,
	ROUND(AVG(ordered_quantity)) as Avg_Orders_By_Month,
	SUM(ordered_quantity) as Total_Monthly_Orders
FROM 
	all_sessions_new asn
JOIN 
	products_new pn ON asn.product_sku = pn.product_sku
GROUP BY 1,2,3
ORDER BY 5 DESC
;
```

**Answer:** 

*Monthly orders - Average & Total*

![image](https://github.com/damie0610/SQL-Project/assets/134011574/5df17841-9bac-4ec9-ace1-047ee027d837)

*Average & Total - By Month & Year*

![image](https://github.com/damie0610/SQL-Project/assets/134011574/e570df9f-ea8a-4ddd-b4eb-ff7ad61b614a)

The monthly orders appear to have risen fairly steadily each month since October 2016, taking a dive in the most recent month (August 2017). Howeverm this data spans August 8, 2016 to August 8, 2017 - it's the beginning of the month and sales would likely pick up during the course of the month given the pattern over the last year.



> # **Question 2: Examine total orders across channel groups on the site.** 

**SQL Queries:**
In this query, I worked with the ```channel_group``` category from analytics_new as it had one category more than the corresponding column in ```all_sessions_new```.

```
SELECT 
	SUM(ordered_quantity) as Total_Orders, 
	ac.channel_groups,
	RANK() OVER (ORDER BY SUM(ordered_quantity) DESC) as rank
FROM 
	all_sessions_new asn
JOIN
	products_new pn ON asn.product_sku = pn.product_sku
JOIN
	analytics_clean ac ON asn.visit_id = ac.visit_id
GROUP BY 2
ORDER BY 3
;
```

**Answer:**

![image](https://github.com/damie0610/SQL-Project/assets/134011574/fe2a5a2a-52c4-4509-bf9e-a3b9591e2a1f)

The highest orders were made by those who organically searched for products.


> # **Question 3: What are the top 10 countries with the highest orders of YouTube products?** 

**SQL Queries:**

```
SELECT 	asn.country, 
		SUM(ordered_quantity) AS "YouTube Orders"
FROM 
	all_sessions_new asn
JOIN
	products_new pn ON asn.product_sku = pn.product_sku
WHERE 
	product_subcategory = 'YouTube'
GROUP BY country
ORDER BY SUM(ordered_quantity) DESC
;
```

**Answer:**

![image](https://github.com/damie0610/SQL-Project/assets/134011574/671731e2-a3bc-4d5f-bf99-c97f78fe345f)



> # **Question 4: From the sentiment attribute, retrieve the ordered products by customers with a positive sentiment score, negative sentiment score and the difference in customer sentiment over time.** 

**SQL Queries:**

```
-- Ordered products by customers with a positive sentiment score

SELECT 	asn.product_name,
		SUM(pn.ordered_quantity) AS total_orders,
		srn.sentiment_category
FROM 
	all_sessions_new asn
JOIN	
	products_new pn ON asn.product_sku = pn.product_sku
JOIN
	sales_report_new srn ON asn.product_sku = srn.product_sku
WHERE 
	srn.sentiment_category = 'Positive'
GROUP BY
	asn.product_name, srn.sentiment_category
HAVING
	SUM(pn.ordered_quantity) > 0
ORDER BY 2 DESC
;
```
```
-- Ordered products by customers with a negative sentiment score

SELECT 	asn.product_name,
		SUM(pn.ordered_quantity) AS total_orders,
		srn.sentiment_category
FROM 
	all_sessions_new asn
JOIN	
	products_new pn ON asn.product_sku = pn.product_sku
JOIN
	sales_report_new srn ON asn.product_sku = srn.product_sku
WHERE 
	srn.sentiment_category = 'Negative'
GROUP BY
	asn.product_name, srn.sentiment_category
HAVING
	SUM(pn.ordered_quantity) > 0
ORDER BY 2 DESC
;
```
```
-- Sentiment score over the time period

SELECT
    sentiment_category,
    TO_CHAR(visit_date, 'YYYY-MM') AS time_period,
    SUM(ordered_quantity) AS total_sales
FROM
    all_sessions_new AS asn
JOIN	
	products_new pn ON asn.product_sku = pn.product_sku
JOIN
	sales_report_new srn ON asn.product_sku = srn.product_sku
GROUP BY
    sentiment_category, TO_CHAR(visit_date, 'YYYY-MM')
ORDER BY
    TO_CHAR(visit_date, 'YYYY-MM');
```

**Answer:**

*Ordered products by customers with a positive sentiment score*

![image](https://github.com/damie0610/SQL-Project/assets/134011574/2debaf1e-9685-4d55-8633-25fe0b7a29be)

*Ordered products by customers with a negative sentiment score*

![image](https://github.com/damie0610/SQL-Project/assets/134011574/7c0fd484-bbb8-4b0e-ae03-0627e1241cc3)

*Sentiment score over the time period*

![image](https://github.com/damie0610/SQL-Project/assets/134011574/7dfa4c98-7f27-4af2-94bb-f4667fc8a3e9)

Orders appear to differ by sentiment_score; with neutral/positive scores, sales appear higher and the sentiment category fluctuates over time. More context would be needed to explain the sentiment of customers that influenced their purchasing patterns, for example, during months with positive scores, were there campaigns or discounts or other factors that contributed to this?



> # **Question 5: Find products with 3 highest total sales (order_quantity) and 3 lowest, excluding products with fewer than 10 visit matches.** 

**SQL Queries:**

```
WITH highest_orders AS(
    SELECT  asn.product_sku,
			asn.product_name,
            SUM(ordered_quantity) AS product_orders,
            RANK() OVER(ORDER BY SUM(ordered_quantity) DESC) AS order_rank
    FROM all_sessions_new asn
	JOIN products_new pn ON asn.product_sku = pn.product_sku
   	GROUP BY asn.product_sku, asn.product_name
    HAVING COUNT(visit_id) >= 10 AND SUM(ordered_quantity) > 0
    ORDER BY product_orders DESC
    LIMIT 3
    ),
    lowest_orders AS(
    SELECT  asn.product_sku,
			asn.product_name,
            SUM(ordered_quantity) AS product_orders,
            RANK() OVER(ORDER BY SUM(ordered_quantity) DESC) AS order_rank
    FROM all_sessions_new asn
	JOIN products_new pn ON asn.product_sku = pn.product_sku
	GROUP BY asn.product_sku, asn.product_name
	HAVING COUNT(visit_id) >= 10 AND SUM(ordered_quantity) > 0
    ORDER BY product_orders ASC
    LIMIT 3
    )

SELECT *
FROM highest_orders

UNION 

SELECT *
FROM lowest_orders

ORDER BY order_rank
;
```

**Answer:**

![image](https://github.com/damie0610/SQL-Project/assets/134011574/bdc69a1d-db98-45ad-bffe-0069b0bbf32b)

