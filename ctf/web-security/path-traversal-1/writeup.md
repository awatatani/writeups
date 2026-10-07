# Path Traversal 1

In this challenge, you are given a web server `/challenge/server`. This serves files from a directory `/challenge/files`.

The `/challenge/files` directory has an `index.html` that it will server by default.

Reading the source of the web server shows us that it has a `flask` backend that exposes a `GET` endpoint at `/docs`.

The flag that we're trying to obtain is at `/flag`.

Here is the main bulk of the code:

```python
@app.route("/docs", methods=["GET"])
@app.route("/docs/<path:path>", methods=["GET"])
def challenge(path="index.html"):
    requested_path = app.root_path + "/files/" + path
```

If you look at how the `requested_path` is crafted, it appends it to the path without any cleansing.

The code then opens and reads from the `requested_path` without any cleaning of the path, so you can traverse outside of the `files` directory
by adding `..`s to the path, like so:

```bash
curl -v 'challenge.localhost:80/docs/../../flag'
```

However, if you leave the curl command as-is, `curl -v` output would resolve the `..`s before it gets sent out, trying to hit the endpoint
`challenge.localhost:80/flag`, which doesn't exist.

Going though the man page for curl, I found the option `--path-as-is` resolves this issue:

```bash
curl -v 'challenge.localhost:80/docs/../../flag --path-as-is'
```

Now, curl shows:

```
\> GET /docs/../../flag HTTP/1.1
```

And this will give you the flag in the html!