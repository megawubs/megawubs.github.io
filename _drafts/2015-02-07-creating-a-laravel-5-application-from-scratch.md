---
title: Creating A Laravel 5 Application From Scratch
layout: post
author: MegaWubs
categories: laravel
---

Laravel 5 just came out and I'm going to create a application with it from scratch and document my process
here. For simplicity I'm going to give the project a code name of Andrew.

## Homestead
Homestead is the default virtual machine that's specifically build for Laravel and other PHP projects. It let's you 
have one central virtual machine that serves all of your local projects. To get started with Homestead you have to 
install [Virtualbox][virtualbox] and [Vagrant][vagrant] because, after all, Homestead is just a Vagrant box. Follow 
the instructions on the site and get back here when you're done installing them.

Before we move onto installing and setting up Homestead, we need to add the Homestead Vagrant box to Vagrant. This 
might be a bit complicated to understand, with the homestead CLI (Command Line Interface) having the same name as the 
box itself. But I'll explain it like this, the Vagrant box called Homestead is a pre configured virtual machine that 
gets managed by the Homestead CLI, so they are two completely different things.

To add the Homestead virtual box to Vagrant, type the following in your terminal:
{% highlight bash %}
$ vagrant box add laravel/homestead
{% endhighlight bash %}

### Composer
To be able to install the Homestead CLI you need to install [Composer][composer]. Composer is the dependency 
manager for PHP. To install Composer globally paste the following in your terminal:

{% highlight bash %}
$ curl -sS https://getcomposer.org/installer | php
$ mv composer.phar /usr/local/bin/composer
{% endhighlight bash %}

Which you can install the Homestead CLI for managing your Homestead virtual box with the 
following command:

{% highlight bash %}
$ composer global require "laravel/homestead=~2.0"
{% endhighlight bash %}


And when it's done installing be sure to add `~/.composer/vendor/bin` to your path! Otherwise the Homestead CLI won't
be usable. You can do this by either editing your `~/.bashrc` or `~/.zshrc` file and add the following line to it:

{% highlight text %}
PATH=$PATH:$HOME/.composer/vendor/bin
{% endhighlight text %}
Save the file and restart your terminal.

### Homestead configuration

Now you can initialize Homestead by typing:
{% highlight bash %}
$ homestead init
{% endhighlight bash %}

This command will create the `Homestead.yaml` file in `~/.homestead`. You can edit it by typing:
 
{% highlight bash %}
$ homestead edit
{% endhighlight bash %}
 
A preview of the file looks like this:
 
{% highlight yaml %}
---
ip: "192.168.10.10"
memory: 2048
cpus: 1
 
authorize: ~/.ssh/id_rsa.pub
 
keys:
    - ~/.ssh/id_rsa
 
folders:
     - map: /Users/megawubs/Code
       to: /home/vagrant/Code
 
sites:
     - map: andrew.app
       to: /home/vagrant/Code/andrew/public
 
databases:
     - homestead
 
variables:
     - key: APP_ENV
       value: local
       
{% endhighlight yaml %}

This file lets you configure your Homestead virtual machine. The first three lines are obvious, just up the 
memory or cpu's when you think you need it. The authorize and key parts are for authentication with the virtual 
machine, if you've made an rsa key once before you probably won't have to do anything here. If not, and you are on a 
Mac or Linux computer you can do it with the following command:

{% highlight bash %}
$ ssh-keygen -t rsa -C "your@email.com"
{% endhighlight bash %}

What's more interesting in the yaml file are the `folders` and `sites` configuration. Under `folders` you can set the
the `map` key to a local folder on the host machine that's mapped to a folder on the virtual/guest machine. I recommend
 to 
chose the folder you place all of your laravel sites in. For me that would be `~/projects/PHP/laravel`. The path 
you provide by `to` will be the path where your code is available inside the virtual machine. This is directly used 
under the `sites` key. Here you provide a name for a site (like `andrew.app`) and under `to` you set the path to the 
public folder of the site inside the virtual machine. So, in the example above it'll be 
`/home/vagrant/Code/andrew/public` and not `/Users/megawubs/Code/andrew/public`, because the last path is on my host 
machine and the `to` path needs to be on the guest machine. Otherwise, Nginx won't be able to find the folder!

When you are done editing your Homestead.yaml file, save it and go back to your terminal. We can now proceed with the 
next step.


### Site configuration
To be able to access your site(s) though your browser, you need to tell the host machine that the url `http://andrew
.app` should be routed to `192.168.10.10` (the ip address of the virtual machine). You can do this by editing your
`/etc/hosts` file and add the following line to it:

{% highlight text %}
192.168.10.10   andrew.app
{% endhighlight text %}

