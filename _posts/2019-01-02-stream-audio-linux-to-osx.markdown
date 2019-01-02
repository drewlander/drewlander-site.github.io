---
layout: post
title:  Streaming Audio from Linux to OSX with Pulseaudio
date:   2019-01-02 7:35:48
categories: linux fedora osx mac pulseaudio
---
This is not for everyone, and I know that there are better things I can do (like hook up a pair of speakers). Let me explain.

I have a Fedora box that I use as a home personal server. It runs various services. However because the more monitors the better, and 
I believe in having as many HUDS and graphs as humanly possible, it IS running X and has a monitor attached to it. I had some music files
on that server I wanted to play while I was working on my Macbook. I could mount the filesystem from my Macbook and play the file, but what fun is that?
Instead, why can't I just tell my computer where to send the audio to and have it work? Turns out, you can!

Install Pulseaudio on OSX:
{% highlight bash %}
brew install pulseaudio
{% endhighlight %}

I am assuming X is running and some kind of DE is configured on the Linux box. 

Start Pulseaudio on OSX:

{% highlight bash %}
pulseaudio --load=module-native-protocol-tcp --exit-idle-time=-1 --daemon
pactl load-module module-native-protocol-tcp auth-ip-acl=<ip of streaming client>
{% endhighlight %}

Tell Linux where to send the audio:
{% highlight bash %}
PULSE_SERVER=<ip of OSX BOx> vlc <filename>
{% endhighlight %}

PROFIT!