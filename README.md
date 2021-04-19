# CP4DBIGSQLS3

### Check the S3 Bucket is accessible form BIGSQL Service

    [bigsql@head /]$ hadoop fs -ls 's3a://icos-cp4d-std/'
    Found 2 items
    -rw-rw-rw-   1 bigsql bigsql          0 2021-01-22 10:08 s3a://icos-cp4d-std/aipdemoext3
    -rw-rw-rw-   1 bigsql bigsql         27 2021-01-22 09:41 s3a://icos-cp4d-std/file1.txt

### Check the Files are accessible 

    [bigsql@head /]$ hadoop fs -ls 's3a://icos-cp4d-std/file1.txt'
    -rw-rw-rw-   1 bigsql bigsql         27 2021-01-22 09:41 s3a://cp4d-big-sql-bucket/file1.txt

### Create a BigSQL Hadoop table pointing to External file on S3

    [bigsql@head /]$ db2 "create hadoop table aipdemoextpk1 (numberid int, personname varchar(20)) LOCATION 's3a://icos-cp4d-std/aipdemoextpk1/'"
    DB20000I  The SQL command completed successfully.

### Check the new external File is created

     [bigsql@head /]$ hadoop fs -ls 's3a://icos-cp4d-std/'
     Found 3 items
     -rw-rw-rw-   1 bigsql bigsql          0 2021-01-22 10:08 s3a://icos-cp4d-std/aipdemoext3
     drwxrwxrwx   - bigsql bigsql          0 2021-01-22 18:19 s3a://icos-cp4d-std/aipdemoextpk1
     -rw-rw-rw-   1 bigsql bigsql         27 2021-01-22 09:41 s3a://icos-cp4d-std/file1.txt

    [bigsql@head /]$ hadoop fs -ls 's3a://icos-cp4d-std/aipdemoextpk1'

### Check the File Content is initially empty 

    [bigsql@head /]$ db2 "select * from aipdemoextpk1"

    NUMBERID    PERSONNAME          
    ----------- --------------------

    0 record(s) selected.

### Insert Data into the External Table

    [bigsql@head /]$ db2 "insert into aipdemoextpk1 values (1, 'pravin')"
    DB20000I  The SQL command completed successfully.

### Check Data

    [bigsql@head /]$ db2 "select * from aipdemoextpk1"
  
    NUMBERID    PERSONNAME          
   ----------- --------------------
             1 pravin              

     1 record(s) selected.

### Check File Content

    [bigsql@head /]$ hadoop fs -ls 's3a://icos-cp4d-std/aipdemoextpk1'
    Found 1 items
    -rw-rw-rw-   1 bigsql bigsql          9 2021-01-22 18:21 s3a://icos-cp4d-std/aipdemoextpk1/i_1611339623126_1891691504_20210122062108177_1.0

### Get the File in S3 with following Contents and Load in S3

     [bigsql@head /]$ hadoop fs -cat 's3a://icos-cp4d-std/file1.txt'
     1,rahul
     2,ramesh
     3,suresh

### Create table on S3 with TEXTFILE, ORC or Parquet

        CREATE [EXTERNAL] HADOOP TABLE <schema>.<name> (-columns block-) 
        LOCATION ’s3a://<bucket>/<schema>/<table>’ STORED AS ORC | Parquet | TEXTFILE;

        CREATE EXTERNAL HADOOP TABLE CP4D.BOBCATDEMOPK1_TEXTFILE (EMPID INT, Name VARCHAR(20)) LOCATION 's3a://icos-cp4d-std/CP4D/BOBCATDEMOPK1_TEXTFILE' STORED AS TEXTFILE;

        CREATE EXTERNAL HADOOP TABLE CP4D.BOBCATDEMOPK2_ORC (EMPID INT, Name VARCHAR(20)) LOCATION 's3a://icos-cp4d-std/CP4D/BOBCATDEMOPK2_ORC' STORED AS ORC;

        CREATE EXTERNAL HADOOP TABLE CP4D.BOBCATDEMOPK3_PARQUET (EMPID INT, Name VARCHAR(20)) LOCATION 's3a://icos-cp4d-std/CP4D/BOBCATDEMOPK3_PARQUET' STORED AS PARQUET;

