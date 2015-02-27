---
layout: post
title:  "Tuning Spamassassin"
date:   2015-02-13 7:35:48
categories: postfix fedora spam spamassassin
---
For those of us who are not large companies where paying another company for spam protection is an option, combatting spam
can be something both fun, and annoying. Hopefully, for any newbies, here are some things
to do that can greatly reduce the amount of spam you receive, as well as minimize
the amount of false positives of mail identified as spam. 

<h3> Have spamassassin learn ham and spam </h3>
Many people I have seen just install spamassassin, run sa-update, then complain 
that they are getting a ton of spam. Spamassassin needs tuned, and tuning is quite
easy. The quick and easy way is with sa-learn.
{% highlight bash %}
sa-learn --ham --username=vscan  --nosync /path/to/maildir/directory/cur
sa-learn --sync
sa-learn --spam --nosync --username=vscan /path/to/maildir/directory/.Junk/cur
sa-learn --sync
{% endhighlight %}
You MUST train both ham and spam to be effective.  I have to use the vscan user, as I run amavis. sa-learn will add the bayesian 
tokens to the datbase of the user executing spamassassin. 

A nice thing to do is to report mail that you have found to be spam. If you have installed the Razmor moudle, it can be reported to 
Vipul's Razor. Assuming in a folder that is full of spam:
{% highlight bash %}
for i in `ls`; do spamassassin -r < $i; done
{% endhighlight %}
[This](http://wiki.apache.org/spamassassin/report_spam.pl) perl script can be run as a cron job. Just point it to a folder that contains only spam.


Ensuring you are doing sa-update frequently, as well as the above, will help spamassassin properly identify spam/ham. 
I am personally just tagging the subject, and putting anything with that tag in the subject into a folder that is
just spam, and running the above commands. This, in addition to running postscreen and a few select rbls, has slowed any spam making it
to my inbox to a trickle. 

Excellent reference: [http://www.spamtips.org/p/ultimate-setup-guide.html](http://www.spamtips.org/p/ultimate-setup-guide.html)

