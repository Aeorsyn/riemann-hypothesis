---
title: "A Brief Introduction to Laravel Envoy"
url: "https://bosnadev.com/2015/01/07/brief-introduction-laravel-envoy"
author: "Mirza Pašić"
date: "2015-01-07"
topic: "Laravel"
tags: ["envoy", "deployment", "automation"]
summary: "Installing Laravel Envoy, defining tasks in Envoy.blade.php and running them on remote servers over SSH, with a simple deploy to staging and production."
---

# A Brief Introduction to Laravel Envoy

> Outdated: Written for Laravel 4.2 in 2015.

[Laravel Envoy](http://laravel.com/docs/4.2/ssh#envoy-task-runner "Laravel Envoy Task Runner") provides you a simple and elegant way to run common tasks on your remote servers. If you have ever used [Fabric](http://www.fabfile.org/ "Fabric - SSH Task Runner and Deployment Tool"), [Capistrano](http://capistranorb.com/ "A remote server automation and deployment tool") or other tools for managing remote tasks, you already have an idea how Envoy tasks will look like.

Contents

- 1 Laravel Envoy Installation
- 2 How Envoy Works
- 3 Running Tasks
- 4 Basic deployment

## Laravel Envoy Installation

To install Laravel Envoy simply run:

```bash terminal
composer global require "laravel/envoy=~1.0"
```

after that, make sure \~/.composer/vendor/bin/envoy is flagged as executable:

```bash terminal
chmod +x ~/.composer/vendor/bin/envoy
```

**Note:** Laravel Envoy requires PHP version 5.4 or greater, and **only** runs on Mac and GNU/Linux operating systems.

For easier access to the envoy command, you should create symbolic link to the \~/.composer/vendor/bin/envoy. On GNU/Linux operating systems, symbolic links are created with ln command:

```bash terminal
mirzap@bosnadev:~$ sudo ln -s ~/.composer/vendor/bin/envoy /usr/bin/envoy
```

Alternatively, you can create an alias for the \~/.composer/vendor/bin/envoy command. Depending on which shell you are using, in your \~/.bashrc or \~/.zshrc file put this:

```bash
alias envoy="~/.composer/vendor/bin/envoy"
```

## How Envoy Works

**Note:** Envoy is not Laravel dependent, which means you can use it on any PHP project you want.

To create tasks you use Blade style syntax. Laravel Envoy doesn’t require Blade template engine, it just uses Blade syntax to define tasks. To start, create an Envoy.blade.php in the root folder of your project. Next, create a simple task:

```blade title="Envoy.blade.php"
@servers(['homestead' => '[email protected]'])

@task('list', ['on' => 'homestead'])
    ls -lah
@endtask
```

As you might suppose, in the @servers declaration we configure our server list. Here you can put your staging, testing or production servers. Then within our @task directive we can tell Envoy on what server we want to execute that task. If you want to execute only on one server, you pass only name you defined in the @servers declaration. If you want to execute task across multiple servers, you simply list them in the @task declaration:

```blade title="Envoy.blade.php" diff
-@servers(['homestead' => '[email protected]'])
+@servers(['homestead' => '[email protected]', 'staging' => '[email protected]'])

-@task('list', ['on' => 'homestead'])
+@task('list', ['on' => ['homestead', 'staging']])
     ls -lah
 @endtask
```

## Running Tasks

To run Envoy tasks you simply use run command:

```bash terminal
envoy run list
```

which will execute the task we defined above. Result is:

## ![Laravel Envoy  Run Command](https://media.bosnadev.com/hub/4ca54b23e6a34d0f/2015-01-envoy-guide.webp)Basic deployment

I tend to have very simple script when doing deployment, less code – less chance for something to go wrong. With that in mind let’s create our deployment.sh script in the project root folder:

```bash title="deployment.sh"
#!/bin/bash

function deploy() {
    # make sure we pull master branch for production environment
    BRANCH=$([ $1 == "production" ] && echo "master" || echo "staging")

    echo "Starting deployment on s <$1> environment"
    composer dump-autoload -o

    # Start SSH agent & add identity to the agent
    killall ssh-agent; eval `ssh-agent`
    ssh-add ~/.ssh/private_key

    echo "Pulling $BRANCH branch..."
    git pull gitlab ${BRANCH}
    php artisan migrate --env=$1
    php artisan migrate --bench=bosnadev/some_component --env=$1
    composer dump-autoload -o
}

function help() {
    echo ""
    echo "   Please specify on what environment you want to deploy: "
    echo "   ./deployment.sh env"
    echo ""
}
## If no argument supplied
if [ -z "$1" ]; then
    echo "No arguments supplied"
fi

## If wrong number of argument has been supplied
if [ ! $# == 1 ]; then
    help $#
else
    # We can deploy only on staging and production
    if [ $1 == 'staging' ] || [ $1 == 'production' ]; then
        deploy $1
    fi
fi
```

And my Envoy.blade.php looks like:

```blade title="Envoy.blade.php"
@servers(['homestead' => '[email protected] -p 2222','staging' => '[email protected] -p 22', 'production' => '[email protected] -p 22' ])

@setup
    $param = isset($param) ? $param : null
@endsetup

@task('artisan', ['on' => 'homestead'])
    cd www/someapp
    php artisan {{ $param }}
@endtask

@task('composer', ['on' => 'homestead'])
    cd www/someapp
    composer {{ $param }}
@endtask

@task('deploy-staging', ['on' => 'staging'])
    cd someapp
    ./deployment.sh staging
@endtask

@task('deploy-production', ['on' => 'production'])
    cd someapp
    ./deployment.sh production
@endtask
```

Now, when you want to deploy to the staging just run:

```bash terminal
envoy run deploy-staging
```

Or if you deploying to the production, run deploy-production task:

```bash terminal
envoy run deploy-production
```

As you can see, it’s pretty straight forward. Of course, you can create much more complex deployment tasks, it’s entirely up to you. Feel free to share your knowledge and experience with Laravel Envoy, or share your best practices when dealing with remote tasks.
