---
layout: post
title:  Get Pianobar Updates On Touchbar With MTMR
date:   2019-01-29 7:35:48
categories: MTMR Pianobar Config Touchbar
---
I found [MTMR](https://github.com/Toxblh/MTMR) and am really liking it so far. However I also enjoy listening to music with [Pianobar](https://6xq.net/pianobar/) but saw there was no love to make the current song playing show in the touchbar. The following is what I did to make it work. Mac OSX 10.14.5 is what I am using, but this can also be made to work on any *nix distro as well. 

My repo of files are located [here](https://github.com/drewlander/configfiles/tree/master/mtmr)

You have to change the config for pianobar to find pianobar-notify.rb. You can do this by adding the following in ~/.config/pianobar/config:

{% highlight bash %}
event_command = /full/path/to/pianobar-notify.rb
{% endhighlight %}

Take note that in the MTMR config (items.json) the weather api is redacted, get your own! Also inside pianobar_notify.rb you need to change the full path to where the file goes.

Other than that, it works for me! Horray!
