# Cross Site Scripting 1

So far, the reason we are able to exploit the server is due to the semantic gap between:

1. Web app and file system (Path traversal)
2. Web app and shell (Command Injection)
3. Web app and database (SQL Injection)

Now we will look at another attack vector:

4. Web app and web browser

For all of these, it is specifically when the web app generates paths, shell commands, and SQL queries that we get
a vulnerability. Let's see how the web app generating HTML leaves gaps for attack.

For the first 3 attacks, the victim of the attack is the web server. But for a XSS attack, the victim is another user
of the web app.

--- 

In this challenge, we have `/challenge/server` that will server us a blog post web app. There is a `/challenge/victim`
that will act as the victim.

Let's take a look at the server:

```python
@app.route("/", methods=["GET"])
def challenge_get():
    page = "<html><body>\nWelcome to pwnpost, the anonymous posting service. Post away!\n"
    page += "<form method=post>Post:<input type=text name=content><input type=submit value=Submit></form>\n"

    for post in db.execute("SELECT content FROM posts").fetchall():
        page += "<hr>" + post["content"] + "\n"

    return page + "</body></html>"
```

We have a fairly simple GET endpoint that will give us a form to post some stuff, as well as a page to be able to
read other people's post.

Now let's look at what the victim (the other user) does here:

```python
num_inputs = len(re.findall(r"<input[^<]*>", requests.get(challenge_url, timeout=1).text))
if num_inputs <= 2:
        print("You did not inject an <input> textbox...")
else:
    print("You got it! Here is your flag:")
    print(flag)
```

It is simply looking for 3 input tags in the page. If it sees 3 or more, it will print us the flag.

How are we able to do this? If you look at the `POST` endpoint of the server, whatever we have in the `content`
argument will be placed in the posts table, which will be displayed when the victim goes to the `GET` endpoint.

So, let's craft the payload like so:

```bash
curl -d "content=<input><input><input>" 'challenge.localhost:80'
```

This stores our payload into the server-side database that will be shown to the client-side when they access
the server, which is why this attack is called a **Stored XSS Attack**.

Now run `/challenge/victim`, who will spit out the flag after seeing the 3 inputs!