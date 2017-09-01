---
title: AWS Account Management
keywords: aws
last_updated: January 26, 2017
tags: [AWS, cloud_basics, console, api, account_management]
summary: "AWS account management"
sidebar: mydoc_sidebar
permalink: aws_account_management.html
folder: aws
---

## Introduction

The purpose of this page is to describe your core AWS account management 
tasks in brief and then again in further detail.  


## Topic: Understanding the purpose and cost of the Config Service with Rule Evaluations.
In our first case study we are using such Rules to detect non-compliant tagging.

To make progress we will need the concept of a Configuration Item. This is abbreviated CI. It is an event associated with AWS resources and it can act as a Rule Evaluation Trigger. So let's start by assuming Config is not enabled yet and go from there.  

First let us enable Config and associate it with resources: EC2 and S3 and RDS and so forth. There is no cost here.

Now: Every time one of these associated resources undergoes a change in its configuration e.g. an S3 bucket goes from unencrypted to encrypted or you change the volume of an EBS: That generates a CI. This costs $0.003 *because* it is associated with the Config service. As a data point this cost us $1.18 this month. Not too many CI items charged.

Now let's create a Config Rule. This costs $2. And you associate it with CIs and the CI is the trigger for the evaluation of this rule. This can happen 20,000 times per month at no cost. Notice that 20,000 CIs would also cost 20,000 x 0.003 = $60 in a month. But it is very unlikely that we would get up in that range; our monthly CI bill is more like $1. 

After those 20,000 no-charge evaluations of the Rule subsequently we are charged $0.10 for every 1000 rule evaluations. 

see also: http://www.dlt.com/sr/contracts/internet2/internet2-faq.pdf



## Links

## Warnings

## Task overview

- Set up your account to have "all green checkmarks". This means you are logged out and back in as an **admin**.
- Start and Stop an EC2 instance: Know what it costs, what EBS is, and log in to it before you Stop it.
- Study up on cloud tech, enough to understand cost, security and capacity; and then make a plan. 

{% include links.html %}
