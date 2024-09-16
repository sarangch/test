# test

## Data model ER diagram

First I imported all the csv files into the Sqlite DB. So the created tables are:

- tbl_brand
- tbl_customer
- tbl_opp
- tbl_product
- tbl_sales_rep

With an intial analysis of the tables, the ER diagram of the data model seems as below:

![alt text](https://github.com/sarangch/test/blob/main/er_diagram.png)


## Analyze the data

To check if there is potential of creating a PRIMARY_KEY for each table, I executed the following SQLs:

#### tbl_brand

```sql
WITH x AS (
  SELECT 
  	brand_id, 
  	COUNT(brand_name) AS cnt 
  FROM tbl_brand 
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
  FROM tbl_customer
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
  FROM tbl_opp
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
  FROM tbl_product
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
  FROM tbl_sales_rep
  GROUP BY emp_id
)
SELECT * FROM x WHERE cnt > 1
```

:x: Duplicates detected!

The emp_id **E658** is used for 2 different sales representatives!

## Possible resolution of the duplicated key 

I checked the tbl_opp and found out that there are quite a few opportunities that are created using emp_id **E658**. 

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

Since I couldn't find any other way of correcting the duplicated record, I decided to keep only one record of **E658** from the tbl_sales_rep.

## Creating the view

I tried to join all the tables and create a view using the following SQL:

```sql
CREATE VIEW insights AS 
SELECT
  opportunity_id, 
  brand_name,
  practice_name, 
  practice_class_desc,
  material_name,
  primary_addon,
  name,
  sales_country,
  sales_region, 
  opportunity_name,
  opportunity_amount,
  stage_of_sale,
  opportunity_created,
  opportunity_closed,
  opportunity_result_reason,
  promotion
FROM
  tbl_opp, 
  tbl_brand, 
  tbl_customer, 
  tbl_product, (
    SELECT 
      emp_id, 
      name, 
      sales_country, 
      sales_region 
    FROM 
      tbl_sales_rep 
    GROUP BY emp_id 
    HAVING min(rowid)
  ) x
WHERE
  tbl_opp.BRAND_ID = tbl_brand.brand_id
  AND
  tbl_opp.CUS_ID = tbl_customer.CUS_ID
  AND
  tbl_opp.MATERIAL_NUM = tbl_product.material_num
  AND
  tbl_opp.EMP_ID = x.emp_id;
```

## Extract the expected insights

1. Find the 2021 sum of opportunity amount for the North America-West region, by sales representative.

```sql
SELECT 
  name, 
  SUM(opportunity_amount) AS total
FROM insights 
WHERE 
  sales_country = 'United States' 
  AND 
  sales_region = 'west' 
  AND 
  strftime('%Y', DATE(opportunity_created)) = '2021'
GROUP BY name
```

The result is: 

name | total
--- | --- 
Jamya Sherman | 4334853

2. Count the lost opportunities by reason. Show the counts by year, where each year is its own column.

The best way to create such insight is to use **pivot_table**. But since the online Sqlite does not support the use of extensions, I used the following SQL to create the insights for the unique years of 2020, 2021 and 2022. 

```sql
WITH x AS (
  SELECT 
    opportunity_id,
    opportunity_result_reason AS reason, 
    strftime('%Y', DATE(opportunity_created)) AS year
  FROM insights 
  WHERE stage_of_sale = 'Loss' 
)
SELECT
  reason,
  COUNT(opportunity_id) FILTER (WHERE year = '2020') AS "2020",
  COUNT(opportunity_id) FILTER (WHERE year = '2021') AS "2021",
  COUNT(opportunity_id) FILTER (WHERE year = '2022') AS "2022"
FROM x
GROUP BY reason
ORDER BY reason;
```

The result is:

reason | 2020 | 2021 | 2022
--- | --- | --- | ---
Brand | 9 | 7 | 14
Contract Terms | 7 | 8 | 9
Hardware | 5 | 11 | 9
Implementation Timing | 8 | 11 | 9
Infrastructure | 10 | 12 | 8
Integrations | 16 | 11 | 5
Pricing | 7 | 8 | 7
Product | 7 | 15 | 12
Relationship | 9 | 5 | 8
Support | 12 | 9 | 8


## Extra insights

1. Show the percentage of won deals and the percentage of total amount won, based on the sales country.

```sql
WITH x AS (
  SELECT 
	sales_country, 
    SUM(opportunity_amount) amount_total, 
    SUM(case WHEN stage_of_sale = 'Win' THEN opportunity_amount ELSE 0 END) AS amount_won,
    COUNT(opportunity_id) AS cnt_total, 
    SUM(case WHEN stage_of_sale = 'Win' THEN 1 ELSE 0 END) AS cnt_won
  FROM insights 
  GROUP BY sales_country
)
SELECT 
  sales_country, 
  100 * cnt_won / cnt_total AS win_percentage, 
  100 * amount_won / amount_total AS win_amount_percentage 
FROM x
```

The result is:

sales_country | win_percentage | win_amount_percentage
--- | --- | ---
Asia  | 66 | 72
Canada | 65 | 68
Europe | 73 | 69
United States | 68 | 70

2. Find the best sales rep of each year

```sql
WITH x AS (
  SELECT 
    name, 
    SUM(opportunity_amount) AS amount, 
    strftime('%Y', DATE(opportunity_created)) AS year 
  FROM insights 
  WHERE stage_of_sale = 'Win' 
  GROUP BY name, strftime('%Y', DATE(opportunity_created))
), y AS (
  SELECT name, year, amount, row_number() OVER (PARTITION BY year ORDER BY amount DESC) rn FROM x
)
SELECT name, year FROM y WHERE rn = 1
```

The result is:

name | year
--- | ---
Cheyanne Salinas | 2020
Anastasia Wells | 2021
Korbin Frazier | 2022


3. Find the average number of days between the creation and closure of an opportunity based on win/loss based on the year

```sql
WITH x AS (
  SELECT 
    stage_of_sale, 
    strftime('%Y', DATE(opportunity_created)) as year, 
    AVG(julianday(opportunity_closed) - julianday(opportunity_created)) as days 
  FROM insights 
  WHERE stage_of_sale IN ('Win', 'Loss') 
  GROUP BY strftime('%Y', DATE(opportunity_created)), stage_of_sale
)
SELECT 
  year, 
  max(days) filter (where stage_of_sale = 'Win') as win, 
  max(days) filter (where stage_of_sale = 'Loss') as loss 
FROM x 
GROUP BY year
```

The result is:

year | win | loss
--- | --- | ---
2020 | 15.774193548387096 | 15.144444444444444
2021 | 15.703056768558952 | 15.371134020618557
2022 | 15.58252427184466 | 16.01123595505618
