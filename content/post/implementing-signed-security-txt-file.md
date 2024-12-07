---
title: "Implementing a signed security.txt file with PGP for your website"
date: 2024-11-25T23:35:37+01:00
draft: false
categories: ["other"]
cover: 
  image: /images/implementing-signed-security-txt-file/implementing-signed-security-txt-file-front.png
---

> _A `security.txt` file is an industry standard used by organizations to provide a clear point of contact for vulnerability reporting. Adding a signed `security.txt` file to your website enhances credibility and ensures trust, as the signature allows others to verify its authenticity. This blog will walk you through implementing a signed `security.txt` file using PGP (Pretty Good Privacy)._

## **Introduction**
The `security.txt` file is typically hosted at `/.well-known/security.txt` on a website. It follows a standardized format to share security contact information, preferred encryption keys, and disclosure policies. However, without proper authentication, anyone could tamper with the file. A signed `security.txt` solves this issue by letting others verify the file’s legitimacy using your PGP key. Here’s how to create, sign, and verify a `security.txt` file for your website.

## **Step 1: Generate a PGP Key Pair**
If you don’t already have a PGP key pair, you’ll need to generate one. A PGP key pair consists of a public key, which you can share publicly, and a private key, which is kept secret.

### **1. Install GPG (GNU Privacy Guard)**
GPG is the most common tool for generating and managing PGP keys. Install it using your package manager:
- **Linux:** `sudo apt-get install gnupg`
- **MacOS (via Homebrew):** `brew install gnupg`
- **Windows:** Download GPG4Win from [https://gpg4win.org](https://gpg4win.org).

### **2. Generate the Key Pair**
Run the following command:
```
gpg --full-generate-key
```

1. Choose **RSA and RSA** for encryption.
2. Select the desired key size (e.g., 4096 bits for stronger security).
3. Set an expiration date (or choose "no expiration" for permanent use).
4. Enter your name and email address when prompted.
5. Set a strong passphrase to secure the private key.

Once complete, your keys are stored in the GPG keyring. You can view your generated key pair by running the following command for both the public and private keys.

- Public key:
```
gpg --list-keys
```

- Private key:
```
gpg --list-secret-keys
```

### **3. Export Your Public Key**
Others will need your public key to verify your signature:
```
gpg --armor --export your_email@example.com > pgp-publickey.txt
```

This will export the public key to a file named `pgp-publickey.txt`. 

## **Step 2: Create and Sign the security.txt File**
1. **Create the security.txt File**
   Create a plain text file named `security.txt` with the following content (customize as needed):
   ```
   # Security address
   Contact: mailto:security@example.com

   # OpenPGP key
   Encryption: https://example.com/.well-know/pgp-publickey.txt

   # Expiration of security.txt file
   Expires: DATE

   # Privacy Policy
   Policy: https://example.com/security-policy.html

   # Languages
   Preferred-Languages: EN, NL
   ```

2. **Sign the File with your Private Key**
   Use the following command to create a cleartext signature:
   ```
   gpg --clearsign security.txt
   ```

   After you enter the passphrase to unlock the OpenPGP secret key, a signature file `security.txt.asc` is created. Copy the contents of `security.txt.asc` into the original `security.txt` file.

3. **Host the files**
   In the `/.well-known/` folder on your website, upload the `security.txt` and `pgp-publickey.txt` files:
   ```
   https://example.com/.well-known/security.txt
   https://example.com/.well-known/pgp-publickey.txt
   ```

## **Step 3: Verifying the Signature**
To confirm the authenticity of the `security.txt` file, anyone can verify it using your public key.

### **1. Import the Public Key**
A user downloads your public key and imports it into their GPG keyring:
```
gpg --import pgp-publickey.txt
```

### **2. Verify the Signature**
The user runs the following command to verify the signature:
```
gpg --verify security.txt
```

If the file is authentic, they’ll see output confirming the signature, such as:
```
gpg: Signature made Fri 23 Nov 2024 10:30:00 AM UTC
gpg:                using RSA key ABC123DEF4567890
gpg: Good signature from "Your Name <your_email@example.com>"
```

The user can trust your public key by running: `gpg --edit-key Key_ID`, typing `trust` at the next prompt, and selecting the option `5 = I trust ultimately`.

## **Conclusion**
Adding a signed `security.txt` file to your website is a simple yet powerful way to build trust with security researchers and users. It ensures that the contact information provided is legitimate and hasn’t been tampered with. By following these steps, you create a clear and verifiable point of contact for handling vulnerabilities responsibly.

## Reference
- [securitytxt.org](https://securitytxt.org/)
- [security.txt is defined in RFC 9116](https://www.rfc-editor.org/rfc/rfc9116#section-2.6)