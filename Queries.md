#Commands 
------------
ssh palla3@144.24.53.159
mkdir kaggle_project

scp 2019-Oct.csv palla3@144.24.53.159:~/kaggle_project

hdfs dfs -mkdir ecommerce
hdfs dfs -put kaggle_project/2019-Oct.csv ecommerce/
hdfs dfs -ls ecommerce/

CREATE EXTERNAL TABLE IF NOT EXISTS ecommerce_original(
eventtime STRING, 
eventtype STRING, 
productid INT, 
category STRING, 
subcategory STRING,
product STRING,
brand STRING,
price INT,
userid INT,
usersession STRING
) 
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
STORED AS TEXTFILE LOCATION 'ecommerce/' 
TBLPROPERTIES ('skip.header.line.count'='1');

show tables;

select * from ecommerce limit 10;

Create table ecommerce as select *  from ecommerce_original where (category != '' or brand != '') and  (category != '' and  brand != '') ;

 hdfs dfs -mkdir ecommerce/tmp
 hdfs dfs -mkdir ecommerce/tmp/data
 hdfs dfs -mkdir ecommerce/tmp/data/stats

CREATE TABLE stats_category 
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
STORED AS TEXTFILE LOCATION 'ecommerce/tmp/data/stats_category' 
AS 
SELECT COUNT(*) count,CATEGORY,EVENTTYPE FROM ECOMMERCE
GROUP BY CATEGORY,EVENTTYPE
ORDER BY CATEGORY;

hdfs dfs -ls ecommerce/tmp/data/stats_category

hdfs dfs -cat ecommerce/tmp/data/stats_category/000000_0

hdfs dfs -get /user/palla3/ecommerce/tmp/data/stats_category/000000_0

scp palla3@144.24.53.159:~000000_0 .


CREATE TABLE highest_selling_category 
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
STORED AS TEXTFILE LOCATION 'ecommerce/tmp/data/highest_selling_category' 
AS 
SELECT category ,count(*) count
FROM ecommerce
where eventtype = 'purchase'
GROUP BY category;

hdfs dfs -getmerge  ecommerce/tmp/data/highest_selling_category/ ~/000123_0
 
scp palla3@144.24.53.159:~/000123_0 hpc123.csv

SELECT brand, category, COUNT(*) AS num_purchases
FROM ecommerce
WHERE category IN (
    SELECT category
    FROM ecommerce
    GROUP BY category
    ORDER BY COUNT(*) DESC LIMIT 1
)
GROUP BY brand, category
ORDER BY num_purchases DESC;

CREATE TABLE BRAND_STATISTICS_HPC
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
STORED AS TEXTFILE LOCATION 'ecommerce/tmp/data/brand_stats_hpc' 
as 
SELECT BRAND, CATEGORY, COUNT(*) FROM ECOMMERCE 
WHERE CATEGORY IN (
SELECT CATEGORY FROM ECOMMERCE 
GROUP BY CATEGORY 
ORDER BY COUNT(*) DESC LIMIT 1)
GROUP BY BRAND , CATEGORY ; 

hdfs dfs -get /user/palla3/ecommerce/tmp/data/brand_stats_hpc/000000_0

scp palla3@144.24.53.159:~/000000_0 bshpc.csv
