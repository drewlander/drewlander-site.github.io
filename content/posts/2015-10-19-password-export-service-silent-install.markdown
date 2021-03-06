---
layout: post
title:  "Active Directory Password Export Service Silent Install"
date:   2015-10-19 8:00:48
categories: active directory password export service automatic silent installation install
---
Password Export Service allows you to export users and passwords from one active
directory domain to another. All the guides I found show how to install it graphically. It took a bit but eventually I was able to install the service
AND import the key created with ADMT all automatically. I used chef but any automation tool will work. It did require some hacking.

You have to edit the password export service yourself via orca, instEd, etc... Something has to alter a datbase table, or else it will ALWAYS prompt and ask if
you who you want to run the service as. You have to edit under InstallExecuteSequence:
<pre>
StartCredentials 0=1
</pre>
Then you can use this msi (or save a transform) and it will no longer prompt as a user to login as.

Run the following
{% highlight bash %}
msiexec /i "c:\password_export_server.msi" /l*v "c:\install.log" /qn REBOOT="ReallySuppress" SENCRYPTIONFILEPATH="c:\key.pes" sKeyPassword="password" sConfPassword="password"
{% endhighlight %}
Though, you really do not need anything after REBOOT. It will NOT import any key at all. Also, you have to reboot anyway to get the service to start properly, otherwise
you will get RPC errors connecting from the destination domain. After you install the service, reboot, then start the Password Export Service. Then run the following
to actually have the key imported:
{% highlight bash %}
"C:\Windows\system32\PesKey.exe C:\key.pes "password"
{% endhighlight %}
Now ADMT _should_ let you migrate the user, and you can do it all automatically. Yay!

