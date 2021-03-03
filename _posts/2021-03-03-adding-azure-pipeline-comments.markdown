---
layout: post
title: Automating Azure DevOps Pull Request Comments
date: 2021-03-03 13:57:20 +0600
description: 
img: message.jpg
tags: [azure, devops, pull, automation]
---

In this article I will explain how you can automatically add comments to an Azure DevOps (ADO) pull request during pipeline execution.  This technique can be useful when you want to add additional information which is only available during pipeline execution.  The approach utilizes the ADO API and could be adjusted to allow the creation of comments from any application/process.

# How it works 

1. Developer initiates a pull request to merge code from one Git branch to another
2. Pull request triggers the execution of a pipeline
3. Task in the pipeline automatically adds a comment to the pull request

# The Pipeline

Before setting up the pipeline, enable the ADO project 'Build Service' permissions allow 'Contribute to pull requests'.  Go to ADO Settings > Repositories > Permissions > Select the project Build Service > Tick 'Contribute to pull requests'.

The pipeline is best defined in YAML format and stored in the ADO code repository.  A task executes a Python script which does something according to your needs and posts the results as a comment to the pull request.  One important point is that a variable 'System.AccessToken' must be assigned to the task, in order to expose the automatically generated token used to authenticate to ADO API. 

*azure-pipelines.yml*
```yaml
# ADO Pipeline Definition
trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:
- script: |
    python do-something.py
  env:
    SYSTEM_ACCESSTOKEN: $(System.AccessToken)
  displayName: 'Python script to execute something and add pull request comment'
```

# Adding A Comment

The following example shows how a Python module is imported, and used to add a comment to the active pull request.  Pull requests comments can be published using markdown format as shown.

*do-something.py*
```python
import pull_request

comment = """
## Useful Table

| Item | Col1  | Col2  | Col3  | Col4  |
|---|---|---|---|---|
| 1 | a | b | c | d |
| 2 | a | b | c | d |
| 3 | a | b | c | d |

Some additional text here
"""
msg = pull_request.Message()
result = msg.add(comment=comment)
if result is False:
    print("Sending message failed")
```

If you now check the pull request you will find that the comment has been added!

![](/assets/img/ado-pull-request/pull-request-comment.png)

# ADO Pull Request Message Python Module

Below is the Python module I wrote which is used to add a comment to the pull request.  As you can see, most of the settings are derived from environment variables which are automatically set by the ADO pipeline.  An API call is used to post the data to the ADO API.

*pull_request.py*
```python
import requests
import os

class Message():
    def __init__(self):
        SYSTEM_COLLECTIONURI = os.getenv('SYSTEM_COLLECTIONURI')
        SYSTEM_PULLREQUEST_PULLREQUESTID = os.getenv('SYSTEM_PULLREQUEST_PULLREQUESTID')
        SYSTEM_TEAMPROJECT = os.getenv('SYSTEM_TEAMPROJECT')
        BUILD_REPOSITORY_ID = os.getenv('BUILD_REPOSITORY_ID')
        self.url = f"{SYSTEM_COLLECTIONURI}{SYSTEM_TEAMPROJECT}/_apis/git/repositories/" \
                   f"{BUILD_REPOSITORY_ID}/pullRequests/{SYSTEM_PULLREQUEST_PULLREQUESTID}" \
                   "/threads?api-version=6.0"
        self.headers = {
            "content-type": "application/json",
            "Authorization": f"BEARER {os.getenv('SYSTEM_ACCESSTOKEN')}"
        }

    def add(self, comment):
        ''' Add a message to Azure DevOps Pull Request'''
        data = {
            "comments": [
                {
                    "parentCommentId": 0,
                    "content": comment,
                    "commentType": 1
                }
            ],
            "status": 1
        }
        r = requests.post(url=self.url, json=data, headers=self.headers)
        if r.status_code == 200:
            return True
        else:
            return False
```