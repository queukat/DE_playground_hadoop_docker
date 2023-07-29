# Oracle Database Docker Setup

This project involves setting up an Oracle Database in a Docker environment. Here's a step-by-step guide:

## Oracle Database Docker Deployment
1. __Pull the Oracle Database Docker image from the Oracle Container Registry.__

Before you can pull the image, you need to accept the terms and conditions for using the image on the Oracle website. You also need to authenticate with your Oracle account. If you don't have an account, you can create one for free.

```bash
docker login container-registry.oracle.com
docker pull container-registry.oracle.com/database/enterprise:latest
```
2. __Run the Oracle Database Docker container.__

Replace __'YourPassword'__ with the password you want to use for the __'SYS'__, __'SYSTEM'__, and __'PDBADMIN'__ accounts. The __'ORACLE_SID'__ environment variable sets the name of the Oracle Database instance.

```bash
docker run -d -p 1521:1521 -p 5500:5500 --name oracle -e ORACLE_SID=YourSID -e ORACLE_PWD=YourPassword container-registry.oracle.com/database/enterprise:latest
```


The Oracle Database Docker container provides several configuration parameters that can be used when starting the container. Here is a detailed docker run command supporting all custom configurations:
<details>
<summary>Click to expand!</summary>

```bash
docker run -d --name <container_name> \
-p <host_port>:1521 -p <host_port>:5500 \
-e ORACLE_SID=<your_SID> \
-e ORACLE_PDB=<your_PDBname> \
-e ORACLE_PWD=<your_database_password> \
-e INIT_SGA_SIZE=<your_database_SGA_memory_MB> \
-e INIT_PGA_SIZE=<your_database_PGA_memory_MB> \
-e ORACLE_EDITION=<your_database_edition> \
-e ORACLE_CHARACTERSET=<your_character_set> \
-e ENABLE_ARCHIVELOG=true \
-v [<host_mount_point>:]/opt/oracle/oradata \
container-registry.oracle.com/database/enterprise:21.3.0.0
```
Here's what each parameter does and their default values if not specified:

- __'--name'__ : The name of the container (default: auto-generated).
- __'-p'__ : The port mapping of the host port to the container port. Two ports are exposed: 1521 (Oracle Listener), 5500 (OEM Express).
- __'-e ORACLE_SID'__ : The Oracle Database SID that should be used (default: ORCLCDB).
- __'-e ORACLE_PDB'__ : The Oracle Database PDB name that should be used (default: ORCLPDB1).
- __'-e ORACLE_PWD'__ : The Oracle Database SYS, SYSTEM, and PDBADMIN password (default: auto-generated).
- __'-e INIT_SGA_SIZE'__ : The total memory in MB that should be used for all SGA components (default: calculated during database creation).
- __'-e INIT_PGA_SIZE'__ : The target aggregate PGA memory in MB that should be used for all server processes attached to the instance (default: calculated during database creation).
- __'-e ORACLE_EDITION'__ : The Oracle Database Edition (default: enterprise).
- __'-e ORACLE_CHARACTERSET'__: The character set to use when creating the database (default: AL32UTF8).
- __'-e ENABLE_ARCHIVELOG'__: To enable archive log mode when creating the database (default: false). Supported 19.3 onwards.
- __'-v /opt/oracle/oradata'__ : The data volume to use for the database. Has to be writable by the Unix "oracle" (uid: 54321) user inside the container. If omitted the database will not be persisted over container recreation.

Replace the placeholders with your desired values. 
For example, replace __'<container_name>'__ with the name you want to give to your Docker container, __'<host_port>'__ with the host port you want to map to the container port, __'<your_SID>'__ with the Oracle Database SID you want to use, and so on.
</details>

3. __Connect to the Oracle Database.__

You can connect to the Oracle Database using SQL*Plus, SQL Developer, or another Oracle Database client. Replace __'YourPassword'__ with the password you specified when you ran the Docker container.

```bash
sqlplus sys/YourPassword@localhost:1521/ORCLCDB as sysdba
```

## Creating and Using Local Users in Oracle Database

In Oracle Database 12c and later versions, you can create and use local users that are specific to a single Pluggable Database (PDB). However, to interact with a specific PDB as a local user, you need to set your session to that PDB. 
###### I DON'T LIKE THIS WAY, PREFER COMMON USERS
Here's how you can do it:
<details>
<summary>Click to expand!</summary>

