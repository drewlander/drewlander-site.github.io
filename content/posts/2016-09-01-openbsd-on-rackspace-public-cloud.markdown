---
layout: post
title:  "Installing OpenBSD On Rackspace Public Cloud"
date:   2016-01-19 05:35:48
categories: openbsd rackspace public cloud linux
---
For the longest time I wanted to see OpenBSD on Rackspace Public Cloud. With their boot.iso you are able to
attempt to boot into a pxe environment and boot OpenBSD 5.7, but it lacks the necessary drivers for xen.
I gave up for a while. but tried with the release of OpenBSD 6. I am happy to report, that you CAN and it WORKS!
To do so, you boot into a linux distro, download bsd.rd, add it to grub, and boot off the RAM disk. Things to note:

* Get ALL the ip information from the box, you will need to setup IPs statically (I just copy /etc/sysconfig/network-settings eth0/eth1 configs to a temp file to read later)
* You will be doing this via a console session, which means you have to use a browser that supports java.

For the purposes of this demonstration, I will be using their Fedora 24 PVHVM image using their General Purpose flavor.

* Boot a Fedora 24 image into the size that you want.
* Login to the server.
* Download the freebsd installer that we will boot from
{% highlight bash %}
wget http://ftp.openbsd.org/pub/OpenBSD/6.0/amd64/bsd.rd 
mv bsd.rd /openbsd.rd
{% endhighlight %}
* Modify /etc/grub.d/40_custom and add:
{% highlight bash %}
  menuentry "Install OpenBSD from RAM disk" {
    set root=(hd0,1)
    kopenbsd /openbsd.rd
  }
{% endhighlight %}
* Edit /etc/default/grub and change GRUB_TIMEOUT=1 to GRUB_TIMEOUT=30
* I remove extlinux and use grub2, because reasons (I like grub2)
{% highlight bash %}
dnf -y remove syslinux-extlinux
rm -f /boot/extlinux.conf
dnf -y install grubby grub2
grub2-mkconfig -o /boot/grub2/grub.cfg
{% endhighlight %}
* Issue the fantastical "reboot" command. It makes the unicorns appear
* Connect via their web interface to a console session of your server. It should be stuck at the grub menu
* Choose "Install OpenBSD from RAM disk"
* At the install screen choose (I)Install
* default keyboard is fine
* set your hostname
* xnf0 will correspond to eth0 settings (public interface). Configure that interface
![OpenBSD network config]({{ site.url }}/assets/openbsd_network.png)
* enter the ip address and netmask. You can enter in your ipv6 info if you feel you can not typo anything (copy/paste does NOT work for me in this virtual console java session)
* choose done (you can configure xnf1 later)
* enter in your default route (gateway)
* enter in your hostname and nameservers
* the rest should be self explanatory. I did not change my console to com0, Timezone I chose US/Easten
* wd0 is the root disk
* I did the auto layout
* for name(s) sets, I did not do anything with X (use - to deselect). Choose done when done
* reboot and enjoy your new system!
![OpenBSD complete]({{ site.url }}/assets/openbsd_complete.png)





