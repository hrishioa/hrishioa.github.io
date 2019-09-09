---
layout: post
title: "Automating Git Commits"
date: 2016-03-11 20:39:37
image: 'gitauto/banner.png'
share_image: 'gitauto/banner.png'
description: "Quick look at the why and the how" 
tags: 
- Git
- Python
categories:
- Hacks of Life
twitter_text:
---

### Why?

There are any number of tools available for the developer that wishes to check up on his project while he's away. I've made use of a number of these tools on occasion, but recently I found myself wanting something more.

The project I wanted to monitor and keep track of was on a [Beaglebone Black](https://beagleboard.org/black), and from experience I knew that the particular device I was using wasn't the most reliable for data fidelity. I needed a way to store the current state of the project on the cloud. 
It's worth noting that Git wasn't my first thought, as Dropbox or any other cloud hosting tool can easily provide this service. But what if you needed version control as well? Even on a binary blob that's being constantly updated, it was useful for me to have rollback - especially when it's time based rollback. What if I could move back four commits and be forty minutes into the past? Now that's something. (Also I was running out of space on my Dropbox)

### How

To start, we need to get rid of authentication when making git commits. In order to do so, we need to first generate an ssh-key. On the host-machine, type:

{% highlight BASH %}
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
{% endhighlight %}

(It is worth nothing that the recommended procedure is slightly different for Github and BitBucket. The one we're following is for Github, bit it should work for Bitbucket without any problems)

The key generation process will ask you for a passphrase. Make sure to leave the field blank (entering a passphrase is more secure, but will make our attempts at circumventing any user input moot) and the key will generated in `~/.ssh/id_rsa.pub`.

Once this is done, you can add the key to your [Github](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/) or [BitBucket](https://confluence.atlassian.com/bitbucket/add-an-ssh-key-to-an-account-302811853.html) account.

Try committing and pushing at this point. If there is a problem, it is likely that you are using `https` instead of `ssh`. Replace your remote url with the following command:

{% highlight BASH %}
git remote set-url git@github.com:USERNAME/REPOSITORY.git
{% endhighlight %}

(Again, the url for Bitbucket is different. You can click on SSH on the web dashboards of either repo manager to find this URL.)

Try pushing again. If all goes well, after you accept the keys from the server on your system, it should now be able to commit and push headlessly.

At this point, our work is simple. While there is a python package to interface with git, I decided this was overkill and just wrote a timed loop:

{% highlight Python %}
import os, sys, calendar, time

while(True):
    print "Adding..."
    os.system("git add .")
    print "Committing..."
    os.system("git commit -m 'Timely commit %d'" % (calendar.timegm(time.gmtime())))
    print "Pushing..."
    os.system("git push --all")
    print "Sleeping..."

    time.sleep(600)
{% endhighlight %}

And that's all. The script will commit every ten minutes, and you now have free cloud hosting with timely version control. 

![commits]({{site.url}}/assets/img/gitauto/commits.png)

Enjoy.
