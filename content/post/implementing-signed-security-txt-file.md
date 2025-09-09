---
title: "Implementing a signed security.txt file with PGP on your webserver"
date: 2024-11-25T23:35:37+01:00
draft: false
categories: ["other"]
cover: 
  image: /images/implementing-signed-security-txt-file/implementing-signed-security-txt-file-front.png
---

> _A `security.txt` file is an industry standard used by organizations to provide a clear point of contact for vulnerability reporting. This blog will walk you through implementing a signed `security.txt` file using PGP (Pretty Good Privacy)._

## Introduction
The `security.txt` file, typically hosted at `/.well-known/security.txt`, provides a standardized way for security researchers to find the correct point of contact for reporting vulnerabilities. It helps ensure that security issues are directed to the right people quickly and efficiently. While it’s recommended to optionally sign the file with a PGP key for added authenticity, the primary goal is to clearly list how to reach your security team or responsible contact. Below, we’ll cover how to create a straightforward, effective `security.txt` file for your site.

## Step 1: Generate a PGP key pair
If you don’t already have a PGP key pair, you’ll need to generate one. A PGP key pair consists of a public key, which you can share publicly, and a private key, which is kept secret.

### 1. Install GPG (GNU Privacy Guard)
GPG is the most common tool for generating and managing PGP keys. Install it using your package manager:
- **Linux (Debian based):** `sudo apt-get install gnupg`
- **MacOS (via Homebrew):** `brew install gnupg`
- **Windows:** `winget install -e --id GnuPG.Gpg4win` or direct download GPG4Win from [https://gpg4win.org](https://gpg4win.org).

### 2. Generate the key pair
Run the following command:
```
gpg --full-generate-key
```

1. Choose **RSA and RSA** for encryption.
2. Select the desired key size (e.g., 4096 bits for stronger security).
3. Set an expiration date (it is not recommended to select **no expiration** for permanent use).
4. Enter your name and email address when prompted.
5. Set a strong passphrase to secure the private key.

Once complete, your keys will be stored in the GPG keyring. You can view your generated key pair, including the corresponding `Fingerprint`, by running the following commands to list the public and private keys:

- Public key:
```
gpg --list-keys
```

- Private key:
```
gpg --list-secret-keys
```

### 3. Export your public key
Others will need your public key to verify your signature:
```
gpg --armor --export your_email@example.com > pgp-publickey.txt
```

This will export the public key to a file named `pgp-publickey.txt`. 

## Step 2: Create and sign the security.txt file
1. **Create the security.txt file**: Create a plain text file named `security.txt` with the following content (customize as needed):
   ```
   # Security address
   Contact: mailto:security@example.com

   # OpenPGP key
   Encryption: https://example.com/.well-known/pgp-publickey.txt

   # Expiration of security.txt file
   Expires: expiration date of your pgp key pair

   # Privacy Policy
   Policy: https://example.com/security-policy.html

   # Languages
   Preferred-Languages: EN, NL
   ```

2. **Sign the file with your Private Key**: Use the following command to create a cleartext signature (replace the `Fingerprint` with the actual fingerprint):
   ```
   gpg --clearsign --local-user Fingerprint security.txt
   ```

   After you enter the passphrase to unlock the OpenPGP Private Key, a signature file `security.txt.asc` is created. Copy the content of `security.txt.asc` into the original `security.txt` file.

3. **Host the files**: In the `/.well-known/` folder on your webserver, upload the `security.txt` and `pgp-publickey.txt` files:
   ```
   https://example.com/.well-known/security.txt
   https://example.com/.well-known/pgp-publickey.txt
   ```

## Step 3: Confirm authenticity and secure communication
To confirm the authenticity of the `security.txt` file, anyone can verify it using your public key.

### 1. Import the Public Key
A user downloads your public key and imports it into their GPG keyring:
```
gpg --import pgp-publickey.txt
```

To trust a public key using GPG, the user can follow these steps:
1. Always verify the `Fingerprint` from trusted sources, such as the key owner’s website or email signature, before trusting their public key: `gpg --fingerprint your_email@example.com`
2.	Open a terminal and run: `gpg --edit-key Fingerprint` (replace the `Fingerprint` with the actual fingerprint)
3.	At the gpg> prompt, type: `trust`
4.	When prompted to choose a trust level, enter: `5` (which stands for “I trust ultimately”), then press Enter.
5.	To exit the editor: On Windows, press Ctrl + C, On Mac, press Command + C (or type `quit` and confirm if prompted)

### 2. Verify the Signature
The user runs the following command to verify the signature:
```
gpg --verify security.txt
```

> The PGP-signed `security.txt` file includes metadata that identifies your fingerprint; GPG then scans the user’s keyring to find your corresponding public key to verify the signature.

If the file is authentic, they’ll see output confirming the signature, such as:
```
gpg: Signature made Fri 23 Nov 2024 10:30:00 AM UTC
gpg:                using RSA key ABC123DEF4567890
gpg: Good signature from "Your Name <your_email@example.com>"
```

### 3. Secure communication
For secure communication, users can use your public key to send an encrypted message to the email address listed in your `security.txt` file, which is cryptographically bound to your private key, an encrypted PGP message will look like:
```
-----BEGIN PGP MESSAGE-----
...
-----END PGP MESSAGE-----
```

To decrypt this message using your private key:
1. Save the encrypted message: `nano message.asc` (paste the entire PGP message and save the file, Ctrl+O, Enter, Ctrl+X)
2. Decrypt the message: `gpg --decrypt message.asc` (the PGP-encrypted message includes metadata showing your fingerprint, which it was encrypted to; GPG then scans your keyring to find the matching private key needed to decrypt it)
3. A successful decryption will look like:
```
gpg: encrypted with rsa4096 key, ID ABCD1234....

This is the decrypted message content.
```

> **NOTE**: While it is a good practice to become familiar with `gpg` commands, I have developed a bash script that streamlines encryption, decryption, signing, and signature verification. It works seamlessly as long as **GnuPG** is installed on your system, available here on my [GitHub repository](https://github.com/vand3rlinden/Bash/blob/main/pgp-buddy/pgp_tool.sh). I have also created a separate bash script for managing PGP keys, including key generation, import, and export. You can download it from the [same repository here](https://github.com/vand3rlinden/Bash/blob/main/pgp-buddy/pgp_key_tool.sh).

## Conclusion
Adding a signed `security.txt` file to your webserver is a simple yet powerful way to build trust with security researchers and users. It ensures that the contact information provided is legitimate and hasn’t been tampered with. By following these steps, you create a clear and verifiable point of contact for handling vulnerabilities responsibly.

## Reference
- [securitytxt.org](https://securitytxt.org/)
- [security.txt is defined in RFC 9116](https://www.rfc-editor.org/rfc/rfc9116)
- [Unsigned vs. Signed security.txt file](https://www.rfc-editor.org/rfc/rfc9116#section-2.6)
- [Digital Trust Center (DTC) - about security.txt - Dutch government](https://www.digitaltrustcenter.nl/securitytxt)
- [Digital Trust Center (DTC) - security.txt is mandatory for the Dutch government](https://www.digitaltrustcenter.nl/nieuws/securitytxt-verplicht-voor-overheid)