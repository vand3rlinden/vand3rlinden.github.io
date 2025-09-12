---
date: 2025-06-13
title: "Secure Communication via PGP"
type: "pages"
---

To ensure our communication remains private and authenticated, I use PGP (Pretty Good Privacy) encryption. My PGP Public Keys allows you to encrypt messages to me and verify my digital signature.

Using PGP is especially recommended when:
- You’re sending sensitive or personal information
- You want to verify that a message genuinely came from me
- You care about privacy and data integrity

If you’d like to communicate securely, please use my public key below.

## PGP Fingerprints and Public Keys
### PGP Fingerprints
- PGP fingerprint for my personal address: `gpg --fingerprint A1C9A7F709D3A889A3539A78C7CEA07966701A9E`
- PGP fingerprint for my security address: `gpg --fingerprint D8B80874001A39C573C4BC3F7A3694918FF5706D`

> **IMPORTANT**: Always verify the `Fingerprint` of a key owner from trusted sources, such as their website or email signature, or through in-person verification.

### PGP Public Keys
- [PGP Public Key for my personal address](https://vand3rlinden.com/encryption/pgp-ricardo-publickey.txt)
- [PGP Public Key for my security address](https://vand3rlinden.com/encryption/pgp-security-publickey.txt)

> **NOTE**: You can save the content to a `.asc` file and import the PGP public key using the following command: `gpg --import publickey.asc`

## PGP Tooling
I use the command-line tool [GnuPG](https://www.gnupg.org/), also known as GPG, which is the most widely adopted utility for generating and managing PGP keys. To streamline tasks like encryption, decryption, signing, and signature verification, I have developed a bash script that works seamlessly if **GnuPG** is installed on your system, available for [download](https://github.com/vand3rlinden/Bash/blob/main/pgp-buddy/pgp_tool.sh) on my GitHub repository.

Additionally, I have created another bash script to manage PGP keys, including key generation, import, and export. This script is also available on my GitHub repository for [download](https://github.com/vand3rlinden/Bash/blob/main/pgp-buddy/pgp_key_tool.sh).

For iOS, I use [Instant PGP](https://apps.apple.com/us/app/instant-pgp/id1497433694) to manage and use PGP keys. To maintain security, make sure iCloud backup is disabled for the Instant PGP app to prevent your private key from being synced to iCloud.