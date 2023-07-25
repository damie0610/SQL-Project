# Final Project: Transforming and Analyzing Data with SQL

## Project/Goals
This project is based on an ```ecommerce database``` which provides data on products offered and orders, customer usage of the ecommerce site - time of visit, location from which the site was accessed, length of time on site and other details regarding the visit and online transactions. Five ```.csv``` (5) tables are provided in this database.

- all_sessions
- analytics
- products
- sales_by_sku
- sales_report

My goal was to explore the attributes of each entity of the database, establish the links between each entity and analyze the relationships between attributes to provide insights regarding the ecommerce patterns/trends.


## Process

### Data Importation & Exploration:
 
Examining and understanding the characteristics and contents within the dataset, the datatypes, issues and anomalies - to determine appropriate paths for importing the data, identify features that are connected and ways to approach data cleaning and analysis.

### Data Cleaning & Transformation:
The raw data was cleaned and *wrangled* into a format suitable for analysis. The process involved addressing duplicates and missing values, data type conversions, insertion of constraints and data standardization among other iterative processes throughout the analysis.

### Data Querying & Quality Assurance:
Usinq SQL queries, information regarding patterns and insights from the ecommerce database was retrieved - using functions, aggregations, joins, unions and other SQL syntax . An iterative QA process was included to ensure validity of queries from data extraction to analysis.


## Results
- Purchase/Order Trends over Time: The data shows how orders have increased over time - using the product orders, averages and totals over the course of the year, the pattern over time was reflected.
- Revenue by Channel Groups: Among the 8 channel groups, the organic search category had the highest number of sales; this, I assume, refers to users visiting the website and making orders through a natural, non-paid search. This is a positive insight as achieving higher rates of organic searches without direct advertising is a goal for many e-commerce businesses.
- Analyzing User Sentiments: Orders appeared to differ by sentiment_score; with neutral/positive scores, sales were higher and the sentiment category fluctuates over time. More context would be needed to explain the sentiment of customers that influenced their purchasing patterns, for example, during months with positive scores, were there campaigns or discounts or other factors that contributed to this?
- Products with the Highest & Lowest Orders: Branded items, particularly YouTube customized decals, had high revenue generation value, across geographical locations as well.

Queries where utilized in linking the information on the database effectively to generate the revenue over time, across geographical locations and product categories and through varying user channels. By examining various aspects of customer interactions, purchase patterns, and preferences, the project's findings offer an interesting understanding of *how customers engage with the business*. These insights enable companies to identify opportunities for growth, optimize marketing strategies, and tailor their product offerings to meet customers' needs effectively.

Analysis of revenue generation sheds light on the most profitable channels and aspects of the user interface, which can aid the business in focusing their resources and efforts on areas that yield the highest returns.

## Challenges 

### Insufficient Context and Unspecified Connotations: 
The database is provided with scanty context around the origin of the data which makes for a lot of assumption in the data transformation and analysis - hardly ideal for real-world data handling. Several columns are not suitable for analysis as their meanings are not clearly defined.

### High Volume of Duplicates: 
One of the datasets had duplicates had an inordinate amount of duplicates which caused constant crashes of the server. 

### Identifying Connections & Creating Primary Keys
The raw data had no unique identifier and required data transformation to create a composite primary key (in 2 of the tables).


## Future Goals
One aspect I would love to explore further given more time, and perhaps more data 😎 is related to the ```ecommerce_action``` columns. It seems to show customers at different levels of transactions - some who reached the checkout or review page and did not complete the transaction. I'm also curious as to the extent to which sentiment analysis could aid in understanding customers and providing better service - perhaps if the basis of the sentiment scores could be provided. Additionally, visualizing the data could provide easier-to-spot patterns and trends in the analysis.
