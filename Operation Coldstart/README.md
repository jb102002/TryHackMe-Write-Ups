# Operation Coldstart — Write-Up

### Scenario

Volt Labs, a small SaaS company, suspects an old staging server has become an exposed liability. The objective is to find a way in and demonstrate full compromise.

---

### Recon

To begin, a basic nmap scan was run against the target to identify open ports and detect any vulnerable service versions.

<img width="1916" height="917" alt="image" src="https://github.com/user-attachments/assets/048239c2-3364-4d99-b37f-5f94c0b97ffa" />

The scan revealed three open ports:

- **Port 21** — FTP running vsftpd 3.0.5
- **Port 22** — SSH running OpenSSH 9.6p1 on Ubuntu
- **Port 80** — HTTP running Gunicorn

---

### Enumeration

With HTTP confirmed, a directory brute force scan was run using Gobuster to enumerate available endpoints.

<img width="1216" height="684" alt="image" src="https://github.com/user-attachments/assets/62781580-c290-4176-b46b-337c9d978d81" />

Two endpoints were discovered: **/admin** and **/preview**. File extension fuzzing for common types such as .env, .txt, and .py did not yield any additional results.

---

### Inspecting the Web Application

<img width="1916" height="964" alt="image" src="https://github.com/user-attachments/assets/971c217e-0778-419f-8712-b6453501efca" />

The application appears to be a URL preview service that fetches external URLs and renders their contents in the browser. The footer reads "© Volt Labs · do not expose externally", confirming this page was never intended to be public facing.

This functionality is a textbook Server-Side Request Forgery (SSRF) target. If exploitable, it could allow us to borrow the server's internal network position to reach services that would otherwise be inaccessible.

---

### Endpoint Analysis

**/admin**

<img width="1276" height="620" alt="image" src="https://github.com/user-attachments/assets/ecac16b5-313d-401f-9f40-83d71772cd07" />

Navigating to /admin redirects to /admin/ and returns an HTTP 403 Forbidden response.

**/preview**

<img width="1278" height="626" alt="image" src="https://github.com/user-attachments/assets/93aaef9f-c822-49b6-ae56-346d151a26d3" />

This further confirms our suspicions of a possible SSRF entry point

The /preview endpoint confirms the SSRF suspicion. The "staging" label visible in the top right corner suggests this is a development environment that may have reduced security controls.

---

### SSRF Exploitation Attempt

Testing a basic SSRF payload against the ?url= parameter returned a block message indicating a server-side allowlist was in place.

<img width="1282" height="626" alt="image" src="https://github.com/user-attachments/assets/2fdd6481-ac5a-4306-a870-b56292b619af" />

All obfuscation payloads attempted were blocked. Header injection was also tested via Burp Suite, including Host header manipulation, but did not prove effective.

---

### FTP Enumeration

With the allowlist proving resistant to bypass, attention shifted to the FTP service identified during reconnaissance.

<img width="1916" height="957" alt="image" src="https://github.com/user-attachments/assets/6fdd169f-1a56-4225-8675-ea74024ff7c3" />

Anonymous login was enabled on the server. A file named **backup.tar.gz** was discovered and downloaded for further inspection.

<img width="809" height="559" alt="image" src="https://github.com/user-attachments/assets/b3967466-6fc0-49d4-a1b7-2006f8840afe" />

---

### Source Code Analysis

After extracting the archive, a directory named **/voltlabs-preview** was found containing three files: README.md, app.py, and requirements.txt.

**README.md**

<img width="656" height="125" alt="image" src="https://github.com/user-attachments/assets/4a956637-63fa-4856-a65b-f7953531ff9b" />

**requirements.txt**

<img width="620" height="81" alt="image" src="https://github.com/user-attachments/assets/00ce4615-8831-40ce-8970-cfa96a1486bd" />

**app.py**

<img width="956" height="657" alt="image" src="https://github.com/user-attachments/assets/f21aefea-ed17-4f2f-a40c-ba7bbfb83e42" />

