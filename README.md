# 🔍 US Retail sales order Data Analysis - Project using Python + SQL

## 📌 Overview
This project demonstrates a complete data analysis pipeline using Python, Pandas, and SQL Server. The dataset is sourced from Kaggle API, cleaned and processed in Python, loaded into a SQL Server database, and finally analyzed using SQL queries.
## Please take a look at the diagram below 👇🏻
![Project Python + SQL Diagram](https://github.com/Gmanas7735/Python-Sql_Project/raw/main/Project%20Python%20%2B%20SQL.drawio%20(1).png)

## 🔄 Workflow
Download Dataset

1. Source: [Kaggle API](https://www.kaggle.com/datasets/manasgouda/retail-sales-order)

- Tool: kaggle CLI

2. Data Cleaning & Processing

  - Tool: Python (pandas)

    - Tasks: Handling missing values, formatting columns, filtering data

3. Load Data into SQL Server

  - Tool: pyodbc / sqlalchemy

    - Method: Insert cleaned data into a SQL Server table

4. Data Analysis using SQL

  - Tool: SQL Server

    - Task: Run complex SQL queries for insights and reporting

## 📁 Python commands step-by-step
### Import the datasets from the Kaggle using API
```Python
!pip install kaggle
import kaggle
!kaggle datasets download manasgouda/retail-sales-order
```
### Extract file from zip file
```Python
import zipfile
zip_refref = zipfile.ZipFile('retail-sales-order.zip') 
zip_ref.extractall() # extract file to dir
zip_ref.close() # close file
```
### Read data from the file and handle null values
```Python
import pandas as pd
df = pd.read_csv('orders.csv', na_values=['Not Available', 'unknown'])
df['Ship Mode'].unique()
```
### Rename columns names ..make them lower case and replace space with underscore
```Python
df.rename(columns={'Order Id':'order_id', 'City':'city'})
df.columns=df.columns.str.lower()
df.columns=df.columns.str.replace(' ', '_')
df.head(10)
```
### Create new columns discount , sale_price and profit
```Python
df['discount']=df['list_price']*df['discount_percent']*.01
df['sale_price']=df['list_price']-df['discount']
df['profit']=df['sale_price']-df['cost_price']
df
```
### Convert order date from object data type to datetime
```Python
df.dtypes
df['order_date']=pd.to_datetime(df['order_date'],format="%Y-%m-%d")
df
```
### Drop unwanted columns (i.e. cost_price, list_price & discount_percent)
```Python
df.drop(columns=['list_price','cost_price','discount_percent'],inplace=True)
```
### Create a conection between Python and Sql_Server
```Python
import sqlalchemy as sal
engine = sal.create_engine('mssql://Manas\\SQLEXPRESS/master?driver=ODBC+DRIVER+17+FOR+SQL+SERVER')
conn = engine.connect()
```
### Create empty table in SQL_Server before load the data

``` sql
CREATE TABLE df_orders (
    [order_id] INT PRIMARY KEY,
    [order_date] VARCHAR(20),
    [ship_mode] VARCHAR(20),
    [segment] VARCHAR(20),
    [country] VARCHAR(20),
    [city] VARCHAR(20),
    [state] VARCHAR(20),
    [postal_code] VARCHAR(20),
    [region] VARCHAR(20),
    [category] VARCHAR(20),
    [sub_category] VARCHAR(20),
    [product_id] VARCHAR(20),
    [quantity] INT,
    [descount] DECIMAL(7, 2),
    [sale_price] DECIMAL(7, 2),
    [profit] DECIMAL(7, 2)
);
```
### Load the data into sql server using append option
```Python
df.to_sql('df_orders', con=conn , index=False, if_exists = 'append')
```

## 🛠️ Tools Used
* Python

* Pandas

* SQL Server

* Kaggle API

* pyodbc / sqlalchemy

## 📈 Requirements in SQL
Key business insights and analytical queries.
### find top 10 highest reveue generating products
```sql
select top 10 product_id, sum(sale_price) as revenue
from df_orders
group by product_id
order by revenue desc
```
### find top 5 highest selling products in each region
```sql
with cte as(
		select region, product_id, sum(sale_price) as revenue
		from df_orders
		group by region, product_id
		)
select * from(
		select *
		, row_number() over(partition by region order by revenue desc) as rn
		from cte) as A
where rn<=5
```
### find month over month growth comparison for 2022 and 2023 sales eg : jan 2022 vs jan 2023
```sql
with cte as (
		select year(order_date) as order_year, month(order_date) as order_month, sum(sale_price) as revenue
		from df_orders
		group by year(order_date) , month(order_date)
		)
select order_month
, sum(case when order_year=2022 then revenue else 0 end) as sales_2022
, sum(case when order_year=2023 then revenue else 0 end) as sales_2023
from cte
group by order_month
order by order_month
```
### for each category which month had highest sales
```sql
with cte as(
		select category,format(order_date, 'yyyyMM') as order_year_month, 
		sum(sale_price) as sales
		from df_orders
		group by category, format(order_date, 'yyyyMM')
	)
select  category, order_year_month from(
		select *
		, row_number() over(partition by category order by sales) as rn
		from cte
	) as a
where rn=1
```
### which sub category had highest growth by profit in 2023 compare to 2022
```sql
with cte as (
		select sub_category, year(order_date) as order_year, sum(sale_price) as revenue
		from df_orders
		group by sub_category, year(order_date)
		)
	, cte2 as (
		select sub_category
		, sum(case when order_year=2022 then revenue else 0 end) as sales_2022
		, sum(case when order_year=2023 then revenue else 0 end) as sales_2023
		from cte
		group by sub_category
		)
select top 1 *
, (sales_2023-sales_2022) as increased_profit
from cte2
order by (sales_2023-sales_2022) desc
```



## 💡 Future Improvements
> [!NOTE]
> Automate the pipeline with Airflow or Prefect

> [!NOTE]
> Add visualizations using Power BI or Tableau

> [!NOTE]
> Integrate with a cloud database (e.g., Azure SQL, AWS RDS)

## 📬 Contact
- [x] For suggestions or collaboration, feel free to reach out via GitHub issues or fork the repo!