1. __Connect to the database as a common user or as a local user with the necessary privileges. For example, you can connect as the SYS user or another common user that has the CREATE SESSION and ALTER SESSION privileges.__

```sql
sqlplus sys/YourPassword@localhost:1521/ORCLCDB as sysdba
```
2. __Switch to the desired PDB.__ You can do this using the __'ALTER SESSION SET CONTAINER = '__ command. Replace __'YourPDB'__ with the name of your PDB.

```sql
ALTER SESSION SET CONTAINER = YourPDB;
```
3. __Create the local user in the PDB.__ You can now create a local user in this PDB. This user will only have privileges in this PDB and won't be able to access other PDBs.

```sql
CREATE USER local_user IDENTIFIED BY local_password;
GRANT CREATE SESSION TO local_user;
```
4. __Connect as the local user.__ You can now connect to the PDB as the local user. Remember to specify the PDB in your connection string.

```bash
sqlplus local_user/local_password@localhost:1521/YourPDB
```
Remember, when you're working with local users, you always need to specify the PDB in your connection string. If you don't, you'll connect to the root container (CDB), where the local user doesn't exist.
</details>

Please note that the Oracle Database Docker image is only for non-production use. For production use, you need to use the Oracle Database software installed directly on a host system or in an Oracle VM or other virtualization environment.

## Creating the SQL File with common user:
 
Create an SQL file (for example, __'setup.sql'__) and add the following code:
<details>
<summary>Click to expand!</summary>

#### Note: This step is optional. If you are not familiar with tablespaces or if you do not need to create a new one, you can skip this step and proceed to the next one.

<details>
<summary>Click to expand!</summary>

```sql
-- Checking the Data Files and Their Associated Tablespaces
SELECT file_name, tablespace_name
FROM dba_data_files;
```

This SQL command allows you to view the data files in your Oracle Database and the tablespaces they are associated with. Data files are the physical files that store the data of the database on disk. Each data file is associated with only one tablespace, and it stores the data for the objects that are located in this tablespace.

```sql
-- Creating a Tablespace
CREATE TABLESPACE test_ts
   DATAFILE '/opt/oracle/oradata/TEST/test_ts01.dbf' SIZE 50M AUTOEXTEND ON,
            '/opt/oracle/oradata/TEST/test_ts02.dbf' SIZE 50M AUTOEXTEND ON,
            '/opt/oracle/oradata/TEST/test_ts03.dbf' SIZE 50M AUTOEXTEND ON,
            '/opt/oracle/oradata/TEST/test_ts04.dbf' SIZE 50M AUTOEXTEND ON,
            '/opt/oracle/oradata/TEST/test_ts05.dbf' SIZE 50M AUTOEXTEND ON,
            '/opt/oracle/oradata/TEST/test_ts06.dbf' SIZE 50M AUTOEXTEND ON,
            '/opt/oracle/oradata/TEST/test_ts07.dbf' SIZE 50M AUTOEXTEND ON,
            '/opt/oracle/oradata/TEST/test_ts08.dbf' SIZE 50M AUTOEXTEND ON,
            '/opt/oracle/oradata/TEST/test_ts09.dbf' SIZE 50M AUTOEXTEND ON,
            '/opt/oracle/oradata/TEST/test_ts10.dbf' SIZE 50M AUTOEXTEND ON;
```
This SQL command creates a new tablespace named __'test_ts'__ with ten data files, each having a size of 50MB and the AUTOEXTEND option enabled. A tablespace is a logical storage unit in an Oracle Database. It groups related logical structures together. Each tablespace in an Oracle Database consists of one or more files called datafiles, which are physical structures that conform to the operating system in which Oracle is running.

The __'AUTOEXTEND ON'__ clause allows the datafile to automatically increase its size when it becomes full. This can be useful to avoid "tablespace full" errors that can occur if the datafile becomes full and cannot extend any further.

Please note that you need to replace the paths to the data files (__'/opt/oracle/oradata/TEST/test_ts0X.dbf'__) with the actual paths on your system.
</details>

#### Next step:

First SQL command creates a new __common__ user named C##TEST_USER. 
- In Oracle Database 12c and later, there are two types of users: common users and local users. 
Common users are users that have the same username and authentication across the entire CDB, including the root and all PDBs. 
Their username must start with __'C##'__ or __'c##'__. Local users are users that exist in a single PDB.

