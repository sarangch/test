# test

First I imported all the csv files into the Sqlite DB. So the created tables are:

- tbl_brand
- tbl_customer
- tbl_opp
- tbl_product
- tbl_sales_rep

With an intial analysis of the tables, the ER diagram of the data model seems as below:

![alt text](https://github.com/sarangch/test/er_diagram.png)

To check if there is potential of creating a PRIMARY_KEY for each table, I executed the following SQLs:

#### tbl_brand

```sql
WITH x AS (
  SELECT 
  	brand_id, 
  	COUNT(brand_name) AS cnt 
  FROM 
  	tbl_brand 
  GROUP BY brand_id
)
SELECT * FROM x WHERE cnt > 1
```

:white_check_mark: No duplicates detected!


#### tem_tbl_customer

```sql
WITH x AS (
  SELECT 
  	cus_id, 
  	COUNT(practice_name) AS cnt 
  FROM 
  	tbl_customer
  GROUP BY cus_id
)
SELECT * FROM x WHERE cnt > 1
```

:white_check_mark: No duplicates detected!

#### tbl_opp

```sql
WITH x AS (
  SELECT 
  	opportunity_id, 
  	COUNT(opportunity_name) AS cnt 
  FROM 
  	tbl_opp
  GROUP BY opportunity_id
)
SELECT * FROM x WHERE cnt > 1
```

:white_check_mark: No duplicates detected!


#### tbl_product

```sql
WITH x AS (
  SELECT 
  	material_num, 
  	COUNT(material_name) AS cnt 
  FROM 
  	tbl_product
  GROUP BY material_num
)
SELECT * FROM x WHERE cnt > 1
```

:white_check_mark: No duplicates detected!


#### tbl_sales_rep

```sql
WITH x AS (
  SELECT 
  	emp_id, 
  	COUNT(name) AS cnt 
  FROM 
  	tbl_sales_rep
  GROUP BY emp_id
)
SELECT * FROM x WHERE cnt > 1
```

:x: Duplicates detected!

The emp_id **E658** is used for 2 different sales representatives!

Then I checked the tbl_opp and found out that there are quite a few opportunities that are created using emp_id **E658**. 

I thought one of the ways of being able to trace the opportunity back to the right sales rep will be to look into the region in which the customer resides to match it with the region of the sales rep. Unfortunately there was no indication of the customer’s region. Hence, I tried to look at the different types of products from tbl_product and I executed the following SQL:

```sql
SELECT DISTINCT material_name FROM tbl_product
```

And the result was:

- Generic Software III Cloud
- Generic Software II
- Migration Services - Extension
- Generic Software II Cloud
- Generic Software III
- Partner A residual
- Administration of Service X
- Generic Software IV Cloud
- Administration of Service Y
- Data Vending Service
- Migration Services
- Generic Software I
- Partner B residual 
- Administration of Service Z


I noticed that we have Service and Software products. I thought that different sales reps are expert in one of these fields. But when I checked the other sales reps, I noticed that they sold both software and service. 

Since I couldn't find any other way of correcting the duplicated record, I decided to keep only one record of **E658** and change the other one to **E658-X** in the tbl_sales_rep.


