---
layout: post
date: 2015-01-21 16:40
title: MongoDB in OpenVZ on Ubuntu 14.04 LTS
published: true
---

I went through a process to get MongoDB running on Ubuntu 14.04 LTS running in an OpenVZ container and wanted to share my findings.  Have you seen scary messages like this?
{% highlight bash %}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
** WARNING: You are running in OpenVZ. This is known to be broken!!!
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
{% endhighlight %}

Fixing the Problem
------------------
For me in fact, it was broken.  I had installed the version of mongodb-server which comes by default with Ubuntu 14.04.01 LTS which at the time of writing (January 2015) was 2.4.9.  Turns out this version still has problems with OpenVZ, but you can thankfully fix that by using the official MongoDB repositories.  They have even nicely packaged these for Ubuntu.  Before we setup those, first uninstall the existing MongoDB.

{% highlight bash %}
sudo apt-get remove mongodb-server mongodb-clients
{% endhighlight %}

Now, we can follow [this nicely laid out guide][1] from the Mongo team in just a few steps.  Using that guide, go ahead and get the right version of MongoDB installed that you need.





[1]: <http://docs.mongodb.org/manual/tutorial/install-mongodb-on-ubuntu/#install-mongodb>


