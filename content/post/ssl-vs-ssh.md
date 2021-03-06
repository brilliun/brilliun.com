---
title: "SSL and SSH"
date: 2017-02-19T22:33:18+09:00
lastmod: 2017-02-19T22:33:18+09:00
draft: false
keywords: []
description: ""
tags: ['SSL/TLS', 'cryptography']
categories: ['Learning']

isCJKLanguage: false

comments: true
showTags: true
showPagination: true
showSocial: true
showDate: true
showMeta: true
showActions: true
---

<!-- toc -->

# Concepts

>**SSL**: Secure Sockets Layer
>
>**TLS**: Transport Layer Security (*TLS is the name of the IETF protocol standard that grew out of SSL 3.0*)
>
> **HTTPS**: HyperText Transfer Protocol over SSL/TLS
> 
> **SSH**: Secure Shell

HTTPS is the secure replacement for HTTP, while SSH is the secure replacement for Telnet.

# Similarity

* Both SSH and SSL use **symmetric** cryptography to preserve the confidentiality of transmitted data.
* They both use **asymmetric** cryptography a.k.a. **public key cryptography** for authentication.
    * Authentication allows one party to verify whether the other party is really who it claims it is.
* They both have data integrity mechanisms



# Difference

One of the most noticeable differences between SSL/TLS and SSH is that SSL normally (yes, there can be exceptions) employs X.509 digital certificates for server and client authentication whereas SSH does not. And because SSL uses digital certificates, it consequently also requires the presence of a public key infrastructure (PKI) and the participation of a certificate authority (CA).

Another big difference is that SSH has more functionality built into it. For instance, on its own, SSH can enable users to login to a server and execute commands remotely. SSL does not have this capability. You would need to pair it with another protocol (e.g. HTTP, FTP, or WebDAV) in order for it to have similar functions.
