---
title: 'Using PGP for secure email communication'
date: 2025-06-06T22:01:22+02:00
draft: false
categories: ["email"]
cover: 
  image: /images/pgp-secure-email-communication/pgp-secure-email-communication-front.png
---

> _In this post, I will explain what PGP is, how it differs from S/MIME, and why encrypting and signing your emails is more important than ever._

## What is PGP?
Email remains one of the least secure forms of communication, unless you take steps to protect it. Pretty Good Privacy (PGP) offers a powerful and accessible way to encrypt and sign your email messages, ensuring that your communication remains private and tamper-proof.

PGP is an encryption program that provides cryptographic privacy and authentication. Originally created by Phil Zimmermann in 1991, PGP is now widely supported and standardized through OpenPGP.

PGP uses a combination of symmetric encryption (a single key for encrypting and decrypting message content) and asymmetric encryption (public/private key pairs). The message is encrypted using symmetric encryption for efficiency, while asymmetric encryption is used to securely exchange the symmetric key, decrypt it on the recipient’s side, and verify digital signatures.

Each user has a key pair:
- A **public key**, which is shared with others and used to encrypt messages to you.
- A **private key**, which is kept secret and used to decrypt messages you receive and to sign messages you send.

When you send a message encrypted with someone’s public key, only their private key can decrypt it. Likewise, when you sign a message with your private key, anyone with your public key can verify that the message came from you and hasn’t been altered.

