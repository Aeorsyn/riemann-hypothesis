---
title: "SSH Authentication With Keys Instead Passwords"
url: "https://bosnadev.com/2015/01/23/ssh-authentication-keys-instead-passwords"
author: "Mirza Pašić"
date: "2015-01-23"
topic: "Server administration"
tags: ["ssh", "security", "linux"]
summary: "Generating an SSH key pair with ssh-keygen, copying the public key to a server with ssh-copy-id, and turning off password logins in sshd_config."
---

# SSH Authentication With Keys Instead Passwords

> Outdated: Written in 2015. Versions, APIs and advice may have changed since.

SSH is a must have tool for every System administrator or DevOps Engineer. It provides you a secure way to access remote servers. But, if you use standard password authentication there is always a chance that someone will break into system due to weak password. This potential risk increases if multiple users have SSH access to the server, because system administrator can’t influence to the users to make a better passwords.

Contents

- 1 The Idea Of Public Key Cryptography
- 2 Generate Keys
- 3 Transfer The Public Key To The Server
- 4 Disable Password Authentication

## The Idea Of Public Key Cryptography

The basic idea behind key-based SSH authentication rely on [asymmetric cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography "asymmetric cryptography") also know as _public key cryptography_. This type of cryptographic algorithm require two separate keys. The first key is a _**secret**_ or _**private**_ key, and second is a _**public**_ key. Public key is used to encrypt _plaintext,_ whereas private key is used to decrypt _ciphertext_. You can learn more about public-key cryptography in [this](https://web.archive.org/web/20150306041524/https://developer.mozilla.org/en/docs/Introduction_to_Public-Key_Cryptography "Introduction to Public-Key Cryptography") MDN (Mozilla Developer Network) article.

## Generate Keys

First what you need to do on the client machine is to generate private/public keys. To generate private/public key set you need to run the following command:

```bash terminal
mirzap@bosnadev:~$ ssh-keygen -t rsa
```

Now you will have an option to name your key pair and set password.

![Public Key Encryption](https://media.bosnadev.com/hub/e86aa1360416b5b4/2015-01-public-key-ecryption-1200x675.webp)

## Transfer The Public Key To The Server

When you have generated private/public key-pair you need to transfer public key to the remote server. This public key will allow you to identify your self to the server. SSH comes with an utility called _**ssh-copy-id**_ that simply copies content of public key to the server’s \~/.ssh/authorized_keys :

```bash terminal
mirzap@bosnadev:~$ ssh-copy-id -i .ssh/mirzap.pub user@some_host_or_ip
```

You should have output similar to this:

```
Number of key(s) added: 1

Now try logging into the machine, with:   "ssh '[email protected]_host_or_ip'"
and check to make sure that only the key(s) you wanted were added.
```

If you don’t have _ssh-copy-id_ utility on your system, you can copy content of the public key and add it to the _authorized_key_ by your self.

## Disable Password Authentication

You can now connect to the remote server using a private key you generated, but your system isn’t secure just yet. You need to disable password-based and allow only key-based authentication. You can improve your server security by applying other measures, like changing default port or enabling host-based authentication, but that is not the topic of this article.

To disable password authentication open /etc/ssh/sshd_config and find the following line:

```text title="/etc/ssh/sshd_config"
#PasswordAuthentication yes
```

uncomment it, and set to _no_ :

```text title="/etc/ssh/sshd_config"
PasswordAuthentication no
```

Now /etc/ssh/sshd_config should look like this:

![Disable Password Authentication](https://media.bosnadev.com/hub/019fb787b429f36f/2015-01-disable-pass-auth.webp)Save and restart SSH server:

```bash terminal
user@secure:~# /etc/init.d/ssh restart
[ ok ] Restarting OpenBSD Secure Shell server: sshd.
```

If you try to connect to the server on the machine without private key, you’ll get this message:

```bash terminal
vagrant@homestead:~$ ssh user@some-host
Permission denied (publickey).
```

But on your local machine, where you created private key, authentication should pass without problems:

```bash terminal
mirzap@bosnadev:~$ ssh user@some-host
Last login: Fri Jan 23 15:24:15 2015 from some-ip
user@secure:~#
```

If you have created password during key generation, you’ll be asked to enter it on each login:

```bash terminal
mirzap@bosnadev:~$ ssh user@some-host
Enter passphrase for key '/home/mirzap/.ssh/mirzap_rsa':
user@secure:~#
```
