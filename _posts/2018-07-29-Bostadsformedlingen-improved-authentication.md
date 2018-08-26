---
layout: post
title: "Bostdsformedlingen improved (authentication)"
date: 2018-08-26
categories: [Python, Bostadsformedlingen, data, Data Science, Stockholm, housing]
---

About a month ago I wrote a summary of my latest project - [Automating the apartment hunt via Bostadsformedlingen]({% post_url 2018-05-12-I-bought-a-custom-domain %}). While not having directly resulted in finding an apartment it has certainly made it _a lot_ easier and almost effortless. However, receiving daily emails and experiencing some of the pain points there were some room for improvement. More specificallly, once listing the appartments it was unclear whether I was eligible for applying, despite filtering away some of the obvious candidates. Furthermore, it was also unclear what position I would have in the queue and hence my chances at getting the apartment at the end.

It turns out that both of these issues can be solved by logging in to the website before downloading all the relevant resources such as the listings. The trick is to perserve the session (and all the cookies) between requesting different resources from the webserver. It turns out that there are two main cookies that needs to be perserved:

```json
{  
   "SMIDENTITY": "abc123",
   "SMSESSION": "xyz456",
   "SMLOCALE": "en-US"
}
```
Using the `requests` library in python this is rather simple. The first step is the initiate a session in the `__init__` of the `BF` class that handles the resource gathering.
```python
self.session = requests.Session()
```
After that the next step is to get a session cookie which is done by requesting the login page (first row below). `requests` makes this incredibly easy by storing the cookies in `self.session.cookies` and they will automatically be sent in the next request that reference the session object. All that is left is to login which is done by authenticating against another server (Stockholms Stad) and posting some form data with user data. Note that are multiple ways of authenticating yourself on the website (Bank ID, Telia & normal user/pass). `'smauthreason': '0'` indicates that we use the normal username/password method which is by far the simplest programatically and does not require any 2-factor steps (I wonder how long they will allow this).

```python
if self.login:
    response = self.session.get(self.url + '/Minasidor/login/')
    logger.info('Getting login cookie...status code: %s' % response.status_code)
    url = 'https://login001.stockholm.se/siteminderagent/forms/login.fcc'
    data = {
        'target': '-SM-https://bostad.stockholm.se/secure/login',
        'smauthreason': '0',
        'smagentname': 'bostad.stockholm.se',
        'USER': username,
        'PASSWORD': password
    }
    response = self.session.post(url, data=data)
    if response.status_code == 200:
        logger.info('Successfully logged in...')
    else:
        logger.error('Failed logging in with status code: %s' % response.status_code)
        raise Exception('Login error')
```