When you add more sites to the `sites` list in the Homestead.yaml file, you also need to add it to your `/etc/hosts` 
file, otherwise Ngnix won't know whitch site to load based on the requested url.

## Laravel CLI

Because we already installed Composer, this step is going to be a lot quicker. First navigate to the folder you want 
to create the new project in (in this example, it would be `/Users/megawubs/Code/`).
To install the Laravel CLI and create your first Laravel project, you only have to execute two commands in the terminal.
 
 {% highlight bash %}
 $ composer global require "laravel/installer=~1.1"
 {% endhighlight bash %}

This will install the laravel installer globally on your computer. This enables you to do:

{% highlight bash %}
$ laravel new andrew
{% endhighlight bash %}

And have a working Laravel installation inside the `andrew` folder within seconds. I recommend this way of installing 
Laravel, as it's much faster than doing `composer create-project laravel/laravel --prefer-dist`.

To view the project in your browser, simply type the following in your terminal:
 
{% highlight bash %}
$ homestead up
{% endhighlight bash %}
 
The first time it'll take a bit longer, because it has to copy over the laravel/homestead virtual machine and 
preform a view actions on it. But when it's done, you should be able to browse to `http://andrew.app` and see the 
Laravel 5 welcom screen!

If something is wrong, you can always destroy the machine with the following command:
 
{% highlight bash %}
$ homestead destroy
{% endhighlight bash %}
 


## Git

The code for this project is going to be managed using Git. To install git for your platform, go to 
[this site][git-link] and follow the instructions. Once you are done installing Git, go to the folder you've just 
installed Laravel 
in and do the following commands:

{% highlight bash %} 
$ git init 
$ git add .
$ git commit -am "Initial Commit - project Andrew"
$ git remote add origin [your git origin url here]
$ git push
{% endhighlight bash %}

## Hosting

For this project I'm going to use an already existing droplet on DigitalOcean and add a new VirtualHost to it. The 
folder structure I use for multiple sites on my server is like this:

{% highlight text %}
home
└── andrew
    └── site
        └── public
     
 {% endhighlight text %}
 
 And inside `/var/www/` I put a symbolic link to the public folder inside the users site directory. The symbolic link
is made with the following command
  
  {% highlight bash %} 
  $ ln -l /home/andrew/site/public /var/www/andrew.com
  {% endhighlight bash %}

I use apache as my web-server, to make sure the right site is loaded when someone enters andrew.com in his browser we
 have to create a VirtualHost. To do this, copy your default site configuration and give it a name like `andrew.com
 .conf` like this: 
 
{% highlight bash %} 
   $ sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/andrew.com.conf
{% endhighlight bash %}
 
This will create a copy of the default configuration. The content of `andrew.com.conf` will look something like this:
 
{% highlight text %}
  <VirtualHost *:80>
      ServerAdmin webmaster@localhost
      DocumentRoot /var/www/html
      ErrorLog ${APACHE_LOG_DIR}/error.log
      CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
{% endhighlight text %} 

First, change the `DocumentRoot` to `/var/www/andrew.com` and add the two directives `ServerName` and `ServerAlias`. 
The fist is our domain `andrew.com` and the second is for defying names that should match as if they were the 
ServerName. Useful for when you want to be able to also catch `www.andrew.com`. 
With he two directives, the file becomes:
{% highlight text %}
 <VirtualHost *:80>
      ServerAdmin webmaster@localhost
      ServerName andrew.com
      ServerAlias www.andrew.com
      DocumentRoot /var/www/html
      ErrorLog ${APACHE_LOG_DIR}/error.log
      CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
{% endhighlight text %} 

We can let apache know that the site should be enabled with the following commands:

{% highlight bash %} 
$ sudo a2ensite andrew.com.conf
$ sudo service apache2 restart
{% endhighlight bash %}

Next, we are going to create a file to quickly test if our setup works. Do the following inside the directory 
`/home/andrew/site/public`

{% highlight bash %} 
$ echo "Hello World" >> index.html
{% endhighlight bash %}

Andrew.com is not a domain I own, but I can test if the setup I have created works by editing my `/etc/hosts` file and 
add the following line

{% highlight text %}
[ip-of-server] andrew.com
{% endhighlight text %} 

after saving the file, going to `www.andrew.com` in your browser should give you a "Hello World!" message!


## PHPStorm Setup

Install IDE helper, Add Homestead as a remote executableTypeTableSeeder


[vagrant]: https://www.vagrantup.com/
[virtualbox]: https://www.virtualbox.org/
[composer]: https://getcomposer.org/
[git-link]: http://git-scm.com/
