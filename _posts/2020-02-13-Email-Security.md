---
title: Hardening Email Security
date: 2020-02-13 00:00:00
featured_image: '/images/email.jpg'
excerpt: Issues with email security features in the wild and how to get them working as a well-oiled machine.
---

![Picture of keyboard keys that spell out the word Email](/images/email.jpg)

## What's Being Seen In The Wild

With phishing and ransomware propagating crazily through email, it would be a no-brainer to harden the transport of our emails as best as possible.

We moved into a "never trust, always verify" mindset for email a few months ago by outright blocking emails that fail either SOFTFAIL or HARDFAIL in SPF. This is not a normal response for those errors. Typically a server will block HARDFAIL, but allow SOFTFAIL with a mark of it being untrusted. SOFTFAIL, listed as ~all in an SPF record, is not best practice. It's designed to temporarily help companies test their applications before switching it back to a HARDFAIL, listed as -all. We have had had to notify countless organizations of misconfigured security settings and guidance on how to resolve it where email would flow again in a verified way.

When we made the decision to block SOFTFAIL as well, I noticed an uptick in legitimate sources being blocked due to misconfigured security settings. Most of the emails being blocked are utilizing one email-hardening tool, SPF, and not taking any additional steps to verify their email, such as DKIM and DMARC.

With the growing abundance of misconfigured email security settings, I wanted to go over what tools are available to operate in a "never trust, always verify" mindset.

## What Are The Tools We Have?

### Sender Policy Framework (SPF)

#### How It Works

SPF uses a TXT DNS record to authenticate allowed email sending servers on your domain. When email is sent, the receiving server pulls the TXT DNS records of the domain located in the ENVELOPE FROM header of the email. If a SPF record is found in the TXT DNS record, it checks the allowed servers/IP's with those allowed in the SPF record. If an email does not pass SPF, it can either be SOFTFAIL'ed (receiving servers can allow softfail if desired and mark as an SPF failure), or can be FAILED completely.

**It is highly recommended to use FAIL (also known as HARDFAIL) instead of SOFTFAIL for your SPF setup as this will help reduce the risk of email spoofing.**

##### Example

A message is sent From emailserver.example.com (192.168.1.250) To testemail@example.org. The example.ORG server checks the DNS TXT records of the domain found in the Envelope From (example.com) for SPF. If one is found, example.org checks the SPF record to see if the sending IP address of the message is listed in the allowed servers for example.COM. 

If the server is included, the message passes. If it does not match, it follows the requested behavior of both the owner of the sending domain and the policies of the recipient's filters.

#### How It Is Set Up

##### DNS Location

A SPF TXT record that is externally viewable is located on the domain or subdomain that will be attached to the ENVELOPE FROM - generally on the same domain as your mail server.

**You can only have one SPF TXT record on your domain.**

##### Common Mechanisms

You can use a combination of mechanisms to add records.

| Mechanism | Purpose |
| :--- | :--- |
| **v=spf1** | Located at the beginning of an SPF record |
| **A** | Use the A record for that domain or subdomain |
| **MX** | Use the MX records for that domain or subdomain |
| **IP4** | Use an IP4 address \(ip4:1.1.1.1\) |
| **IP6** | Use an IP6 address |
| **INCLUDE** | Use the SPF records found at this domain |
| **-ALL** | FAIL all emails that do not pass the other mechanisms \(Can not be used with SOFTFAIL\) |
| **~ALL** | SOFTFAIL all emails that do not pass the other mechanisms \(Can not be used with FAIL\) |

##### Example

 v=spf1 a mx ip4:192.168.1.250 include:\_spf.google.com -all

 **NOTE:** I intentionally included an unrouteable IP address for this example. For your SPF records, you will be using publicly routable IP addresses

##### Limits

The limit of DNS lookups an SPF record can have is ten. the "all", "ip4" and "ip6" mechanisms do not count against this limit as they do not require a DNS lookup request.

