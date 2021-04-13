---
layout: post
title:  "Dual boot Windows 10 and FreeBSD with ZFS"
date:   2020-04-13 06:00:00
categories: FreeBSD Windows ZFS
---

I followed the guide located [here](https://forums.freebsd.org/threads/uefi-gpt-dual-boot-how-to-install-freebsd-with-zfs-alongside-another-os-sharing-the-same-disk.75734/#post-493993)

I am currently dual booting FreeBSD 13.RC5 (13-RELEASE should be out extremely soon as of this post) and Windows 10. I have had no issues.

But can you watch DRM enabled content like Netflix? You bet you can! Download and run [https://github.com/mrclksr/linux-browser-installer](https://github.com/mrclksr/linux-browser-installer). Works like a charm. It actually loads faster than native Firefox. 

In addition, I have bhyve with an Ubuntu 20.04 guest. I am using the tool vm to manage virtual machines. I will make that another post.

All in all, I can say I am quite pleased. My laptop has the dual graphics, and if I previously did have it using the Nvidia graphics card, but I just have it using the built in Intel graphics and it works great. Most of the games I like are pre-2010 so the card handles it great!

Hardware of note:

* Intel Wireless 3165
* GP107M [GeForce GTX 1050 Mobile]
* Intel HD Graphics 630

The one regret is that it [does NOT support](https://wiki.freebsd.org/dev/ath(4)) my ALFA AR9271. Maybe one day. 