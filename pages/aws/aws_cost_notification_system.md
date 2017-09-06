---
title: AWS Email Notification System
keywords: aws, lambda, sns
last_updated: September 5, 2017
tags: [AWS, Lambda, SNS, cost]
summary: "AWS Email Notification System"
sidebar: mydoc_sidebar
permalink: aws_cost_notification_system.html
folder: aws
---

## Introduction
The Email notification system is designed to send cost summary on a daily basis to subscribed emails. This system is based on AWS Lambda serverless computing framework, Simple Notification Service (SNS) and Simple Storage Service (S3). The figure below illustrates the system architecture.

![pic0001](/documentation/images/aws/aws_cost_notification_system_001.png)

DLT has set up a pipeline that keeps billing record in an S3 bucket, named as 509248752274-dlt-utilization in the US East(N.Virginia) region. This bucket will gets updated on a daily basis. All the billing information is recorded in compressed csv format. The daily update will trigger the Lambda service, which is currently called "billing_test_jin" in the US East(N.Virginia) region. This service consists of a Python script that will download and parse the billing record from the S3 bucket. The script will aggregate cost information based on tags and resource type (e.g. Elastic Computing Service). For untagged billing, the script will summarize cost by resource ID if the resource ID appears in the file. Once the Lambda done with these, the SNS will be triggered. The SNS is currently named as "Billing_Notification" in the same region with the S3 bucket and the Lambda. The SNS will send Lambda cost summary to subscribed emails.

A detailed walkthrough of this system can be seen from the following.

## 1, Billing Information in the S3 bucket

The billing record is located in the "509248752274-dlt-utilization" S3 bucket of the US East(N.Virginia) region. The following screenshot shows the outline of this bucket. The billing information of a certain day can be found in the compressed csv file with the corresponding date in the file name. For example, the billing information of Aug, 9th 2017 will appear in the file named as "509248752274-aws-billing-detailed-line-items-with-resources-and-tags-2017-08.csv.zip". However, the billing information of the first day of every month might appear in the zipped csv file of the previous month.
![pic0001](/documentation/images/aws/aws_cost_notification_system_002.png)


## 2, 

## Links

## Heads-up
- The S3 bucket for billing information does not always get updated daily. The update might take a vacation during the holidays!!! So should we consider to send 2 day or 3 day summary information to users in order to avoid missing anything?

- All time related information is based on UTC time zone!

## Contact



{% include links.html %}