   
> ### **Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:
```
-- Using revenues as transaction_revenue
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
```
-- Using total orders by city and country as there are more data points
SELECT 	
	city, 
	country, 
	SUM(CASE WHEN total_ordered NOT BETWEEN 0 AND 999999 THEN NULL ELSE total_ordered END) as transaction_revenue,
	RANK() OVER (ORDER BY SUM(total_ordered) DESC) AS rank
FROM 
	all_sessions_new asn
JOIN 
	sales_by_sku_new ssn ON asn.product_sku = ssn.product_sku
WHERE 
	city != 'Unknown' AND country != 'Unknown'
GROUP BY 1, 2
HAVING 
	SUM(total_ordered) > 0;

```


Answer:
*Using revenue data*

![Q1A](https://github.com/damie0610/SQL-Project/assets/134011574/7addf939-6f7f-4c40-bb3e-74697aebb57c)


*Using ordered quantity as it has more data points*
![Q1B](https://github.com/damie0610/SQL-Project/assets/134011574/c3153526-57b3-4de8-84af-794b16819d3f)




> ### **Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:
```
SELECT	'City' AS Locale,
		city AS Locale_Name,
		"Avg Product Orders"
FROM (
	SELECT 	DISTINCT(city),
		ROUND(AVG(ordered_quantity) OVER (PARTITION BY city)) AS "Avg Product Orders"
	FROM 
		all_sessions_new asn
	JOIN
		products_new pn ON asn.product_sku = pn.product_sku
	WHERE
		city <> 'Unknown'
	GROUP BY 
		city, ordered_quantity
	ORDER BY 1
	) AS sub

UNION

SELECT	'Country' AS Locale,
		country AS Locale_Name,
		"Avg Product Orders"
FROM (
	SELECT 	DISTINCT(country),
			ROUND(AVG(ordered_quantity) OVER (PARTITION BY country)) AS "Avg Product Orders"
	FROM 
		all_sessions_new asn
	JOIN
		products_new pn ON asn.product_sku = pn.product_sku
	WHERE
		country <> 'Unknown'
	GROUP BY 
		country, ordered_quantity
	ORDER BY 1
	) AS sub2
	
ORDER BY 3 DESC
;
```


Answer:

![Q2](https://github.com/damie0610/SQL-Project/assets/134011574/89a1a367-6152-4f40-bd3d-a1f9144f0269)





> ### **Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:
```
SELECT 	product_category,
		city,
		country,
		total_ordered_quantity,
		country_rank
FROM (
	SELECT
		product_category,
		city,
		country,
		SUM(ordered_quantity) AS total_ordered_quantity,
		RANK() OVER(PARTITION BY product_category ORDER BY SUM(ordered_quantity) DESC) as country_rank
	FROM
		all_sessions_new asn
	JOIN
		products_new pn ON asn.product_sku = pn.product_sku
	WHERE
		product_category != 'Unknown' and city <> 'Unknown' AND country <> 'Unknown'
	GROUP BY
		product_category, city, country
	HAVING 
		SUM(ordered_quantity) > 0
	ORDER BY
		product_category, total_ordered_quantity DESC) AS sub
WHERE
	country_rank = 1
;
```


Answer:

![Q3](https://github.com/damie0610/SQL-Project/assets/134011574/3f3fc373-837a-4d6c-8d7f-63efb60b6f67)





> ### **Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:
```
/*Using revenue data*/

WITH cte_country AS (
		SELECT 	country AS Locale_Name,
				asn.product_name,
				SUM(revenue) AS sales,
				RANK() OVER(PARTITION BY country ORDER BY SUM(revenue) DESC) AS rank_by_country
		FROM
			all_sessions_new asn
		JOIN
			analytics_clean ac ON asn.visit_id = ac.visit_id
		WHERE 
			country <> 'Unknown'
		GROUP BY 
			country, asn.product_name
		HAVING 
			SUM(revenue) > 0
		ORDER BY
			country),
