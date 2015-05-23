---
layout: post
title:  "Modular Project Environments With Chroot"
date:   2015-05-21 22:05:44
categories: projects
permalink: /modular-project-environments-with-chroot/
excerpt: Work conveniently on multiple projects by isolating project environments within chroot jails
---

Wouldn't it be nice to have all your project files and dependencies in one neat little package?

I'm not talking about just your source files and repositories, I'm talking about /etc and the zillion packages that you installed but never bothered to track. Yes, all those stupid things, like a folder you created somewhere, or a small patch you made on init.d scripts to properly start or shutdown a service. 

I'm talking about all of that, and a couple months back, I stumbled across a simple, but great solution for this.

Enter humble, often overlooked command `chroot`. What chroot lets you do is treat a folder in your system like its the root filesystem. When you do `chroot /myroot`, chroot drops you into a shell in which `/` is actually `/myroot`, and you can do anything within the chroot from creating users to installing packages to changing configs, pretty much anything, and it will all stay within the `/myroot` folder, isolated from the rest of the operating system.

What chroot lets you do is create a fully isolated root filesystem which houses everything your project needs from odd files and folders, to users and packages, all neatly packed into one folder which you can backup and restore in minutes, or use to switch between machines at work, or machines between work and home too.

Here's how I use chroot to isolate my django projects. I'm going to be assuming a Virtualbox debian environment for development, though the same procedure can be used with physical disks like pend drives or USB hard disks.

Step 1: Server setup
====================

- Create two virtual disks
    - base.vdi for the base operating system, in my case, debian.
    - myproject.vdi for the project chroot

- Install base operating system OR add myproject.vdi to an existing system

- Install debootstrap 

{% highlight bash %}sudo apt-get install binutils debootstrap{% endhighlight %}

- Make filesystem on myproject.vdi disk (/dev/sdb in my case)

{% highlight bash %}mkfs.ext4 /dev/sdb{% endhighlight %}

- Mount myproject.vdi

{% highlight bash %}mount /dev/sdb /myroot{% endhighlight %}

- create a base chroot environment with debootstrap

{% highlight bash %}debootstrap jessie /myroot http://http.debian.net/debian{% endhighlight %}

- Make a setup folder

{% highlight bash %}
mkdir /myroot/setup
mkdir /myroot/setup/to_root_os/
{% endhighlight %}


Step 2: Create scripts to setup chroot and manage services within it
====================================================================

- Create a setup script to setup the chroot `vi /myroot/setup/to_root_os/setup_chroot`

{% highlight bash %}
#!/bin/bash

if mount | grep /myroot ; then
	echo "! chroot already setup"
else
	mount /dev/sdb /myroot
	mount --bind /proc /myroot/proc
	mount --bind /sys /myroot/sys
	mount --bind /dev/pts /myroot/dev/pts
fi

chroot /myroot && /setup/start_services
{% endhighlight %}

- Setup an ssh server for the chroot environment

{% highlight bash %}
chroot /myroot
useradd myuser
passwd myuser
apt-get install dropbear
{% endhighlight %}

- Install services you want within the chroot `/myroot`

{% highlight bash %}
chroot /myroot
apt-get install myfavoritedatabase
apt-get install myfavoritemessagequeue
{% endhighlight %}

- Create a script to start services within the /myroot chroot `vi /myroot/setup/start_services`

{% highlight bash %}
#!/bin/bash

service dropbear start
service myfavoritedatabase start
service myfavoritemessagequeue start
{% endhighlight %}

- Create a script to shutdown services within /myroot chroot `vi /myroot/setup/stop_services`

{% highlight bash %}
#!/bin/bash

service myfavoritemessagequeue stop
service myfavoritedatabase stop
service dropbear stop
{% endhighlight %}

- Create a script to remove the chroot project disk safely `vi /myroot/setup/to_root_os/remove_chroot`

{% highlight bash %}
#!/bin/bash

chroot /myroot && /setup/stop_services

umount /myroot/proc
umount /myroot/sys
umount /myroot/dev/pts
umount /myroot

{% endhighlight %}

Step 3: Setup chroot setup during boot (optional)
=================================================

- copy `to_root_os` scripts to the base root filesystem(NOT /myroot)

{% highlight bash %}
mkdir -p /etc/myroot/
cp -R /myroot/setup/to_root_fs/* /etc/myroot/
{% endhighlight %}

- test chroot scripts

{% highlight bash %}
/etc/myroot/remove_chroot
/etc/myroot/setup_chroot
{% endhighlight %}

- add chroot setup scripts to startup scripts in rc.local

- symlink /etc/myroot/remove_chroot to /etc/rc0.d/S00remove_chroot to remove chroot on shutdown.

Step 4: Login to your chroot via SSH and setup Django
=====================================================

At this point we're all set to start working on django or any project we need to. 

Just `ssh` into the chroot and start working!

`ssh myuser@host`

I currently still use `virtualenv` within the chroot though it is strictly not necessary.

I tend to work on multiple projects so what i do is create vdi's for each project while maintaining
the same base os, so to work on different projects, I simply change the virtual machine setup and switch
vdi's which is pretty nifty and space-saving.

I only keep around 4 GB of space for the project vdi's, it helps me keep things optimized while not putting too much constraints on stuff I'd like to install, which also means that in production environments, the DB needs to be stored in another instance or disk.

All in all, I'm extremely happy with this setup. Its a nice feeling to know that everything you need for a project is in one folder which you can backup, forget about and restore in seconds.





