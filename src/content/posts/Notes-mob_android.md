---
title: Android Pentesting Workflow
published: 2025-12-16
description: "⭐ NANO IS HERE ⭐"
image: "./mob.png"
tags: ["Android", "Mobile Pentesting", "Frida", "Security"]
category: Android Security
draft: false
---

> [Source](./mob.png)

⭐ **NANO IS HERE** ⭐  
--------------------

**اللهم صلِّ وسلم وبارك على سيدنا محمد ﷺ.**

# Android Pentesting: A Practical Guide

## Introduction

Android applications are everywhere today, and many of them handle sensitive information such as **credentials**, **tokens**, **personal data**, and **financial information**.

Unlike traditional applications, Android apps run on devices controlled by users. This creates a unique security challenge because an attacker can analyze, modify, and interact with the application locally.

In this article, we will follow a practical **Android penetration testing workflow**, including:

- **Lab Setup**
- **APK Reconnaissance**
- **Static Analysis**
- **Reverse Engineering**
- **Runtime Testing**
- **Common Android Security Issues**

> Always perform security testing only on applications where you have proper authorization.

---

# 1. Setting Up The Lab

A basic **Android Pentesting environment** requires:

- **Android Emulator** or **Rooted Device**
- **Android SDK / ADB**
- **JADX**
- **APKTool**
- **Frida**
- **Burp Suite**

Recommended setup:

```
Android Version: Android 13+
Architecture: x86_64
Root Access: Enabled
```

Having a rooted testing environment helps during security assessments because it allows access to:

- **Application private files**
- **Runtime memory**
- **System information**

---

# 2. APK Recon & Static Analysis

The first step in any Android assessment is understanding the application structure.

Before testing vulnerabilities, we need to identify:

- What permissions the application requests
- Which components are exposed
- How sensitive data is handled
- Where security controls exist


Important APK files:

```
AndroidManifest.xml
classes.dex
resources/
assets/
```

---

# AndroidManifest.xml Analysis

The **AndroidManifest.xml** file defines how the application interacts with the Android system.

It contains:

- **Permissions**
- **Activities**
- **Services**
- **Broadcast Receivers**
- **Content Providers**


Example:

```xml
<activity
    android:name=".AdminActivity"
    android:exported="true"/>
```

The **exported** attribute determines whether other applications can interact with this component.

However, `exported=true` is not always a vulnerability.

The real problem happens when:

- A sensitive component is exposed
- No authentication check exists
- Authorization is handled only on the client side


Example:

```java
Intent intent = new Intent();

intent.setClassName(
"com.target.app",
"com.target.app.AdminActivity"
);

startActivity(intent);
```

An attacker application may directly launch the activity if proper security checks are missing.

---

# 3. Reverse Engineering APKs

After collecting basic information, the next step is analyzing the application logic.

Common tools:

- **JADX**
- **APKTool**


## JADX Analysis

**JADX** converts Android bytecode into readable Java code.

Example:

```bash
jadx-gui target.apk
```

During analysis, I usually search for sensitive keywords:

```
password
token
api_key
secret
firebase
debug
admin
```

Possible findings:

- **Hardcoded credentials**
- **API keys**
- **Hidden functionality**
- **Debug features**
- **Sensitive endpoints**


Example:

```java
String apiKey = "AIzaXXXXXXXX";
```

Hardcoded secrets inside APK files should be considered exposed because anyone can download and analyze the application.

---

# APKTool Analysis

While **JADX** is useful for reading code, **APKTool** helps analyze application resources and modify APK files.

Decompile APK:

```bash
apktool d target.apk -o output
```

Generated structure:

```
output/

├── AndroidManifest.xml
├── smali/
├── res/
└── assets/
```

APKTool can be used to analyze:

- **Resources**
- **Smali code**
- **Application configurations**

---

# 4. Insecure Data Storage

One common issue in Android applications is storing sensitive information locally without proper protection.

Common storage locations:

```
/data/data/<package_name>/
```


Things to check:

- **SharedPreferences**
- **SQLite databases**
- **Cache files**
- **Logs**


Example:

```
shared_prefs/user.xml
```

If the application stores:

- **Passwords**
- **Tokens**
- **Session IDs**
- **Encryption keys**

in plaintext, it may lead to sensitive data exposure.

---

# 5. Runtime Analysis

Runtime analysis focuses on observing the application's behavior while it is running.

