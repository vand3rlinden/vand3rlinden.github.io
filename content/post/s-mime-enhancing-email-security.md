---
title: "Understanding S/MIME: Enhancing Email Security"
date: 2024-04-06T17:09:18+02:00
draft: false
categories: ["email"]
cover: 
  image: /images/s-mime-enhancing-email-security/s-mime-enhancing-email-security-front.png
---

> _Unlocking the Power of S/MIME: This article will help you understand S/MIME and how to request, configure, and use S/MIME on your devices._

## What is S/MIME?
In today's digital landscape, the security of sensitive information transmitted via email is critical. One method of safeguarding email communications is through the use of S/MIME _(Secure/Multipurpose Internet Mail Extensions)_. S/MIME is a widely adopted protocol that provides end-to-end encryption and digital signatures, ensuring the confidentiality, integrity, and authenticity of email messages. 

In addition to DMARC-compliant email, S/MIME provides proof that the sender is the sender by signing the message with a personal certificate that exists only on the sender's devices _(the private key of the certificate)_. In practice, when we see DMARC-compliant email being sent with malicious content, these mailboxes are likely compromised and therefore able to send DMARC-compliant email from the sender. If the compromised mailbox normally signs the messages with S/MIME, the recipient may see this as a red flag and ignore the message.

## How does S/MIME work?
At its core, S/MIME is a security standard that enables users to send and receive secure e-mail using cryptographic techniques and provides non-repudiation for e-mail communications. These techniques include the use of Public Key Infrastructure (PKI), which uses asymmetric encryption algorithms.

1. **Digital Certificate (Confidentiality):** When a user sends an email using S/MIME, their email client **encrypts the message** using the recipient's public key. This means that only the intended recipient, who has the corresponding private key, can decrypt and read the message. This process ensures that even if the email is intercepted during transmission, its contents remain inaccessible to unauthorized parties.

2. **Digital Signature (Integrity):** S/MIME also allows users to digitally sign their e-mail. This involves using the sender's private key to create a **unique digital signature** for each message. When the email is received, the recipient's email client verifies the signature against the sender's public key. If the signature is valid, it provides assurance that the email has not been tampered with in transit and is indeed from the sender.

## Requesting an S/MIME email certificate
Under the hood, the S/MIME email certificate is a code signing certificate. Instead of specifying a domain name in the `Common Name (CN)` section of the `CSR`, you must specify your full name instead. We will request the CSR by using OpenSSL, OpenSSL is a command-line program for creating and managing certificates that is widely used by BSD Unixes* and Linux distributions, but there are also versions available for Windows.

_*BSD Unixes such as Apple MacOS, OpenBSD, FreeBSD use LibreSSL (a fork of OpenSSL) as the primary SSL library as of 2021. LibreSSL also interacts with the `openssl` program name. This was chosen by the developers (OpenBSD team) of LibreSSL to make it easy for people, scripts, and systems that previously used OpenSSL to switch to LibreSSL._

**Note:** In this practical we will use the `/tmp` folder to temporarily store the CSR and private key. Please find a safe place for the private key once you have exported it.

1. Open a terminal application on your computer

2. Execute the following command:
```
openssl req -nodes -newkey rsa:4096 -keyout /tmp/certificate.key -out /tmp/certificate.csr
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
**Note:** If you are requesting an S/MIME e-mail certificate for your personal e-mail, enter `N/A` in the `Organization Name` and `Organizational Unit Name` fields.

4. The `certificate.csr` file is now created in the `/tmp` directory. You can upload the `CSR` file to a trusted Certificate Authority (CA), such as [Sectigo](https://sectigostore.com/id/email-signing-certificate).

## Configuring and installing the S/MIME email certificate
After the CA signs the certificate, you can download the certificate, for example, in `.crt` format. This `.crt` file is your public key.

1. Place your public key `certificate.crt` in the `/tmp` directory along with the previous private key `certificate.key`.

2. Execute the following command to combine the public key with the private key in a `.pfx` bundle file (`PKCS #12`).
```
openssl pkcs12 -inkey /tmp/certificate.key -in /tmp/certificate.crt -export -out /tmp/your-email-com.pfx
```
**Note:** If you want to import the certificate in BSD Unixes such as MacOS, you must add the `-legacy` flag at the end of the above command, if your are using OpenSSL `V3`. This OpenSSL version changed the default algorithm and it's not compatible with the default LibreSSL library.

3. Your S/MIME email certificate is now ready to be installed on your devices, here are some external instructions on how to install the certificate on your device and email client.
- [S/MIME on iOS](https://formsmarts.com/iphone-smime-encrypted-email)
- [S/MIME for Outlook for iOS and Android](https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/outlook-for-ios-and-android/smime-outlook-for-ios-and-android)
- [S/MIME on Mac](https://support.apple.com/guide/mail/use-personal-certificates-mlhlp1179/mac)
- [S/MIME on Outlook for Mac](https://support.microsoft.com/en-us/office/encrypt-email-messages-using-s-mime-in-the-new-outlook-for-mac-623f5b72-4a8c-4293-a8a2-1f2ea223fde5)
- [S/MIME for Windows](https://learn.microsoft.com/en-us/windows/security/operating-system-security/data-protection/configure-s-mime)
- [S/MIME in Outlook on the web](https://support.microsoft.com/en-us/office/encrypt-messages-by-using-s-mime-in-outlook-on-the-web-878c79fc-7088-4b39-966f-14512658f480)
- [S/MIME in Outlook for Windows](https://formsmarts.com/outlook-smime-encrypted-email)
- [Configure S/MIME in Exchange Online](https://learn.microsoft.com/en-us/exchange/security-and-compliance/smime-exo/configure-smime-exo)

Once you have imported the S/MIME e-mail certificate into your device, you can configure the e-mail client to encrypt and/or sign your messages.

![IMAGE](/images/s-mime-enhancing-email-security/s-mime-enhancing-email-security1.png)

After you send the email to your recipient, the recipient's email client verifies the signature using your public key.

![IMAGE](/images/s-mime-enhancing-email-security/s-mime-enhancing-email-security2.png)

## To Summarize
S/MIME plays a critical role in enhancing email security by providing encryption, digital signatures, and authentication mechanisms. As cyber threats continue to evolve, implementing robust security measures such as S/MIME is essential to protecting sensitive information and maintaining trust in digital communications.