---
title: 'Reverse Engineering WannaCry Ransomware: A Deep Dive'
date: '2026-02-20 00:00:00 +0530'
categories:
  - Malware
  - Ransomware
tags:
  - WannaCry
  - Reverse Engineering
  - Static Analysis
  - Dynamic Analysis
  - EternalBlue
description: >-
  A hands-on reverse engineering walkthrough of the infamous WannaCry ransomware
  — covering static analysis, kill switch discovery, encryption routine, and
  IOCs.
---
## Overview

**WannaCry** (also known as WannaCrypt, WCry) is one of the most destructive pieces of ransomware ever deployed. It hit systems worldwide in **May 2017**, infecting over 230,000 machines across 150 countries in a single day.

What made WannaCry unique wasn't just its encryption — it was a **self-propagating worm** that exploited the **EternalBlue** SMB vulnerability (MS17-010), a zero-day initially developed by the NSA and later leaked by the Shadow Brokers group.

---

## Sample Information

| Property | Value |
|----------|-------|
| **MD5** | `84c82835a5d21bbcf75a61706d8ab549` |
| **SHA256** | `24d004a104d4d54034dbcffc2a4b19a11f39008a575aa614ea04703480b1022c` |
| **File Type** | PE32 Executable (Windows) |
| **File Size** | 3.4 MB |
| **Packer** | None (unpacked sample) |
| **First Seen** | 2017-05-12 |

---

## Static Analysis

### Strings Extraction

Running `strings` on the binary reveals some juicy artifacts:

```bash
$ strings wannacry.exe | grep -i http
http://www.iuqerfsodp9ifjaposdfjhgosurijfaew.com
```

> 💡 **This URL is the kill switch!** Marcus Hutchins (MalwareTech) registered this domain for ~$10.69 and accidentally stopped the global outbreak.

Other interesting strings found:

```
tasksche.exe
icacls . /grant Everyone:F /T /C /Q
attrib +h .
cmd.exe /c "%s"
WannaDecryptor
WANACRY!
```

### PE Header Analysis (PEStudio)

The import table reveals key Windows API calls:

```
KERNEL32.dll
  - CreateFileA / WriteFile / ReadFile       → File I/O (encryption targets)
  - CreateThread                             → Multi-threaded encryption
  - VirtualAlloc / VirtualProtect            → Memory allocation

ADVAPI32.dll
  - CryptGenKey / CryptEncrypt               → RSA/AES key generation
  - CryptAcquireContextA

WS2_32.dll
  - connect / send / recv                    → C2 communication
  - WSAStartup
```

### Entropy Analysis

The `.text` section shows **normal entropy (~6.2)**, confirming the sample is **not packed**. The resource section contains an embedded ZIP file (encrypted tools).

---

## Dynamic Analysis

### Environment Setup

- **Host OS:** Windows 10 (isolated VM — no network)
- **Tools:** x64dbg, Process Monitor, Wireshark, Regshot
- **Network:** INetSim to simulate internet responses

### Execution Behavior

On first execution (captured with **Process Monitor**):

```
1. Checks for kill-switch domain → DNS query to iuqerfsodp9ifjaposdfjhgosurijfaew.com
2. If domain unreachable → continues execution
3. Drops files to C:\Windows\mssecsvc.exe
4. Creates Windows service: "mssecsvc2.0"
5. Scans LAN for port 445 (SMB)  ← worm propagation
6. Encrypts files on disk
7. Drops ransom note: @WanaDecryptor@.exe
```

### Registry Changes (Regshot diff)

```
HKLM\SYSTEM\CurrentControlSet\Services\mssecsvc2.0
  ImagePath = C:\Windows\mssecsvc.exe -m security
  Start = 2 (Automatic)
```

---

## Encryption Routine (Reverse Engineered)

Using **Ghidra** to decompile the encryption logic:

```c
// Pseudocode from Ghidra decompilation
void encrypt_file(char* filepath) {
    // Generate per-file AES-128 key
    BYTE aes_key[16];
    CryptGenRandom(hProv, 16, aes_key);
    
    // Encrypt file content with AES-128-CBC
    encrypt_aes128(filepath, aes_key);
    
    // Encrypt AES key with attacker's RSA-2048 public key
    RSA_encrypt(aes_key, attacker_pubkey, encrypted_key_blob);
    
    // Append encrypted key to .WNCRY file
    append_to_file(filepath + ".WNCRY", encrypted_key_blob);
    
    // Delete original (NOT securely wiped!)
    DeleteFile(filepath);
}
```

> ⚠️ **Key Finding:** WannaCry uses `DeleteFile()` rather than secure wiping — meaning forensic tools like Recuva could potentially recover **some** original files!

---

## Network Traffic (Wireshark)

```
[SMB Probe] → Port 445 scan on 192.168.x.x/24
[EternalBlue] → SMBv1 exploit payload sent to vulnerable hosts
[DoublePulsar] → Backdoor installed via kernel shellcode
[C2 Beacon] → POST to Tor hidden service (for key exchange)
```

---

## YARA Detection Rule

```yara
rule WannaCry_Ransomware {
    meta:
        description = "Detects WannaCry ransomware"
        author = "Mayank Bhardwaj"
        date = "2026-02-20"
        
    strings:
        $kill_switch = "iuqerfsodp9ifjaposdfjhgosurijfaew.com" ascii
        $magic = "WANACRY!" ascii
        $tasksche = "tasksche.exe" ascii
        $ransom_ext = ".WNCRY" ascii wide
        
    condition:
        uint16(0) == 0x5A4D and  // PE file
        3 of ($kill_switch, $magic, $tasksche, $ransom_ext)
}
```

---

## Indicators of Compromise (IOCs)

### File Hashes
```
tasksche.exe   MD5: 7bf2b57f2a205768755c07f238fb32cc
mssecsvc.exe   MD5: 2ca2d550e603d74dedda03156023135b
```

### Registry
```
HKLM\SYSTEM\CurrentControlSet\Services\mssecsvc2.0
```

### Network
```
Domain: iuqerfsodp9ifjaposdfjhgosurijfaew.com (kill switch — now sinkholed)
Port:   445/TCP (EternalBlue propagation)
Port:   9001, 9003 (Tor C2)
```

---

## Mitigation

1. **Patch MS17-010** (KB4012212) — disable SMBv1 immediately
2. **Block port 445** at the network perimeter
3. **Offline backups** — tested and verified regularly
4. Deploy **YARA rule above** in your EDR/SIEM

---

## Conclusion

WannaCry remains a masterclass in destructive APT-level malware. What started as an NSA offensive tool ended up crippling hospitals, banks, and government agencies. The accidental kill-switch discovery by Marcus Hutchins is one of the most remarkable events in cybersecurity history.

**Next post:** I'll be reverse engineering **NotPetya** — which had no kill switch and was designed purely to destroy, not extort.

---

*Tools used: PEStudio, Detect-It-Easy, Ghidra, x64dbg, Process Monitor, Wireshark, YARA*
