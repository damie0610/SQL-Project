   
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
```
Using revenue data:
![Q1A](https://github.com/damie0610/SQL-Project/assets/134011574/901202a7-1ad2-4c4c-9e80-83fe78ebe7f6)





> ### **Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:



Answer:





> ### **Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:



Answer:





> ### **Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:



Answer:





> ### **Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:







