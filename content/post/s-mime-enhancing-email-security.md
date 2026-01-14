---
title: "Understanding S/MIME: Enhancing Email Security"
date: 2024-04-06T17:09:18+02:00
draft: false
categories: ["email"]
cover: 
  image: /images/s-mime-enhancing-email-security/s-mime-enhancing-email-security-front.png
---

> _Unlocking the Power of S/MIME (Secure/Multipurpose Internet Mail Extensions): This article will help you understand S/MIME and how to request, configure, and use S/MIME on your devices._

## What is S/MIME?
S/MIME is an email security standard that allows messages to be encrypted and digitally signed. Encryption protects the content of emails, so only the intended recipient can read them, while digital signatures verify the sender’s identity and ensure the message has not been altered. When properly implemented, S/MIME helps keep email communications private and secure from unauthorized access.

> Email encrypted with S/MIME **cannot be read** by big tech companies or governments.

## S/MIME vs. Outbound email authentication
In addition to DMARC-compliant email, which validates the sending source through SPF and DKIM checks, S/MIME provides cryptographic proof of the sender’s identity by digitally signing the message with a personal certificate, whose private key exists only on the sender’s device. In practice, when DMARC-compliant email is used to deliver malicious content, it often indicates that the sender’s mailbox has been compromised, allowing an attacker to send messages that still pass DMARC checks. However, if the compromised mailbox normally signs messages with S/MIME and the malicious message lacks this signature, the recipient may recognize this inconsistency as a red flag and choose to ignore the message.

## How does S/MIME work?
S/MIME provides confidentiality, integrity, authentication, and non-repudiation (often grouped as the CIANA triad) for email messages by leveraging Public Key Infrastructure (PKI) and a combination of asymmetric encryption (public/private key pairs) and symmetric encryption (a single key for both encryption and decryption). It uses digital certificates, electronic credentials issued by trusted Certificate Authorities (CAs), to bind a user’s identity to their public key. These certificates are essential for enabling encryption and digital signatures, helping establish trust between communicating parties.

S/MIME can be used for:

1. Encryption (Confidentiality):
When a user sends an email using S/MIME, their email client encrypts the message using the recipient’s public key. Only the intended recipient, who possesses the corresponding private key, can decrypt and read the message. This ensures that the message content remains confidential, even if intercepted during transmission.

2. Digital Signature (Authentication and Integrity):
S/MIME also enables users to digitally sign emails. The sender’s email client generates a digital signature using the sender’s private key. Upon receiving the message, the recipient’s email client uses the sender’s public key to verify the signature. A valid signature confirms that the message has not been altered (integrity) and authenticates the sender’s identity.

By combining encryption and digital signatures, S/MIME ensures that email communications is secure and resistant to tampering or impersonation.

## Requesting an S/MIME email certificate
Under the hood, the S/MIME email certificate is a code signing certificate. Instead of specifying a domain name in the `Common Name (CN)` section of the `CSR`, you must specify your full name instead. We will request the CSR by using OpenSSL, OpenSSL is a command-line program for creating and managing certificates that is widely used by BSD Unixes* and Linux distributions, but there are also versions available for Windows.

_*BSD Unixes such as Apple MacOS, OpenBSD, FreeBSD use LibreSSL (a fork of OpenSSL) as the primary SSL library as of 2021. LibreSSL also interacts with the `openssl` command. This was chosen by the developers (OpenBSD team) of LibreSSL to make it easy for people, scripts, and systems that previously used OpenSSL to switch to LibreSSL._

> **Note:** In this practical we will use the `/tmp` folder to temporarily store the CSR and private key. Please find a safe place for the private key once you have exported it.

1. Open a terminal application on your computer

2. Execute the following command:
```
openssl req -out /tmp/certificate.csr -newkey rsa:4096 -nodes -sha256 -keyout /tmp/certificate.key
```

3. Fill in the required information for the displayed fields with your own data:
```
Country Name (2 letter code) [AU]:NL
State or Province Name (full name) [Some-State]:Your State
Locality Name (eg, city) []:Your City
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Your Organization 
Organizational Unit Name (eg, section) []:Your Department
Common Name (e.g. server FQDN or YOUR name) []:Your Fullname
Email Address []:your@email.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []: **LEAVE BLANK**
An optional company name []: **LEAVE BLANK**
```
> **Note:** If you are requesting an S/MIME e-mail certificate for your personal e-mail, enter `N/A` in the `Organization Name` and `Organizational Unit Name` fields.

4. The `certificate.csr` file is now created in the `/tmp` directory. You can upload the `CSR` file to a trusted Certificate Authority (CA), such as [Sectigo](https://sectigostore.com/id/email-signing-certificate).

## Configuring and installing the S/MIME email certificate
After the CA signs the certificate, you can download the certificate, for example, in `.crt` format. This `.crt` file is your public key.

1. Place your public key `certificate.crt` in the `/tmp` directory along with the previous private key `certificate.key`.

2. Execute the following command to combine the public key with the private key in a `.pfx` bundle file (`PKCS #12`).
```
openssl pkcs12 -inkey /tmp/certificate.key -in /tmp/certificate.crt -export -out /tmp/your-email-com.pfx
```
**Note:** If you want to import the certificate on BSD-based systems like macOS, you must add the `-legacy` flag at the end of the command when using OpenSSL v3. This version of OpenSSL has changed the default algorithm, making it incompatible with the default LibreSSL library used in BSD-based systems.

3. Your S/MIME email certificate is now ready to be installed on your devices or MUAs, here are some external instructions on how to install the certificate on your device and email client.
- [S/MIME on iOS](https://formsmarts.com/iphone-smime-encrypted-email)
- [S/MIME for Outlook for iOS and Android](https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/outlook-for-ios-and-android/smime-outlook-for-ios-and-android)
- [S/MIME on Mac](https://support.apple.com/guide/mail/use-personal-certificates-mlhlp1179/mac)
- [S/MIME on Outlook for Mac](https://support.microsoft.com/en-us/office/encrypt-email-messages-using-s-mime-in-the-new-outlook-for-mac-623f5b72-4a8c-4293-a8a2-1f2ea223fde5)
- [S/MIME for Windows](https://learn.microsoft.com/en-us/windows/security/operating-system-security/data-protection/configure-s-mime)
- [S/MIME in Outlook on the web](https://support.microsoft.com/en-us/office/encrypt-messages-by-using-s-mime-in-outlook-on-the-web-878c79fc-7088-4b39-966f-14512658f480)
- [S/MIME in Outlook for Windows](https://formsmarts.com/outlook-smime-encrypted-email)
- [Configure S/MIME in Exchange Online](https://learn.microsoft.com/en-us/exchange/security-and-compliance/smime-exo/configure-smime-exo)
- [Using S/MIME in Thunderbird](https://support.mozilla.org/en-US/kb/instructions-smime-certificate-using-csr#w_import-the-certificate-into-certificate-manager-and-back-it-up)

To send signed messages, you need to have your personal S/MIME certificate installed in your keychain. To send encrypted messages, the recipient’s certificate must also be stored in your keychain, this applies, for example, on macOS. Once you’ve imported your S/MIME email certificate into your device, you can configure your email client to digitally sign your outgoing messages and decrypt incoming messages that were encrypted with your public key.

![IMAGE](/images/s-mime-enhancing-email-security/s-mime-enhancing-email-security1.png)

After you send the email to your recipient, the recipient's email client verifies the signature using your public key.

![IMAGE](/images/s-mime-enhancing-email-security/s-mime-enhancing-email-security2.png)

## Sharing your S/MIME public key
Once you send a signed email using your S/MIME certificate, the recipient can save your public key and then send encrypted emails to you. However, unlike PGP with WKD (Web Key Directory), S/MIME does not provide a standardized discovery mechanism for public keys.

There are, however, alternative approaches using HTTPS, DNS TXT records, and optionally SMIMEA records. Please note that SMIMEA records are used only for verification and require DNSSEC.

> **IMPORTANT**: Based on my research, there is currently no standardized method for sharing S/MIME public keys, and mail clients do not automatically discover them. This means out-of-band instructions are required.

### HTTPS hosting
You can publish your (or your users) S/MIME public key certificates on your domain, for example: `https://example.com/.well-known/smime/firstname_lastname.crt`

### Publish a DNS TXT record
You can publish the `HTTPS` URL of the certificate in a DNS `TXT` record so it can be queried by others:
  - **Host**: `firstname.lastname._smime.example.com`
  - **Value**: `https://example.com/.well-known/smime/firstname_lastname.crt`
  - **Type**: `TXT`

### Optional: Publish a DNS SMIMEA record for integrity (DNSSEC required)
An SMIMEA record allows email senders to validate a recipient’s S/MIME certificate. These records are designed to be used with DNSSEC so the information can be trusted. SMIMEA is defined in [RFC 8162](https://www.rfc-editor.org/info/rfc8162) and currently has *experimental* status.

#### Implementing a DNS SMIMEA record
1. Hash the local part of the email address: `echo -n "firstname.lastname" | openssl dgst -sha256`
2. Extract the public key from the (downloaded) certificate and hash it: `openssl x509 -in firstname_lastname.crt -noout -pubkey | openssl pkey -pubin -outform DER | openssl dgst -sha256`
3. Create the SMIMEA record:
   - **Host**: `hash._smimecert.example.com`
   - **Value**: `3 1 1 hash` (See my [earlier post](https://vand3rlinden.com/post/dnssec-dane-explained/#implementation-of-dane) on DANE for an explanation of usage, selector, and matching types)
   - **Type**: `SMIMEA`

> **IMPORTANT**: Unfortunately, most DNS providers do not yet support SMIMEA records.

### How to use this discovery method
If you want to send an encrypted email to `firstname.lastname@example.com` and check whether an S/MIME certificate is available, follow these steps:

1. Query the `._smime` `TXT` record: `dig firstname.lastname._smime.example.com TXT +short`
2. Download the `.crt` file from the `HTTPS` URL provided in the `TXT` record
3. **Optional**: Validate the S/MIME certificate by hashing the local part of the email address and the public key for integrity: `dig hash._smimecert.example.com SMIMEA +short`

While this approach is not standardized, I am researching a way to improve the S/MIME public key discovery, similar to what PGP offers with WKD. This because sending encrypted email is more important than ever for your digital sovereignty, especially when you want to protect your communications from prying eyes, whether they belong to big tech companies or governments. With S/MIME, you remain the only holder of your private key and the only one able to decrypt your messages.

## To Summarize
S/MIME plays a critical role in enhancing email security by providing encryption, digital signatures, and authentication mechanisms. As cyber threats continue to evolve, implementing robust security measures such as S/MIME is essential to protecting sensitive information and maintaining trust in digital communications.