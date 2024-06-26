---
title: "Using Portainer for Privilege Escalation in Docker Containers"
date: 2022-06-15 12:01:07
categories: [PrivilegeEscalation]
tags: [Dockers, Containers, PrivilegeEscalation, Portainer]
---

## Introduction

In this article, we explore how to perform privilege escalation using containers with the help of Portainer, a GUI-based container management tool that simplifies managing and deploying cloud-native applications.

## What is Portainer?

Portainer is a graphical interface that eases container administration, allowing users to manage their applications and container infrastructure efficiently and securely. It facilitates everything from creating and managing images to monitoring the health of running containers.

## Step-by-Step Privilege Escalation Guide

### 1. Accessing Portainer:
First, access the Portainer portal and verify the available images for use.

### 2. Creating the Volume and Container:
Create a Volume with these configurations (Add Drive Option:) below:  
- **Additional Volume Options:**
  - **device:** `/`
  - **o:** `bind`
  - **type:** `none`

 This setup allows the container to directly access a path in the host's file system. Here's a breakdown of each part:

- **device:** This is the path on the host's file system. `/` refers to the root directory of the host.
- **o: bind:** This indicates the mounting method, which in this case is "bind". It means that the directory specified in `device` will be mounted directly into the container.
- **type: none:** This specifies that it is not a block device being mounted, but rather a directory or file.

  These settings are used to ensure that the container has access to specific files or directories on the host, which can be useful for various purposes such as data persistence, managing configuration files, or logs, among others.
  
  After that create a New Container, select and copy the ID of the any image, create a container based on this image. This is done in the container section of Portainer by clicking "Add Container".

### 3. Configuring the Container:
While configuring the container:
- Name the container and insert the copied image ID (ensure this ID is a previously created SHA hash).
- Choose the console to be interactive and TTY (-i -t).
- Important: configure the volume mapping to mount the host's /root directory to /mnt/root in the container

### 4. Setting Privileges:
Ensure to set the container to operate in "Privileged Mode" in the Security/Host section. This allows the container extended access to the host system.

### 5. Starting the Container:
After setting up, click "Create" to launch the container. Wait for the confirmation that the container has been successfully created.

### 6. Accessing the Container:
Once the container is operational, access the console by clicking on the container's name and then on the >_Console menu. Select /bin/bash and click "Connect".

### 7. Exploring the Host System:
In the container's console, navigate to /mnt/root to access the host's filesystem. Here, you can explore various directories and files, including the host's /etc/hosts.

## Conclusion

Using tools like Portainer for managing containers provides powerful administrative capabilities but also requires caution to prevent abuses, such as the demonstrated privilege escalation. It is crucial to implement safe container management practices to protect cloud environments.
