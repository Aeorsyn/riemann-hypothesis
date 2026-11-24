---
title: "Setting Up Laravel Environments"
url: "https://bosnadev.com/2014/12/26/setting-laravel-environments"
author: "Mirza Pašić"
date: "2014-12-26"
topic: "Laravel"
tags: ["configuration", "environments", "nginx"]
summary: "Three ways to set the environment in Laravel 4: by machine host name, by a server variable in Apache or Nginx, and with the .env.local.php files of 4.2."
---

# Setting Up Laravel Environments

> Outdated: Written for Laravel 4.2 in 2014. Current Laravel reads the environment from .env.

Setting up a development environment is an important task you should do before starting any project. We all know that Laravel has a lot of cool features and one of things I like a lot is how easy is to set environments on your machine. Official [documentation](http://laravel.com/docs/4.2/configuration#environment-configuration "Laravel Environments") page provides you a lot of useful information about it, so make sure you check it.

Typically for each project you would need to have development, staging and production environments. All of these should be on separated servers, so configuring each environment can be a bit tricky. But not with Laravel.

### Laravel Environments

Before _L4.1_ release it was possible to set Laravel environments by domain host names. But, because of security concerns, that “feature” was removed and Taylor’s recommendation was that instead of using domain host names we use machine host name.

So, you can configure your environments to something like this:

```php
$env = $app->detectEnvironment(array(
    'local' => array('local-machine-host-name'),
    'staging' => array('staging-machine-host-name'),
    'production' => array('production-machine-host-name')
));
```

 

That’s good and it will work fine. But, problem appears when your application runs multiple environments on the same machine. For example, if you have _**dev.mycoolapp1.com**_ and _**staging.mycoolapp1.com**_ on one machine, application will not be able to use correctly these environments because _**dev**_ and _**staging**_ are on the same machine.

### Server Environment Variables

To solve problem I mentioned above, we utilize server environment variables. For example, if you run your application on Apache2 web server, you set your environment variable like this:

```
# Set local environment
SetEnv LARAVEL_ENV local
```

You can put this code in your **.htaccess** file or append it to Apache2 virtual host. If you changed virtual host file, make sure to restart Apache.

However, if you use Nginx with php-fpm, you need to put this into your _location \~_ part of the config:

```
# Local env
fastcgi_param LARAVEL_ENV local;
```

Now we can update _**detectEnvironment**_ to read this variable as follows:

```php
$env = $app->detectEnvironment(function()
{
    // We set local if LARAVEL_ENV is not set
    return getenv('LARAVEL_ENV') ?: 'local';
});
```

### Dot Env Files

From 4.2 version it’s possible to add your [Laravel Environment](http://laravel.com/docs/4.2/configuration#protecting-sensitive-configuration "Laravel Environment Configuration") variables into .env files. For example, for local environment you need to create _**.env.local.php**_ file within the root directory of your Laravel project. Typical configuration file is much like:

```php title=".env.local.php"
<?php
return [
    'LARAVEL_ENV' => 'local'
];
```

All returned key-value pairs will be available via the **$\_ENV** and **$\_SERVER** PHP [**superglobal**](https://php.net/manual/en/language.variables.superglobals.php "PHP Superglobal variables") variables and you can reference them within your configuration:

```php
$env = $app->detectEnvironment(function()
{
    // We set local if LARAVEL_ENV is not set
    return $_ENV['LARAVEL_ENV'] ?: 'local';
});
```

You can use whatever method you like, depending on your needs. If you are a beginner in the Laravel world, I hope this short tutorial will help you to get started with.

If you have any questions, feel free to post them in the comment section bellow.
