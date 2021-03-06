---
layout: post
title: My GnuPG Configuration
category:
  - blog
  - en
---

[GnuPG (GPG in short)][gpg] is a well-known cryptographic utility that enables you to encrypt and sign stuff using strong cryptographic methods (RSA, DSA, ECC, etc). [GnuPG][gpg] is also highly-configurable -- that means you can modify its behavior easily. Here I want to share my configuration of [GnuPG][gpg] to you.

**Note:** I am using Linux, and if you're using Windows, you can also find the corresponding configuration file in `%SystemDrive%\Users\%User%\.gnupg`, where `%SystemDrive%` means your system drive, and `%User%` means your user name.

**Update Oct. 19, 2019:** I have updated my `gpg.conf` for some time, so I think it's time to update this blog post too.

## ~/.gnupg/gpg.conf

```conf
# GPG Configuration File
# Junde Yhi <lmy441900@aosc.xyz>

# I have many private keys
default-key D274286F672C800A

# Secure memory, or quit
require-secmem

# Ask trust level during signing
ask-cert-level

# Show a short key ID for easy referencing
# (don't worry, for identity checking the full fingerprint is used)
keyid-format short

# Show the full fingerprint for comparison
with-fingerprint

```

### default-key

This is used when you have multiple secret keys, and you want to choose a default key.

**This is usually unnecessary.** Since your UID contains your email address, GPG can easily recognize which key should be used (unless you have multiple keys with the same UID, which is my case).

### ask-cert-level

This option let GPG ask the signature level when you're signing other's key. For example:

```
lmy441900 [ ~ ] $ LANG=C gpg --sign-key veracrypt@idrix.fr

pub  rsa4096/EB559C7C54DDD393
     created: 2014-06-27  expires: never       usage: SCE
     trust: unknown       validity: unknown
[ unknown] (1). VeraCrypt Team <veracrypt@idrix.fr>

gpg: using "4008E282" as default secret key for signing

pub  rsa4096/EB559C7C54DDD393
     created: 2014-06-27  expires: never       usage: SCE
     trust: unknown       validity: unknown
 Primary key fingerprint: 993B 7D7E 8E41 3809 828F  0F29 EB55 9C7C 54DD D393

     VeraCrypt Team <veracrypt@idrix.fr>

How carefully have you verified the key you are about to sign actually belongs
to the person named above?  If you don't know what to answer, enter "0".

   (0) I will not answer. (default)
   (1) I have not checked at all.
   (2) I have done casual checking.
   (3) I have done very careful checking.

Your selection? (enter '?' for more information):
```

This option may not be important for you. However, it does indicate how much you trust this key (with the corresponding UID), as well as a reference for others who are going to sign the key too.

### armor (or armour)

This let GPG put ASCII-armored results. By deafult GPG puts binary results (and even to `stdout`!). Use ASCII text makes your encrypted or signed file more explicit about what it is.

Update: ~~GPG is now smart enough...~~ **GPG still puts binary data to stdout...** The ASCII armoured text is more bloated than the binary format, so if you are encrypting or signing a big file, change it back.

### expert

**Update: I don't recommend this anymore. This option also disables some check on "insane" operations, so for key generation I recommend adding `--expert` manually.**

This option enables GPG to use new features (though those will be not so recommended, for compability reasons), such as ECC keypair generating.

For example, when using `gpg --full-gen-key` to generate a new key pair, we can see:

```
lmy441900 [ ~ ] $ LANG=C gpg --full-gen-key
gpg (GnuPG) 2.1.15; Copyright (C) 2016 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection?
```

with `--expert` option enabled, we can see:

```
lmy441900 [ ~ ] $ LANG=C gpg --full-gen-key --expert
gpg (GnuPG) 2.1.15; Copyright (C) 2016 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
   (9) ECC and ECC
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
Your selection?
```

You can setup your ECC key and authentication-use key only in expert mode. If you want GPG be more simple, you can ignore this option, and add it when you need it.

PS: [ECC](https://en.wikipedia.org/wiki/Elliptic_curve_cryptography) is a more modern algorithm, which provides stronger encryption with shorter key length. (Use curve25519!)

### require-secmem

This make GPG run only in a secure memory environment. GPG will alert you when it is in an "insecure" environment. Usually when running in a not-lockable memory or a flash memory disk you will receive such message, but GPG will not terminate operations. This option causes GPG refuse to proceed in this situation.

### with-fingerprint

This option turns the long ID displayed in `--list-keys` into the full key fingerprint. It's easier to read.

### with-subkey-fingerprint

Let GPG show subkeys' fingerprints too. **I don't use this option anymore, since it makes the key list longer and harder to read.**

## ~/.gnupg/dirmngr.conf

In new [GnuPG][gpg], key server is connected through `dirmngr`. So key server configurations are in `~/.gnupg/dirmngr.conf`.

```
keyserver hkps://sks.ustclug.org
keyserver hkps://pgp.mit.edu
```

This server is run by [USTC LUG](https://lug.ustc.edu.cn/wiki/). It's fast in mainland China, and has joined the [SKS Kerserver Pool](https://sks-keyservers.net/), so upload once, and every keyserver in the pool will receive the key.

You can use these protocols:

- `hkp://`: HTTP Keyserver Protocol
- `hkps://`: Secure HTTP Keyserver Protocol
- `ldap://`: LDAP. I don't know how to use it :P

~~Note that you need a certificate in `.pem` format if you want to use `hkps://` protocol. Using `hkps://` is recommended, if you can get the certificate in `.pem`, yet I can't find one for USTC LUG.~~ _(It is usable now)_

[gpg]: https://gnupg.org/
