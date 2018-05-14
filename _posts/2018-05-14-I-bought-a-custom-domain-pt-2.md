---
layout: post
title: "I bought a custom domain pt. 2"
date: 2018-05-14
categories: [DNS, GitHub Pages, Google Domains, https]
---

[Two days ago]({% post_url 2018-05-12-I-bought-a-custom-domain %}) about setting up a my custom domain name and enforcing HTTPS by redirecting HTTP. As suggested in [this](https://github.community/t5/Pages/How-to-enable-https-support-on-custom-domains/m-p/6923#M463) thread it's likely that the certificate generation has just been put on an error queue somewhere in GitHub's backend. The suggested solution was to create a ticket with the suport team in order for GitHub to "nudge" the certificate generation ahead in line and within a few hours my site was redirecting HTTP to HTTPS 🙂.


<p align="center">
  <img  src="{{ "/img/cert.png" | absolute_url }}">
</p>


Upon inspection everything seemed fine until I visited the blog page where Chrome was indicating that the connection was unsafe. I immediately expected mixed content which after using the [JitBit](https://www.jitbit.com/sslcheck/) SSL webcrawler. Unsurprisingly the result gave away that linked content was still using HTTP.

```
Done. Total pages crawled: 5

Pages with unsecure content:
https://mmodin.com/ ?
http://mmodin.com/img/me.png
http://mmodin.com/feed.xml

https://mmodin.com/blog/ ?
http://mmodin.com/feed.xml

https://mmodin.com/blog/2018-05-12/I-bought-a-custom-domain ?
http://mmodin.com/img/cname.png
http://mmodin.com/img/enforcehttps.png
http://mmodin.com/feed.xml

https://mmodin.com/blog/2018-05-10/WhatsApp-text-analysis ?
http://mmodin.com/feed.xml

Pages failed to crawl (error returned from the server):
https://mmodin.com/blog/2018-05-12/letsencrypt.org
https://mmodin.com/blog/2018-05-12/letsencrypt.org
```

The solution to the problem was to edit `_config.yml` and include `url: https://mmodin.com`. You would subsequently link to images using e.g. `absolute_url`:

```
<p align="center">
  <img  src="{% raw %}{{ "/img/cert.png" | absolute_url }}{% endraw %}">
</p>

```
or link to permalinked (`permalink: /blog/:year-:month-:day/:title`) blog posts using :
```
[Two days ago]({% raw %}{% post_url 2018-05-12-I-bought-a-custom-domain %}{% endraw %})
```