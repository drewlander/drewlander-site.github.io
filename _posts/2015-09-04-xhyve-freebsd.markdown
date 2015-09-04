---
layout: post
title:  "FreeBSD on Xhyve"
date:   2015-04-18 8:00:48
categories: xhyve freebsd
---
[xhyve](https://github.com/mist64/xhyve) is essentially a port of FreeBSD's [bhyve](http://bhyve.org) to OSX. The instructions
show how to run Ubuntu (with which you can use to boot other linux distro's), but lack any documentation on how to run FreeBSD with it. Here is how to do it:
<h3>Clone and build the latest xhyve</h3>
{% highlight bash %}
git clone git@github.com:mist64/xhyve.git
cd hyve
make
mkdir freebsd
cd freebsd
{% endhighlight %}
<h3> Download latest FreeBSD virtual machine image</h3>
{% highlight bash %}
wget http://ftp10.freebsd.org/pub/FreeBSD/releases/VM-IMAGES/10.2-RELEASE/amd64/Latest/FreeBSD-10.2-RELEASE-amd64.raw.xz
unxz FreeBSD-10.2-RELEASE-amd64.raw.xz
tar xf FreeBSD-10.2-RELEASE-amd64.raw.xz
cd ../
{% endhighlight %}
<h3> Setup the runfreebsd script </h3>
Create a file call runfreebsd with the following contents:
<pre>
#!/bin/sh

# FreeBSD
# this is provided in the xhyve sources
USERBOOT="test/userboot.so"
BOOTVOLUME="freebsd/FreeBSD-10.2-RELEASE-amd64.raw"
KERNELENV=""

MEM="-m 4G"
#SMP="-c 2"
NET="-s 2:0,virtio-net"
IMG_HDD="-s 4,virtio-blk,freebsd/FreeBSD-10.2-RELEASE-amd64.raw"
PCI_DEV="-s 0:0,hostbridge -s 31,lpc"
LPC_DEV="-l com1,stdio"
ACPI="-A"

# FreeBSD
build/xhyve $ACPI $MEM $SMP $PCI_DEV $LPC_DEV $NET $IMG_CD $IMG_HDD $UUID -f fbsd,$USERBOOT,$BOOTVOLUME,"$KERNELENV"
</pre>
<h3>Run the script<h3>
{% highlight bash %}
bash runfreebsd.sh
{% endhighlight %}
Stuck with ~20GB box but hey, its freebsd w/xhve!
