# test

First I imported all the csv files into the Sqlite DB by adding a prefix as `temp_` to the table names. So the created tables are:
- temp_tbl_brand
- temp_tbl_customer
- temp_tbl_opp
- temp_tbl_product
- temp_tbl_sales_rep

To check if there is potential of creating a PRIMARY_KEY for each table, I executed the following SQLs:

#### temp_tbl_brand

```sql
WITH x AS (
  SELECT 
  	brand_id, 
  	COUNT(brand_name) AS cnt 
  FROM 
  	temp_tbl_brand 
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
  	temp_tbl_customer
  GROUP BY cus_id
)
SELECT * FROM x WHERE cnt > 1
```

:white_check_mark: No duplicates detected!

#### temp_tbl_opp

```sql
WITH x AS (
  SELECT 
  	opportunity_id, 
  	COUNT(opportunity_name) AS cnt 
  FROM 
  	temp_tbl_opp
  GROUP BY opportunity_id
)
SELECT * FROM x WHERE cnt > 1
```

:white_check_mark: No duplicates detected!


#### temp_tbl_product

```sql
WITH x AS (
  SELECT 
  	material_num, 
  	COUNT(material_name) AS cnt 
  FROM 
  	temp_tbl_product
  GROUP BY material_num
)
SELECT * FROM x WHERE cnt > 1
```

:white_check_mark: No duplicates detected!


#### temp_tbl_sales_rep

```sql
WITH x AS (
  SELECT 
  	emp_id, 
  	COUNT(name) AS cnt 
  FROM 
  	temp_tbl_sales_rep
  GROUP BY emp_id
)
SELECT * FROM x WHERE cnt > 1
```

:x: Duplicates detected!

The emp_id **E658** is used for 2 different sales representatives!

