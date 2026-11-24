---
title: "How to allow remote connections to PostgreSQL database server"
url: "https://bosnadev.com/2015/12/15/allow-remote-connections-postgresql-database-server"
author: "Mirza Pašić"
date: "2015-12-15"
topic: "Databases"
tags: ["postgresql", "configuration", "security"]
summary: "Allowing remote connections to PostgreSQL 9.4: set listen_addresses in postgresql.conf, add a host line to pg_hba.conf and restart the server."
---

# How to allow remote connections to PostgreSQL database server

> Outdated: Written for PostgreSQL 9.4 in 2015.

After installing PostgreSQL database server, remote access mode is disabled by default for security reasons. However, sometimes you may want to allow remote connections to PostgreSQL database server from other locations, your home or office for example. In the next few lines I’ll guide you to do just that.

##### Connect to the remote server

First things first, you need to login to the remote server:

```bash terminal
mirzap@bosnadev:~$ ssh root@remote-host
```

##### Change The Listen Address

By default, PostgreSQL DB server listen address is set to the 'localhost' , and we need to change it so it accepts connection from any IP address; or you can use comma separated list of addresses. Here is how it looks by default:

```bash terminal
root@fe35577e9f8b:/# grep listen /etc/postgresql/9.4/main/postgresql.conf
listen_addresses = 'localhost'		# what IP address(es) to listen on;
```

Open your postgresql.conf file in your editor:

```bash terminal
root@fe35577e9f8b:/# vim /etc/postgresql/9.4/main/postgresql.conf
```

search for listen_addresses , and set it to '\*' :

```text title="/etc/postgresql/9.4/main/postgresql.conf" {7}
#------------------------------------------------------------------------------
# CONNECTIONS AND AUTHENTICATION
#------------------------------------------------------------------------------

# - Connection Settings -

listen_addresses = '*'         # what IP address(es) to listen on;
```

or if you want to set connection restrictions to a few IP’s, then you should set listen_addresses to something like this:

```text title="/etc/postgresql/9.4/main/postgresql.conf" {7}
#------------------------------------------------------------------------------
# CONNECTIONS AND AUTHENTICATION
#------------------------------------------------------------------------------

# - Connection Settings -

listen_addresses = '192.168.1.100,192.168.1.101,192.168.1.110'         # what IP address(es) to listen on;
```

To find out more about _connections and authentication_ and available parameters, check the official [documentation](https://www.postgresql.org/docs/current/static/runtime-config-connection.html) page.

##### Open PostgreSQL to the world

In this step, you need to allow remote connections to actually reach your PostgreSQL server. Open pg_hba.conf :

```bash terminal
root@fe35577e9f8b:/# vim /etc/postgresql/9.4/main/pg_hba.conf
```

To allow connections from absolutely any address with password authentication add this line at the end of pg_hba.conf file:

```text title="/etc/postgresql/9.4/main/pg_hba.conf"
host all all 0.0.0.0/0 md5
```

You can also use your network/mask instead just 0.0.0.0/0 .

##### Restart

You have made it! Just make sure to restart your PostgreSQL instance before leaving remote SSH session:

```bash terminal
root@fe35577e9f8b:/# /etc/init.d/postgresql restart
```

Now you should be able to connect to the PostgreSQL instance with any of DB tools.
