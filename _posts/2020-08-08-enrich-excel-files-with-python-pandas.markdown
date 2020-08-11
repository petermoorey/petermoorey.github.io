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
<iframe
  src="https://carbon.now.sh/embed?bg=rgba(171%2C%20184%2C%20195%2C%201)&t=material&wt=none&l=auto&ds=true&dsyoff=20px&dsblur=68px&wc=true&wa=true&pv=56px&ph=56px&ln=false&fl=1&fm=Hack&fs=14px&lh=133%25&si=false&es=2x&wm=false&code=import%2520requests%250Aimport%2520pandas%2520as%2520pd%250Aimport%2520logging%250A%250A%2523%2520setup%2520logging%252C%2520so%2520we%2520can%2520view%2520what%27s%2520happening%2520as%2520it%2520executes%250Alogging.getLogger().addHandler(logging.StreamHandler())%250Alogging.getLogger().setLevel(logging.INFO)%250A%250A%2523%2520define%2520the%2520file%252Fsheet%2520names%250Aexisting_file%2520%253D%2520%27endpoint-report.xlsx%27%250Anew_file%2520%253D%2520%27endpoint-report-with-locations.xlsx%27%250Asheet_name%2520%253D%2520%27Endpoint%2520Data%27%250A%250A%250A%2523%2520reusable%2520function%2520to%2520get%2520the%2520location%2520for%2520a%2520given%2520IP%2520address%250Adef%2520make_api_call(ip_address%253A%2520str)%253A%250A%2520%2520%2520%2520%2522%2522%2522Return%2520the%2520location%2520for%2520a%2520given%2520IP%2520address.%2522%2522%2522%250A%2520%2520%2520%2520url%2520%253D%2520f%2522https%253A%252F%252Fmywebsite%252Fapi%252F%253Fip%253D%257Bip_address%257D%2522%250A%2520%2520%2520%2520resp%2520%253D%2520requests.get(url%253Durl)%250A%2520%2520%2520%2520if%2520resp.status_code%2520%253D%253D%2520200%253A%250A%2520%2520%2520%2520%2520%2520%2520%2520location%2520%253D%2520resp.json()%255B%27location%27%255D%250A%2520%2520%2520%2520%2520%2520%2520%2520return%2520location%250A%2520%2520%2520%2520else%253A%250A%2520%2520%2520%2520%2520%2520%2520%2520raise%2520HTTPError(resp.status_code%252C%2520resp.text)%250A%250A%250Aif%2520__name__%2520%253D%253D%2520%27__main__%27%253A%250A%250A%2520%2520%2520%2520%2523%2520open%2520the%2520Excel%2520document%2520and%2520import%2520as%2520a%2520Pandas%2520data%2520frame%250A%2520%2520%2520%2520df%2520%253D%2520pd.read_excel(existing_file%252C%2520sheet_name%253Dsheet_name)%250A%250A%2520%2520%2520%2520%2523%2520get%2520the%2520values%2520of%2520the%2520column%2520%27IPv4%2520Address%27%250A%2520%2520%2520%2520ip_addresses%2520%253D%2520df%255B%27IPv4%2520Address%27%255D%250A%250A%2520%2520%2520%2520%2523%2520create%2520an%2520empty%2520list%2520to%2520store%2520the%2520location%2520values%250A%2520%2520%2520%2520locations%2520%253D%2520%255B%255D%250A%250A%2520%2520%2520%2520%2523%2520get%2520the%2520total%2520rows%2520in%2520the%2520data%2520frame%2520and%2520create%2520a%2520counter%2520to%2520track%2520progress%250A%2520%2520%2520%2520total_ips%2520%253D%2520len(ip_addresses)%250A%2520%2520%2520%2520processed_ips%2520%253D%25200%250A%250A%2520%2520%2520%2520%2523%2520loop%2520over%2520each%2520IP%2520address%250A%2520%2520%2520%2520for%2520ip%2520in%2520ip_addresses%253A%250A%2520%2520%2520%2520%2520%2520%2520%2520%2523%2520query%2520API%2520to%2520get%2520the%2520location%250A%2520%2520%2520%2520%2520%2520%2520%2520location%2520%253D%2520make_api_call(ip)%250A%250A%2520%2520%2520%2520%2520%2520%2520%2520%2523%2520log%2520to%2520screen%2520the%2520IP%2520and%2520the%2520discovered%2520location%250A%2520%2520%2520%2520%2520%2520%2520%2520logging.info(f%2522%257Bprocessed_ips%257D%252F%257Btotal_ips%257D%253A%2520%257Bip%257D%2520--%253E%2520%257Blocation%257D%2522)%250A%250A%2520%2520%2520%2520%2520%2520%2520%2520%2523%2520add%2520the%2520discovered%2520location%2520to%2520the%2520list%2520of%2520location%2520values%250A%2520%2520%2520%2520%2520%2520%2520%2520locations.append(location)%250A%250A%2520%2520%2520%2520%2520%2520%2520%2520%2523%2520increment%2520the%2520number%2520of%2520processed%2520IP%2520addresses%250A%2520%2520%2520%2520%2520%2520%2520%2520processed_ips%2520%252B%253D%25201%250A%250A%2520%2520%2520%2520%2523%2520create%2520a%2520new%2520column%2520in%2520the%2520dataframe%2520called%2520%27location%27%2520and%2520assign%2520the%2520values%2520from%2520the%2520now%2520populated%2520list%250A%2520%2520%2520%2520df%255B%27location%27%255D%2520%253D%2520locations%250A%250A%2520%2520%2520%2520%2523%2520save%2520the%2520data%2520frame%2520to%2520a%2520new%2520Excel%2520document%250A%2520%2520%2520%2520df.to_excel(new_file%252C%2520sheet_name%253Dsheet_name)"
  style="width: 1024px; height: 1284px; border:0; transform: scale(1); overflow:hidden;"
  sandbox="allow-scripts allow-same-origin">
</iframe>

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
