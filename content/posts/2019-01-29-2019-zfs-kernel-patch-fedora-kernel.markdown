---
layout: post
title:  ZFS kmod Does Not Build On Fedora Kernel 4.20.4-200
date:   2019-01-29 7:35:48
categories: linux fedora zfs kernel
---
After updating my Fedora box, I noticed that the ZFS kernel module was no longer loaded.
The error (from /var/lib/dkms/zfs/0.7.12/build/make.log) was:

{% highlight bash %}
/var/lib/dkms/zfs/0.7.12/build/include/zpios-ctl.h:186:11: error: implicit declaration of function ‘current_kernel_time’; did you mean ‘current_kernel_time64’? [-Werror=implicit-function-declaration]        
  ts_now = current_kernel_time();                                                                                                                                                                              
           ^~~~~~~~~~~~~~~~~~~                                                                                                                                                                                 
           current_kernel_time64                                                                                                                                                                               
/var/lib/dkms/zfs/0.7.12/build/include/zpios-ctl.h:186:9: error: incompatible types when assigning to type ‘struct timespec’ from type ‘int’                                                                   
  ts_now = current_kernel_time();  
{% endhighlight %}

The easiest fix is to:

Download [zfs-linux_0.6.5.11-1ubuntu3.6.debian.tar.xz ](https://mirrors.edge.kernel.org/ubuntu/pool/main/z/zfs-linux/zfs-linux_0.6.5.11-1ubuntu3.6.debian.tar.xz)
extract and get debian/patches/3204-Add-4.20-timespec-compat-fix.patch

Next you need to patch /var/lib/dkms/zfs/0.7.12/source/include/zpios-ctl.h with that patch
{% highlight bash %}
run
dkms autoinstall
modprobe zfs
{% endhighlight %}
PROFIT!