***

### DomainKeys Identified Mail (DKIM)

#### How It Works

DKIM relies on a public/private key pair to sign and verify email messages. A public key is posted on DNS as a TXT record under _**selector**_**.\_domainkey.example.com.** Your sending server would then sign the email messages with the private key that can later be verified by the sending server with the public key. Once verified, the receiving server can be sure that the message was authorized and was not modified in transit.

 **NOTE:** Appended headers and footer messages \(compliance disclosures or cybersecurity reminders\) must be attached at the signing sender or receiving server in order to not fail DKIM. This can not be done in the middle of transit.

#### Example DKIM Public Record

TXT DNS - **selector.\_domainkey.example.com:**

"v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAsiv4mVODIx6hAfsjPoxRl
+h0AvN8MviDRoMiJxe0NzpslN2NH2xofQgQpIVBkkexZpC9m0E6gp3ES3tPutyglcS2pps
+DHbOegfi/nY3l69JVhBjNXl0whN20/FC3Adh0CETyRtKpI0HJStzr1A4
+Q9cAGW4hmAguJ1iwZ1FsFEOrI8Qe/ZtxZo8V1x4oWauhk9XOU
+ZUGRYDXTXeuEWyvtlfr5WN7N9I5tlD3BFMyB74k/LaVw6/
kRC2fm6ug8MoM11hYxt2f1s0jLeDSLh7Jyju9jHSaVJy1nRRDsf4veyA/
JCtmvwPJE3teYWlXIqTksMciWij4vyCTgAGmopxQIDAQAB"

 **NOTE**: Not an actual record for that domain and a random key generated just for this example.

***

### Domain-based Message Authentication, Reporting, and Conformance (DMARC)

#### How It Works

After you have SPF and DKIM set up on your domain, you can use DMARC to further set up your policy on how a receiving server should handle emails claiming to be from your domain. DMARC works as a TXT DNS entry set up on a specific subdomain: **\_dmarc**.example.com. The mechanisms added to the DNS define the policy. DMARC takes action and performs the "**p="** policy action when an email fails either SPF or DKIM. 

Postmark, a popular email sending service, provides a free tool to help companies with their DMARC reports. It creates a nice aggregated report weekly to show where your email is being originated from and how much of it is verified through SPF and DKIM.

<a href="https://dmarc.postmarkapp.com/"> Postmark DMARC Tool </a>

#### How It Is Set Up

##### Typical DMARC mechanisms

| Mechanism | Purpose |
| :--- | :--- |
| **v=DMARC1** | Located at the beginning of a DMARC record |
| **p=** | Determines policy response. Can be none, quarantine, or reject  |
| **rua=mailto:** | Tells the receiving server what email to send aggregate reports of DMARC failures |
| **ruf=mailto:** | Tells the receiving server what email to send forensic reports of DMARC failures |
| **rf=** | Tells what kind of report the policyholder wants delivered. **afrf** is default |
| **pct=100** | Tells the receiving server what percent of the mail was subject to the DMARC policy. |
| **sp=** | Determines if policy applies to subdomains. **none** is default |
| **adkim=** | DMARC strict or relaxed - **r** is default |
| **aspf=** | SPF strict or relaxed - **r** is default |

##### Example DNS Record

 **\_dmarc**.example.com

"v=DMARC1; p=reject; pct=100; rua=mailto:dmarc@example.com; sp=none; aspf=r;"

## References, Resources, and Citations

<a href="https://knowledge.ryangarr.com/it-systems/email/email-concepts" class="button button--small">Check Out The Knowledge Guide</a>

<a href="https://www.pexels.com/photo/email-blocks-on-gray-surface-1591062/">Photo by Miguel Á. Padriñán from Pexels</a>

<a href="https://tools.ietf.org/html/rfc7208"> RFC 7208 - Sender Policy Framework (SPF) for Authorizing Use of Domains in Email, Version 1