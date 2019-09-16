---
layout: post
title:  "Mailonacircle: Mailinabox(ish) for Centos 7 With Selinux=Enabled"
date:   2015-10-19 8:00:48
categories: centos postfix dovecot dkim spamasassain clamav postfixadmin chef chef-solo selinux postgrey postgrey-milter mariadb
---
Recently I saw the latest mailinabox, and man is it fancy. I looked, and there was nothing like that for centos.
I wanted a way to help the lazy people out there setup a centos 7 box with postfixadmin, dovecot, spamasassin, clamav-milter, postgrey-milter, mariadb, and dkim with ease. So in very short time in between down time, I wrote [mailonacircle](https://github.com/drewlander/mailonacircle). It uses chef-solo, and the attributes are basic and do not make a lot of sense at first glance, but it is a start. With a quick copy/pasta, you can take a fresh centos 7 server and install all of the above, with selinux enabled. By default, there is no default domain (and of course no open relay), do if someone were to just copy/pasta, no damage done. You have to add the domains and mailboxes in postfixadmin. Also smtpauth should work as well.

Reasons I did it this way:

* Why Chef? Why not bash/ansible/fancypants thing:
  * I know Chef very well
  * Because I have a hammer, everything looks like a nail
    * To be fair, chef solo is perfect for this. Run it once, "make this box look like X", and you can remove chef-solo if you wanted. 
* You did not use any community cookbooks! You should feel bad
  * I went for simplicity. Why do I need an entire separate cookbook to install mariadb, create a db and create a few users? 
* Selinux enabled is hard
  * [http://stopdisablingselinux.com/](http://stopdisablingselinux.com/)

This is not as fancy as mailinabox, no fancy web ui, and this assumes at least basic troubleshooting/following directions. What is still left:

* Attributes need to make more sense
* There are some hardcoded references to the mail database (specifically called mail in the sql maps for postfix)
* Would love to solve the  postfixadmin password hash manual setup. 
* Would like to add more tweaks for configuration.
* Need to NOT chmod 777, EVER
* Why did I enabled ports AND unix sockets for milters to listen on? (127.0.0.1, but STILL)
* Clean up recipes (single responsibility)
