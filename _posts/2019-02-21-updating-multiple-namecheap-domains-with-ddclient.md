---
layout: post
title:  "Updating Multiple Namecheap Domains With ddclient"
date:   2019-02-21 18:00:00 -0500
updated: 2020-09-30 18:00:00 -0500
categories: tech
tags: [server management, linux]
comments: true
---
## Introduction

If you are having trouble trying to update multiple namecheap domains from a single ddclient config: you aren't alone, and have no fear, there are solutions.
If you are interested in the complicated history behind this problem, see [this post](updating-multiple-namecheap-domains-with-ddclient-a-history).
Otherwise, the following are to-the-point instructions on how to get it working.

First I'll cover what the config should look like for updating multiple namecheap domains, then follows install instructions to make sure you have a version of ddclient that allows multiple namecheap domain updates.

Credit goes to Robert Ian Hawdon for his [blog post](https://robertianhawdon.me.uk/2010/09/03/making-ddclient-work-with-multiple-domains-on-namecheap/) on this subject.
All I am really doing is providing a 2019 update to his original work.

## The ddclient Config

The config is very similar to what is shown in the [official namecheap docs](https://www.namecheap.com/support/knowledgebase/article.aspx/583/11/how-do-i-configure-ddclient), except for a few details.
If you are totally new to ddclient configuration, you may want to read the [ddclient docs](https://sourceforge.net/p/ddclient/wiki/usage/).

```
#### Global Settings
# How often to update (seconds)
daemon=600
ssl=yes
use=web
web=dynamicdns.park-your-domain.com/getip
protocol=namecheap
server=dynamicdns.park-your-domain.com

#### domain1.com
login=domain1.com
password='dynamicdns-password-for-domain1'
@.domain1.com

#### domain2.com
login=domain2.com
password='dynamicdns-password-for-domain2'
@.domain2.com, www, blog, mail
```

The main difference is that we need to fully specify the root domain using `@.domain1.com` rather than just `@`.
This is because ddclient needs to be able to tell the difference between `@.domain1.com` and `@.domain2.com`.
If they were both specified as just `@`, ddclient would only save the latest config instance of the `@` (resulting in only updating `@.domain2.com`).

The above config results in the following updates:
- `domain1.com`
- `domain2.com`
- `www.domain2.com`
- `blog.domain2.com`
- `mail.domain2.com`

Next we look at two options for installing a version of ddclient that can parse the above config properly.
To check what version of ddclient you may already have installed, use the following:
```
ddclient -help | grep "ddclient version"
```

## Easiest Install Option (3.9.1)
The easiest solution is to simply install ddclient version 3.9.1.
If you need an older version of ddclient, see the next section.

Version 3.9.1 of ddclient was released in January 2020, so your best option is to use my install script until it gets included in later linux distributions.
I've also just noticed that ddclient now has an excellent ["Packaging Status" list on their github page](https://github.com/ddclient/ddclient#distribution-package). This should help you decide if you can just install ddclient from your package manager or use my script.

### Ubuntu/Debian users:

1. Remove your currently installed ddclient (if installed via apt).
```
sudo apt remove ddclient
```

1. Install ddclient 3.9.1 using my script:
```
wget https://raw.githubusercontent.com/icolwell/install_scripts/master/ddclient_install.bash
bash ddclient_install.bash
```

1. Edit the config at `/etc/ddclient/ddclient.conf`.

1. Restart ddclient:
```
sudo service ddclient restart
```

### Other distro users:

If you aren't running Ubuntu, [download ddclient 3.9.1](https://github.com/ddclient/ddclient/archive/v3.9.1.tar.gz) and follow the install instructions in the provided `README.md`.

## Other Install Option (3.9.0)
If you need ddclient 3.9.0, it can be patched to add the parsing support needed for multiple namecheap domains.

### Ubuntu/Debian users:

1. Install ddclient using my install script. It will patch ddclient as required.
```
wget https://raw.githubusercontent.com/icolwell/install_scripts/master/ddclient_install.bash
bash ddclient_install.bash -v 3.9.0
```

2. Edit the config at `/etc/ddclient/ddclient.conf`.

3. Restart ddclient:
```
sudo service ddclient restart
```

### Other distro users:

1. [Download ddclient 3.9.0](https://github.com/ddclient/ddclient/archive/v3.9.0.tar.gz) and follow the install instructions in the provided `README.md`.

2. Download the patch file:
```
wget -O /tmp/ddclient.patch https://raw.githubusercontent.com/icolwell/install_scripts/master/ddclient-3.9.0.patch
```

3. Stop ddclient service using distro service tools.

4. Apply the ddclient patch:
```
cd /usr/sbin
sudo patch --verbose < /tmp/ddclient.patch
```

5. Restart ddclient using distro service tools.

## Related Links
- [Making DDClient work with multiple domains on namecheap](https://robertianhawdon.me.uk/2010/09/03/making-ddclient-work-with-multiple-domains-on-namecheap/) (old patch method)
- [Linux Make ddclient Work with Multiple Namecheap Domains](https://thornelabs.blog/posts/linux-make-ddclient-work-with-multiple-namecheap-domains.html) (old patch method)
- [How do I configure DDClient?](https://www.namecheap.com/support/knowledgebase/article.aspx/583/11/how-do-i-configure-ddclient) (namecheap docs)
- [ddclient distribution packaging](https://github.com/ddclient/ddclient#distribution-package) (List of ddclient versions available on many linux distros)