cte_city AS (
		SELECT 	city AS Locale_Name,
				asn.product_name,
				SUM(revenue) AS sales,
				RANK() OVER(PARTITION BY city ORDER BY SUM(revenue) DESC) AS rank_by_city
		FROM
			all_sessions_new asn
		JOIN
			analytics_clean ac ON asn.visit_id = ac.visit_id
		WHERE 
			city <> 'Unknown'
		GROUP BY 
			city, asn.product_name
		HAVING 
			SUM(revenue) > 0
		ORDER BY
			city)

SELECT 	Locale_Name,
		product_name,
		sales,
		'Country' AS Locale
FROM cte_country
WHERE rank_by_country = 1

UNION

SELECT 	Locale_Name,
		product_name,
		sales,
		'City' AS Locale
FROM cte_city
WHERE rank_by_city = 1

ORDER BY sales DESC
;
```
```
/*Using ordered_quantity*/

WITH cte_country AS (
		SELECT 	country AS Locale_Name,
				asn.product_name AS top_selling_product,
				SUM(ordered_quantity) AS sales,
				RANK() OVER(PARTITION BY country ORDER BY SUM(ordered_quantity) DESC) AS rank_by_country
		FROM
			all_sessions_new asn
		JOIN
			products_new pn ON asn.product_sku = pn.product_sku
		WHERE 
			country <> 'Unknown'
		GROUP BY 
			country, asn.product_name
		HAVING 
			SUM(ordered_quantity) > 0
		ORDER BY
			country),
cte_city AS (
		SELECT 	city AS Locale_Name,
				asn.product_name AS top_selling_product,
				SUM(ordered_quantity) AS sales,
				RANK() OVER(PARTITION BY city ORDER BY SUM(ordered_quantity) DESC) AS rank_by_city
		FROM
			all_sessions_new asn
		JOIN
			products_new pn ON asn.product_sku = pn.product_sku
		WHERE 
			city <> 'Unknown'
		GROUP BY 
			city, asn.product_name
		HAVING 
			SUM(ordered_quantity) > 0
		ORDER BY
			city)

SELECT 	Locale_Name,
		top_selling_product,
		sales,
		'Country' AS Locale
FROM cte_country
WHERE rank_by_country = 1

UNION

SELECT 	Locale_Name,
		top_selling_product,
		sales,
		'City' AS Locale
FROM cte_city
WHERE rank_by_city = 1

ORDER BY sales DESC, locale_name
;
```

Answer:

*Using revenue data*
![Q4A](https://github.com/damie0610/SQL-Project/assets/134011574/02cbb2a4-acc3-4dcb-9780-ea6b0f61b712)


*Using ordered quantity*
![Q4B](https://github.com/damie0610/SQL-Project/assets/134011574/71a2ed49-15f5-452e-a6a5-f44fdc1a8ae9)


> ### **Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:
```
/*Using revenue data*/

SELECT 	city,
	country,
	SUM(revenue) AS Revenue_Impact
FROM
	all_sessions_new asn
JOIN
	analytics_clean ac ON asn.visit_id = ac.visit_id
WHERE
	city <> 'Unknown' AND country <> 'Unknown'
GROUP BY
	city, country
ORDER BY 3 DESC, 2, 1
;

```
```
-- Using total_ordered to get impact of revenue by country --

SELECT 	country,
	SUM(ordered_quantity) AS Revenue_Impact
FROM
	all_sessions_new asn
JOIN
	products_new pn ON asn.product_sku = pn.product_sku
WHERE
	country <> 'Unknown'
GROUP BY
	country
ORDER BY 2 DESC, 1
;
```


Answer:

*Using revenue data*
![Q5A](https://github.com/damie0610/SQL-Project/assets/134011574/3a04776b-c5ff-4a05-b415-769b0fe835b2)

*Using ordered quantity*
![Q5B](https://github.com/damie0610/SQL-Project/assets/134011574/a15f5fc0-7995-425d-b34c-2dd21c080415)






