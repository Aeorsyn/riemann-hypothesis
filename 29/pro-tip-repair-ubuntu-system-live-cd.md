---
title: "Pro tip: How to repair an Ubuntu system with a Live CD"
url: "https://bosnadev.com/2014/12/29/pro-tip-repair-ubuntu-system-live-cd"
author: "Mirza Pašić"
date: "2014-12-29"
topic: "Pro tips"
tags: ["ubuntu", "recovery", "linux"]
summary: "A colleague ran chown -R on the whole root file system. Booting an Ubuntu Live CD, mounting the installed partition and using chroot to fix the owners."
---

# Pro tip: How to repair an Ubuntu system with a Live CD

> Outdated: Written in 2014. Versions, APIs and advice may have changed since.

Few days I ago, I had a funny situation. Colleague of mine accidentally executed:

```bash
sudo chown -R www-data:www-data /*
```

so instead for current directory ./\* he changed owner for entire root file system. Sure, he tried to change back owner to the root:root , but root did not have any permissions anymore, since /bin is owned by another user.

Fortunately, this kind of mess on Linux systems we can fix very quickly using Live CD. So reboot your system and run Live CD.

## Mounting a File System

First you need to mount the partition your Ubuntu installation is on. It will be something like sda1 or sda6 , if you are not sure about drive letter or partition number you can check it using _GParted_ which is included in the Live CD. You mount partition simply by execution mount command:

```bash terminal
sudo mkdir /mnt/ubuntu
sudo mount /dev/sdXY /mnt/ubuntu
```

## Chrooting

Now you need to bind these directories to the chroot:

```bash terminal
sudo mount --bind /dev /mnt/ubuntu/dev &&
sudo mount --bind /dev/pts /mnt/ubuntu/dev/pts &&
sudo mount --bind /proc /mnt/ubuntu/proc &&
sudo mount --bind /sys /mnt/ubuntu/sys
```

And finally you can enter in using chroot:

```bash terminal
sudo chroot /mnt/ubuntu /bin/bash
```

Now you can fix whatever you want in your system, in this case I restored root as owner.
