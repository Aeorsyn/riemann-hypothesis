---
title: "Install Elasticsearch on Laravel Homestead"
url: "https://bosnadev.com/2014/09/12/install-elasticsearch-on-laravel-homestead"
author: "Mirza Pašić"
date: "2014-09-12"
topic: "Laravel"
tags: ["elasticsearch", "homestead", "vagrant", "search"]
summary: "Installing Elasticsearch 2.1 inside the Laravel Homestead Vagrant box with apt, forwarding its port and checking from the host that it responds."
---

# Install Elasticsearch on Laravel Homestead

> Outdated: Written for Elasticsearch 2.1 and the Laravel Homestead box of the time.

Recently I started working on a Laravel project that requires integration with [Elasticsearch](http://www.elasticsearch.org/ "Elasticsearch") search engine. I didn’t want to change super cool development environment that Taylor (creator of Laravel) so kindly provided to us, so I installed Elasticsearch on [Homestead](http://laravel.com/docs/5.1/homestead "Laravel Homestead") instead of creating a new Vagrant box specially for this project. Here is a little snippet that will help you if you want to use Elasticsearch in you Laravel application.

```bash terminal
# Login as root
root@homestead:~# sudo -s

# Update Aptitude
root@homestead:~# apt-get update

# Install Java
root@homestead:~# apt-get install openjdk-7-jre-headless -y

# Download and install Elasticsearch Public Signing Key
root@homestead:~# wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

# Add repository
root@homestead:~# echo "deb http://packages.elastic.co/elasticsearch/2.x/debian stable main" | sudo tee -a /etc/apt/sources.list.d/elasticsearch-2.x.list

# Update Aptitude
root@homestead:~# apt-get update

# Install Elasticsearch
root@homestead:~# apt-get install elasticsearch

# Set Elasticsearch to run on startup
root@homestead:~# update-rc.d elasticsearch defaults 95 10

# Start Elasticsearch server
root@homestead:~# /etc/init.d/elasticsearch start
```

 

After that you can configure port forwarding, edit your homestead.rb file and add following line just below other port forwards:

```ruby title="homestead.rb"
config.vm.network "forwarded_port", guest: 9200, host: 62000
```

You need to reload vagrant box:

```bash terminal
vagrant reload
```

Now, on your host, try to make curl request:

```bash terminal
curl -X GET localhost:62000
```

and you should have the following response:

```json
{
  "name" : "Penance",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "2.1.0",
    "build_hash" : "72cd1f1a3eee09505e036106146dc1949dc5dc87",
    "build_timestamp" : "2015-11-18T22:40:03Z",
    "build_snapshot" : false,
    "lucene_version" : "5.3.1"
  },
  "tagline" : "You Know, for Search"
}
```

That’s it, now when we have installed Elasticsearch on Laravel Homestead, in one of the next tutorials we will see how to index and search data.

 

**Update (04.12.2015.)** :\
Tutorial has been updated to show installation process of the latest stable release of Elesticsearch at this time (2.1.0).
