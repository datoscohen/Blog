---
title: "Sending Google Cloud Logging Notifications to Slack"
date: 2024-01-30T17:33:49Z
draft: false

categories: ['GCP']
tags: ['cloud logging', 'slack']
toc: true
author: 'Juan Cruz Parise'

featuredImage: 'articles/cloud-logging-slack/portada.png'
---

> TL;DR: Configure automatic notifications from Google Cloud Logging to Slack to receive specific alerts from your applications on GCP.

<!--more-->

When managing projects on Google Cloud Platform (GCP), having effective control of what happens in your project is essential. In this article, we will explore the step-by-step process of notifying Cloud Logging log alerts directly to a Slack channel.

## What is Google Cloud Logging?

It is a tool that allows real-time log management, with search, analysis, and storage capabilities.

To access Google Cloud Logging:

1. Go to [Google Cloud](https://console.cloud.google.com/)
2. Search for "Logging"
3. Once inside, you will see a Log Explorer console and real-time logs

### Logs Explorer

The console has a [specific syntax](https://cloud.google.com/logging/docs/view/building-queries) and allows filtering logs easily with words or [regular expressions](https://cloud.google.com/blog/products/management-tools/cloud-logging-gets-regular-expression-support).

![Logs explorer](/articles/cloud-logging-slack/log-explorer.png).

## Preliminary steps to create an alert

### Google Cloud Monitoring in Slack

To add Google Cloud Monitoring to Slack and invite it to a channel, you must follow the steps in this [link](https://cloud.google.com/monitoring/support/notification-options#slack)

### Necessary Permissions

The GCP permissions required for this procedure are:

- [Error Reporting User](https://cloud.google.com/iam/docs/understanding-roles#errorreporting.admin).
- [Monitoring Editor](https://cloud.google.com/iam/docs/understanding-roles#monitoring.editor).

### Add Slack Channel to Cloud Logging

To add a channel, click on **Manage Notification Channel**. It will open a new window where you can add a new Slack channel.

![Button to create alert](/articles/cloud-logging-slack/notification-channel.png)

Then, allow Google Cloud Monitoring to access Slack.

![Approval](/articles/cloud-logging-slack/aprobacion.png)

Once allowed, enter the name of the channel you want to add.

## Parameters for creating an alert

Once you've identified the logs you want to notify in the Log Explorer panel, proceed to create the alert. Simply click the 'Create Alert' button located in the same panel.

![Button to create alert](/articles/cloud-logging-slack/button-create-alert.png)

Then, complete a form, which includes the following parameters:

### 1: Alert details

- `Alert Policy Name`: The name that will appear in the Slack alert.
- `Policy Severity Level`: Select the severity level. Options are "No Severity," "Critical," "Error," and "Warning."
- `Documentation`: Documentation to include with the message, such as how to resolve the alert or referencing a wiki. You can also [format messages](https://cloud.google.com/monitoring/alerts/doc-variables?_ga=2.148459511.-194555348.1625487700) and include specific fields about the alert.

### 2: Choose logs to include in the alert**asd12

- `Define log entries to alert on`: Filters that Cloud Logging will consider to find the alert. For example, we'll look for logs indicating that the memory limit of an app running on CloudRun has been exceeded.
  
  ```yaml
  resource.type = "cloud_run_revision"
  severity=ERROR
  textPayload: "Memory limit of"
  ```
  
### 3: Set notification frequency and autoclose duration

- `Set notification frequency and autoclose duration`: The frequency with which notifications are sent, with a minimum time of 5 minutes.
- `Incident autoclose duration`: The period before an incident is automatically closed.

**Section 4: Who should be notified?**

- `Who should be notified?`: Mark the Slack channel where the notification will be sent.

![Step by step](/articles/cloud-logging-slack/parametros.png)

## Result

When the alert is triggered, a Slack message will be sent with the content (link to the pre-filtered log in Log Explorer, documentation, alert name, etc.), similar to the following example. 🚀

![Slack Message](/articles/cloud-logging-slack/resultado.png)
