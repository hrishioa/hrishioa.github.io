---
layout: post
title: "Tracking Word Documents with Git"
date: 2015-10-25 05:03:02
image: 'TextUtilGit/banner.png'
share_image: 'TextUtilGit/banner.png'
description: "Version controlling meaningful blobs"
tags:
- Git
- TextUtil
- Word
- Pandoc
categories:
- Hacks of Life
- Building Stuff
twitter_text:
---
 
For some time now, I have been using Git to keep track of my work - both source code and otherwise. Even when you're not using it for code, Git is perfect - easy backups to a Git server in the cloud along with the possibility of maintaining multiple working versions of my papers and writing projects is just great.

Except - Git treats my Word files as binaries. Different versions can be accessed just fine, but I have no access to diff functionality because Git doesn't understand what has changed between two versions. Not to mention, most programmers will flinch [uncontrollably][stkovflowq] at my mention of storing binaries in version control - even if it's only the stupid play I'm writing.

The solution is to use something that Git can understand out of the box - like Markdown - or find a way to help Git interpret `.docx` files, which you know deep (deep) down are text files. Choosing not to give up the flexibility of Word (and mostly out of habit), I started looking at ways to implement the latter.

[Martin Fenner][mfbio] has a very informative [post][mfpost] on one way of doing this. He uses [pandoc][pandoclink] to convert docx files to markdown that git can then use to compare two versions of the same file. However, I ran into problems when using pandoc to convert files, and had to use a different implementation. I'm including both methods below, so feel free to try either.

## Using Pandoc to interpret Word Documents

Martin recommends the following steps:

Before you begin, install [pandoc][pandoclink]. On a Mac, if you have [HomeBrew][hblink] or [MacPorts][mplink], this is as simple as typing `brew install pandoc` or `port install pandoc`. For a PC, you can find an installer on the [downloads][pddownloads] page. If you use LaTex, there's a good chance you already have it.

+ Create a `.gitattributes` file in your project directory (if you don't have one already) and add the following:

{% highlight BASH %}
*.docx diff=pandoc
{% endhighlight %}

This will let git know what to do when it encounters files with a `.docx` extension.

+ Append the following to the .gitconfig file (On Mac, this should be `~/.gitignore`. For Windows, this will depend on your installation, but for default settings, it should be in your home folder - or `%HOMEDRIVE%%HOMEPATH%`) - 

{% highlight BASH %}
[diff "pandoc"]
  textconv=pandoc --to=markdown
  prompt = false
[alias]
  wdiff = diff --word-diff=color --unified=1
{% endhighlight %}

We're adding pandoc to the config, along with creating an alias for a specific *diff* call.

+ Once this is done, you can make sure everything works by commiting a Word file, and running the following command:

{% highlight BASH %}
git wdiff <your_file>.docx
{% endhighlight %}

Unfortunately, this didn't work for me. For every file I tried, pandoc failed with a 'UTF-8' error. Some Googling showed that this was a common error with pandoc and some versions of Word. There were a few fixes, but none solved my problem. Not wanting to trawl the manpage for answers, I moved on to door number two.

## Using a custom converter for Word in Git

If pandoc worked for you, then this is really unnecessary and you can move on to actual work. But it didn't work for me, so I started looking for docx to markdown converters. There are a few online that are pretty good. There's a [Ruby Gem][rgmarkdown] that does it pretty well. There's even a [Visual Basic Script][vbscript] that does it, if you're looking for integration into Word itself. For Mac, the included `textutil` does docx conversions pretty easily. It comes pre-installed, and is pretty light-weight. Unfortunately, it doesn't support markdown, but if you really need to track stylistic changes, there is a [Ruby Gem][rgmarkdown2] that uses textutil for docx to markdown conversion that shouldn't be hard to use. Just install it and replace the direct calls to textutil in the directions. 
Converting to plain text was simple enough for me. I'm mostly using Terminal to track changes to my work, so textutil's txt conversion seemed enough (at least for the time being). It also meant that I didn't have to install new software on all my workstations.

To implement textutil-based tracking, do the following:

+ Create and add the following to `.gitattributes` in your project directory (same as before):

{% highlight BASH %}
*.docx diff=textutil
{% endhighlight %}

+ Add these lines to `~/.gitconfig`:

{% highlight BASH %}
[diff "textutil"]
  textconv=textutil -convert txt -stdout
  prompt = false
[alias]
  wdiff = diff --word-diff=color --unified=1
{% endhighlight %}

The `-convert txt` switch lets us convert the file to plain text, and the `-stdout` switch makes sure we're printing to standard out instead of to file.

+ Once this is done, you can run `git wdiff <filename>.docx` to see the results.

![screenshot]({{site.url}}/assets/img/TextUtilGit/screenshot.png)

It should now be easy to modify Git to work with complex file formats, now that you know how.

[rgmarkdown2]:https://gist.github.com/ttscoff/3861434
[vbscript]:https://gist.github.com/hawkrives/2305254
[rgmarkdown]:https://github.com/benbalter/word-to-markdown
[pddownloads]:https://github.com/jgm/pandoc/releases/tag/1.15.1.1
[hblink]:https://brew.sh
[mplink]:https://www.macports.org/
[stkovflowq]:http://stackoverflow.com/questions/29674006/is-it-good-practice-to-store-binary-dependencies-in-source-control
[mfbio]:http://blog.martinfenner.org/about.html
[mfpost]:http://blog.martinfenner.org/about.html
[pandoclink]:http://pandoc.org/