Unlike static analysis, which examines the APK structure and source code, runtime testing allows testers to analyze the application during execution.

During runtime analysis, testers can monitor:

- Application behavior
- API communication
- Memory usage
- Security controls
- Runtime values

Common tools used in this phase:

- **ADB**
- **Frida**
- **Burp Suite**

Runtime analysis helps identify issues that may not appear during static analysis, such as insecure runtime behavior, weak client-side controls, and unexpected application interactions.

---

# 6. Common Android Security Issues

Android applications may contain different security weaknesses depending on their implementation and design.

During an Android security assessment, testers usually focus on the following areas:

- **Application Components**
- **Data Storage**
- **Authentication Controls**
- **Network Communication**
- **WebView Configuration**
- **Client-Side Security Mechanisms**

Common Android security issues include:

- **Exported Components**
- **Weak Authentication Controls**
- **Insecure Data Storage**
- **Hardcoded Secrets**
- **WebView Vulnerabilities**
- **Root Detection Issues**
- **SSL Pinning Misconfiguration**
- **Weak Network Security Configuration**

Identifying these issues helps determine whether an application exposes sensitive data or allows unauthorized access to protected functionality.

---

---
---

# 7. Android Debug Bridge (ADB)

**Android Debug Bridge (ADB)** is one of the first tools I use during an Android assessment. It provides direct communication with the device, making it possible to inspect applications, collect information, and interact with Android from the command line.

Some of the most useful commands include:

```bash
# List connected devices
adb devices

# Install an APK
adb install target.apk

# Uninstall an application
adb uninstall com.target.app

# Open an interactive shell
adb shell

# View application logs
adb logcat

# List installed packages
adb shell pm list packages

# Launch an Activity
adb shell am start -n com.target.app/.MainActivity
```

ADB is commonly used to:

- Install or remove applications.
- Access the Android shell.
- Extract files from the device.
- Monitor application logs during testing.

For example:

```bash
adb shell am start \
-n com.target.app/.AdminActivity
```

This command can be used to test whether an Activity is properly protected against unauthorized access.
Another useful command is:

```bash
adb logcat
```
Many developers accidentally leave debugging statements in production builds. **Sensitive information** such as **tokens**, **API** responses, or even user **credentials** may appear inside the logs.

# 8. Dynamic Analysis with Frida

Static analysis explains how an application is built, but dynamic analysis shows how it behaves while running.

**Frida** allows testers to hook Java methods, modify return values, and observe application behavior without rebuilding the APK.

Typical use cases include:

- Bypassing Root Detection
- Bypassing SSL Pinning
- Monitoring sensitive methods
- Modifying application logic
- Inspecting runtime values

Example:

```javascript
Java.perform(function () {

    var RootDetection = Java.use(
        "com.target.app.RootDetection"
    );

    RootDetection.isRooted.implementation = function () {

        console.log("[+] Root Detection Bypassed");

        return false;

    };

});
```

Run the script:

```bash
frida -U -f com.target.app -l bypass.js
```

In this example, the application normally checks whether the device is rooted.

Original behavior:

```java
if(isRooted()){
    showError();
}
```

After hooking the method, **Frida** forces `isRooted()` to always return **false**, allowing the application to continue as if the device were not rooted.



---

# 9. Intercepting Network Traffic

Most Android applications communicate with backend services through REST APIs.

Intercepting this traffic helps identify:

- Authentication weaknesses
- Missing authorization
- Sensitive information leakage
- Insecure API design

The most common proxy used during Android assessments is **Burp Suite**.

Basic setup:

1. Start Burp Suite.
2. Configure the emulator to use Burp as its proxy.
3. Install Burp's CA certificate.
4. Launch the application.

If everything is configured correctly, requests similar to the following should appear:

```http
POST /api/login HTTP/1.1

{
    "username":"admin",
    "password":"password"
}
```

Being able to inspect requests allows testers to modify parameters, replay requests, and analyze server-side validation.

---

# 10. SSL Pinning

Installing Burp's certificate is often not enough.

Many modern Android applications implement **SSL Pinning**, meaning they trust only specific certificates embedded inside the application.

Instead of accepting any trusted certificate installed on the device, the application verifies that the server presents the expected certificate or public key.

As a result, HTTPS interception fails.

A common technique for testing applications protected by SSL Pinning is using **Frida**.

