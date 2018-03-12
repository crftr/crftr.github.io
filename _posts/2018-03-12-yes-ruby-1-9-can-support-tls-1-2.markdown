---
layout: post
title: Yes, Ruby 1.9 can support TLSv1.2!
modified:
categories:
excerpt:
tags: [tutorial, ruby, tls]
comments: true
date: 2018-03-12T10:09:55-07:00
---

Did you receive an email indicating that a critical service is deprecating support for TLSv1.0 or TLSv1.1? Even worse, is your app running ruby 1.9?

All hope is not lost! Let's get you on TLSv1.2!

# Background

Ruby 1.9.0 was first released on Christmas of 2007. Support for the 1.9 branch ended on February 23, 2015. *It had a good run.*

The 1.9 branch supported SSLv2, SSLv2.3, SSLv3 and TLSv1. *Perfectly acceptable for the time.* But 1.9 apps still exist, and the world of services around them are changing for PCI compliance and other valid security concerns.


# Is your OpenSSL version, adequate?

{% highlight bash %}
#!/bin/sh

openssl version
{% endhighlight %}

If the system is running an OpenSSL version >= `1.0.1` there is no immediate need to upgrade. **[Skip to the ruby section](#rebuild-ruby)**

If the system is running an OpenSSL version < `1.0.1` then you need to upgrade OpenSSL to support TLSv1.2

## Upgrade OpenSSL

You now know that you must upgrade OpenSSL. *I take zero responsibility for knowing your environment, so please investigate your appropriate procedures.* Here are a few options for popular systems:

{% highlight bash %}
#!/bin/sh

# Ubuntu 12.04 and newer
sudo apt-get update
sudo apt-get install --only-upgrade openssl libssl-dev

# RedHat Enterprise v6 or CentOS
sudo yum update openssl libcurl

# OS X (using homebrew)
brew update
brew install openssl

{% endhighlight %}




# Rebuild ruby

I currently enjoy using [rbenv][3] to manage my local and server ruby installations. Check to ensure that your version of rbenv can install your desired ruby version

{% highlight bash %}
#!/bin/sh

rbenv install -l
{% endhighlight %}

**Are you ready for the big show?!** The following assumes that we're attempting to build ruby version 1.9.2-p180.

{% highlight bash %}
#!/bin/sh

# rbenv
rbenv install --force --patch 1.9.2-p180 < <(curl -sSL https://gist.githubusercontent.com/crftr/7a8f8873fcb1469c1d02c572e00a13a9/raw/ee187756fbc855ec7447d1a476515a2c443260d1/openssl_tls_1.2.patch)
rbenv rehash

# for users of RVM
rvm install 1.9.2-p180 --patch https://gist.githubusercontent.com/crftr/7a8f8873fcb1469c1d02c572e00a13a9/raw/ee187756fbc855ec7447d1a476515a2c443260d1/openssl_tls_1.2.patch

{% endhighlight %}



# Let's test!

The following [curl][1] command will return HTML and JavaScript if and only if the server can communicate over TLSv1.2

{% highlight bash %}
#!/bin/sh

curl --tlsv1.2 https://www.google.com
{% endhighlight %}

For our ruby test we will use [HTTParty][2]. It's tried and true. If it's available try the following in a ruby REPL of your choice

{% highlight ruby %}
# IRB, the Rails console, or whatever is appropriate for your app

require 'httparty'

HTTParty.get(
  'https://www.google.com',
  ssl_version: :TLSv1_2,
  debug_output: $stdout
)
{% endhighlight %}


# Final notes

Let's be honest, running ruby 1.9 is risky. It is in your best interest to upgrade as soon as possible. But legacy apps exist in the wild. I hope that all critical applications eventually receive the attention and upgrades they deserve -- but only do so after adequately planning the procedure. There's no need to rush through an upgrade for the TLS support.

Happy hacking!


[1]: https://curl.haxx.se
[2]: https://github.com/jnunemaker/httparty
[3]: https://github.com/rbenv/rbenv