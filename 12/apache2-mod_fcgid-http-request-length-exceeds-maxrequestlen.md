---
title: "Apache2 mod_fcgid: HTTP request length exceeds MaxRequestLen"
url: "https://bosnadev.com/2014/12/12/apache2-mod_fcgid-http-request-length-exceeds-maxrequestlen"
author: "Mirza Pašić"
date: "2014-12-12"
topic: "Server administration"
tags: ["apache", "php", "configuration"]
summary: "Large uploads to Pydio failed with \"HTTP request length exceeds MaxRequestLen\". Raising the PHP upload limits and FcgidMaxRequestLen in Apache fixed it."
---

# Apache2 mod_fcgid: HTTP request length exceeds MaxRequestLen

> Outdated: Written in 2014. Versions, APIs and advice may have changed since.

Few days ago I found a very good Open Source project for sharing and synchronization (Dropbox alternative) called [Pydio](https://www.pydio.com/ "Pydio"). Heard before for this project, but I have never tried to install it before. I tried the demo and decided to install Pydio on my own server and check this little bit more. Installation procedure went smoothly and in 5 minutes I had my sharing web site ready to use.

But, when I tried to upload multiple files, all of them larger than 3Mb, browser simply returned:

```
Error 500, Internal server error
```

Then I checked Apache2 log, and here is what I found:

```
[Thu Dec 11 14:45:58 2014] [warn] [client xxx.xxx.xxx.xxx] mod_fcgid: HTTP request length 15735832 (so far) exceeds MaxRequestLen (15728640), referer: https://mynicesite.dev
```

It seams that [MaxRequestLen](https://httpd.apache.org/mod_fcgid/mod/mod_fcgid.html#fcgidmaxrequestlen "FcgidMaxRequestLen Directive") directive of mod_fccgid is too low. To fix this, you need to update few options in your php.ini file. Search for [_upload_max_filesize_](https://php.net/manual/en/ini.core.php#ini.upload-max-filesize "Upload Max Filesize") which puts limit on any single file, default value should be set to this:

```ini title="php.ini"
upload_max_filesize = 2M
```

change this value to whatever you want, but I strongly suggest you to set >50M. Then search for [_post_max_size_](https://php.net/manual/en/ini.core.php#ini.post-max-size "Post Max Size"), default value is:

```ini title="php.ini"
post_max_size = 8Mb
```

this is the size of the entire body of the request, which could include multiple files. Set this value at lest 2-3x as upload_max_filesize.

Now, to avoid _“ HTTP request length exceeds MaxRequestLen”_ you need to update [FcgidMaxRequestLen](https://httpd.apache.org/mod_fcgid/mod/mod_fcgid.html#fcgidmaxrequestlen "FcgidMaxRequestLen Directive") directive. Default value is set to:

```
FcgidMaxRequestLen 131072
```

which is, as you can guess, very very small. To change this, update your virtual host configuration file and add FcgidMaxRequestLen directive value based on your application requirement, for example:

```
FcgidMaxRequestLen 200000000
```

that is equal to roughly 200 MB.Finally restart Apache:

```bash terminal
/etc/init.d/apache2 restart
```

 

\[GARD]
