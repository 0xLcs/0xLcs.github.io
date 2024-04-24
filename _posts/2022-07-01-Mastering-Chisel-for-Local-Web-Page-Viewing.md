---
title: "Using Portainer for Privilege Escalation in Docker Containers"
date: 2022-07-01 14:06:24
categories: [portforwarding, tunnel]
tags: [chisel, portforwarding, pentest, OSCP]
---

# Mastering Chisel for Local Web Page Viewing: A Comprehensive Guide

Chisel is a powerful tool that allows developers to safely expose local resources to the internet in a controlled manner. It operates as a reverse tunnel, facilitating the interaction with applications running in local development environments without the need to expose them directly to the web. This detailed guide will show you how to use Chisel to view web pages locally, with a deep dive into its advanced commands.

## What is Chisel?

Chisel is a reverse HTTP tunnel, written in Go, that facilitates secure network traffic forwarding. It is perfect for developers who need to access services running on `localhost` from external devices or networks.

## Setting Up Chisel

Before exploring the commands, you need to install and configure Chisel. Follow these steps:

1. **Installation:**
   - Download Chisel from the [GitHub repository](https://github.com/jpillora/chisel).
   - Compile or download the pre-compiled binary appropriate for your operating system.

2. **Running the Chisel Server:**
   - On the server that will receive the connections, run:
    
      chisel server --port 8080 --reverse
     

3. **Connecting the Chisel Client:**
   - On your local machine, connect to the server using:
     
     chisel client http://server-IP:8080 R:8080:localhost:80
     

## Advanced Commands and Their Functions

### 1. Specific Port Forwarding

**Command:**

   chisel client http://server-IP:8080 R:remote-port:localhost:local-port

**Description:**
This command creates a tunnel between a specific server port and a port on your local machine. It's useful for exposing local web applications to a broader network.

**Example:**
Suppose you have an application running on port 3000 in your local environment and want to access it externally on port 8080 of the server:

   chisel client http://server-IP:8080 R:8080:localhost:3000


### 2. Multiple Ports Forwarding

**Command:**

   chisel client http://server-IP:8080 R:8080:localhost:3000 R:5000:localhost:5000

**Description:**
Allows the forwarding of multiple ports simultaneously, facilitating the exposure of multiple local services.

### 3. Using SOCKS5 Proxy

**Command:**

   chisel client http://server-IP:8080 --socks5

**Description:**
Activates a SOCKS5 proxy server on the client side, allowing you to route your internet traffic through the Chisel server. This is useful for testing the appearance of a website from different geographic locations.

**Example:**

   chisel client http://server-IP:8080 --socks5

After execution, configure your browser to use the SOCKS proxy at `localhost:1080`.

**To configure proxychains to use this SOCKS5 proxy, add the following line to your `proxychains.conf`:**

   socks5  127.0.0.1 1080


## Conclusion

Chisel is a versatile tool that offers robust solutions to common problems faced by developers while working with local applications. The ability to securely forward ports and configure a SOCKS5 proxy are just the tip of the iceberg. Try integrating Chisel into your development workflow to see how it can simplify your access to local resources from anywhere in the world.