Reviewing app.py revealed the allowlist configuration:

```python
ALLOWED_HOSTS = {"kestrel.thm"}
```

A comment in the source code confirmed that **kestrel.thm** resolves to 127.0.0.1 via the server's /etc/hosts file. This meant requests to kestrel.thm would originate from localhost on the server side which was exactly what was needed to bypass both the allowlist and the /admin/ 403.

Further inspection of the admin route revealed a hidden endpoint:

```python
if p == "notes":
    with open("/opt/voltlabs-preview/admin_notes.txt") as f:
        return "<pre>" + f.read() + "</pre>"
```

---

### SSRF Confirmed - Accessing /admin/notes

<img width="1122" height="888" alt="image" src="https://github.com/user-attachments/assets/a18e90f3-9781-4d5b-b8cd-0c565327a0f3" />

Passing kestrel.thm through the ?url= parameter successfully bypassed the allowlist, confirming the SSRF vulnerability. Navigating to **/admin/notes** via the same vector returned the web developer's SSH credentials.

<img width="1116" height="885" alt="image" src="https://github.com/user-attachments/assets/f323bbd0-fa3a-4d9a-ade3-d7e8a5527b47" />

**Finding:** SSRF confirmed. The allowlist was bypassable using a hostname that resolved internally to localhost, granting access to the protected /admin route and exposing credentials stored in a plaintext notes file.

---

### Initial Access

Using the recovered credentials, SSH access to the target machine was established. The first flag was located in the user's home directory.

<img width="813" height="564" alt="image" src="https://github.com/user-attachments/assets/5f2348d6-f481-44dd-811b-9fc1d6a62b56" />

---

### Privilege Escalation

**Enumeration**

Local enumeration revealed a writable directory at **/opt/backups**. Further inspection uncovered a cron job at **/etc/cron.d/voltlabs-backup** running as root every minute:
```
# Volt Labs staging backup - runs as root
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

* * * * * root cd /opt/backups && tar czf /var/backups/uploads.tgz *
```
The wildcard `*` combined with write access to /opt/backups made this vulnerable to a tar wildcard injection attack.

**Exploitation**

A malicious script was created to copy bash to /tmp and set the SUID bit on it, allowing it to be executed with root privileges:

```bash
echo 'cp /bin/bash /tmp/rootbash && chmod u+s /tmp/rootbash' > shell.sh
chmod +x shell.sh
```

Tar option filenames were then created to trick tar into executing the script as root:

```bash
touch -- "--checkpoint=1"
touch -- "--checkpoint-action=exec=sh shell.sh"
```

When the cron job fired, tar expanded the wildcard, interpreted the filenames as command-line flags, and executed shell.sh as root. This copied bash to /tmp/rootbash with the SUID bit set.

**Root Flag**

The SUID bash binary was then executed with the -p flag to preserve root privileges:

```bash
/tmp/rootbash -p
```

With a root shell obtained, the final flag was retrieved.

**Finding:** The tar wildcard injection succeeded because the cron job ran as root over a world-writable directory with no input sanitisation. Write access to the backup directory was all that was needed to escalate to full root.

<img width="705" height="150" alt="image" src="https://github.com/user-attachments/assets/356c7579-3848-47cb-b24a-37006cb1ec77" />

---

### Summary

Operation Coldstart illustrates how a chain of individually low-severity misconfigurations can combine into full system compromise. No single vulnerability was catastrophic on its own (anonymous FTP, an internal hostname in a backup archive, plaintext credentials in an admin note, and an unsanitised cron job). Each finding enabled the next.

The SSRF vulnerability could not have been exploited without the source code from the FTP server. The credentials could not have been retrieved without the SSRF. Root access could not have been obtained without the tar wildcard exploit. The attack path was entirely dependent on the backup archive being publicly accessible, making that the most critical finding to remediate.
