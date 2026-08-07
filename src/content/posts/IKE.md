---
title: IKE _IPSEC( Port 500/4500) Explained
published: 2025-06-22
description: ⭐ NANO IS HERE ⭐
image: "./Ike_gu.png"
tags: [IKE, IPsec, NetworkSecurity, TechGuide, Encryption]
category: Network
draft: false
---
> Cover image source: [Source](./Ike_gu.png)

IKE _IPSEC( Port 500/4500) Explained
=====================================


⭐ NANO IS HERE ⭐
----------------

> _Port 500 is used by IPsec for IKE to set up secure VPN connections over UDP._
> 
> _When passing through NAT, IPsec switches to port 4500 using NAT Traversal._

What is IPsec?
--------------

**IPsec = Internet Protocol Security**

It is a technology that protects data on the network by:

*   Encrypting the data (Confidentiality)
*   Making sure the data is not changed (Integrity)
*   Verifying the identity of the sender (Authentication)
*   Preventing tampering or replay attacks

Used mainly in:

*   Site-to-Site VPN
*   Remote Access VPN
*   Protecting traffic between servers
*   Government and enterprise networks

Main Components of IPsec (3 Core Parts)
---------------------------------------

AH — Authentication Header
--------------------------

Provides:

*   Identity verification
*   Data integrity
*   No encryption

AH stops modification of data, but does not hide it.

Used when confidentiality is not required

ESP — Encapsulation Security Payload
------------------------------------

This is the main protocol used in IPsec.

It provides:

*   Full encryption (Confidentiality)
*   Integrity
*   Authentication
*   **Anti-Replay** ← **It prevents replay attacks**

Most VPNs use **ESP**.

IKE — Internet Key Exchange
---------------------------

Responsible for:

*   Exchanging encryption keys
*   Creating Security Associations
*   Choosing encryption algorithms
*   Selecting protocols and settings

Without **IKE**, IPsec cannot work.

IPsec Modes
-----------

IPsec works in two modes:

1️ Transport Mode
-----------------

*   Encrypts **only the payload**
*   Original IP header stays visible
*   Used for **end-to-end** communication Examples:
*   Server ←→ Server Host←→Host

2️ Tunnel Mode
--------------

*   Encrypts the **entire packet** (Header + Data)
*   Wraps it inside a new packet
*   Used in **VPNs** Examples: Firewall ←→Firewall Route ←→Router

This is the highest security level for public networks.

Security Association (SA)
-------------------------

For IPsec to work, devices must create an **SA (Security Association)**.

An SA contains:

*   Encryption type (AES, 3DES, and more)
*   Integrity method (SHA256, and more)
*   Mode (Tunnel / Transport)
*   Protocol (ESP / AH)
*   Key lifetime

Each direction has **its own SA**.

IPsec Encryption Tools
----------------------

IPsec uses 3 main components:

1.  Encryption Algorithms

*   AES
*   3DES
*   Blowfish
*   ChaCha20

1.  Integrity Algorithms

*   SHA1
*   SHA256
*   SHA512

1.  Diffie–Hellman (DH) Groups

Used for secure key exchange:

*   Group 2
*   Group 5
*   Group 14
*   Group 21 (ECC — provides high security with greater efficiency)

IPsec VPN Phases
----------------

Phase 1 — IKE Phase 1
---------------------

Creates a secure channel (The **IKE SA**):

*   Builds encrypted tunnel
*   Authenticates both sides
*   Chooses encryption settings

Modes (IKEv1): Main Mode / Aggressive Mode

Phase 2 — IKE Phase 2
---------------------

*   Creates the **IPsec SA** (using Quick Mode in IKEv1)
*   Selects ESP or AH
*   Defines what traffic goes through the tunnel
*   Generates new keys

**Note on IKEv2:** IKEv2 is the modern, more efficient version, which streamlines these two phases into fewer steps.

Main Branches of IPsec
----------------------

Protocols
---------

*   AH
*   ESP
*   IKEv1
*   IKEv2

Modes
-----

*   Transport
*   Tunnel

Architectures
-------------

*   Site-to-Site VPN
*   Remote Access

Encryption Families
-------------------

*   AES
*   DES/3DES
*   ECC

Authentication Types
--------------------

*   PSK (Pre-Shared Key)
*   Certificates
*   RSA
*   ECDSA

Simple Example
--------------

Imagine two entities:

They want secure communication.

IPsec will:

*   Use **Tunnel Mode**
*   Encrypt everything with **ESP**
*   Use **IKE** to negotiate keys
*   Wrap the original packet inside a new encrypted one
*   Prevent spying or spoofing

Summary
-------

FLOW

```
| Component          | Function                                |
| ------------------ | --------------------------------------- |
| **AH**             | Identity + Integrity                    |
| **ESP**            | Encryption + Integrity + Authentication |
| **IKE**            | Negotiation + Key exchange              |
| **Transport Mode** | Encrypts data only                      |
| **Tunnel Mode**    | Encrypts entire packet                  |
| **SA**             | Security agreement                      |
```

