# PairipJNI Self‑Dumper

<p align="center"> 
<a href="https://t.me/rktechnoindians"><img src="https://readme-typing-svg.herokuapp.com?font=Fira+Code&weight=800&size=35&pause=1000&color=F74848&center=true&vCenter=true&random=false&width=435&lines=PairipJNI" /></a>
 </p>

## 📖 Overview

**A Zygisk-free, root-free PairipJNI runtime dumper that runs inside the target process itself. load it via `system.loadlibrary()` and it automatically finds `libPairipDumper.so`, dump all fields String & reflect Method with PairipJNI APIs, and writes a `pairip.json` to the app's own files directory.**

**Extracts ( `java.lang.String` & `java.lang.reflect.Method` ) every fields String value & reflect Method, directly from the live PairipJNI runtime memory.**

**You can view JNI logs in Logcat using the filter key — `PairipJNI`**

**I haven't considered making the repo open source right now — I don't want this method to get patched and Pairip's security to become more tighter.**

**Special Thanks — [**NullRE**](https://t.me/NullRE)**

---

## 📋 Table of Contents

1. [Special Advantages](#-1-special-advantages)  
   1.1 [Encrypted Metadata Support](#-encrypted-metadata-support)  
   1.2 [Container / Virtual Machine Native Support](#-container--vm-native-support)  
   1.3 [Zero Impact on App Behaviour](#-zero-impact-on-app-behaviour)  
2. [Security Perspective](#%EF%B8%8F-3-security-perspective)  
   2.1 [What This Library Can Access](#-what-this-library-can-access)  
   2.2 [What This Library Does NOT Do](#-what-this-library-does-not-do)  
   2.3 [Policy Compliance](#%EF%B8%8F-policy-compliance)
3. [Output Locations & Fallback Order](#-4-output-locations--fallback-order)
4. [Usage Guide](#-4-usage-guide)  
   4.1 [Build Instructions](#%EF%B8%8F-build-instructions)  
5. [Usage JSON](#-5-usage-json)  
   5.1 [**RKPairip**](https://github.com/TechnoIndian/RKPairip#installation-method)  
   5.2 [**Flag `-t` ➸ Pairip Restored String Translate**](https://github.com/TechnoIndian/RKPairip#flag--t--pairip-restored-string-translate--if-you-already-have-pairipjson-)

---

## 🚀 1. Special Advantages

### ✅ Encrypted Metadata Support

The library **does not decrypt anything**. It waits until PairipJNI itself has loaded the decrypted metadata into memory, then dumps the live state. Works with any encryption scheme — no key extraction, no file parsing.

### ✅ Container / VM Native Support

>🔥 **Special Feature**
>Works inside **any container or virtual machine**. Dumps official apps without tampering their integrity.

>- Implement the library inside the container
>- Clone and run the target app inside the container
>- The `pairip.json` is written directly to a shared folder, accessible from the host

### ✅ Zero Impact on App Behaviour

- ❎ No method hooking
- ❎ No bytecode modification
- 🧵 The dump thread runs detached at default `SCHED_OTHER` priority
- ❄️ All polling loops use `sleep(3)` — no CPU busy‑wait, no battery drain

---

## 🛡️ 2. Security Perspective

### 🔑 What This Library Can Access

The library operates **entirely within the existing security boundary** of the target process. It:
- ✅ Reads `/proc/self/cmdline` — accessible to every process by design `( But i'm use only for packageName detect for output directory )`
- ✅ Writes to the app’s own files directory — storage the process already owns
- ✅ Makes JNI calls using the app’s own `JavaVM` — standard Android API

**No system call is made that the app itself could not make.**

### ❎ What This Library Does NOT Do

- ❌ Does not escalate privileges
- ❌ Does not communicate over the network
- ❌ Does not access other apps’ data
- ❌ Does not modify **any** memory (no hooks, no patches, no code injection)
- ❌ Does not persist after the process exits
- ❌ Does not request additional Android permissions

### 🏛️ Policy Compliance

Because the library loads as part of the app’s own process under the app’s UID, it operates entirely within Android’s standard application sandbox. No security policy is violated. The library is functionally equivalent to an app examining its own runtime state — a normal and permitted operation.

---

## 📁 3. Output Locations & Fallback Order

The library tries the following directories **in order**. If one fails (e.g., permission denied, mount namespace issue), it moves to the next.

**Storage Permission Granted (`targetSdkVersion < 28 -`)**

| Step | Strategy | Example path |
|---|---|---|
| 1 | External Storage | `/storage/emulated/0/MT2/dictionary/` |
| 2 | Legacy External | `/sdcard/MT2/dictionary/` |

**Storage Permission Denied (`targetSdkVersion < 29 +`)**

| Step | Strategy | Example path |
|---|---|---|
| 1 | External Storage | `/storage/emulated/0/Android/data/<pkg>/files/dictionary/` |
| 2 | Legacy External | `/sdcard/Android/data/<pkg>/files/dictionary/` |
| 3 | Multi-User Private Storage | `/data/user/0/<pkg>/files/dictionary/` |
| 4 | Context.getFilesDir() | `/data/data/<pkg>/files/dictionary/` |

---

Rebuild the library and the dump will appear exactly at that hardcoded path.

---

## 💡 4. Usage Guide

### 🔀 Implementation Instructions

The native library must be loaded **as early as possible** — ideally in the app’s `Application` class static initialiser or its Smali equivalent.

#### Patch an existing APK (Smali)

**1. Smali code to load the library:**
```smali
.method static constructor <clinit>()V
    .registers 1

    const-string v0, "PairipDumper"

    invoke-static {v0}, Ljava/lang/System;->loadLibrary(Ljava/lang/String;)V

    return-void
.end method
```

**2. Add the native libraries:**
```
lib/armeabi-v7a/libPairipDumper.so
lib/arm64-v8a/libPairipDumper.so
lib/x86/libPairipDumper.so
lib/x86_64/libPairipDumper.so
```