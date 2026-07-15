---
title : "Introduction"
date : 2024-01-01 
weight : 1 
chapter : false
pre : " <b> 5.1. </b> "
---


#### AWS Serverless & AI Introduction
+ **AWS Serverless** is a cloud computing execution model that allows you to build and run applications without provisioning or managing physical servers. The system scales automatically and charges only for actual resource consumption.
+ **Artificial Intelligence (AI)** integrated via Amazon Bedrock provides powerful Foundation Models to process smart features such as multi-language translation and automated food recommendations for customers.

#### Workshop overview
In this workshop, you will build the **SmartMenu** system using a Serverless architecture integrated with AI on AWS, featuring the following key components:
+ **Frontend:** Client and Admin web interfaces developed with ReactJS & Vite, hosted on Amazon S3 and distributed via Amazon CloudFront.
+ **Backend & API:** Amazon API Gateway routes incoming requests to AWS Lambda, which handles business logic such as menu management, ordering, and translation.
+ **Database:** Amazon DynamoDB stores menu and order data with high performance and low latency.
+ **AI Integration:** Amazon Bedrock is used to connect with Foundation Models to power automated translation and food recommendations.

![overview](/images/5-Workshop/5.1-Workshop-overview/smartmenu_architecture.png)
