---
layout:     post
title:      V for Vagrant
date:       2015-08-02
summary:    If you are a web developer, a operations engineer or a designer then
categories: blogs
---

If you are a web developer, an operations engineer or a designer then ```vagrant``` is your best friend.
You write code that works on your machine but when it goes on production something went wrong.

###Why this shit happens???

Your teammates are using mac osx, windows, linux having different versions of all
dependancies. So there are lots of chances of breaking code on any of the machine. Sometimes your code doesn't work on production may be because you have different version of database than production or may be your local machine have different operating system than production or there may be lots of scenarios. 

### Solution
Vagrant is a very useful tool that helps you to create development environment as same as production. It allows you to work from your favourite operating sytem but runs code in the operating system that is used on production or any other.
It also helps to run instance on AWS or rackspace and can stop after your testing is done in single command.

## Installation
You will need [Virtualbox](https://www.virtualbox.org/){:target="_blank"} and then [Vagrant](http://www.vagrantup.com/downloads){:target="_blank"}.

Now choose the box from [ATLAS](https://atlas.hashicorp.com/boxes/search){:target="_blank"}. This box is nothing but your operating system where you want to run your code. It is JeOS(just enough operation system). It is ~200mb depending on what box you choose.  
Now, from terminal goto your project directory and run

    $ vagrant init ubuntu/trusty64

This will create vagrant file which is configuration file.  
The ```ubuntu/trusty64``` is the box name from ATLAS which is nothing but ubuntu 64 bit.
You can automate a process of installation of all softwares which is required for your project.  
Following is a sample shell script file named as ```setup.sh``` in same directory to install postgresql, redis, git and nginx.

    #!/usr/bin/env bash
    apt-get update
    apt-get install -y postgresql-9.1
    wget http://download.redis.io/releases/redis-2.8.9.tar.gz
    tar xzf redis-2.8.9.tar.gz
    cd redis-2.8.9
    make
    make install
    cd utils
    ./install_server.sh
    apt-get install -y git
    apt-get install -y nginx
    service nginx start


Add path of your shell script and forward port

    Vagrant.configure(2) do |config|
      config.vm.box = "ubuntu/trusty64"
      config.vm.provision :shell, path: "setup.sh"
      config.vm.network :forwarded_port, guest: 80, host: 4567
    ...

And run 

    $ vagrant up --provider virtualbox

It will download OS ( if you are firing this command very first time), boots ubuntu and install dependancies. 

Port forwarding means niginx is running on port 80 inside VM and you can access it on port 4567 on host.

Goto browser and hit ```http://localhost:4567```

It will o/p something like this

![Welcome to nginx!](/images/nginx.png)

Hurrey!. You are done.

Using vagrant you can change files from your host machine using your favourite IDE and that changes will reflect inside VM because vagrant shares your project folder where you have initiated vagrant.  

## Some Useful commands

You can login to your VM and play
    
    $ vagrant ssh

Shutdown VM

    $ vagrant halt

Remove box from machine

    $ vagrant destroy

## References
* [slides](http://www.slideshare.net/sagarjunnarkar/v-for-vagrant){:target="_blank"} on same topic from pune ruby meetup group [PRUG](http://www.punerb.org/){:target="_blank"}.  
* [vagrantup.com](http://www.vagrantup.com){:target="_blank"}