### Check the normal File Formart

    [bigsql@head /]$ hadoop fs -cat 's3a://icos-cp4d-std/aipdemoextpk1/i_1611339623126_1891691504_20210122062108177_1.0'
    1pravin

### Create table with delimited format

       CREATE HADOOP TABLE CP4D.TEST (
             id INT,
             name VARCHAR(10)
        )
        LOCATION 's3a://icos-cp4d-std/CP4D/TEST'
        ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
        STORED AS TEXTFILE;
 
 ### Insert data from existing table
  
     insert into cp4d.TEST SELECT * FROM CP4D.BOBCATDEMOPK3_PARQUET;
  
### Create Table based on existing data file 

    [bigsql@head /]$ db2 -tvf /tmp/2.sql 
    CREATE EXTERNAL HADOOP TABLE country ( SALESCOUNTRYCODE INT, COUNTRY VARCHAR(40), ISOTHREELETTERCODE VARCHAR(3), ISOTWOLETTERCODE VARCHAR(2), ISOTHREEDIGITCODE VARCHAR(3),     CURRENCYNAME VARCHAR(50), EUROINUSESINCE TIMESTAMP ) LOCATION 's3a://icos-cp4d-std/country'
    DB20000I  The SQL command completed successfully.

    [bigsql@head /]$ db2 -tvf /tmp/3.sql
    CREATE HADOOP TABLE staff ( ID SMALLINT, NAME VARCHAR(9), DEPT SMALLINT, YEARS SMALLINT, SALARY DECIMAL(7,2), COMM DECIMAL(7,2) ) PARTITIONED BY (JOB VARCHAR(5)) LOCATION 's3a://icos-cp4d-std/staff'
    DB20000I  The SQL command completed successfully.

### Check the files on S3 for the tables created above

    [bigsql@head /]$ hadoop fs -ls 's3a://icos-cp4d-std/'
    Found 5 items
    -rw-rw-rw-   1 bigsql bigsql          0 2021-01-22 10:08 s3a://icos-cp4d-std/aipdemoext3
    drwxrwxrwx   - bigsql bigsql          0 2021-01-22 19:12 s3a://icos-cp4d-std/aipdemoextpk1
    drwxrwxrwx   - bigsql bigsql          0 2021-01-22 19:12 s3a://icos-cp4d-std/country
    -rw-rw-rw-   1 bigsql bigsql         27 2021-01-22 09:41 s3a://icos-cp4d-std/file1.txt
    drwxrwxrwx   - bigsql bigsql          0 2021-01-22 19:12 s3a://icos-cp4d-std/staff

### MANY MORE CREATE HADOOP TABLE FORMATS AND EXAMPLES

https://www.ibm.com/docs/en/db2-big-sql/7.1?topic=statements-create-table-hadoop

### LOAD HADOOP EXAMPLES FOR QUICK AND DIRTY ANALYSIS

        CREATE EXTERNAL HADOOP TABLE CP4D.T1 (id INT, name VARCHAR(40))
        LOCATION 's3a://icos-cp4d-std/CP4D/PRAVINTEST.csv'
        ROW FORMAT DELIMITED  FIELDS TERMINATED BY ',' ;

        select * from CP4D.T1;

### LOAD HADOOP JSON Example

        CREATE EXTERNAL HADOOP TABLE CP4D.GOOGLEREVIEWJSON ( JSONDATA VARCHAR(4096) )
        LOCATION 's3a://icos-cp4d-std/CP4D/GoogleReviews_sample.json'
        ROW FORMAT DELIMITED  FIELDS TERMINATED BY ',' ;
    
        select * from CP4D.GOOGLEREVIEWJSON;
