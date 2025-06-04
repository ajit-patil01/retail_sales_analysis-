# Retail Sales Analysis SQL 


## Project Overview

**Project Title**: Retail Sales Analysis  
**Level**: Beginner  
**Database**: `p1_retail_db`

This project serves as a practical showcase of the core SQL abilities and techniques that data analysts frequently employ to investigate, refine, and interpret retail sales information. Participants will have the opportunity to establish a retail sales database from scratch, carry out exploratory data analysis (EDA) to uncover insights, and tackle targeted business inquiries using SQL queries. It is especially well-suited for individuals who are new to the field of data analysis and are eager to develop a strong grounding in SQL.

## Objectives

1. **Set up a retail sales database**: Create and populate a retail sales database with the provided sales data.
2. **Data Cleaning**: Identify and remove any records with missing or null values.
3. **Exploratory Data Analysis (EDA)**: Perform basic exploratory data analysis to understand the dataset.
4. **Business Analysis**: Use SQL to answer specific business questions and derive insights from the sales data.

## Project Structure

### 1. Database Setup

- **Database Creation**: The project starts by creating a database named `p1_retail_db`.
- **Table Creation**: A table named `retail_sales` is created to store the sales data. The table structure includes columns for transaction ID, sale date, sale time, customer ID, gender, age, product category, quantity sold, price per unit, cost of goods sold (COGS), and total sale amount.

```sql
CREATE DATABASE p1_retail_db;

CREATE TABLE retail_sales
(
    transactions_id INT PRIMARY KEY,
    sale_date DATE,	
    sale_time TIME,
    customer_id INT,	
    gender VARCHAR(10),
    age INT,
    category VARCHAR(35),
    quantity INT,
    price_per_unit FLOAT,	
    cogs FLOAT,
    total_sale FLOAT
);
```

### 2. Data Exploration & Cleaning

- **Record Count**: Determine the total number of records in the dataset.
- **Customer Count**: Find out how many unique customers are in the dataset.
- **Category Count**: Identify all unique product categories in the dataset.
- **Null Value Check**: Check for any null values in the dataset and delete records with missing data.

```sql
SELECT COUNT(*) FROM retail_sales;
SELECT COUNT(DISTINCT customer_id) FROM retail_sales;
SELECT DISTINCT category FROM retail_sales;

SELECT * FROM retail_sales
WHERE 
    sale_date IS NULL OR sale_time IS NULL OR customer_id IS NULL OR 
    gender IS NULL OR age IS NULL OR category IS NULL OR 
    quantity IS NULL OR price_per_unit IS NULL OR cogs IS NULL;

DELETE FROM retail_sales
WHERE 
    sale_date IS NULL OR sale_time IS NULL OR customer_id IS NULL OR 
    gender IS NULL OR age IS NULL OR category IS NULL OR 
    quantity IS NULL OR price_per_unit IS NULL OR cogs IS NULL;
```

### 3. Data Analysis & Findings

The following SQL queries were developed to answer specific business questions:

1. **Write a SQL query to retrieve all columns for sales made on '2022-11-05**:
```sql
SELECT *
FROM retail_sales
WHERE sale_date = '2022-11-05';
```

2. **Write a SQL query to retrieve all transactions where the category is 'Clothing' and the quantity sold is more than 4 in the month of Nov-2022**:
```sql
SELECT 
  *
FROM retail_sales
WHERE 
    category = 'Clothing'
    AND 
    TO_CHAR(sale_date, 'YYYY-MM') = '2022-11'
    AND
    quantity >= 4
```

3. **Write a SQL query to calculate the total sales (total_sale) for each category.**:
```sql
SELECT 
    category,
    SUM(total_sale) as net_sale,
    COUNT(*) as total_orders
FROM retail_sales
GROUP BY 1
```

4. **Write a SQL query to find the average age of customers who purchased items from the 'Beauty' category.**:
```sql
SELECT
    ROUND(AVG(age), 2) as avg_age
FROM retail_sales
WHERE category = 'Beauty'
```

5. **Write a SQL query to find all transactions where the total_sale is greater than 1000.**:
```sql
SELECT * FROM retail_sales
WHERE total_sale > 1000
```

6. **Write a SQL query to find the total number of transactions (transaction_id) made by each gender in each category.**:
```sql
SELECT 
    category,
    gender,
    COUNT(*) as total_trans
FROM retail_sales
GROUP 
    BY 
    category,
    gender
ORDER BY 1
```

7. **Write a SQL query to calculate the average sale for each month. Find out best selling month in each year**:
```sql
SELECT 
       year,
       month,
    avg_sale
FROM 
(    
SELECT 
    EXTRACT(YEAR FROM sale_date) as year,
    EXTRACT(MONTH FROM sale_date) as month,
    AVG(total_sale) as avg_sale,
    RANK() OVER(PARTITION BY EXTRACT(YEAR FROM sale_date) ORDER BY AVG(total_sale) DESC) as rank
FROM retail_sales
GROUP BY 1, 2
) as t1
WHERE rank = 1
```
**Second mehtod** 

