---
title: "Deploy your Laravel 4 application on Heroku"
url: "https://bosnadev.com/2014/09/16/deploy-your-laravel-4-application-on-heroku"
author: "Mirza Pašić"
date: "2014-09-16"
topic: "Laravel"
tags: ["heroku", "deployment", "paas"]
summary: "Deploying a Laravel 4 application to Heroku with git, the Heroku Toolbelt and a one-line Procfile that serves the public/ folder with Apache."
---

# Deploy your Laravel 4 application on Heroku

> Outdated: Written for Laravel 4 and the Heroku Toolbelt in 2014.

Heroku is a popular PaaS that allows you to deploy your application’s in no time. But, learning how to do that can be confusing, especially for PHP developers. In this tutorial I’ll cover basic steps which will help you to deploy your Laravel 4 application on Heroku. First let’s download Laravel:

```bash terminal
composer create-project laravel/laravel --prefer-dist laravel
```

After that, enter in laravel directory:

```bash terminal
cd laravel
```

and initialize new git repository:

```bash terminal
git init
```

Now, create **.gitignore** file and put everything you want to ignore from git tracking. You can use Laravel’s default **.gitignore** file, just remove composer.lock from it:

```text title=".gitignore"
/bootstrap/compiled.php
/vendor
composer.phar
.env.*.php
.env.php
.DS_Store
Thumbs.db
/.idea
```

And make a commit:

```bash terminal
git add -A
git commit -m 'Initial commit'
```

I assume that you have installed the [Heroku Toolbelt](https://devcenter.heroku.com/articles/heroku-cli "Heroku Toolbelt - Instsallation Instructions"), if you don’t have Heroku Toolbelt installed then just look at the install instructions and then continue with this tutorial.

Now, you need to login:

```bash terminal
heroku login
```

you need to enter your credentials and we’re ready to go. Let’s create our app:

```bash terminal
heroku create shop-up --region eu --stack cedar --buildpack https://github.com/heroku/heroku-buildpack-php
```

heroku create command is used to create application called shop-up, with [**region** parameter](https://devcenter.heroku.com/articles/regions "Heroku | Regions") you can select region, [**stack** parameter](https://devcenter.heroku.com/articles/stack "Heroku | The Celadon Cedar Stack") allows you to select on which stack you want to deploy your application and finally with [**buildpack** parameter](https://devcenter.heroku.com/articles/buildpacks "Heroku | Buildpacks") we can specify which buildpack we want to use for our application. In this case I used Heroku’s default PHP buildpack, and you can use whatever you want.

After executing this command we get following message:

```text
Creating shop-up... done, region is eu
BUILDPACK_URL=https://github.com/heroku/heroku-buildpack-php
http://shop-up.herokuapp.com/ | git@heroku.com:shop-up.git
Git remote heroku added
```

Now, you just need to create Heroku Procfile. You need to tell Apache2 web server that document root of your application is public/. Procfile should be placed in the root folder of your application. We just need to add one line to it:

```text title="Procfile"
web: vendor/bin/heroku-php-apache2 public
```

Commmit changes and push you application to heroku repository:

```bash terminal
git add Procfile
git commit -m 'added Procfile'
git push heroku master
```

After minute or two, you should get a message that tells you that your application is now deployed to Heroku:

```text
       Generating optimized autoload files
       Generating optimized class loader
-----> Preparing runtime environment...
-----> Discovering process types
       Procfile declares types -> web

-----> Compressing... done, 70.2MB
-----> Launching... done, v11
       http://shop-up.herokuapp.com/ deployed to Heroku
```

And that’s it. Here is list of some useful commands that you may need:

```bash terminal
heroku run php artisan <some-artisan-command>
heroku run composer <some-composer-command>
heroku apps # List of your Heroku applications
heroku config --app <your-app> # List of your config/environment variables for specified application
heroku addons --app <your-app> # List of add-ons you have enabled on your application
```

In the next tutorial I’ll show you how you can add and use PostgreSQL database on your Heroku instance. Stay tuned.
