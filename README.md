# CP4DBIGSQLS3

### Check the S3 Bucket is accessible form BIGSQL Service

  [bigsql@head /]$ hadoop fs -ls 's3a://cp4d-big-sql-bucket/'
  Found 2 items
  -rw-rw-rw-   1 bigsql bigsql          0 2021-01-22 10:08 s3a://cp4d-big-sql-bucket/aipdemoext3
  -rw-rw-rw-   1 bigsql bigsql         27 2021-01-22 09:41 s3a://cp4d-big-sql-bucket/file1.txt

### Check the Files are accessible 

  [bigsql@head /]$ hadoop fs -ls 's3a://cp4d-big-sql-bucket/file1.txt'
  -rw-rw-rw-   1 bigsql bigsql         27 2021-01-22 09:41 s3a://cp4d-big-sql-bucket/file1.txt

### Create a BigSQL Hadoop table pointing to External file on S3

  [bigsql@head /]$ db2 "create hadoop table aipdemoextpk1 (numberid int, personname varchar(20)) LOCATION 's3a://cp4d-big-sql-bucket/aipdemoextpk1/'"
  DB20000I  The SQL command completed successfully.

### Check the new external File is created

  [bigsql@head /]$ hadoop fs -ls 's3a://cp4d-big-sql-bucket/'
  Found 3 items
  -rw-rw-rw-   1 bigsql bigsql          0 2021-01-22 10:08 s3a://cp4d-big-sql-bucket/aipdemoext3
  drwxrwxrwx   - bigsql bigsql          0 2021-01-22 18:19 s3a://cp4d-big-sql-bucket/aipdemoextpk1
  -rw-rw-rw-   1 bigsql bigsql         27 2021-01-22 09:41 s3a://cp4d-big-sql-bucket/file1.txt

  [bigsql@head /]$ hadoop fs -ls 's3a://cp4d-big-sql-bucket/aipdemoextpk1'

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

  [bigsql@head /]$ hadoop fs -ls 's3a://cp4d-big-sql-bucket/aipdemoextpk1'
  Found 1 items
  -rw-rw-rw-   1 bigsql bigsql          9 2021-01-22 18:21 s3a://cp4d-big-sql-bucket/aipdemoextpk1/i_1611339623126_1891691504_20210122062108177_1.0

### Get the File in S3 with following Contentes and Load in S3

  [bigsql@head /]$ hadoop fs -cat 's3a://cp4d-big-sql-bucket/file1.txt'
  1,rahul
  2,ramesh
  3,suresh

### Check the normal File Formart

  [bigsql@head /]$ hadoop fs -cat 's3a://cp4d-big-sql-bucket/aipdemoextpk1/i_1611339623126_1891691504_20210122062108177_1.0'
  1pravin

### Create Table based on existing data file 

    [bigsql@head /]$ db2 -tvf /tmp/2.sql 
    CREATE EXTERNAL HADOOP TABLE country ( SALESCOUNTRYCODE INT, COUNTRY VARCHAR(40), ISOTHREELETTERCODE VARCHAR(3), ISOTWOLETTERCODE VARCHAR(2), ISOTHREEDIGITCODE VARCHAR(3),     CURRENCYNAME VARCHAR(50), EUROINUSESINCE TIMESTAMP ) LOCATION 's3a://cp4d-big-sql-bucket/country'
    DB20000I  The SQL command completed successfully.

    [bigsql@head /]$ db2 -tvf /tmp/3.sql
    CREATE HADOOP TABLE staff ( ID SMALLINT, NAME VARCHAR(9), DEPT SMALLINT, YEARS SMALLINT, SALARY DECIMAL(7,2), COMM DECIMAL(7,2) ) PARTITIONED BY (JOB VARCHAR(5)) LOCATION 's3a://cp4d-big-sql-bucket/staff'
    DB20000I  The SQL command completed successfully.

### Check the files on S3 for the tables created above

    [bigsql@head /]$ hadoop fs -ls 's3a://cp4d-big-sql-bucket/'
    Found 5 items
    -rw-rw-rw-   1 bigsql bigsql          0 2021-01-22 10:08 s3a://cp4d-big-sql-bucket/aipdemoext3
    drwxrwxrwx   - bigsql bigsql          0 2021-01-22 19:12 s3a://cp4d-big-sql-bucket/aipdemoextpk1
    drwxrwxrwx   - bigsql bigsql          0 2021-01-22 19:12 s3a://cp4d-big-sql-bucket/country
    -rw-rw-rw-   1 bigsql bigsql         27 2021-01-22 09:41 s3a://cp4d-big-sql-bucket/file1.txt
    drwxrwxrwx   - bigsql bigsql          0 2021-01-22 19:12 s3a://cp4d-big-sql-bucket/staff