``` sql
 WITH monthly_avg_sales AS (
  SELECT
    EXTRACT(YEAR FROM sale_date) AS year,
    EXTRACT(MONTH FROM sale_date) AS month,
    AVG(total_sale) AS avg_monthly_sale
  FROM retail_sales
  GROUP BY EXTRACT(YEAR FROM sale_date), EXTRACT(MONTH FROM sale_date)
),
ranked_months AS (
  SELECT *,
         RANK() OVER (
           PARTITION BY year
           ORDER BY avg_monthly_sale DESC
         ) AS rank_
  FROM monthly_avg_sales
)
SELECT year, month, avg_monthly_sale
FROM ranked_months
WHERE rank_ = 1; 
```



8. **Write a SQL query to find the top 5 customers based on the highest total sales **:
```sql
SELECT 
    customer_id,
    SUM(total_sale) as total_sales
FROM retail_sales
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5
```

9. **Write a SQL query to find the number of unique customers who purchased items from each category.**:
```sql
SELECT 
    category,    
    COUNT(DISTINCT customer_id) as cnt_unique_cs
FROM retail_sales
GROUP BY category
```

10. **Write a SQL query to create each shift and number of orders (Example Morning <12, Afternoon Between 12 & 17, Evening >17)**:
```sql
WITH hourly_sale
AS
(
SELECT *,
    CASE
        WHEN EXTRACT(HOUR FROM sale_time) < 12 THEN 'Morning'
        WHEN EXTRACT(HOUR FROM sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
        ELSE 'Evening'
    END as shift
FROM retail_sales
)
SELECT 
    shift,
    COUNT(*) as total_orders    
FROM hourly_sale
GROUP BY shift
```


11. **Monthly Sales Growth/Decline**
# Calculate the month-over-month percentage change in total_sale for each category**:
    
```sql
with Monthly_sales as (
select 
extract(year from sale_date) as year , 
extract(month from sale_date) as month , 
category  , Sum(total_sale) as total_sales 
from retail_sales 
group by extract(year from sale_date) , 
extract(month from sale_date) , 
category

 )  , 
lag_month as (
select year , month , category ,
total_sales , 
lag(total_sales ,1,0) over (partition by category order by year , month ) as previous_sales 
from   Monthly_sales
)
SELECT
    year ,
  month,
    category,
    previous_sales , 
    case 
    when previous_sales > 0 then ((total_sales  - previous_sales)/ previous_sales) * 100 
    else null 
    end as percentage_chng
    from 
    Lag_month;
```

12. **Gender-Based Purchasing Habits**
# Identify the top 3 categories by total_sale for each gender.
```sql 
select 
gender , category , 
sum(total_sale) as total_sales ,
rank() over(partition by gender order by sum(total_sale) desc) as rank_
from retail_sales
group by gender ,category  ;
```

13. **Sales in peak hour**
  
  ```sql
 select extract(hour from sale_time) as sales_hour ,
sum(total_sale) as total_sales , 
rank () over(order by sum(total_sale) desc ) as rnk
from retail_sales
group by sales_hour 
order by total_sales desc;
 ```




## Findings

### 1. **Customer Demographics**
The dataset encompasses a diverse customer base, spanning a wide range of age groups. This diversity allows for a more comprehensive understanding of customer behavior across different life stages. Sales are distributed among various product categories, with notable activity in areas such as **Clothing** and **Beauty**. This distribution suggests that these categories are significant drivers of retail sales and may be more appealing to certain age groups or customer segments.

### 2. **High-Value Transactions**
The analysis reveals a number of transactions where the total sale amount exceeds **1,000**. These high-value transactions highlight the presence of premium purchases within the dataset. Such transactions could indicate purchases of luxury items, bulk buying, or heightened customer engagement during promotional periods. Identifying these transactions is valuable for targeting high-value customers and tailoring marketing strategies to maximize revenue from this segment.

### 3. **Sales Trends and Seasonality**
A monthly review of sales data uncovers fluctuations in transaction volumes and revenue, pointing to distinct **sales trends** and **seasonal patterns**. These variations help pinpoint **peak seasons**—periods when sales surge—as well as slower months. Understanding these trends enables businesses to optimize inventory management, staffing, and marketing efforts in anticipation of higher demand during peak times.

### 4. **Customer Insights**
Through detailed analysis, the project identifies the **top-spending customers**, providing insight into customer loyalty and purchasing power. Additionally, the analysis highlights the **most popular product categories**, revealing which items or categories are preferred by the customer base. These insights are crucial for developing targeted promotions, improving customer retention strategies, and making informed decisions about product assortment and merchandising.

---

In summary, these findings offer a comprehensive view of customer behavior, sales performance, and product popularity, equipping businesses with actionable intelligence to drive growth and customer satisfaction.

## Reports

- **Sales Summary**: A detailed report summarizing total sales, customer demographics, and category performance.
- **Trend Analysis**: Insights into sales trends across different months and shifts.
- **Customer Insights**: Reports on top customers and unique customer counts per category.

## Conclusion

This project serves as a comprehensive introduction to SQL for data analysts, covering database setup, data cleaning, exploratory data analysis, and business-driven SQL queries. The findings from this project can help drive business decisions by understanding sales patterns, customer behavior, and product performance.