## PGP vs. S/MIME: What’s the difference?
In an [earlier blog](https://vand3rlinden.com/post/s-mime-enhancing-email-security/), I explained how S/MIME works, what it does, and how to request and configure it. Both PGP and S/MIME offer email encryption and digital signing, but they differ in architecture and how they are used.

- **S/MIME**: Uses X.509 certificates issued by a trusted Certificate Authority (CA) within a Public Key Infrastructure (PKI) model. It is centrally managed and commonly integrated into enterprise email platforms like Outlook. S/MIME enables encryption and digital signing of emails using public key cryptography.
- **PGP**: Uses a web-of-trust model, where users manually validate each other’s keys using ASCII-armored text files (`.asc`). It is a decentralized system, meaning no central authority is required. While PGP provides greater flexibility, it also involves manual key management, which can be more complex for users.

In general, S/MIME is easier to deploy since it is built into many enterprise email platforms and allows certificate management through a central authority. PGP, on the other hand, is more popular among privacy-conscious individuals because of its decentralized nature and the transparency it offers.

PGP provides confidentiality, integrity, authentication, and non-repudiation (often grouped as the CIANA triad) for email communication, in the same way as S/MIME. However, non-repudiation in PGP is more easily challenged, especially if a private key is compromised or weakly tied to a verifiable identity. Since PGP is decentralized, there is often no authority backing to prove that a specific key belongs to a particular person. For example, if someone claims, _“That wasn’t me, someone else used my key”_, it can be difficult to disprove. In contrast, S/MIME relies on Certificate Authorities (CAs) that may perform identity verification (e.g., checking a passport or company ownership), providing stronger legal assurance in such cases. This is where S/MIME’s centralized approach can offer an advantage.

## Why encrypt and sign emails with PGP?
1. **Keep your email private**: Most email is sent in plain text, your actual messages are stored unencrypted on the email provider’s servers. That means providers, or anyone who gains access to their systems, can read your email content. While email is often transmitted over encrypted connections using TLS, this only protects the message in transit, not at rest. Once the message reaches the recipient’s mail server, it is typically stored unencrypted and accessible to that provider. Some providers may advertise end-to-end encryption within their own ecosystem, but the encryption keys are usually managed by the provider, not by you. This means the provider, or anyone who gains access to their systems, can potentially read or decrypt your emails. PGP encryption ensures that only the intended recipient can decrypt your message, even if the email provider is compromised or subject to government surveillance or backdoors.

2. **Prevent tampering**: By signing your emails with your private key, recipients can verify that the message came from you and hasn’t been altered in transit. This protects against man-in-the-middle attacks or spoofing.

3. **Protect against government backdoors**: In many countries, governments have pressured or compelled tech companies to include backdoors in their services, hidden ways for authorities to access user data. Even if you’re not doing anything wrong, the existence of these backdoors means your private communications could be accessed without your knowledge. PGP encryption is end-to-end and user-controlled, meaning even your email provider can’t decrypt your messages, and neither can a government agency, unless they gain access to your private key.

> _Arguing that you don't care about the right to privacy because you have nothing to hide is no different from saying you don't care about free speech because you have nothing to say._ – **Edward Snowden**

4. **Establish digital identity**: Your public PGP key serves as a unique digital signature tied to your identity. As more people verify and associate your key with you, it becomes increasingly difficult for impersonators to send fake emails in your name. As I explained in my [earlier blog](https://vand3rlinden.com/post/s-mime-enhancing-email-security/#smime-vs-outbound-email-authentication) on S/MIME, particularly in the context of outbound email authentication, consistently signing your messages helps establish trust. If a message suddenly lacks your usual signature, the recipient may see it as a red flag and choose to ignore it, even if it passes outbound authentication for the sending domain.

## Setting up secure email communication with OpenPGP
OpenPGP is the most widely used standard for email encryption. There are [many tools](https://www.openpgp.org/software/) and user-friendly applications that support it, such as [GPG Suite](https://gpgtools.org/) and various email plugins. In this blog, we’ll use the command-line tool [GnuPG (GNU Privacy Guard)](https://www.gnupg.org/), commonly known as GPG, which is the most widely used utility for generating and managing PGP keys.

**Let's begin**:

### 1. Install GnuPG (GNU Privacy Guard) using your package manager
- **Linux (Debian based):** `sudo apt-get install gnupg`
- **MacOS (via Homebrew):** `brew install gnupg`
- **Windows:** Download GPG4Win from [https://gpg4win.org](https://gpg4win.org)

### 2. Generate the key pair
Run the following command:
```
gpg --full-generate-key
```

1. Choose **RSA and RSA** for encryption
2. Select the desired key size (e.g., 4096 bits for stronger security)
3. Set an expiration date (it is not recommended to select **no expiration** for permanent use)
4. Enter your name and email address when prompted
5. Set a strong passphrase to secure the private key

Once complete, your keys will be stored in the GPG keyring. You can view your generated key pair, including the corresponding `Fingerprint`, by running the following commands to list the public and private keys.

- Show Public keys:
```
gpg --list-keys --keyid-format LONG
```

- Show Private keys:
```
gpg --list-secret-keys --keyid-format LONG
```

### 3. Export your public key
Others will need your public key to verify your digital signature and/or to send you encrypted messages. You can share your public key via your website or other trusted channels.

- Export Public key:
```
gpg --armor --export your_email@example.com > pgp-publickey.asc
```

### 4. Encrypt a message
1. Save the recipient’s public key: `nano pgp-publickey.asc` (paste the entire public key block and save the file with Ctrl+O, Enter, Ctrl+X)
2. Import the public key: `gpg --import pgp-publickey.asc`
3. Verify the `Fingerprint` from trusted sources like the recipient’s website or email signature: `gpg --fingerprint recipient@domain.com`
4. Create your message: `nano message.txt`
5. Encrypt the message using the recipient’s public key: `gpg --encrypt --armor --recipient recipient@domain.com message.txt`, this will output an ASCII-armored encrypted file, e.g., `message.txt.asc`
6. The output will look like this:
```
-----BEGIN PGP MESSAGE-----
hQIMAxOeBV
-----END PGP MESSAGE-----
```

### 5. Sign the message
1. Digitally sign the encrypted message using your private key: `gpg --clearsign --local-user Fingerprint message.txt.asc` (Replace `Fingerprint` with your actual key fingerprint)
2. Enter your private key’s passphrase. This will generate a new ASCII-armored signed file, e.g., `message.txt.asc.asc`
3. The output will resemble:
```
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA256

- -----BEGIN PGP MESSAGE-----
hQIMAxOeBV
- -----END PGP MESSAGE-----
-----BEGIN PGP SIGNATURE-----

iQIzBAEBCAAdFiEEocmn9wnTqI
-----END PGP SIGNATURE-----
```
4. You can now copy and paste this signed and encrypted message into your email client, and ready to send for secure email communication

### 6. The recipient verifies your digital signature
1. The recipient has already imported your public key
2. They paste the received text into a file named `signedmessage.asc`
3. To verify the signature: `gpg --verify --local-user YourFingerprint signedmessage.asc`.
4. A successful verification will look like:
```
gpg: Good signature from "Your Name <your@email.com>"
```

### 7. The recipient decrypts the message
1. The recipient must have their private key on the local system
2. They paste the PGP-encrypted content into a file called `encryptedmessage.asc`
3. To decrypt the message: `gpg --decrypt --local-user RecipientFingerprint encryptedmessage.asc`
4. A successful decryption will look like:
```
gpg: encrypted with rsa4096 key, ID ABCD1234....

This is the decrypted message content.
```

> **NOTE**: While it is a good practice to become familiar with `gpg` commands, I have developed a bash script that streamlines encryption, decryption, signing, and signature verification. It works seamlessly as long as **GnuPG** is installed on your system, available here on my [GitHub repository](https://github.com/vand3rlinden/Bash/blob/main/pgp-buddy/pgp_tool.sh). I have also created a separate bash script for managing PGP keys, including key generation, import, and export. You can download it from the [same repository here](https://github.com/vand3rlinden/Bash/blob/main/pgp-buddy/pgp_key_tool.sh).

## Summerize
PGP remains one of the most effective tools for securing email communication. By using strong encryption and digital signatures, it helps protect your messages from surveillance, tampering, and impersonation, even across untrusted networks or email providers. While it may not rely on centralized authorities like S/MIME, PGP empowers individuals with control over their own security and privacy through a decentralized trust model.

Whether you’re a privacy advocate, a journalist, or just someone who values confidential communication, learning to use PGP is a practical step toward regaining control of your digital conversations.

> _In a world where email remains a vulnerable medium, PGP helps ensure your messages stay truly private and authentically yours._ - **Ricardo van der Linden**