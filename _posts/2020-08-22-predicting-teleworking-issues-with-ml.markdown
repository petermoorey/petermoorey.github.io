---
layout: post
title: Predicting Teleworking Issues With Machine Learning
date: 2020-08-22 16:32:20 +0600
description: 
img: machine-learning.jpg
tags: [google, machine learning, ml, automl]
---

This is my first machine learning project.  The goal is to predict if teleworkers are experiencing network issues using network performance and configuration data collected from each computer.  How the data is collected falls outside the scope of this article.  The prediction will indicate the source of the problem (Wi-Fi, wired LAN, or Internet).  The sample data set I will be using to train the model contains details of 1,500 computers including the following 'features' for each:

- Latency to corporate network test page (10th percentile - last 24hrs)
- Latency to corporate network test page (90th percentile - last 24hrs)
- Latency to Internet test page (10th percentile - last 24hrs)
- Latency to Internet test page (90th percentile - last 24hrs)
- Latency to LAN gateway (10th percentile - last 24hrs)
- Latency to LAN gateway (90th percentile - last 24hrs)
- Wi-Fi Channel Number
- Wi-Fi Signal Strength Percentage (average - last 24hrs)
- VPN gateway name
- Issue (None, LAN, WiFi, Internet)

What I am trying to predict is the type of 'Issue' for each computer, this is also known as the 'Target' in Google ML speak.  This type of machine learning model is known as multi-class classification, meaning it predicts one class (label) from a series of three or more discrete classes.  Other types of model include binary (two classes), or regression (numberical prediction).

I chose to use [Google AutoML Tables](https://cloud.google.com/automl/) because it abstracts away much of the complexity, making it extremely quick to get started.

# Data Preperation

This is the most important and time-consuming part of the process. I created a SQL query to retrieve the data into the correct structure, essentially there is a row of data for each computer, with columns for each feature (listed above).  I then provided the target data (answers) for the data that would be used for training.  The more data you provide, the more accurate your model will be.  The minimum number of rows is 1,000 for Google Auto ML tables.  

_Here's a sample of the data.  Highlighted in yellow are some indicators of network performance issues:_

<table class="table table-bordered table-hover table-condensed">
<thead><tr><th title="Field #1">percentile_10_corporate</th>
<th title="Field #2">percentile_90_corporate</th>
<th title="Field #3">percentile_10_internet</th>
<th title="Field #4">percentile_90_internet</th>
<th title="Field #5">percentile_10_lan</th>
<th title="Field #6">percentile_90_lan</th>
<th title="Field #7">wlan_signal</th>
<th title="Field #8">wlan_in_use</th>
<th title="Field #9">wlan_channel</th>
<th title="Field #10">vpn_gateway</th>
<th title="Field #11">issue</th>
</tr></thead>
<tbody><tr>
<td align="right">62</td>
<td align="right">102</td>
<td align="right">202</td>
<td align="right">2165</td>
<td align="right">24</td>
<td align="right" bgcolor="yellow">344</td>
<td align="right">49</td>
<td>TRUE</td>
<td align="right">6</td>
<td>location1</td>
<td bgcolor="yellow">wifi</td>
</tr>
<tr>
<td align="right">183</td>
<td align="right">2967</td>
<td align="right">355</td>
<td align="right" bgcolor="yellow">2592</td>
<td align="right">0</td>
<td align="right">5</td>
<td align="right">99</td>
<td>TRUE</td>
<td align="right">6</td>
<td>location1</td>
<td bgcolor="yellow">internet</td>
</tr>
<tr>
<td align="right">277</td>
<td align="right">311</td>
<td align="right">585</td>
<td align="right">455</td>
<td align="right">1</td>
<td align="right">171</td>
<td align="right">83</td>
<td>TRUE</td>
<td align="right">36</td>
<td>location4</td>
<td>none</td>
</tr>
<tr>
<td align="right">103</td>
<td align="right">138</td>
<td align="right">196</td>
<td align="right">308</td>
<td align="right">16</td>
<td align="right" bgcolor="yellow">229</td>
<td align="right">93</td>
<td>FALSE</td>
<td align="right">4</td>
<td>location6</td>
<td bgcolor="yellow">lan</td>
</tr>
<tr>
<td align="right">170</td>
<td align="right">409</td>
<td align="right">353</td>
<td align="right" bgcolor="yellow">1018</td>
<td align="right">1</td>
<td align="right">11</td>
<td align="right">0</td>
<td>FALSE</td>
<td align="right">0</td>
<td>location8</td>
<td bgcolor="yellow">internet</td>
</tr>
<tr>
<td align="right">225</td>
<td align="right">384</td>
<td align="right">466</td>
<td align="right">219</td>
<td align="right">1</td>
<td align="right">15</td>
<td align="right">77</td>
<td>TRUE</td>
<td align="right">6</td>
<td>location10</td>
<td>none</td>
</tr>
<tr>
<td align="right">228</td>
<td align="right">303</td>
<td align="right">472</td>
<td align="right">488</td>
<td align="right">1</td>
<td align="right">4</td>
<td align="right">99</td>
<td>TRUE</td>
<td align="right">11</td>
<td>location1</td>
<td>none</td>
</tr>
<tr>
<td align="right">200</td>
<td align="right">211</td>
<td align="right">430</td>
<td align="right">456</td>
<td align="right">2</td>
<td align="right">64</td>
<td align="right">78</td>
<td>TRUE</td>
<td align="right">6</td>
<td>location14</td>
<td>none</td>
</tr>
</tbody></table>

Once imported the data is (by default) split into groups, 80% for training, 10% for validation and 10% for testing.

# Training

AutoML Tables uses Supervised Training (example data with known results) to improve accuracy.  As ML performs training it verifies the outcome of the predictions against the validation (10%) data.  The precision of the ML is determined by using the test (10%) data.

Once the training process completes you can view the results.  Using the sample network performance data I uploaded, the outcome of ML training was 86% precision.  This means 86% of the time ML successfully predicted the classification (Issue) and exactly matched the known test data.

![](/assets/img/predicting-teleworking-issues-with-ml/training-results-summary.png)

# Test and Use

Now that we have created the machine learning model we can start to use it.  In Google AutoML Tables there are three options to use the model:

- Batch Prediction - manual, providing data in CSV format or BigQuery table)
- Online Prediction - creates a API endpoint hosted in Google Cloud
- Export Model -  download the model as a TensorFlow package for local use

I used batch prediction and uploaded data for 5,000 computers for which I didn't know the 'Issue' and ran the data through the ML model.  After downloading the results new columns were available, one for each category (None, WiFi, LAN and Internet in my case).  For each computer/row, there is a score with the probability for each category.  The batch took four minutes to execute, and based on manual validation the results look very promising.

_Highlighted in green is the category (Issue) which had the highest probability.  As we know from the training data, the accuracy should be around 86%_

![](/assets/img/predicting-teleworking-issues-with-ml/scores.png)


