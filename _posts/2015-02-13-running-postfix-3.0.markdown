---
layout: post
title:  "Running postfix 3.0"
date:   2015-02-13 7:35:48
categories: postfix fedora new
---
Postfix 3.0 stable was released a few days ago (Feb 8). I run a few personal mailservers and decided it would be good if I were to upgrade the one running on Fedora21 (postfix + mysql + spamass-milter + greylisting), This would be a deviation as I would be compling it from source, and who knows what would break with 3.0! Here is a very brief update to what I did to get it working on Fedora 21:

{% highlight bash %}
yum remove postfix -y
wget ftp://postfix.mirrors.pair.com/postfix-release/official/postfix-3.0.0.tar.gz
tar -xzvf postfix-3.0.0.tar.gz
cd postfix-3.0.0
{% endhighlight %}
I put the following in a file so I could keep building it until I got all the right libs/deps
{% highlight bash %}
make tidy
#This builds postfix with mysql+tls+sasl support
make -f Makefile.init makefiles 'CCARGS=-DHAS_MYSQL -I/usr/include/mysql  -DUSE_SASL_AUTH -DDEF_SERVER_SASL_TYPE=\"dovecot\" -DUSE_TLS `pkg-config --cflags openssl`'   'AUXLIBS_MYSQL=-L/usr/lib64/mysql -lmysqlclient -lz -lm' 'AUXLIBS=-L/usr/lib64 -lssl -lcrypto'  shared=yes dynamicmaps=yes
make
make install
{% endhighlight %}
For the install part, I used the default paths given.
{% highlight bash %}
/usr/sbin/postfix start
{% endhighlight %}

Do not forget when testing with telnet (if using postscreen) to turn on crlf
{% highlight bash %}
telnet your.mailserver.com 25
Trying 127.0.0.1...
Connected to your.mailserver.com.
Escape character is '^]'.
220 your.mailserver.com ESMTP Postfix
^]
telnet> set crlf
Will send carriage returns as telnet <CR><LF>.
{% endhighlight %}

For fedora 21, (I do not recall off the top of my head) but these are the packages you need to build with mysql+tls:
{% highlight bash %}
mariadb-devel
openssl-devel
{% endhighlight %}
The nice thing too is that when I did a yum -y remove postfix, it left my main.cf intact, just saved it as main.cf.rpmsave. So before starting postfix don't foget to do:
{% highlight bash %}
cp /etc/postfix/main.cf.rpmsave /etc/postfix/main.cf
{% endhighlight %}
Also, I left backwards compatability on with postfix, have not tried turning it off. When I do, I will take note of what I had and what needed to be changed.

No idea how to do comments yet, so email [drewski@drewstud.com](mailto:drewski@drewstud.com) with comments/questions
