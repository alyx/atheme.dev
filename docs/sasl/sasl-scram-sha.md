---
id: sasl-scram-sha
title: SCRAM-SHA Support for SASL Logins
sidebar_label: SCRAM-SHA
---

Atheme IRC Services version 7.3 and above supports SASL SCRAM-SHA logins.

Some work needs to be performed by the prospective IRC network administrator
to enable this. The 5 main steps to perform are:

### 1) Build Atheme with GNU libidn support (`./configure --with-libidn`)

### 2) Load `modules/crypto/pbkdf2v2` **before** any other crypto module

This ensures that it will become the primary crypto provider

### 3) Decide which SCRAM mechanism you want to use.

It is highly recommended that you choose **SCRAM-SHA-256**. You cannot enable
more than one. SCRAM-SHA-1 is only supported to comply with RFC 5802, which
states that supporting SHA-1 is required. However, all modern client SASL
libraries that support SCRAM, support SCRAM-SHA-256 (RFC 7677), and any new
client implementations are expected to as well. You should only choose
SCRAM-SHA-1 if you have a large user base that wants to use SCRAM, but whom
cannot use SCRAM-SHA-256 or SCRAM-SHA-512. SCRAM-SHA-512 is not officially
specified, and so is not widely implemented, but RFC 5802 Section 4
indirectly allows it, and the default PBKDF2v2 digest was (and remains)
SHA2-512, which will allow SCRAM-SHA-512 logins without services having to
recompute any PBKDF2 digests for users who reidentify (by plain text
password) or change their password. This enables a seamless transition to
SCRAM logins if you are still using the default algorithm. If you were using
the pbkdf2v2 module previously with SHA2-256 digests, choose SCRAM-SHA-256
instead. If you were not previously using the pbkdf2v2 module, choose
SCRAM-SHA-256 instead.

### 4) Configure pbkdf2v2 to generate SCRAM-SHA hashes (atheme.conf):

```
   crypto {
       pbkdf2v2_digest = "SCRAM-SHA-256";
       /* or "SCRAM-SHA-512" or "SCRAM-SHA-1" */
       #pbkdf2v2_rounds = ...;          /* between 10000 and 65536 ** */
   }
```

**Inclusive. The popular Cyrus SASL client library will refuse to perform a
PBKDF2 calculation with an iteration count greater than 65536, and the
pbkdf2v2 crypto module will refuse an iteration count lower than 10000.
The default is 64000, so you can continue to omit this parameter from your
configuration file if you are doing so already.

### 5) Load `modules/saslserv/scram-sha`