- Common users are typically used for administrative tasks that need to be performed across the entire CDB. However, they can also be useful for testing purposes, as they allow you to perform operations in multiple PDBs without having to switch between different local users.

```sql
-- Creating an Oracle common User
CREATE USER C##TEST_USER IDENTIFIED BY MyOraclePassword123;

-- Granting Privileges to the User
BEGIN
FOR R IN (SELECT * FROM DBA_SYS_PRIVS WHERE GRANTEE = 'SYSTEM') LOOP
EXECUTE IMMEDIATE 'GRANT ' || R.PRIVILEGE || ' TO c##TEST_USER';
END LOOP;
END;
/

-- Creating and Populating the Table
DECLARE
sql_stmt VARCHAR2(4000);
random_string VARCHAR2(50);
random_date DATE;
BEGIN

    -- Creating the Table
    EXECUTE IMMEDIATE 'CREATE TABLE big_table_new (
        col_string1 VARCHAR2(50),
        col_string2 VARCHAR2(50),
        col_date1 DATE,
        col_date2 DATE,
        col_number1 NUMBER(38,0),
        col_number2 NUMBER(38,0),
        col_number3 NUMBER(10,2),
        col_number4 NUMBER(10,2),
        col_float1 FLOAT,
        col_float2 FLOAT,
        col_num1 NUMBER,
        col_num2 NUMBER,
        col_num3 NUMBER(5,2),
        col_num4 NUMBER(5,2),
        col_double1 DOUBLE PRECISION,
        col_double2 DOUBLE PRECISION,
        col_int1 INTEGER,
        col_int2 INTEGER,
        col_boolean1 NUMBER(1),
        col_boolean2 NUMBER(1)
    )"TABLESPACE test_ts"';

    -- Populating the Table with Data
    FOR i IN 1..2000000 LOOP
        sql_stmt := 'INSERT INTO big_table_new VALUES (';
        FOR j IN 1..20 LOOP
            -- Generating Random Data Depending on the Column Type
            IF j <= 2 THEN
                random_string := DBMS_RANDOM.STRING('A', 50);
                sql_stmt := sql_stmt || '''' || random_string || '''';
            ELSIF j BETWEEN 3 AND 4 THEN
                random_date := TRUNC(SYSDATE) + DBMS_RANDOM.VALUE(0, 365);
                sql_stmt := sql_stmt || 'TO_DATE(''' || TO_CHAR(random_date, 'YYYY-MM-DD') || ''', ''YYYY-MM-DD'')';
            ELSIF j BETWEEN 5 AND 8 THEN
                sql_stmt := sql_stmt || ROUND(DBMS_RANDOM.VALUE(0, 100));
            ELSIF j BETWEEN 9 AND 10 THEN
                sql_stmt := sql_stmt || ROUND(DBMS_RANDOM.VALUE(0, 1000000));
            ELSIF j BETWEEN 11 AND 14 THEN
                sql_stmt := sql_stmt || TO_CHAR(TRUNC(DBMS_RANDOM.VALUE(0, 100), 2), 'FM9999999990.00');
            ELSIF j BETWEEN 15 AND 16 THEN
                sql_stmt := sql_stmt || TO_CHAR(TRUNC(DBMS_RANDOM.VALUE(0, 100000), 2), 'FM9999999990.00');
            ELSIF j BETWEEN 17 AND 18 THEN
                sql_stmt := sql_stmt || ROUND(DBMS_RANDOM.VALUE(0, 1000));
            ELSIF j BETWEEN 19 AND 20 THEN
                sql_stmt := sql_stmt || ROUND(DBMS_RANDOM.VALUE(0, 1));
            END IF;

            IF j < 20 THEN
                sql_stmt := sql_stmt || ', ';
            END IF;
        END LOOP;
        sql_stmt := sql_stmt || ')';

        EXECUTE IMMEDIATE sql_stmt;
    END LOOP;

    COMMIT;
END;
/
```
</details>

This SQL file creates a user __'C##TEST_USER'__ , grants them all the privileges that the __'SYSTEM'__ user has, creates a table __'big_table_new'__, and populates it with random data.

__Note:__ To execute this SQL file, you need to connect to the Oracle Database running in the Docker container. You can use Oracle SQL Developer or any other tool that supports Oracle Database connections.

__Important:__ Please replace MyOraclePassword123 with a secure password before running this script.