LIKE THIS FOR bypass SSL i made this for explain  Script:
```
Java.perform(function () {

    var SSLPinning = Java.use(
        "com.target.app.security.SSLValidator"
    );

    SSLPinning.verifyCertificate.implementation = function () {

        console.log(
            "[+] Custom SSL Pinning Bypass"
        );

        return true;

    };

});
```
and after that push with runtime 

Example:

```bash
frida -U -f com.target.app -l script.js
```
Where `script.js` contains the required SSL Pinning bypass hooks.
Once SSL Pinning has been bypassed, Burp Suite can inspect encrypted traffic normally.

This allows testers to analyze:

- Authentication tokens
- Session cookies
- API requests
- API responses
- Sensitive transmitted data

---

# 11. Common Testing Checklist

Before completing an Android assessment, verify the following:

- **Permissions** have been reviewed.
- **Exported Activities** have been tested.
- **Services** have been analyzed.
- **Broadcast Receivers** have been inspected.
- **Content Providers** have been reviewed.
- **SharedPreferences** have been inspected.
- **SQLite Databases** have been analyzed.
- **Hardcoded Secrets** have been identified.
- **Application Logs** have been reviewed.
- **Root Detection** has been evaluated.
- **SSL Pinning** has been tested.
- **WebView** configuration has been analyzed.
- **Network Traffic** has been intercepted and inspected.

---

---

# 12. Android Component Testing with Drozer

**Drozer** is an Android security assessment framework designed to interact with application components from outside the target application.

Instead of manually writing an Android application to test exported components, Drozer provides ready-to-use modules that simplify the process.

It is mainly used to assess:

- **Activities**
- **Services**
- **Broadcast Receivers**
- **Content Providers**

Before using Drozer, install the **Drozer Agent** on the Android device, then forward the communication port:

```bash
adb forward tcp:31415 tcp:31415

drozer console connect
```

After connecting, start by identifying the target application:

```bash
run app.package.list
```

Display basic information about the application:

```bash
run app.package.info -a com.target.app
```

To enumerate exported Activities:

```bash
run app.activity.info -a com.target.app
```

If an exported Activity is found, try launching it directly:

```bash
run app.activity.start \
--component com.target.app \
com.target.app.AdminActivity
```

If the Activity exposes sensitive functionality without proper authentication or authorization checks, this may indicate an **Authentication Bypass** or **Broken Access Control** vulnerability.

Drozer can also enumerate other Android components, including **Services**, **Broadcast Receivers**, and **Content Providers**, making it useful for quickly identifying exposed attack surfaces during an assessment.

Although many testers now prefer combining **ADB** and **Frida** for advanced testing, Drozer remains a valuable tool for Android component enumeration and access control testing.



---

# 13. Android Pentesting Summary

Android penetration testing is a complete process that combines static analysis, reverse engineering, runtime analysis, and network testing to identify security weaknesses inside Android applications.

A typical Android pentesting workflow includes:

- **Reconnaissance**  
  Understanding the application structure, permissions, and exposed components.

- **Static Analysis**  
  Reviewing APK files, source code, configurations, and identifying security issues.

- **Reverse Engineering**  
  Analyzing application logic, resources, and sensitive information inside the APK.

- **Runtime Analysis**  
  Observing application behavior during execution using tools such as ADB and Frida.

- **Network Testing**  
  Inspecting API communication, authentication mechanisms, and data transmission.

- **Security Testing**  
  Testing common vulnerabilities such as insecure storage, exported components, weak authentication, SSL Pinning issues, and improper access controls.

Common tools used during Android security assessments include:

- **JADX**
- **APKTool**
- **ADB**
- **Frida**
- **Burp Suite**
- **Drozer**
- **MobSF**

A successful Android security assessment requires combining multiple techniques to analyze the application from different perspectives, including its code, runtime behavior, device interaction, and backend communication.

Always perform security testing only on applications where you have proper authorization.
**THere is part two BUT not basics more it will be advanced inshallah**


Mohem مهم جدا انك تسرش وتحل تحديات وتختبر الابليكشن بيدك ترفيوو السورس كود وان الشخص يتعلم مينفعش من مصدر واحد الحاجه تتعلمها من اكتر من مصدر لان كل عقليه غير التاني والمعلومات  محدش معاه المعلومه كامل ممكن تسمع من حد تاني لاقيه شرح حاجه انت متعرفهاش 
ربنا يبارك فيكم وفينا ويرحمنا برحمته يارب 
=============================