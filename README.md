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



### Other Helpful links

* [Data Objects in the Databricks Lakehouse](https://docs.databricks.com/lakehouse/data-objects.html)

### Recommended books

* [Delta Lake: The Definitive Guide by O’Reilly](https://www.databricks.com/resources/ebook/delta-lake-the-definitive-guide-by-oreilly)
