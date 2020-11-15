---
title: "Generate secure passwords with Linux or Mac OS and store them safely …at paper"
date: "2020-11-15"
tags:
    - shell
    - cli
    - gpg
    - security
---

![Render generateg secret shares as QP codes](/assets/bash-script-split-generate-password-and-render-qr-codes.png)

This article covers problem or storing passphrase securely. It's usually a common use case create strength password, but it's hard to remember them. Very often, such passwords are stored inside text files in a plain form. Good, if such files are located on encrypted storage.

How to solve this problem? Secret sharing to the rescue!

Secret sharing is a method of splitting a secret into several parts and distributing them across multiple locations or even secret holders. We will use Shamir's Secret Sharing algorithm to split secret for further sharing. This algorithm adding additional layer of security, where original secret parts encoded into hashes and secret holders do not know even part of original plain secret. Later, secret can be reconstructed from parts.

Let's get started!

### Install required dependencies

**Ubuntu/Debian**

```bash
sudo apt install ssss qrencode gpg
```

**Mac OS**

```shell
brew install ssss qrencode gpg
```

### Generate password secret and split secret into shares

First, generate password using [GPG](https://gnupg.org/).

```shell
SECRET_PASSWORD=$(gpg --armor --gen-random 2 18)
```

Command generates binary random data and transforms it into printable ASCII characters (ASCII armor) with base64 encoding. I'm used 18 chars for password length to avoid adding `==` output padding characters to the final string.

Next, split secret into 4 parts, with 3 shares necessary to reconstruct secret. Option `-w secret` prefixes related shared together. At least 3 valid parts are required to successfully reconstruct secret.

```shell
echo $SECRET_PASSWORD | ssss-split -n 4 -t 3 -w secret
```

Command output:

```txt
Generating shares using a (3,4) scheme with dynamic security level.
Enter the secret, at most 128 ASCII characters: Using a 192 bit security level.
secret-1–4ca1b67c6f42b2f55f37a114ca1da1659d8fd0c6dc21fb6e
secret-2–60537b3ff6a8be9693ad6e1bcaa42cae81e4d4eb5c44ec83
secret-3-abd2834094905b0e04f3c815b2435f761cf7460f51f0a586
secret-4–8805337204530fb2aa05416e64074810ef13a3b790d03e75
```

Render hashes into QR codes

```shell
qrencode "secret-1–4ca1b67c6f42b2f55f37a114ca1da1659d8fd0c6dc21fb6e" -t ANSIUTF8
```

Try to reconstruct secret from part, with command `ssss-combine -t 3`. Use any 3 shares in any order to decrypt secret (note: parts are joined with new line `\n`).

```shell
echo "secret-4-8805337204530fb2aa05416e64074810ef13a3b790d03e75\nsecret-1-4ca1b67c6f42b2f55f37a114ca1da1659d8fd0c6dc21fb6e\nsecret-2-60537b3ff6a8be9693ad6e1bcaa42cae81e4d4eb5c44ec83" | ssss-combine -t 3 -q
```

### Automate secret split with shell script

Simple bash script to split secret into shares and encode them as QR coded PNG images.

```shell
#!/usr/bin/env bash

PASSWORD=$(gpg --armor --gen-random 2 18)
SHARES=($(echo ${PASSWORD} | ssss-split -q -n 4 -t 3 -w secret))

for share in ${SHARES[@]}
do
    qrencode -t PNG $share -o "${share}.png"
done
```

Print these QR codes on paper and store them in different places.

## Wrap up

In this article, we generated password with **GnuPG** and used Shamir's Secret Sharing cryptographic algorithm to split secret into several shares, addition, shares were rendered as QR codes for printing them and storing offline.
