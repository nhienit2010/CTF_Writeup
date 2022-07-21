> # Imaginary CTF 2022
<hr />


# maas - Web Challenge

Based on the source code provided, we see that the web has a few basic functions as follows: allows users to create their own account, has the function to allow login to the newly created account and will display a flag. for `admin user`.

The admin user will be created first when the site starts running

![](https://i.imgur.com/jBkWVpl.png)

Registration with an existing account is not allowed, the cookie is set to the SHA256 of the password, use this cookie field to identify the user and store information such as the user's username, uuid

![](https://i.imgur.com/6q8J44V.png)

The login function seems a bit strange, because it does not need to care about the username but only the password, does not check if the user exists or not and only SHA256 encrypts the password that has just been sent to set for cookie field.

![](https://i.imgur.com/2EC7tIj.png)

Besides, there is a function to list users, and we can see their uuid.

![](https://i.imgur.com/CJ8BIWS.png)

Finally, only the admin can read the flag =)

![](https://i.imgur.com/xNvBUUx.png)

Based on our analysis we somehow managed to get the exact time at the start of the admin user account creation via the uuid field, but how?

The first way we tried to brute-force was by seeding the time to find the uuid that matched the admin's uuid. But after a while, nothing changed, and realized that

![](https://i.imgur.com/S0lIvuV.png)

The `uuid.uuid1()` function is generated depending only on the hostId and the current time, so using the `brute-force` attack is completely impossible. Because `uuid.uuid1()` is not affected by `random.seed()`

We then found [a post](https://versprite.com/blog/universally-unique-identifiers/) talking about this UUID1, and as shown below

![](https://i.imgur.com/6ECQeb3.png)

This `uuid1()` function is strongly discouraged, as an attacker can take advantage of the UUID generated from this `uuid1()` function to find the exact time of birth.

The formula calculated by the UUID1 function is based on the figure below:

![](https://i.imgur.com/oaJ4RV4.png)

The code to find the generated time is as follows:

```python
import time, datetime, random, uuid

u = uuid.UUID('9b680d66-07f1-11ed-b789-3623a06e2560', version=1) // admin uuid
t = datetime.datetime.fromtimestamp((u.time - 0x01b21dd213814000)*100/1e9)
sec = int(time.mktime(t.timetuple()))

print(sec)
```

![](https://i.imgur.com/hj6cfeF.png)

Here we can find the time of creation as an integer, but when it is randomly generated, the server uses 2 more digits in the decimal part. So the problem here, is simply we brute-force those decimal places, and seed the time we just found and create a password dictionary.

```python
import time, datetime, random, uuid

sec = 1658296979
for i in range(0, 100):
	s = ""
	if len(str(i)) == 1:
		s = str(sec) + f".0{str(i)}"
	else:
		s = str(sec) + f".{str(i)}"

	random.seed(float(s))
	password = "".join([random.choice("abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789") for _ in range(30)])

	print(password)

```

![](https://i.imgur.com/o82HpmZ.png)

Finally, use Burp Intruder to check the admin's cookie through SHA256 encryption of the password you just found.

![](https://i.imgur.com/qcj7bNG.png)

> FLAG: ictf{d0nt_use_uuid1_and_please_generate_passw0rds_securely_192bfa4d}

# 1337 - Web challenge

In this challenge we can easily see its source code at:
[Source code](http://1337.chal.imaginaryctf.org/source)

At the same time, it is easy to see the [Dockerfile](http://1337.chal.imaginaryctf.org/docker), based on the Dockerfile we can realize that the name of the flag file has been changed.

Looking at the main thread of execution, at the `/` endpoint, the server receives 2 main params, `text` and `dir`

![](https://i.imgur.com/grsUUiF.png)

then these 2 parameters are passed through a function called `leetify` for processing and then rendered by the web server

![](https://i.imgur.com/QvEEJNL.png)

The `leetify` function has a fairly simple function of filtering out blacklisted characters and replacing the corresponding alphanumeric characters in the preset dictionary,

![](https://i.imgur.com/vZwrMcE.png)

and can be converted back.  
![](https://i.imgur.com/hmrHj1s.png)


The server uses the [mojojs template](https://github.com/mojolicious/template.js) library, so here comes the `SSTI` vulnerability.

To bypass we just need to pass the param `dir=from` to not be converted from characters to numbers and use the `text` param to insert the template there. 

![](https://i.imgur.com/tEFOAbO.png)

then I tried exploitable cases leading to RCE like using `require`, `child_process`,... but everything seems not available in the current context. And throw with error:

```
ReferenceError: require is not defined
```

but it looks like we omitted an important variable `ctx`, then we focused on this more, hoping to get a lot of things needed to be able to escape the sandbox.

First we need to dump all the properties and methods from the variable `ctx`

Payload: `?text=<%= console.log(ctx) %>&dir=from`

![](https://i.imgur.com/1RO7czx.png)

As we can see that there are many properties and methods in the variable `ctx`. We've tried accessing each property, but there doesn't seem to be anything special here.

We then began to dig deeper into the library's source code, and realized that the variable `ctx` also had an attribute `home`

![](https://i.imgur.com/J7Gw0fn.png)

Payload: `?text=<%= console.log(ctx.home) %>&dir=from`

![](https://i.imgur.com/aemEtJO.png)

the result returned is the instance of the `Path module`, 
and luckily this is exactly what we were looking for

![](https://i.imgur.com/WanmdaP.png)

Based on the Path module, we can read any file and can also list existing files on a given path.

Because `ctx.home` returns an instance with the path being the current path, here we only need to list the files that are present and which contain the flag file.

![](https://i.imgur.com/APcYmfq.png)

Payload: `?text=<%= a=ctx.home %><% for await (const file of a.list({recursive: true})) { =%><%= file.toString() %><% } =%>&dir=from`


![](https://i.imgur.com/g1UUJ4E.png)

Flag stored at: `/app/app/Dockerfile/app/FL46_7BVY31.7X7`

To read the file we need to use `ctx.home` to create a new `Path` instance with the path being the filename to read through the file scheme.

![](https://i.imgur.com/POU5aMP.png)

But besides that, we cannot use quotes to create strings, because in the scheme file there are special characters that will cause syntax errors. One way to fix it is to use `regex` to get the string,

Example: `/abc/.source === "abc"`

and the path to the flag is obtained via the URL param.

Payload: `?text=<%= a=ctx.home.constructor.fromFileURL(ctx.req._query.get(/file/.source)).createReadStream() %><%= ctx.res.send(a) %>&dir=from&file=file:///app/FL46_7BVY31.7X7&dir=from`

![](https://i.imgur.com/3tiwPce.png)

> Flag: ictf{M0J0_15N7_0N_P4YL04D54LL7H37H1N65}

