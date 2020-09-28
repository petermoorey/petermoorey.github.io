---
layout: post
title: Predicting Teleworking Issues With Machine Learning
date: 2020-08-22 16:32:20 +0600
description: 
img: machine-learning.jpg
tags: [google, machine learning, ml, automl]
---

The goal of this experiment is to predict and classify network issues that teleworkers may experience whilst working from home. It will be achieved by analyzing network performance and configuration data collected from each computer using machine learning technology. I won't go into details about how the data is collected, that's a seperate subject. The machine learning prediction will classify issues such as Wi-Fi, wired LAN, Internet or None (in case there is no issue detected).

The method of machine learning model that will be applied is known as multi-class classification, meaning it predicts one class (label) from a series of three or more discrete classes. Other types of model include binary (two classes), or regression (numberical prediction).

I selected to use Google AutoML because it is very quick and simple to get started.

# Data Features

Machine Learning requires 'Features' to build the model; these are the values used to determine the prediction. The dataset includes the following features for each computer:

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

# Data Preperation

This is the most important and time-consuming part of the process. I created a SQL database query to retrieve the data in a structure where there is a row for each computer, with columns for each feature (listed above). I then manually provided the target data (answers) for the 'Issue' column which will be used for training. For supervised machine learning you must provide the 'correct answers' in the training data. The more data you provide, the more accurate the model will be.

The minimum number of rows required is 1,000 for Google AutoML. The more features you add, the more training data you should provide.
Below is a sample of the training data, note that the 'Issue' column on the far right has been populated by myself. The training data set contains information about 1,600 computers.

Here's a sample of the data.  Highlighted in yellow are some indicators of network performance issues:

<table>
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

# Import the training data

Once the training data is ready it can be uploaded directly (in CSV format) from your computer, or imported from a Google Cloud storage bucket. Alternatively, you can provide the data in a BigQuery database table.

Screenshot showing import of data from Google storage bucket
[](/assets/img/predicting-teleworking-issues-with-ml/data_import.png)

# Configure the training the model

AutoML uses supervised training (example data with known results) to improve accuracy. As ML performs training it verifies the outcome of the predictions against the validation (10%) data. The precision (accuracy) of the machine learning is determined by using the remaining test (10%) data.

Screenshot showing data training settings
![](/assets/img/predicting-teleworking-issues-with-ml/data_training.png)

# Evaluate the training results

Once the training completes you can evaluate how accurate the model is. Precision incidates the percentage of tests where machine learning successfully identified the correct classification for the test (10%) portion of the training dataset.

Using the sample network performance data I uploaded, the outcome of ML training was 86% precision. This means 86% of the time ML successfully predicted the classification (Issue) and exactly matched the known test data.

Screenshot showing results of training

![](/assets/img/predicting-teleworking-issues-with-ml/data_results.png)

# Utilize the machine learning model

Now that the machine learning model has been created, we have three options to use it:

- Batch Prediction - manual, providing data in CSV format or BigQuery table
- Online Prediction - creates a API endpoint hosted in Google Cloud
- Export Model - download the model as a TensorFlow package for local use

For this experiment I used batch prediction and uploaded data for 5,000 computers for which I didn't know the 'Issue' and ran the data through the ML model. After downloading the results new columns were available, one for each category (None, Wi-Fi, LAN and Internet).

For each computer/row, and for each classification (Issue) there is a score with the probability. The batch took four minutes to execute.

_Highlighted in green is the category (Issue) which had the highest probability.  As we know from the training data, the accuracy should be around 86%_

![](/assets/img/predicting-teleworking-issues-with-ml/scores.png)

# Analysis of Internet versus LAN latency

I knew from the results of the machine learning that the Internet and LAN latency 90th percentile features were the most important factors. To check if the machine learning results made sense I decided to create a scatter graph and compare the two most important features.

```python
import plotly.express as px
import plotly.offline as pyo

pyo.init_notebook_mode()

# create a scatter graph with 90pc latency for LAN versus Internet
fig = px.scatter(
    df.query('percentile_90_internet < 4000 & percentile_90_lan <4000'),
    x="percentile_90_internet", y="percentile_90_lan",
    title="Internet versus LAN latency by Issue",
    color="Issue",
    labels = {
        'percentile_90_internet': 'Internet Latency (MS)',
        'percentile_90_lan': 'LAN Gateway Latency (MS)',
        }
    )
# display graph
fig.show()
```

![](/assets/img/predicting-teleworking-issues-with-ml/data_int_lan_latency.png)


From the chart I was able to validate that:
- When both LAN and Internet latency are low no issue is detected
- When Internet latency is high, but LAN is low the issue is Internet (as expected)
- When LAN latency is high the issue is LAN or Wi-Fi (I infer that if LAN issues would cause Internet to be poor too)

# Analysis of Wi-Fi signal strength versus channel number

The local network is under the control of the teleworker; it's the component of the network that they can have the most influence over. I was curious to see what they could do to improve their connectivity. I expected there to be a correlation between poor LAN performance and Wi-Fi signal strength/channel.

```python
# create a scatter graph with Wi-Fi channel versus signal strength
fig = px.scatter(
    df.query('Issue != "Internet" and Issue != "LAN"'),
    x="wlan_channel",
    y="wlan_signal",
    title="Wi-Fi Signal versus Channel by Issue (excluding Internet)",
    color="Issue",
    labels = {
        'wlan_signal': 'Wi-Fi Signal Strength (%)',
        'wlan_channel': 'Wi-Fi Channel Number',
        }
    )
# display graph
fig.show()
```

![](/assets/img/predicting-teleworking-issues-with-ml/data_wlan_channel_signal.png)

I found that the majority of Wi-Fi issues occur on 2.4Ghz channels, which is expected. Interestingly, many issues affect 2.4Ghz users with good signal strength. This indicates that high LAN latency could be caused by interference. I can also conclude that for those users with strong Wi-Fi signal strength it would be safe to advise switching to 5Ghz which is often less congested.

# Conclusion

At the end of this experiment I'm happy with the progress I made, given minimal time investment. It was a fun project to try machine learning for the first time. Google AutoML makes it incredibly easy to get started, and accuracy of 86% is pretty good!.

I'd like to find a way to generate training data automatically, and also provide feedback to the machine learning model for continous improvement. Machine learning will become more important as the dataset grows and more features are added which require analysis.
