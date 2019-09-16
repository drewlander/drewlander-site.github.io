---
layout: post
title:  "Centos 7 and systemd-nspawn"
date:   2015-04-18 8:00:48
categories: centos systemd nspawn systemd-nspawn centos7 containers
---
With systemd, comes systemd-nspawn. (I will not talk about love/hate of systemd, not in this post at least. Let's just say
if you start out saying you are writing an init system, why does your init system takeover acpi?). Anyway, part of the systemd
monstrosity is something called nspawn. In short, its a "more powerful" chroot, but still should not be relied upon soley for security
(think defense in depth people!). Why would you use this over lxc? If you just want to run something in a chroot, its great for it.
If you need a full container, lxc is probably a better choice. Because I like to try new things, I chose systemd-nspawn to run certain
applications (http, mysql, minecraft). I also did not see many tutorials specifically targeting centos 7 which seems to be using
and older verson of systemd-nspawn/machinectl so some features are missing in those tools.

<h3> Create a basic centos 7 bootstrap</h3>
{% highlight bash %}
yum -y --releasever=7 --nogpg --installroot=/srv/yaystuff install systemd passwd yum fedora-release vim-minimal
{% endhighlight %}
This will install a basic bootstrap centos 7 environment
<h3> Reset the password in the container </h3>
If you are a good person, you are running selinux in enforcing mode. However there is a known bug in centos 7 that will not
let you change the password if you "boot" your container. So currently, you can be fancy and make modules, or be only a slightly
bad person and disable selinux, change the password, re-enable (things work fine after this point with selinux enabled)
{% highlight bash %}
setenforce 0
systemd-nspawn -D /srv/yaystuff
passwd
exit
setenforce 1
{% endhighlight %}
<h3> Boot your container and install your fancypants stuff</h3>
{% highlight bash %}
systemd-nspawn -D /srv/yaystuff/ -b
{% endhighlight %}
This will drop you into a shell, where you can yum install/configure to your hearts desire.

Note: This will also by default make your "container" listen on your network interfaces. So if you have port 80
open in your firewall, and have httpd started and listening in your container (but not on your host), it will be as if you
are serving httpd on your host. 

When you are done installing fancy stuff, I just issue 
{% highlight bash %}
halt
{% endhighlight %}
And it stops the container and takes you out of your chroot. 

You can also bind paths outside of your container into the container's chroot
{% highlight bash %}
systemd-nspawn -D /srv/yaystuff/ -b --bind=/path/on/host:/path/in/container
{% endhighlight %}

<h3>Start your container with systemd </h3>
edit the following file: /etc/systemd/system/yaystuff.service
<pre>
[Unit]
Description=yaystuff container

[Service]
ExecStart=/usr/bin/systemd-nspawn -jbD /srv/yaystuff/ -bind=/path/on/host:/path/in/container
KillMode=process
</pre>
Then reload systemd and start the container

{% highlight bash %}
systemd start yaystuff.service
{% endhighlight %}

Now the version of machinectl on centos 7.0 does not have a login feature. So you have to use nsenter. To use nsenter, you have to get the pid of your container. 
That is logged in /var/log/messages (or if you started your container from the cli)

{% highlight bash %}
grep -A 1 "Spawning namespace container on /srv/yaystuff" /var/log/messages
{% endhighlight %}

Grab the pid there and then execute:
{% highlight bash %}
nsenter -m -u -i -n -p -t $PID
{% endhighlight %}
To leave just type exit. 

Excellent reference: [http://0pointer.de/blog/projects/changing-roots.html](http://0pointer.de/blog/projects/changing-roots.html)

## UPDATE:

Redhat 7.2 (CentOS Linux release 7.2.1511) has a newer version of machinectl that has login. No need to use nsenter anymore.

{% highlight bash %}
machinectl list
MACHINE                          CONTAINER SERVICE         
php-fpm                          container nspawn          
mysql                            container nspawn          
webstuff                         container nspawn          

machinectl login mysql
Connected to container mysql. Press ^] three times within 1s to exit session.

Debian GNU/Linux stretch/sid drewstud.com pts/0

drewstud login: 
{% endhighlight %}
