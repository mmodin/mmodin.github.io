---
layout: post
title: "I bought a custom domain"
date: 2018-05-12
categories: [DNS, GitHub Pages, Google Domains]
---

I decided to buy a custom domain from [Google Domains](https://domains.google) because why the hell not. [mmodin.com](/) only cost me $12 for a year and hopefully gives me some incentives to continue writing posts. Either way, prior to this I had very little experience with configuring DNS' and I ultimately had to spend some time "researching" how to get things working. This is what I did.

1. Add a file called `CNAME` in the root directory of my github repository with the entry `mmodin.com`.
2. Configured an A record, CAA and CNAME with Google Domains.

The 2nd step took me a while to figure out as I wanted to ensure that the new domain automatically used (and redirected) all traffic to HTTPS. Luckily, Github has very recently partnered ([read announcement here](https://blog.github.com/2018-05-01-github-pages-custom-domains-https/)) with [letsencrypt.org](letsencrypt.org) to support HTTPS natively with custom domains. This means that there's no need to generate RSA keys or fiddling around.

![googledomains]({{ "/img/cname.png" | absolute_url }})

Note how the CAA entry reads `0 issue "letsencrypt.org"`. Once this was configured all you have to do is enforce it via the the GitHub settings, however, before this is possible GitHub/letsencrypt will need to generate the certificates which can take some time. You'll see this in the meantime:

![enforcehttps]( {{ "/img/enforcehttps.png" | absolute_url }})

As of writing I've been waiting for 36 hours since I enabled letsencrypt as CA via Google Domains.