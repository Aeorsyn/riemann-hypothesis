---
title: "*sshole: how * became a password in OpenSSH on musl"
url: "https://bosnadev.com/2026/11/05/sshhole-openssh-musl-star-password"
author: "Mirza Pašić"
date: "2026-11-05"
topic: "Server administration"
tags: ["security", "linux", "openssh"]
summary: "sshd on Alpine and other musl systems took any 257 to 1,024 byte password for accounts with * in /etc/shadow. Who is exposed, why, and what to do."
---

# *sshole: how * became a password in OpenSSH on musl

For ten years, `sshd` on some musl Linux systems accepted a password for accounts that have no password. Send any 257 to 1,024 bytes and the account opens. [Alpine's advisory](https://alpinelinux.org/posts/2026-11-05-openssh-password-authentication-bypass/) puts it plainly: "It takes one connection, works every time, and needs no retries."

**If you run `sshd` on Alpine, update `openssh` and then restart `sshd`.** The upgrade alone leaves the old daemon running.

Özgün Kültekin of [Wakefield](https://wakefield.co) found it and named it [\*sshole](https://sshhole.com/). OpenSSH shipped a workaround in 10.6p2 on 5 November 2026. No CVE had been assigned when Alpine published, and I haven't found one since.

Nobody wrote this bug on purpose. Different people made five sensible decisions between 1980 and 2022, and one character connects them.

## Am I affected?

Answer these five questions. You are exposed only if all five are true.

You are exposed only if all five of these are true:

1. Does sshd run on musl libc? Alpine, and OpenWrt, Buildroot or Yocto builds that use OpenSSH. `ldd /usr/sbin/sshd` shows it.
2. Does sshd check passwords without PAM? Alpine's openssh-server is built without PAM. openssh-server-pam with UsePAM no counts too.
3. Are password logins on? PasswordAuthentication yes is the default. cloud-init images turn it off.
4. Does an account have exactly * in /etc/shadow? Key-only users, root on Alpine 3.16 and later, the alpine user on tiny-cloud images. Not !, which stays locked.
5. Does another account have a real $5$ or $6$ hash? One user with a password is enough, such as an admin who set a console password.

These commands check them on a running box:

```bash
ldd /usr/sbin/sshd | grep -i musl
sshd -T | grep -Ei '^(usepam|passwordauthentication|permitrootlogin)'
awk -F: '$2 == "*" {print $1}' /etc/shadow
awk -F: '$2 ~ /^\$[56]\$/ {print $1}' /etc/shadow
```

A build without PAM prints no `usepam` line at all.

The fifth condition surprises people. One user with a password makes every `*` account on the machine reachable. Some early coverage also overstates the reach, so three details:

- **Containers:** running an Alpine image doesn't expose you by itself. The advisory leaves out containers that don't run `sshd`.
- **Root:** Alpine's root has had `*` since about 3.16, but the default `PermitRootLogin prohibit-password` protects it. Root is reachable directly only with `PermitRootLogin yes`. An attacker who gets into another account can still become root, for example with `doas`.
- **Cloud images:** the `alpine` user in tiny-cloud images has a `*` field and passwordless `doas`. The advisory says these images are safe on first boot and become exploitable as soon as any `$6$` hash exists, for example when an admin sets a console password.

Not affected: images set up by cloud-init (it writes `PasswordAuthentication no`), glibc systems, non-Linux systems, `UsePAM yes`, and Dropbear, which is the default SSH server on OpenWrt. Accounts locked with `!` can't log in with a password or a key, so they're out too.

## Five decisions, one character

The [\*sshole write-up](https://sshhole.com/) tells this story as a comic, and it's worth reading. Here's the short version, with the details checked in the source.

**five decisions, 1980 to 2022**

1. **1980: * means nobody logs in here**

   Early Unix kept password hashes in `/etc/passwd`. In 3BSD the `daemon` account has a `*` in that field. A `*` is not a valid hash, so no password can produce it. The write-up credits Bill Joy, and Unix still uses the convention.

2. **1997: crypt() returns NULL on failure**

   The Single Unix Specification says so. Plenty of programs never checked, compared the result anyway and crashed.

3. **9 August 2012: musl stops returning NULL**

   Rich Felker changed musl so that a failed `crypt()` returns a string that can't match. [The commit](https://git.musl-libc.org/cgit/musl/commit/?id=b3c4cc121f70faea45389fe7ddc1127ed5cbd8bb) calls this "reportedly a better behavior than failing on error". In `crypt_des.c` the failure path is one line:

   ```c
   return (setting[0]=='*') ? "x" : "*";
   ```

   If the setting is `*`, it returns `x`, so the result never equals the setting it was given. In 2012 that was enough, because callers passed the stored hash as the setting.

4. **29 August 2012: a 256-byte cap**

   [Twenty days later](https://git.musl-libc.org/cgit/musl/commit/?id=88bf5a8a8d7d796f63cca8589f4de67aa8345f1a) musl added SHA-256 and SHA-512 crypt. These get slower as the password gets longer, so musl refuses long keys. The source says why: "key limit is not part of the original design, added for DoS protection". Above 256 bytes these functions fail, and failure now means `*`.

5. **15 July 2016: OpenSSH borrows a salt**

   [CVE-2016-6210](https://github.com/openssh/openssh-portable/commit/9286875a) was a timing leak: send long passwords, time the replies, and you learn which usernames exist. Darren Tucker's fix hashes with a salt taken from a real account, so every failure takes as long.

   The fix was meant for invalid users. One condition also sends real accounts down the new path when their stored field is too short to be a salt:

   ```c
   if (authctxt->valid && pw_password[0] && pw_password[1])
           salt = pw_password;
   encrypted_password = xcrypt(password, salt);
   ```

   Before this, a one-character field got the DES salt `xx`, and DES on musl has no key limit. After it, a `*` account gets a borrowed `$6$` salt. The setting and the stored field are now different strings, so musl's `x` guard never fires. The bug starts here.

6. **2019 to 2022: Alpine moves root to ***

   CVE-2019-5021 was an empty root password in Alpine's Docker images. Alpine locked root with `!`. Without PAM, `!` also blocks SSH keys, so Alpine 3.16 changed root to `*` in 2022. That didn't cause the bug, but it made `*` accounts normal on the distribution most likely to meet the other conditions.

As far as anyone knows, nobody noticed for ten years. The write-up gives the reasons: OpenSSH tests on OpenBSD and glibc, and musl doesn't test `sshd`. Most distributions use PAM, which never reaches this code. And a box with no real password hash is safe, because the salt falls back to `xx`.

Two threads came close. In 2013 Thorsten Glaser of Debian [asked glibc](https://sourceware.org/legacy-ml/libc-alpha/2013-07/msg00046.html) to return `*` on failure the way musl does, in [Debian bug 714219](https://bugs.debian.org/714219). Debian later moved to libxcrypt, which returns `*0`, one character away. In December 2024, a [musl list thread](https://www.openwall.com/lists/musl/2024/12/29/1) dealt with `passwd` crashing on Alpine when PAM tried to overwrite musl's `*` after a password longer than 256 characters. Felker [defended the limit](https://www.openwall.com/lists/musl/2024/12/31/2). Same `*`, same limit, and nobody mentioned `sshd`.

## The bug in six lines

Without PAM, `sshd` checked passwords like this. I shortened the names from [`auth-passwd.c`](https://github.com/openssh/openssh-portable/commit/4353fdc43ef646c43a8dda7025cc1a898caf1482):

```c
char *stored = shadow_pw(pw);          /* "*" */
char *salt = NULL;
if (valid && stored[0] && stored[1])   /* one character: not a salt */
        salt = stored;
char *out = xcrypt(password, salt);    /* no salt: borrow one */
return out != NULL && strcmp(out, stored) == 0;
```

Step a 300-byte password through it:

**Diagram: One 300-byte password through sshd without PAM, on glibc and on musl.** Three shared steps, then two lanes. A 300-byte password arrives for an account whose shadow field is *. The field is too short to be a salt, so sshd borrows a $6$ salt from another account. On glibc, crypt() returns a real hash, which does not equal *, and the login is rejected. On musl, crypt() refuses keys over 256 bytes and returns *, which equals the stored field, and the login is accepted.

1. **A password arrives.** Someone sends 300 bytes as the password for deploy, an account that has * in /etc/shadow. sshd accepts up to 1,024 bytes.
2. **The stored field is not a salt.** sshd uses the stored field as the salt when it has at least two characters. * has one, so sshd has no salt of its own.
3. **sshd borrows a salt.** Since 2016, sshd hashes with a salt taken from another account, so a failed login takes as long as a real one. Here it finds bob's $6$ salt.
4. **crypt() runs.** glibc hashes the 300 bytes like any other password. musl's sha512 crypt refuses keys over 256 bytes and returns its error string, *.
5. **strcmp() compares.** sshd compares what crypt() returned with the stored field. On glibc that is a full $6$ hash against *. On musl it is * against *.
6. **The verdict.** glibc rejects the login. musl accepts it, for any password from 257 to 1,024 bytes.

*one 300-byte password, glibc and musl*

I didn't want to take the libc part on trust, so I ran `crypt()` on real musl and on real libxcrypt, in `alpine:3.24` and `ubuntu:24.04` containers:

```c
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

int main(void) {
    char key[1025];
    memset(key, 'A', 256); key[256] = 0;
    puts(crypt(key, "$6$saltsalt$"));   /* 256 bytes: a hash */
    memset(key, 'A', 257); key[257] = 0;
    puts(crypt(key, "$6$saltsalt$"));   /* 257 bytes: "*" on musl */
    puts(crypt(key, "*"));              /* a bare star as setting: "x" */
}
```

```bash terminal
docker run --rm -v "$PWD":/w -w /w alpine:3.24 \
  sh -c 'apk add -q build-base && cc star.c -o star && ./star'
```

One extra byte turns a full hash into a single star. The wider run, 7 to 1,024 bytes:

| Setting        | Password    | musl                   | glibc with libxcrypt |
| -------------- | ----------- | ---------------------- | -------------------- |
| `$6$saltsalt$` | 256 bytes   | real hash              | real hash            |
| `$6$saltsalt$` | 257 bytes   | `*`                    | real hash            |
| `$5$saltsalt$` | 257 bytes   | `*`                    | real hash            |
| `$6$saltsalt$` | 512 bytes   | `*`                    | `*0`                 |
| `$1$saltsalt$` | 1,024 bytes | real hash              | `*0`                 |
| `xx` (DES)     | 1,024 bytes | real hash              | `*0`                 |
| `*`            | any length  | `x`                    | `*0`                 |
| `*LK*`         | any length  | DES hash starting `*L` | `*0`                 |

The table explains the odd numbers in the advisories:

- **Why 257 bytes:** the first length past musl's limit.
- **Why 1,024 bytes:** `MAX_PASSWORD_LEN` in `sshd`, which rejects anything longer before it hashes.
- **Why glibc is safe:** libxcrypt fails from 512 bytes too, but its error string is `*0`, and `*0` isn't `*`.
- **Why `!` is safe:** `crypt()` still returns `*`, and `*` isn't `!`.
- **Why `*LK*` is safe:** it has four characters, so `sshd` uses it as the salt, and musl computes an ordinary DES hash that starts `*L` and can't equal `*LK*`.

Try it yourself. This runs the same decisions in your browser with stand-in hashes, and nothing leaves the page:

| Password | Debian · glibc + libxcrypt | Alpine · musl |
|---|---|---|
| 7 bytes | a real hash, denied | a real hash, denied |
| 256 bytes | a real hash, denied | a real hash, denied |
| 257 bytes | a real hash, denied | crypt() returns `*`, **accepted** |
| 1024 bytes | crypt() returns `*0`, denied | crypt() returns `*`, **accepted** |
| 1025 bytes | never hashed, denied | never hashed, denied |

*A `*` account on a box where another account has a `$6$` hash, before the fix. Interactive in the article: try any password and stored field.*

I tested the libc here, not a full login against a live `sshd`. The login path comes from reading the code, the write-up and Alpine's advisory.

## The fix, and whose bug it is

The security part of [the OpenSSH commit](https://github.com/openssh/openssh-portable/commit/4353fdc43ef646c43a8dda7025cc1a898caf1482) is one new condition:

```c
#define MIN_PASSWORD_HASH_LEN 4

ret = encrypted_password != NULL &&
    strlen(pw_password) >= MIN_PASSWORD_HASH_LEN &&
    strcmp(encrypted_password, pw_password) == 0;
```

A stored field shorter than four characters now can't match anything. `sshd` still runs `crypt()` first, so the 2016 timing fix stays in place.

Neither side claims the root cause. The commit message says: "We do not consider the root cause to be within OpenSSH". Felker's [advisory to musl users](https://www.openwall.com/lists/musl/2026/11/05/4) says: "It is my position that this is not a bug in musl just an erroneous use by the caller."

Both have a case:

- **Against musl:** the specification says `crypt()` returns NULL on failure. `sshd` checked for NULL, so a change made to protect careless programs took the signal away from a careful one.
- **Against OpenSSH:** musl's guard promises that the result never equals the setting. Since 2016, `sshd` has compared the result with a different string, and that promise never covered it.

I think each side kept its own contract and the failure sits between them. The last change to musl's crypt code in a [mirror of its history](https://github.com/kraj/musl) dates from 2020, and the mirror runs to 25 October, so `*` was still the error value then. `sshd` is the only known victim, but any program on musl that hashes with one string and compares with another has the same problem.

## One day, no embargo, and a lot of agents

OpenSSH learned of the bug the day before it shipped the fix. The commit is dated 5 November at 08:18 UTC+11, the evening of 4 November in UTC. Damien Miller [explained the speed](https://lists.mindrot.org/pipermail/openssh-unix-dev/2026-November/042837.html):

> We are making this release as fast as possible (we learned of the problem yesterday) because we have recently observed multiple cases of rapid bug rediscovery by AI agents, sometimes within hours, and so now consider long embargo periods to be a dangerous practice that primarily benefits attackers.

OpenSSH had said as much a month earlier. The [10.6 announcement](https://seclists.org/oss-sec/2026/q4/58) of 6 October noted that bugs found by AI tools were being found again by other researchers, which suggests attackers who never report bugs can find them too.

The discovery method is the part I keep thinking about. Kültekin built `sshd` every way possible: glibc and musl, with and without PAM, with and without shadow. Then came about 17,600 odd logins against the builds. The result fits in one line of the write-up: "every build agreed on every login. except musl without pam." On who did the work: "i had agent swarms. a lot of them."

Differential testing is an old idea, and agents made it cheap. Kültekin didn't have to suspect `crypt()`. The builds disagreed, and the disagreement pointed at the bug.

Code review alone missed it. On 30 October, [a commit](https://github.com/openssh/openssh-portable/commit/84f8f4666b536dcd3959ab27171caecd5e80947b) fixed `pick_salt()` for bcrypt hashes, crediting the report Anthropic ANT-2026-7T6M8N4X. That change touched the salt logic at the centre of this story, and the bypass stayed. No single file looks wrong. The bug shows up only when all the parts run together.

Fast disclosure has a cost, and small operators pay it. The full write-up went public on the day of the fix, so admins had hours. I found no reports of exploitation and no count of exposed hosts so far, which doesn't say much on day one.

## What to do now

**1. Update and restart.** On Alpine:

```bash terminal
apk upgrade -U -a
rc-service sshd restart
```

Don't go by the upstream version number. Alpine backported the patch, so a fixed 3.24 box still says 10.3. [Alpine's advisory](https://alpinelinux.org/posts/2026-11-05-openssh-password-authentication-bypass/) lists these fixed packages:

| Alpine branch | Fixed openssh package |
| ------------- | --------------------- |
| edge          | 10.6_p2-r0            |
| 3.24          | 10.3_p1-r2            |
| 3.23          | 10.2_p1-r1            |
| 3.22          | 10.0_p1-r11           |
| 3.21          | 9.9_p2-r1             |
| 3.20          | 9.7_p1-r6             |
| 3.19          | 9.6_p1-r3             |
| 3.18          | 9.3_p2-r4             |
| 3.17          | 9.1_p1-r7             |
| 3.16          | 9.0_p1-r6             |

Alpine 3.21 and older are past their end of support, 3.21 since 1 November. The fixed packages are there, but plan to move off those branches.

On other musl systems, build OpenSSH 10.6p2 or wait for your vendor. OpenWrt's OpenSSH package [moved to 10.6p2](https://github.com/openwrt/packages/pull/31158) the same day. Turn password logins off until then, and rebuild any images you ship that contain `sshd`.

**2. If you can't update, turn password logins off.** First check that your key works, and keep your current session open:

```bash terminal
mkdir -p /etc/ssh/sshd_config.d
echo 'PasswordAuthentication no' > /etc/ssh/sshd_config.d/00-no-passwords.conf
sshd -t && rc-service sshd restart
sshd -T | grep -i passwordauthentication
```

The last command must print `passwordauthentication no`. Alpine's default config reads that directory only from 3.19 onward. On older releases, or with `Match` blocks, use the `sed` command in the advisory.

**3. Check exposed machines for a break-in.** Look for password logins to accounts that have no password:

```bash terminal
grep 'sshd.*Accepted password for' /var/log/messages
```

Such a line for `alpine`, `root` or a key-only user is a bad sign. Also review `doas` use, new users, new files and `authorized_keys`. Your logs may not go back far, and the bug is ten years old. Alpine's advice: if you can't rule out a login, treat the machine as compromised.

**4. Don't rule yourself out on a technicality.** The conditions above are the ones published today. If your server runs `sshd` on musl without PAM, update it even if one condition looks false.

## What I take from it

`*` meant "no login" in the data and "failure" in a return value. Those two meanings lived apart from 2012 to 2016, until one comparison put them on either side of a `strcmp`. Any `-1`, `""` or `"unknown"` your code returns as an error can meet a stored value the same way.

musl promised that the result never equals the setting. `sshd` needed a different promise: that the result never equals the stored field. The two were the same until a timing fix separated them. When you change what you pass into a function, check what its guarantees still cover.

A `*` in `/etc/shadow` feels like "this account has no password", but `sshd` only ever saw a string to compare. If you want no password logins, set `PasswordAuthentication no` and let the server refuse them before any hash runs.

## Sources

- [\*sshole](https://sshhole.com/), the original write-up by Özgün Kültekin, with a browser demo
- [Alpine Linux security advisory](https://alpinelinux.org/posts/2026-11-05-openssh-password-authentication-bypass/), 5 November 2026
- [OpenSSH fix commit 4353fdc](https://github.com/openssh/openssh-portable/commit/4353fdc43ef646c43a8dda7025cc1a898caf1482)
- [Damien Miller, urgent security release: 10.6p2](https://lists.mindrot.org/pipermail/openssh-unix-dev/2026-November/042837.html), openssh-unix-dev, 5 November 2026
- [OpenSSH 10.6p2 announcement](https://lists.mindrot.org/pipermail/openssh-unix-announce/2026-November/000170.html)
- [OpenSSH 10.6 announcement](https://seclists.org/oss-sec/2026/q4/58), 6 October 2026
- [OpenSSH commit 9286875a](https://github.com/openssh/openssh-portable/commit/9286875a), the 2016 salt change for CVE-2016-6210
- [OpenSSH commit 84f8f466](https://github.com/openssh/openssh-portable/commit/84f8f4666b536dcd3959ab27171caecd5e80947b), the October 2026 `pick_salt()` fix
- [musl commit b3c4cc12](https://git.musl-libc.org/cgit/musl/commit/?id=b3c4cc121f70faea45389fe7ddc1127ed5cbd8bb), `crypt()` stops returning NULL
- [musl commit 88bf5a8a](https://git.musl-libc.org/cgit/musl/commit/?id=88bf5a8a8d7d796f63cca8589f4de67aa8345f1a), SHA-256 and SHA-512 crypt with the 256-byte limit
- [Rich Felker, advisory for OpenSSH+musl users](https://www.openwall.com/lists/musl/2026/11/05/4), musl list, 5 November 2026
- [musl list, crypt(3) returns string literal "\*", crashing PAM on Alpine](https://www.openwall.com/lists/musl/2024/12/29/1), December 2024
- [LWN: Critical security vulnerability affects OpenSSH on Alpine Linux systems](https://lwn.net/Articles/1105464/), 5 November 2026
