# Databricks

## Data Enginnering Associate Certification material

* [Preparation Videos](https://databricks.com/p/thank-you/databricks-certification-preparation-on-demand)

* [Preparation Slides](https://pages.databricks.com/rs/094-YMS-629/images/Databricks-Certification-Preparation-Associate-DE.pdf)

* [Code](https://github.com/databricks-academy/data-engineering-with-databricks-english) 

* [Spreadsheet for making the practice exam](https://docs.google.com/spreadsheets/d/1F-YfU_32impN7izNavcLGsf5CutKmdVqWRgESH0LCRw/edit#gid=0)

* [Practice exam](https://files.training.databricks.com/assessments/practice-exams/PracticeExam-DataEngineerAssociate.pdf)


### Notes

#### Exam questions are distributed into five categories:

  * Databricks Lakehouse Platform – 24%
  * ELT with Spark SQL and Python – 29%
  * Incremental Data Processing – 22%
  * Production Pipelines – 16%
  * Data Governance – 9%

## Certification Preparation

  * Lakehouse architecture and fundamentals
  
    One simple platform to unify all of your data, analytics and AI workloads
  
  * [3.1 Databricks and tables](https://github.com/databricks-academy/data-engineering-with-databricks-english/tree/published/03%20-%20Relational%20Entities%20on%20Databricks) 
   
   Database creation
   
    
    CREATE DATABASE IF NOT EXISTS database_name
    
    CREATE DATABASE IF NOT EXISTS database_name LOCATION 'path_to_custom_location'
    
    # note that the location of the first database is in the default location under "dbfs:/user/hive/warehouse/" and the database directory is the name of the database with the .db extension
    
    
    DESCRIBE DATABASE EXTENDED database_name

    DESCRIBE EXTENDED table_name
    
    
    # Create an external (unmanaged) table from sample data from CSV and then we want to create a delta table with location
    
    CREATE OR REPLACE TEMPORARY VIEW temp_delays
    USING CSV OPTIONS (......)
    
    CREATE OR REPLACE delays_external_table 
    LOCATION '....' AS
    SELECT * FROM temp_delays;
    
    # select directly from file - SELECT * from file_format.'path_to_file_or_folder'
    SELECT * FROM json.'file_location'
    
    SELECT * FROM json.'folder_location'

* [08 - Delta Live Tables](https://github.com/databricks-academy/data-engineering-with-databricks-english/tree/published/08%20-%20Delta%20Live%20Tables) 
  
   
   CREATE OR REFRESH STREAMING LIVE TABLE sales_order_raw 
   COMMENT "sales raw table"
   AS SELECT * FROM cloud_files("location", "json", map("cloudfiles.inferColumnTypes", "True")
   

* [2.3 - Advanced Delta Lake Features](https://github.com/databricks-academy/data-engineering-with-databricks-english/blob/published/02%20-%20Delta%20Lake/DE%202.3%20-%20Advanced%20Delta%20Lake%20Features.sql)

   Transaction logs 
       **`_delta_log`**
   
   Compacting Small Files and Indexing
   `OPTIMIZE` will replace existing data files by combining records and rewriting the results.
    When executing **`OPTIMIZE`**, users can optionally specify one or several fields for **`ZORDER`** indexing.

   Delta lake transaction

     
     DESCRIBE HISTORY students
     
     Time travel
     
     SELECT * 
     FROM students VERSION AS OF 3


     Rollback Versions
     
     DELETE FROM students;

     SELECT * FROM students

     RESTORE TABLE students TO VERSION AS OF 8 
     
     
     Cleaning Up Stale Files
     
     If you wish to manually purge old data files, this can be performed with the **`VACUUM`** operation.
     retention of **`0 HOURS`** to keep only the current version:

     VACUUM students RETAIN 0 HOURS

* [4.4 - Writing to Tables](https://github.com/databricks-academy/data-engineering-with-databricks-english/blob/published/04%20-%20ETL%20with%20Spark%20SQL/DE%204.4%20-%20Writing%20to%20Tables.sql)

 ** Complete Overwrites
 Spark SQL provides two easy methods to accomplish complete overwrites.
 

` CREATE OR REPLACE TABLE events AS
SELECT * FROM parquet.`${da.paths.datasets}/ecommerce/raw/events-historical`
` 

` INSERT OVERWRITE sales
SELECT * FROM parquet.`${da.paths.datasets}/ecommerce/raw/sales-historical/`
` 

`INSERT OVERWRITE`  provides a nearly identical outcome as above: data in the target table will be replaced by data from the query. 
  Can only overwrite an existing table, not create a new one like our CRAS statement
  Can overwrite only with new records that match the current table schema -- and thus can be a "safer" technique for overwriting an existing table without disrupting downstream consumers
  Can overwrite individual partitions

** Append rows
 We can use **`INSERT INTO`** to atomically append new rows to an existing Delta table. This allows for incremental updates to existing tables, which is much more efficient than overwriting each time.
 
` INSERT INTO sales
SELECT * FROM parquet.`${da.paths.datasets}/ecommerce/raw/sales-30m`
`


* Merge Updates
 You can upsert data from a source table, view, or DataFrame into a target Delta table using the **`MERGE`** SQL operation. Delta Lake supports inserts, updates and deletes in **`MERGE`**, and supports extended syntax beyond the SQL standards to facilitate advanced use cases.


* Insert-Only Merge for Deduplication
This optimized command uses the same **`MERGE`** syntax but only provided a **`WHEN NOT MATCHED`** clause.

` MERGE INTO events a
USING events_update b
ON a.user_id = b.user_id AND a.event_timestamp = b.event_timestamp
WHEN NOT MATCHED AND b.traffic_source = 'email' THEN 
  INSERT * `
  
  
* Load Incrementally
 
 **`COPY INTO`** provides SQL engineers an idempotent option to incrementally ingest data from external systems.

Note that this operation does have some expectations:
 - Data schema should be consistent
 - Duplicate records should try to be excluded or handled downstream
  
  This operation is potentially much cheaper than full table scans for data that grows predictably.
 
 ` COPY INTO sales
FROM "${da.paths.datasets}/ecommerce/raw/sales-30m"
FILEFORMAT = PARQUET `


* [4.7 Advanced SQL Transformations](https://github.com/databricks-academy/data-engineering-with-databricks-english/blob/published/04%20-%20ETL%20with%20Spark%20SQL/DE%204.7%20-%20Advanced%20SQL%20Transformations.sql)

Spark SQL has built-in functionality to directly interact with JSON data stored as strings. We can use the **`:`** syntax to traverse nested data structures.

` SELECT value:device, value:geo:city 
FROM events_strings
`

Spark SQL also has the ability to parse JSON objects into struct types (a native Spark type with nested attributes).

` 
SELECT value 
FROM events_strings 
WHERE value:event_name = "finalize" 
ORDER BY key
LIMIT 1
`

Spark SQL also has a **`schema_of_json`** function to derive the JSON schema from an example. Here, we copy and paste an example JSON to the function and chain it into the **`from_json`** function to cast our **`value`** field to a struct type.

`
CREATE OR REPLACE TEMP VIEW parsed_events AS
  SELECT from_json(value, schema_of_json('{"device":"Linux","ecommerce":{"purchase_revenue_in_usd":1075.5,"total_item_quantity":1,"unique_items":1},"event_name":"finalize","event_previous_timestamp":1593879231210816,"event_timestamp":1593879335779563,"geo":{"city":"Houston","state":"TX"},"items":[{"coupon":"NEWBED10","item_id":"M_STAN_K","item_name":"Standard King Mattress","item_revenue_in_usd":1075.5,"price_in_usd":1195.0,"quantity":1}],"traffic_source":"email","user_first_touch_timestamp":1593454417513109,"user_id":"UA000000106116176"}')) AS json 
  FROM events_strings;


SELECT * FROM parsed_events `



 We can interact with the subfields in this field using standard **`.`** syntax similar to how we might traverse nested data in JSON.
 
 `SELECT ecommerce.purchase_revenue_in_usd 
FROM events
WHERE ecommerce.purchase_revenue_in_usd IS NOT NULL
`

 Spark SQL has a number of functions specifically to deal with arrays.
 For example, the **`size`** function provides a count of the number of elements in an array for each row.
 Let's use this to filter for event records with arrays containing 3 or more items.
 
 `
SELECT user_id, event_timestamp, event_name, items
FROM events
WHERE size(items) > 2
`


**Explode Arrays**

The **`explode`** function lets us put each element in an array on its own row.

`
SELECT user_id, event_timestamp, event_name, explode(items) AS item
FROM events
WHERE size(items) > 2
`

**Collect Arrays**

The **`collect_set`** function can collect unique values for a field, including fields within arrays.
  The **`flatten`** function allows multiple arrays to be combined into a single array.
  The **`array_distinct`** function removes duplicate elements from an array.

`
SELECT user_id,
  collect_set(event_name) AS event_history,
  array_distinct(flatten(collect_set(items.item_id))) AS cart_history
FROM events
GROUP BY user_id
`

**Join Tables**

Spark SQL supports standard join operations (inner, outer, left, right, anti, cross, semi).

`CREATE OR REPLACE VIEW sales_enriched AS
SELECT *
FROM (
  SELECT *, explode(items) AS item 
  FROM sales) a
INNER JOIN item_lookup b
ON a.item.item_id = b.item_id;

SELECT * FROM sales_enriched
`


 **Pivot Tables**
 The **`PIVOT`** clause is used for data perspective. We can get the aggregated values based on specific column values, which will be turned to multiple columns used in **`SELECT`** clause. The **`PIVOT`** clause can be specified after the table name or subquery.
**`SELECT * FROM ()`**: The **`SELECT`** statement inside the parentheses is the input for this table.
`
CREATE OR REPLACE TABLE transactions AS

SELECT * FROM (
  SELECT
    email,
    order_id,
    transaction_timestamp,
    total_item_quantity,
    purchase_revenue_in_usd,
    unique_items,
    item.item_id AS item_id,
    item.quantity AS quantity
  FROM sales_enriched
) PIVOT (
  sum(quantity) FOR item_id in (
    'P_FOAM_K',
    'M_STAN_Q',
    'P_FOAM_S',
    'M_PREM_Q',
    'M_STAN_F',
    'M_STAN_T',
    'M_PREM_K',
    'M_PREM_F',
    'M_STAN_K',
    'M_PREM_T',
    'P_DOWN_S',
    'P_DOWN_K'
  )
);

SELECT * FROM transactions
`

**Higher Order Functions**
Higher order functions in Spark SQL allow you to work directly with complex data types. When working with hierarchical data, records are frequently stored as array or map type objects. Higher-order functions allow you to transform data while preserving the original structure.

 Higher order functions include:
 - **`FILTER`** filters an array using the given lambda function.
 - **`EXISTS`** tests whether a statement is true for one or more elements in an array. 
 - **`TRANSFORM`** uses the given lambda function to transform all elements in an array.
 - **`REDUCE`** takes two lambda functions to reduce the elements of an array to a single value by merging the elements into a buffer, and the apply a finishing function on the final buffer.

**Filter**
Remove items that are not king-sized from all records in our **`items`** column. We can use the **`FILTER`** function to create a new column that excludes that value from each array.

**`FILTER (items, i -> i.item_id LIKE "%K") AS king_items`**
 - **`FILTER`** : the name of the higher-order function <br>
 - **`items`** : the name of our input array <br>
 - **`i`** : the name of the iterator variable. You choose this name and then use it in the lambda function. It iterates over the array, cycling each value into the function one at a time.<br>
 - **`->`** :  Indicates the start of a function <br>
 - **`i.item_id LIKE "%K"`** : This is the function. Each value is checked to see if it ends with the capital letter K. If it is, it gets filtered into the new column, **`king_items`**


`
-- filter for sales of only king sized items
SELECT
  order_id,
  items,
  FILTER (items, i -> i.item_id LIKE "%K") AS king_items
FROM sales
`

**Transform**
Built-in functions are designed to operate on a single, simple data type within a cell; they cannot process array values. **`TRANSFORM`** can be particularly useful when you want to apply an existing function to each element in an array. 
Compute the total revenue from king-sized items per order.
  **`TRANSFORM(king_items, k -> CAST(k.item_revenue_in_usd * 100 AS INT)) AS item_revenues`**

In the statement above, for each value in the input array, we extract the item's revenue value, multiply it by 100, and cast the result to integer. Note that we're using the same kind as references as in the previous command, but we name the iterator with a new variable, **`k`**.

`
-- get total revenue from king items per order
CREATE OR REPLACE TEMP VIEW king_item_revenues AS

SELECT
  order_id,
  king_items,
  TRANSFORM (
    king_items,
    k -> CAST(k.item_revenue_in_usd * 100 AS INT)
  ) AS item_revenues
FROM king_size_sales;
`


### Other Helpful links

* [Personal notes - Fundamentals of the Databricks Lakehouse Platform Accreditation](https://github.com/ahmenezes/databricks/blob/main/Fundamentals%20of%20the%20Databricks%20Lakehouse%20Platform%20Accreditation%20-%20v2.md)
* [Data Objects in the Databricks Lakehouse](https://docs.databricks.com/lakehouse/data-objects.html)

### Recommended books

* [Delta Lake: The Definitive Guide by O’Reilly](https://www.databricks.com/resources/ebook/delta-lake-the-definitive-guide-by-oreilly)
