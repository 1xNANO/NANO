---
title: iOS Pentesting Notes
published: 2026-03-19
description: ⭐ NANO IS HERE ⭐
tags: [Penetration Testing, iOS Security, Mobile Security, Reverse Engineering, Frida, API Security]
image: "./IOs-notes.png"
category: IOS Security
draft: false
---

> Cover image source: [Source](./IOs-notes.png)

 Introduction
---------------

## Introduction

iOS pentesting can look confusing at first.

You open an `.ipa`, see a bunch of files, hear about **Frida**, **Objection**, **Mach-O**, **Keychain**, **Swift**, **Objective-C**, and suddenly everything feels like a lot.

This cheat sheet is basically a collection of the things I would want in front of me while testing an iOS application.

The goal isn't to memorize commands.

The goal is to understand **what you're looking for and why you're looking for it**.

---

# iOS Pentesting Workflow

A simple workflow to keep in mind:

```text
Recon
  ↓
Get the App
  ↓
Static Analysis
  ↓
Install & Run
  ↓
API / Network Testing
  ↓
Local Storage
  ↓
Runtime Analysis
  ↓
Authentication & Authorization
  ↓
iOS-specific Attack Surfaces
  ↓
Validate Impact
  ↓
Report
```

Don't start with Frida just because everyone talks about Frida.

First understand the application.

---

# 1. Lab Setup

For a proper iOS pentesting lab, macOS is highly recommended.

You'll probably want:

* Xcode
* Burp Suite
* Frida
* Objection
* Ghidra
* LLDB
* Python
* An iOS device or simulator

For example:

```bash
brew install python
brew install frida
brew install --cask burp-suite
```

For Objection:

```bash
pipx install objection
```

You don't need every tool immediately.

Start with the basics and add tools when you actually need them.

---

# 2. Getting the Application

Depending on the engagement, you might receive:

```text
.ipa
.app
TestFlight build
Development build
```

If you're working with an authorized device setup, you can inspect installed applications.

For example:

```bash
ideviceinstaller -l
```

Once you have the application, the next step is understanding what is actually inside it.

---

# 3. Inspecting an IPA

An `.ipa` is basically a ZIP archive.

So:

```bash
unzip application.ipa
```

You'll normally get something like:

```text
Payload/
└── Example.app/
    ├── Example
    ├── Info.plist
    ├── Frameworks/
    ├── PlugIns/
    └── ...
```

The main executable is usually:

```text
Payload/Example.app/Example
```

That's one of the files we'll spend a lot of time looking at.

---

# 4. Info.plist

One of the first things I check is:

```text
Info.plist
```

It can tell you quite a lot about the application.

Things worth checking:

```text
Bundle Identifier
URL Schemes
Associated Domains
Permissions
Background Modes
ATS Configuration
Application Configuration
```

You can inspect it with:

```bash
plutil -p Info.plist
```

Or convert it to XML:

```bash
plutil -convert xml1 Info.plist -o -
```

---

# 5. Strings

This is one of the simplest tricks, but it's still useful.

Extract strings:

```bash
strings Example > strings.txt
```

Then search for interesting stuff:

```bash
grep -i "password" strings.txt
grep -i "token" strings.txt
grep -i "secret" strings.txt
grep -i "apikey" strings.txt
grep -i "debug" strings.txt
```

You might find:

```text
API endpoints
Debug messages
Internal URLs
Feature flags
Class names
Error messages
```

But don't make the mistake of seeing a string called `api_key` and immediately calling it a vulnerability.

You need to figure out what that value actually is and whether it has any security impact.

---

# 6. Mach-O

iOS applications use the Mach-O executable format.

You can start with:

```bash
file Example
```

Then inspect linked libraries:

```bash
otool -L Example
```

And inspect the Mach-O header:

```bash
otool -hv Example
```

For deeper reverse engineering, you can move to:

```text
Ghidra
Hopper
IDA
LLDB
```

---

# 7. Static Analysis

When looking through the binary, don't just browse randomly.

Have questions.

For authentication, search for things related to:

```text
login
authenticate
session
token
refresh
logout
password
```

For sensitive information:

```text
token
secret
key
authorization
cookie
password
```

For network functionality:

```text
URLSession
NSURLSession
Alamofire
https://
certificate
pinning
```

For development functionality:

```text
debug
test
staging
development
internal
mock
```

You're basically trying to build a mental map of the application.

---

# 8. Local Storage

This is something I wouldn't skip.

Look for places where the application stores data:

```text
NSUserDefaults
Keychain
SQLite
.plist
Caches
Documents
Library
Temporary files
```

The important question isn't:

> "Does the app store data?"

Almost every app does.

The question is:

> "What exactly is being stored?"

For example:

```text
Username → probably fine
Theme preference → fine
Access token → interesting
Password → very interesting
Encryption key → very interesting
Sensitive personal data → investigate
```

---

# 9. Keychain

The Keychain is designed for sensitive data.

Things you might find there:

