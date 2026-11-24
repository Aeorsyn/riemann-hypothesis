---
title: "Virtual hosts overlap on 443, the first has precedence"
url: "https://bosnadev.com/2014/12/04/virtual-hosts-overlap-443-first-precedence"
author: "Mirza Pašić"
date: "2014-12-04"
topic: "Server administration"
tags: ["apache", "ssl", "virtual-hosts"]
summary: "Fixing the Apache 2 warning that virtual hosts overlap on port 443 after enabling SSL on several domains: add NameVirtualHost *:443 to ports.conf."
---

# Virtual hosts overlap on 443, the first has precedence

> Outdated: Written for Apache 2.2 in 2014. Since Apache 2.4, NameVirtualHost has no effect.

![Apache2 Logo](https://media.bosnadev.com/hub/c017c81737f4a4a9/2014-12-apache-logo.webp)Today I got strange error on my production server when I tried to enable SSL on three domains. Since I use ISPConfig as hosting control panel I assumed it will work without problems. But I was wrong. I did little bit of research on this topic, and I found a simple solution.

If you look at _**/etc/apache2/ports.conf**_ file, you will see:

```apache title="/etc/apache2/ports.conf"
NameVirtualHost *:80
Listen 80
```

This way Apache knows which virtual host to serve based on the different host names. And it works fine for \*:80 port. But, for 443 port it’s disabled, so you need to update _**ports.conf**_ as follows to make it work:

```apache title="/etc/apache2/ports.conf"
<IfModule mod_ssl.c>
NameVirtualHost *:443
Listen 443
</IfModule>
```

Make sure to restart Apache and it will work fine.

I hope this tip will save you some time.

#### Resources

More about Apache2 virtual hosts you can learn [here](https://httpd.apache.org/docs/current/vhosts/ "Apache2 Virtual Hosts"). Other useful links I found while I was looking for solution:

[Apache error: _default_ virtualhost overlap on port 443](https://stackoverflow.com/a/11615249 " Apache error: _default_ virtualhost overlap on port 443")\
[Virtual hosts overlap on 443, first has precedence](https://web.archive.org/web/20150509232529/http://blog.breidert.net/virtual-hosts-overlap-443-first-has-precendence/ "Virtual hosts overlap on 443, first has precedence")\
VirtualHost Overlap on Port 443
