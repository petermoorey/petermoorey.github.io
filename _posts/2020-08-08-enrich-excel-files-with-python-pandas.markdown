---
layout: post
title: Enriching Excel Files with Python Pandas
date: 2020-08-09 11:32:20 +0600
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes.
img: panda.jpg
fig-caption: Photo courtesy of https://www.chinadaily.com.cn/
tags: [python, pandas, excel]
---

I was asked by one of my colleagues to discover the locations of ~2,000 endpoints in a global enterprise network.  They provided a detailed Excel document which included the IPv4 address of each endpoint.  To translate the IP addresses to locations I used a web service/API which accepted an IP address as input and responded with the location.  I'm not aware of a way to make API calls natively with Excel, perhaps it can be done with TypeScript in Excel, I'm not sure. Anyway, I used Python and 'pandas' data analysis library to update the Excel document with each individual location automatically.  

The script took 12 minutes to write, performing the task manually wouldn't have been possible, given the effort required.

To learn about <a href="https://pandas.pydata.org/" target="_blank">pandas</a> I recommend the <a href="https://pandas.pydata.org/docs/index.html" target="_blank">official documentation</a>, it's well written and easy to understand. 


## Preparation

First, you'll need Python and Python PIP (package manager).  You can get python from <a href="https://www.python.org/downloads/" target="_blank">here</a>.  Instructions for installing Python PIP are available <a href="https://pip.pypa.io/en/stable/installing/" target="_blank">here</a>.  Next, create a requirements file, we'll need a few Python modules that are not built-in:

*requirements.txt*
```
requests
pandas
xlrd
openpyxl
```

Install the modules using Python PIP:

```
pip install -r requirements.txt
```

## The Script

High level process:

- Open Excel file as Pandas data frame
- Get the data from the IPv4 Address column
- Loop over each IP address and call the web service to get the location
- Store the locations in a list
- Create a new column in the data frame and assign the location values
- Save the data frame as a new file

Here's the Python script, hopefully you can follow the comments to understand the flow.  

*excel-enrichment.py*
<script src="https://gist.github.com/petermoorey/f1a8a4c461fae52262c004970b8e4a81.js"></script>

Whilst it executes you will see output like the example below, and a new Excel document will be created in the same folder as your script:

```
12/1798: 10.200.4.202 --> mylocation1
13/1798: 10.200.4.204 --> mylocation1
14/1798: 10.200.4.206 --> mylocation6
15/1798: 10.100.99.201 --> mylocation9
16/1798: 10.200.5.205 --> mylocation8
17/1798: 10.200.4.198 --> mylocation1
18/1798: 10.200.4.199 --> mylocation1
```

I hope this was useful!