```text
Access Tokens
Refresh Tokens
Credentials
Session Identifiers
Encryption Keys
```

But don't stop at:

> "It's in Keychain, so it's secure."

Look at how it's stored and what protection is applied.

Also check whether the same secret is being copied somewhere less secure.

For example:

```text
Keychain
   +
NSUserDefaults
   +
Logs
```

Storing something securely in one place doesn't help much if the application also leaks it somewhere else.

---

# 10. API Testing

This is where Burp Suite becomes very useful.

The basic setup is:

```text
iPhone
   ↓
Burp Suite
   ↓
Internet
   ↓
API
```

Start by using the application normally.

Capture:

```text
Login
Register
Profile
Password reset
File upload
Payments
Logout
```

Don't immediately attack everything.

First understand how the API works.

---

# 11. What I Check in API Requests

For every interesting request, I ask myself:

```text
Who am I?
What resource am I accessing?
What proves I'm authenticated?
What proves I'm authorized?
Can I change the ID?
Can I remove parameters?
Can I replay the request?
Does the server validate the request?
```

For example:

```http
GET /api/users/123/profile
Authorization: Bearer <token>
```

If I'm testing with two authorized accounts, I might check whether changing the object identifier allows one account to access another account's data.

That's the kind of test that can reveal an authorization issue.

---

# 12. Authentication vs Authorization

These are two different things.

### Authentication

> Who are you?

Examples:

```text
Password
OTP
Session token
Biometric authentication
```

### Authorization

> What are you allowed to access?

For example:

```text
User A
   ↓
Requests User B's invoice
   ↓
Server says NO
```

If the server says YES, that's where things get interesting.

A lot of API bugs are authorization problems rather than authentication problems.

---

# 13. TLS / Certificate Pinning

Normally, you want to inspect the application's HTTPS traffic through your proxy.

Sometimes the application uses certificate pinning.

Then you might see something like:

```text
App → API

but

Burp sees nothing useful
```

If you're testing an authorized application, runtime instrumentation can help you understand where the TLS validation is happening.

This is one of the places where Frida becomes useful.

---

# 14. Frida

Frida is one of the most important tools in mobile application testing.

List processes:

```bash
frida-ps -U
```

Attach to an application:

```bash
frida -U -n "ExampleApp"
```

Spawn an application:

```bash
frida -U -f com.example.app
```

Load a JavaScript script:

```bash
frida -U -f com.example.app -l script.js
```

The important part isn't memorizing these commands.

You want to use Frida when you have a specific runtime question.

For example:

> "What value is this function receiving?"

or:

> "Is this security check actually being called?"

---

# 15. Objection

Objection is built around Frida and makes a lot of runtime testing easier.

Start an application:

```bash
objection -g com.example.app explore
```

From there you can explore the application using Objection's available commands.

I personally think it's easier to understand Objection **after** learning some Frida basics.

Otherwise it can feel like you're just copying commands without understanding what's happening.

---

# 16. Runtime Analysis

Static analysis tells you what the application **contains**.

Dynamic analysis tells you what the application **does**.

For example:

```text
Static:

"There's a function checking isPremium."
```

Then runtime testing asks:

```text
"What happens if this value changes?"
```

That's the mindset you want.

---

# 17. Objective-C Runtime

If you're testing an Objective-C application, you'll see classes and methods such as:

```text
ClassName
methodName:
init
viewDidLoad
application:didFinishLaunchingWithOptions:
```

Frida can be used to inspect Objective-C classes and methods during runtime.

This becomes especially useful when you're trying to understand how a specific feature works.

---

# 18. Swift

A lot of modern iOS applications are written in Swift.

You'll run into:

```text
Swift symbols
Mangling
Protocols
Generics
Closures
Frameworks
```

You don't need to become a professional Swift developer before starting iOS pentesting.

But learning the basics of:

```text
Swift
Objective-C
Mach-O
ARM64
iOS architecture
```

will make reverse engineering much easier.

---

# 19. URL Schemes

Check the URL schemes defined by the application.

For example:

```text
myapp://profile
myapp://reset
myapp://payment
```

Then ask:

```text
Can anyone open this?
Does it require authentication?
Can parameters be modified?
Can a sensitive action be triggered?
```

Don't assume that because something comes through a URL, it is trusted.

---

# 20. Universal Links

Universal Links are another interesting attack surface.

For example:

```text
https://example.com/reset
https://example.com/profile
https://example.com/payment
```

Check whether manipulating parameters changes the application's behavior.

Especially interesting:

```text
Account actions
Password reset
Authentication
Payment flows
Sensitive resources
```

---

# 21. WebViews

If the application uses WebViews, pay attention to the bridge between native code and web content.

Things worth checking:

```text
JavaScript
Custom URL handlers
Native bridges
Local files
External content
Authentication data
```

WebViews can become interesting when the native side exposes powerful functionality to untrusted web content.

---

# 22. Biometric Authentication

If the app uses Face ID / Touch ID, don't just check whether the popup appears.

