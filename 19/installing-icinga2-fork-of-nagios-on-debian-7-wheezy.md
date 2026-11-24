---
title: "Installing Icinga2 (fork of Nagios) on Debian 7 (Wheezy)"
url: "https://bosnadev.com/2015/03/19/installing-icinga2-fork-of-nagios-on-debian-7-wheezy"
author: "Mirza Pašić"
date: "2015-03-19"
topic: "Server administration"
tags: ["monitoring", "debian", "icinga"]
summary: "Installing Icinga 2 and the Nagios plugins on Debian 7 from the debmon repository, with MySQL through IDO, and Icinga Web with a fix for a package bug."
---

# Installing Icinga2 (fork of Nagios) on Debian 7 (Wheezy)

> Outdated: Written for Debian 7 (Wheezy) and the first Icinga 2 packages in 2015.

Contents

- 1 Preface
- 2 Prerequisits
- 3 Icinga and nagios plugins
- 4 Icinga Web
  - 4.1 Hans Kohl

## Preface

[Icinga](https://icinga.com/ "Icinga") is a fork of [Nagios](http://www.nagios.org/ "Nagios") monitoring system. There are lots of changes and upgrades compared to Nagios, especially in version 2. The main visible difference is UI which is built on ext js. Other significant differences are in hosts and services definitions (it will be covered in part 2).

For this tutorial we’re going to use [Digital Ocean](https://www.digitalocean.com/ "Digital Ocean") smallest droplet with installed Debian 7.8.

## Prerequisits

We’ll use debmon repository

At the end of file “/etc/apt/source.list” add the following entry:

```text title="/etc/apt/source.list"
#ICINGA
deb http://debmon.org/debmon debmon-wheezy main
```

Repository key:

```bash terminal
wget -O - http://debmon.org/debmon/repo.key 2>/dev/null | apt-key add -
```

Now run:

```bash terminal
apt-get update && apt-get -y upgrade
```

If there was a kernel upgrade (like in my case), reboot the server before proceeding to the next step.

## Icinga and nagios plugins

```bash terminal
apt-get install icinga2 nagios-plugins -y
```

Starting the service

```bash terminal
root@icinga-mon:~# /etc/init.d/icinga2 start
[ ok ] checking Icinga2 configuration.
[ ok ] Starting icinga2 monitoring daemon: icinga2.
root@icinga-mon:~#
```

We also want icinga integration with the database.

Install MySQL and IDO-MySQL

```bash terminal
apt-get install mysql-server
aptitude install icinga2-ido-mysql -y
```

And enable manually ido-mysql:

```bash terminal
root@icinga-mon:/etc/icinga2# icinga2 feature enable ido-mysql
Enabling feature ido-mysql. Make sure to restart Icinga 2 for these changes to take effect.
root@icinga-mon:/etc/icinga2# /etc/init.d/icinga2 restart
```

## Icinga Web

```bash terminal
apt-get install icinga-web -y
```

Choose dbconfig-common for db setup and it will prompt you for web interface password.\
After the setup icinga-web will bi available on: http\://SERVER/icinga-web/

username:root\
password: (set during the installation)

There seems to be a bug in icinga-web package from debmon.

![Screenshot from 2015-03-19 14:07:33](https://media.bosnadev.com/hub/498d999aed5abb95/2015-03-screenshot-from-2015-03-19-140733-282x60.webp)

After checking with chrome inspect tool, the url with this issue was:\
http\://SERVER/icinga-web/modules/cronks/reloadStatus/json?\_dc=1426770390818

```
PDO Connection Error: SQLSTATE[28000] [1045] Access denied for user 'icinga'@'localhost' (using password: YES)
```

The issue is in config file that is (for some reason) used for db connection:

```
/usr/share/icinga-web/app/config/databases.xml
```

Line 39:

```xml
<ae:parameter name="dsn">mysql://icinga_web:[email protected]:3306/icinga_web</ae:parameter>
```

Change with data from:

```
/etc/icinga-web/conf.d/database-web.xml
```

Line 90:

```xml
<ae:parameter name="dsn">mysql://icinga:[email protected]:3306/icinga</ae:parameter>
```

Change with data from:

```
/etc/icinga2/features-available/ido-mysql.conf
```

Ingraph

```bash terminal
apt-get install ingraph -y
```

In the next part of this tutorial we’ll go through configuration for new hosts and services.
