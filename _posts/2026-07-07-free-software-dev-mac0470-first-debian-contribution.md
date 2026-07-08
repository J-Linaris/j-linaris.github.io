---
layout: post
title: Free Software Dev (MAC0470) First Debian Contribution
date: 2026-07-07 18:01 -0300
description: 'Comments on my first contribution to the linux kernel as part of the discipline "Free and Open Source Software Development" (MAC0470) of IME-USP' 
image: 
category: academic
tags: 
published: true
sitemap: true
---

# Linux Kernel Contribution

## Introduction

The end for my series on "Free and Open Source Software Development" (MAC0470) of IME-USP has come and will be related in this brief post. To wrap this journey, me and my pair have chosen to make our first contribution to Debian. As you'll see, it was a simple and really beginner friendly way to introduce us to this Debian Dev world. The Debian Brasil Community's [packaging documentation](https://debianbrasil.org.br/pt-br/empacotamento) guided us through the whole process seamlessly.

## The contribution: Change in packaging

A good way for us just to get to know how all this contribution process works is to let us fix deprecations (that's exactly what we did). Just like previously, we decided each one would take a package to modify, as this is better to accelerate our pace.

In this context, it is important to know that there are two types of debian packages: the source and the binary (.deb) packages. The source contains all the actual source code and a `debian/` folder containing the content necessary for packaging. Hence, the source package can generate multiple binary packages in its construction. We will foccus, in this contribution, in source packages.

Actually, we will modify the `control` file in `debian/`. In this context, three topics are of major importance:
* `control` file: Concentrates the metadata for both source package and binary packages. The first paragraph always refers to the source package, while the other, to the binary constructed by the source.
* `Priority`  is a field of source package metadata (in `control` files) that specifies if the package must be installed when Debian is installed. The possibles values are: {optional, required, important, standard}
* `changelog` file: Presents one entry per modification made in the package.

As you may imagine, most of the packages aren't obligatory in Debian installation, therefore, in `control`, we add `Priority:optional` to indicate this.

**Contribution:** Currently, however, Debian tools have incorporated 'optional' as the standard value for the `Priority` field. Hence, there is no need anymore to restate that in the file, so the task was to delete it.

Before doing this, I had to create an account in Debian Brazil community's [Salsa](https://salsa.debian.org/debian-brasil-team), the gitlab instance used for debian development, choose a package to contribute to and clone the repo in my local machine. The tutorial perfectly described how to do this initial setup. 

To select the package, I searched for the first one among the recommended packages that contained Golang in its named and took it. In this process, I ended up choosing a POP3 client package for golang, called "golang-github-knadh-go-pop3". POP3 is a protocol (*Post Office Protocol*) used to enable email clients. It basically lets the client receive all entries for the user in the mail server linearly using TCP ports. To know more about, make sure to read [its wiki](https://pt.wikipedia.org/wiki/Post_Office_Protocol).

Moving forward, it all actually started with the change:

```
control file:
----------------------------------------------------------------------------------------------------
Source: golang-github-knadh-go-pop3
Section: golang
- Priority: optional
(...)
```

Then, I committed it and added an entry in the changelog, by simply running `gbp dch --commit`, which uses my last commit to create this registry:

```
changelog file:
----------------------------------------------------------------------------------------------------
golang-github-knadh-go-pop3 (1.0.0-2) UNRELEASED; urgency=medium

  * d/control: drop Priority field, optional is default now. // my commit message

 -- Joao Paulo Linaris <joaolinaris@gmail.com>  Tue, 07 Jul 2026 15:04:13 -0300
(...)
```

Indeed, soon, I'll be the author of the last change made in golang-github-knadh-go-pop3 debian package ;) 

Finally, in theory, we simply push using `gbp push --debian-tag=''`. However, when I ran it, a invalid refs/head error popped. Then, I noticed that it was trying to push to an `upstream` branch, although I was in `debian/sid` branch. So, I just added the flag `--upstream-branch='debian/sid'` and, again, an error.

It said the remote counterpart had changes I didn't have locally (which didn't make any sense), so I spent around 10 minutes trying to fix it. After this period, I simply tried to create a merge request and, guess what: THE COMMITS WERE THERE. Sometimes, git simply works in misterious ways...

After all, I have a merge request. Now, the reviewers approval is the only thing pending, but, as I talked to him, apparently, there is nothing more I can do. It is worth stating that executing `gbp buildpackage <my_package_repo_clone_path>` lintian is failing due to the `UNRELEASED` tag in the changelog, which was expected to be `unstable` (referring to the debian release it would be applied on). This will be changed by the reviewer futurely when he accepts the merge request.

## Conclusion

I can not describe how much better it is to contribute to Debian compared to the Linux Kernel. Not because of the code itself, but due to how welcoming and friendly is the Debian community. They are always willing to help and provide extensive documentation, two profound advantages over Linux Kernel reviewers. It was really nice to get in contact with this new community and I really look forward to be a member of it futurely (I changed from Ubuntu to Debian 13 as a proof of commitment).

My huge thanks to both Guilherme Dias, my pair, and Carlos Henrique, the reviewer. This subject has really expanded my notion about free software and really made me much more interested than I was when it started, thanks, as well, to [FLUSP](https://flusp.ime.usp.br/)! 
