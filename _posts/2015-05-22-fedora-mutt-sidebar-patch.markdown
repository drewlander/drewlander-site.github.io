---
layout: post
title:  "Fedora 22 mutt sidebar patch rpm"
date:   2015-05-22 8:00:48
categories: fedora22 mutt kmod-wl sidebar patch
---

<h3> Fedora 22 mutt with sidebar patch</h3>
So I recently installed fedora 22 on my Macbook pro (Dual booting using GRUB2, no refind). 
I use mutt for my email because I should never even have a chance of having one of those fancy moving pictures
running in my email client. I really like the sidebar patch for mutt, but it is not in the rpmfusion or default repos.

So I built it by downloading the mutt SRPM, editing the spec and adding the patch from [Lunar Linux](http://www.lunar-linux.org/mutt-sidebar/).
I did not do any other modifications to the SRPM apart from adding the patch. Maybe when I get more time I will do so. This means that 
when fedora releases a new version of mutt, it will overwrite the version I have installed. That is fine, it takes a very small time to
download and patch. Maybe when I get more time I will modify the package so it doesn't happen. Anyway, I do have a repo with the rpm.

[http://repo.drewstud.com](http://repo.drewstud.com). Also available via SSL (need to update cert since logjam, it is also self signed).

In this repo is also the module for the broadcom wireless driver (since as of the time of this writing, rpmfusion only has it for kernel version
4.0.2) for Fedora 22. Once rpmfusion gets its non-free repostory back in shape, this will no longer be needed.

Anyway, wanted to share my mutt rpm if anyone else out there wanted to use it.  
