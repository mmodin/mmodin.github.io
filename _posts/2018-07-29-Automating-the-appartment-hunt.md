---
layout: post
title: "Automating the apartment hunt via Bostadsformedlingen"
date: 2018-07-29
categories: [Python, Bostadsformedlingen, data, Data Science, Stockholm, housing]
---

As you may or may not be familiar with, finding an apartment in Stockholm is not very easy. Your alternatives are basically buying your apartment or renting either via subletting contracts or utilzing the public utlity company [Bostadsformedlingen](https://bostad.stockholm.se/). It turns out that neither of the latter alternatives are optimal. While leasing you often up with overpriced subpar apartments (despite rent control) whereas you're faced with an unrealistically long queue using Bostadsformedlingen.

I've personally been in the queue for an apartment with Bostadsformedlingen for aproximately 10 years and no luck at finding an apartment so far. Mostly because it's a time consuming task (mind you the user experience on their website is terrible) as aparments are listed for only a few days at the time. Hence I thought - why not (semi) automate this task.

While casually browsing [Bostadsformedlingen](https://bostad.stockholm.se/) I found that it loads a resource called `AllaAnnonser` (all listings) at the following URL: [https://bostad.stockholm.se/Lista/AllaAnnonser](https://bostad.stockholm.se/Lista/AllaAnnonser). 

![devtools_bostadsformedlingen]( {{ "/img/devtools_bostadsformedlingen.png" | absolute_url }})

This conveniently GETs all the currently listed apartments and hence the idea to automate the task of filtering and downloading this. I decided to write a simple Python program ([Github repo here](https://github.com/mmodin/bostadsformedlingen)) to do just this.

The resource `AllaAnnonser` convinelty wraps the data in a json list which can easlily be parsed in Python.

```json
[
   {
      "AnnonsId":144964,
      "Stadsdel":"Flemingsberg",
      "Gatuadress":"Rontgenvagen 5",
      "Kommun":"Huddinge",
      "Vaning":3,
      "AntalRum":1,
      "Yta":28,
      "Hyra":4055,
      "AnnonseradTill":"2018-07-31",
      "AnnonseradFran":"2018-07-28",
      "KoordinatLongitud":17.937958782410504,
      "KoordinatLatitud":59.223580253558971,
      "Url":"/Lista/Details/?aid=144964",
      "Antal":1,
      "Balkong":false,
      "Hiss":true,
      "Nyproduktion":false,
      "Ungdom":false,
      "Student":true,
      "Senior":false,
      "Korttid":false,
      "Vanlig":false,
      "Bostadssnabben":false,
      "Ko":"Bostadskon",
      "KoNamn":"Bostadskon",
      "Lagenhetstyp":"Studentlagenhet",
      "HarAnmaltIntresse":false,
      "KanAnmalaIntresse":false,
      "HarBraChans":false,
      "HarInternko":false,
      "Internko":false,
      "Externko":false,
      "Omraden":[
         {
            "Id":306,
            "PlatsTyp":2
         },
         {
            "Id":8,
            "PlatsTyp":1
         },
         {
            "Id":76,
            "PlatsTyp":0
         }
      ],
      "ArInloggad":false,
      "LiknadeLagenhetStatistik":{
         "KotidFordelningQ1":3,
         "KotidFordelningQ3":6
      }
   },
   ...
]

```

In my case I decided to build a class to handle all the data management but the data download and parsing can easily be captured by a simple program.

```python
import Requests

response = requests.get('https://bostad.stockholm.se/Lista/AllaAnnonser')
if response.status_code == 200:
    data = response.json()
```

The next steps are rather straight forward. I store the data as a [pandas.DataFrame](https://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.html) and save it to disk by ["Pickling"](https://docs.python.org/2/library/pickle.html) it in order to determine what listings are "new" since the program ran the last time. My initial idea was to store the data as csv but I ended up having precision issues with the map coordinates while stored as floating points which caused the slight differences and I wanted to get away with as little data manipulation as possible.

Now I had the option to host the full program on my local desktop computer (which is typically not always on) or use some sort of cloud server. I opted for the latter and spun up a [Compute Engine](https://cloud.google.com/compute/) with Google Cloud. My idea was simply to email myself the an HTML table version of the data every day for me to look at. It turns out that cloud providers are very much subject to abuse where people use the services to send mass emails with malware or ads and Google has hence blocked the standard SMTP ports. It's all very well document [here](https://cloud.google.com/compute/docs/tutorials/sending-mail/) and they offer workarounds by utilizing thrid-party email providers that you can call via standard http requests. I decided to go with [MailJet](https://www.mailjet.com/) as the sign up process was simple (even for the free tier) and that they offer a very simple Python module for authenticating and sending the emails.

I created a simple [HTML template](https://github.com/mmodin/bostadsformedlingen/blob/master/scripts/table_template.html) and use convert the pandas DataFrame to html with [pandas.DataFrame.to_html](https://pandas.pydata.org/pandas-docs/version/0.22/generated/pandas.DataFrame.to_html.html). The final result looks something like:

<head><style>#listings {    font-family: "Trebuchet MS", Arial, Helvetica, sans-serif;    border-collapse: collapse;    width: 100%;}#listings td, #listings th {    border: 1px solid #ddd;    padding: 8px;}#listings tr:nth-child(even){background-color: #f2f2f2;}#listings tr:hover {background-color: #ddd;}#listings th {    padding-top: 12px;    padding-bottom: 12px;    text-align: left;    background-color: #4CAF50;    color: white;}</style></head><body><table id="listings">  <thead>    <tr style="text-align: right;">      <th></th>      <th>id</th>      <th>district</th>      <th>municipality</th>      <th>sqm</th>      <th>rooms</th>      <th>type</th>      <th>rent</th>      <th>Q3</th>      <th>fromDate</th>      <th>toDate</th>    </tr>  </thead>  <tbody>    <tr>      <th>54</th>      <td>144961</td>      <td>Johanneshov</td>      <td>Stockholm</td>      <td>37.0</td>      <td>1.0</td>      <td>Hyresratt</td>      <td>5949.0</td>      <td>13.0</td>      <td>2018-07-27</td>      <td>2018-07-31</td>    </tr>    <tr>      <th>55</th>      <td>144965</td>      <td>Kristineberg</td>      <td>Stockholm</td>      <td>29.0</td>      <td>1.0</td>      <td>Korttidskontrakt</td>      <td>5105.0</td>      <td>12.0</td>      <td>2018-07-28</td>      <td>2018-07-31</td>    </tr>    <tr>      <th>68</th>      <td>144956</td>      <td>Norrmalm</td>      <td>Stockholm</td>      <td>37.0</td>      <td>1.0</td>      <td>Hyresratt</td>      <td>7795.0</td>      <td>18.0</td>      <td>2018-07-28</td>      <td>2018-07-31</td>    </tr>    <tr>      <th>79</th>      <td>144952</td>      <td>Stadshagen</td>      <td>Stockholm</td>      <td>40.0</td>      <td>1.5</td>      <td>Korttidskontrakt</td>      <td>5920.0</td>      <td>NaN</td>      <td>2018-07-28</td>      <td>2018-07-31</td>    </tr>    <tr>      <th>81</th>      <td>145023</td>      <td>Sodermalm</td>      <td>Stockholm</td>      <td>89.0</td>      <td>2.0</td>      <td>Hyresratt</td>      <td>9163.0</td>      <td>23.0</td>      <td>2018-07-31</td>      <td>2018-08-01</td>    </tr>    <tr>      <th>83</th>      <td>145002</td>      <td>Sodermalm</td>      <td>Stockholm</td>      <td>40.0</td>      <td>1.0</td>      <td>Hyresratt</td>      <td>6775.0</td>      <td>18.0</td>      <td>2018-07-31</td>      <td>2018-08-01</td>    </tr>    <tr>      <th>86</th>      <td>144966</td>      <td>Sodermalm</td>      <td>Stockholm</td>      <td>73.0</td>      <td>2.0</td>      <td>Korttidskontrakt</td>      <td>6871.0</td>      <td>21.0</td>      <td>2018-07-28</td>      <td>2018-07-31</td>    </tr>  </tbody></table></body>

It probably doesn't look the best on this page as it uses dynamic sizing in a limited sized blog post but I will probably end up re-purposing this the function for presenting further data on this blog in the future. I ended up having to replace some junk produced by `to_html()` but the basics of it all goes something like:

```python
import pandas

def html_table(df):
    # Load table template
    with open('table_template.html', 'r') as f:
        template = f.read().replace('\n', '')

    # Replace formatting options that come with to_html()
    html_str = df.to_html()\
        .replace('\n', '')\
        .replace('<table border=\"1\" class=\"dataframe\">', '')\
        .replace('</table>', '')
    return template % html_str

```

I'll happily answer any questions or help out if you want to implement something similar via my email address below.