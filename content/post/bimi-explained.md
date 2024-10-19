---
title: "Enhance your email's visual identity with BIMI"
date: 2024-10-19T23:30:32+02:00
draft: false
categories: ["email", "DNS"]
cover: 
  image: /images/bimi-explained/bimi-explained-front.png
---

>_In this post, you'll learn what BIMI is, how it works, and the benefits it brings to your domain, including increased trust and brand visibility._

In today’s digital age, making a memorable first impression is crucial. With the increasing volume of emails, standing out in the inbox can be challenging. That’s where BIMI (Brand Indicators for Message Identification) comes in, transforming how brands interact with recipients through email.

## Why BIMI Matters
BIMI allows your brand’s logo to appear alongside your emails, providing visual trust cues to recipients. This simple addition can significantly enhance brand recognition and trust, making your communications more engaging and trustworthy.

## Getting Started with BIMI
To begin your BIMI journey, you must first meet certain prerequisites. A crucial requirement is having a DMARC policy set to `p=quarantine` or `p=reject` on your organizational domain. It’s essential to avoid using `sp=none` or `pct<100` to ensure full protection.

## How BIMI Works
BIMI works by using your domain’s DNS records to display your verified logo in the email clients that support BIMI. This logo provides recipients with an immediate visual confirmation of your brand’s identity, reducing the risk of phishing and enhancing the overall user experience.

## Steps to Implement BIMI
1. **Authenticate Your Emails**: Ensure that your emails are authenticated using SPF, DKIM, and DMARC. These protocols verify the legitimacy of your emails and prevent spoofing.
   
> For more information about SPF, DKIM, and DMARC, and how to implement a DMARC-compliant domain, see an [earlier blog post](https://vand3rlinden.com/post/spf-dkim-dmarc-explanation/).

2. **Create a BIMI-Compliant Logo**: Your logo must be in SVG Tiny PS format. This format ensures that the logo is scalable and of high quality.

3. **Obtain a Verified Mark Certificate (VMC)**: A [VMC](https://www.digicert.com/tls-ssl/verified-mark-certificates#vmc_basic) is crucial for verifying your logo’s authenticity. It confirms that the logo is indeed associated with your domain.

4. **Publish Your BIMI Record**: Add a BIMI record to your DNS, specifying the URL of your logo. This record tells email clients where to find your logo.

### Example BIMI Record
Here's what a sample BIMI DNS record might look like:

`default._bimi.example.com IN TXT "v=BIMI1; l=https://example.com/bimilogo.svg; a=https://example.com/vmc.pem"`

- `v=BIMI1;` indicates the version of BIMI.
- `l=https://example.com/bimilogo.svg;` is the URL of your BIMI-compliant logo.
- `a=https://example.com/vmc.pem` is the URL to your Verified Mark Certificate.

Your logo might appear Like this:
![IMAGE](/images/bimi-explained/bimi-explained-front1.png)

## The Role of a Verified Mark Certificate (VMC)
A VMC is essential for implementing BIMI. It acts as proof that your logo is valid and associated with your domain, giving email clients the confidence to display it. Obtaining a VMC involves a verification process by a trusted third party, ensuring that only legitimate logos are used.

## Benefits of BIMI
- **Enhanced Brand Visibility**: Your logo in the recipient’s inbox makes your emails instantly recognizable.
- **Increased Trust and Engagement**: A familiar logo encourages recipients to open your emails, knowing they’re from a trusted source.
- **Phishing Prevention**: BIMI helps reduce phishing attacks by making it difficult for malicious actors to impersonate your brand.

## Conclusion
Implementing BIMI is a strategic move for any brand looking to enhance email engagement and security. By following the steps outlined above, you can ensure that your logo is prominently displayed in recipient inboxes, building trust and reinforcing your brand’s identity with every email sent.

## Reference
- [Verified Mark Certificates (VMC) and BIMI](https://bimigroup.org/verified-mark-certificates-vmc-and-bimi/)
- [Mailbox providers which supports BIMI](https://bimigroup.org/bimi-infographic/)
- [BIMI Group](https://bimigroup.org/)