Ask:

> What exactly does the biometric check protect?

Test normal scenarios:

```text
Login
Open protected feature
Fail authentication
Successfully authenticate
Restart application
Lock device
Unlock device
Expire session
```

A biometric prompt appearing on the screen doesn't automatically mean the underlying action is properly protected.

---

# 23. Jailbreak Detection

Some applications try to detect jailbroken devices.

From a pentesting perspective, the interesting question isn't simply:

> "Can I bypass jailbreak detection?"

Instead:

> "What security decision depends on this detection?"

For example:

```text
Jailbreak detected
       ↓
Application blocks sensitive functionality
```

If bypassing the check gives access to something sensitive, then you have something worth investigating.

A bypass by itself isn't automatically a vulnerability.

---

# 24. Client-Side Security

Be suspicious of security decisions made entirely on the client.

Things like:

```text
isAdmin
isPremium
isVerified
isAuthenticated
debugMode
```

If the client says:

```text
isAdmin = true
```

that doesn't necessarily mean the server should trust it.

The server should enforce important authorization decisions.

---

# 25. Logs

Don't forget logs.

Look for:

```text
Passwords
Tokens
Session IDs
Personal information
API responses
Internal URLs
Debug information
```

A secret appearing in logs can become a real security issue depending on the environment and who can access those logs.

---

# 26. Backups & Files

Sensitive information can also end up in:

```text
Caches
Temporary files
Databases
Screenshots
Backups
Exported files
Application documents
```

Again, don't just search for the word:

```text
password
```

Look for actual sensitive data and determine whether exposing it matters.

---

# 27. Quick Testing Checklist

```text
[ ] IPA structure
[ ] Info.plist
[ ] Hardcoded secrets
[ ] Static analysis
[ ] API endpoints
[ ] Authentication
[ ] Authorization
[ ] Session handling
[ ] Token storage
[ ] Keychain
[ ] Local databases
[ ] Logs
[ ] Backups
[ ] TLS
[ ] Certificate pinning
[ ] Deep links
[ ] Universal Links
[ ] WebViews
[ ] Biometric authentication
[ ] Jailbreak detection
[ ] Client-side controls
[ ] Debug functionality
```

---

# 28. Reporting

Finding a weird behavior is not the end.

You need to prove it.

A simple report structure:

```markdown
## Title

Short and specific title.

## Severity

Medium

## Description

Explain what is happening.

## Steps to Reproduce

1. Login as User A.
2. Perform the required action.
3. Capture the request.
4. Modify the relevant parameter.
5. Send the request.
6. Observe the response.

## Expected Result

Explain what should happen.

## Actual Result

Explain what actually happens.

## Impact

Explain what an attacker could realistically do.

## Recommendation

Explain how the application should fix the issue.
```

Keep the report technical, but easy to reproduce.

---

# 29. My Small Toolkit

You don't need 50 different tools.

A basic toolkit can look like this:

| Tool       | Main Use                    |
| ---------- | --------------------------- |
| Burp Suite | API / HTTP testing          |
| Frida      | Runtime instrumentation     |
| Objection  | Runtime exploration         |
| Xcode      | iOS development & debugging |
| LLDB       | Debugging                   |
| Ghidra     | Reverse engineering         |
| otool      | Mach-O inspection           |
| strings    | Quick static analysis       |
| Python     | Automation                  |

---

# 30. Learning Path

If you're starting from zero, I wouldn't try to learn everything at the same time.

Go roughly like this:

```text
iOS Basics
   ↓
Swift / Objective-C Basics
   ↓
IPA Structure
   ↓
Info.plist
   ↓
Static Analysis
   ↓
Burp + API Testing
   ↓
Local Storage
   ↓
Keychain
   ↓
Frida
   ↓
Objection
   ↓
Mach-O
   ↓
ARM64 Basics
   ↓
Advanced Runtime Analysis
```

You don't need to master one topic completely before touching the next.

Learn enough to understand what you're doing, then practice.

---

# Final Thoughts

The biggest mistake I see beginners make is collecting commands.

They know:

```bash
frida ...
objection ...
otool ...
strings ...
```

but when you ask:

> "Why are you running this?"

they don't have an answer.

Try doing the opposite.

Start with a question.

For example:

> Where is the application storing my token?

Then choose the tool.

Or:

> Is authorization actually enforced by the server?

Then intercept the request.

Or:

> What happens when this function receives a different value?

Then use runtime instrumentation.

The tools are just tools.

The real skill is learning how to **ask the right security questions**.

---

## References

* [OWASP MASVS](https://mas.owasp.org/MASVS/)
* [OWASP MASTG](https://mas.owasp.org/MASTG/)
* [Frida Documentation](https://frida.re/docs/)
* [Apple Developer Documentation](https://developer.apple.com/documentation/)
* [Ghidra](https://ghidra-sre.org/)
* [Burp Suite](https://portswigger.net/burp)

> Use these techniques only on applications and devices you own or have explicit permission to test.
