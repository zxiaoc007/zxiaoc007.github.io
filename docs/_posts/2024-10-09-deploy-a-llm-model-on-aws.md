---
layout: post
title: "Deploy a LLM model on AWS"
date: 2024-10-09 20:57:00 -0000
categories: LLM, Deployment, Cloud
---

# System Design of Deploying a LLM model on AWS

Training a LLM model is difficult and time-resouces consuming. Most of the time, people just use an open-source model from facehugging, or from OpenAI API, to support their own applications. 

This blog is an application of using a Meta Llambda Large Language model, to build an application that can detect harmful ingredients from a label photo taken by your phone. Also launch the application through AWS, so that users can use it anytime that they want.

The application website is https://bit.ly/Artificialingredients

A quick demo of the application (TBA)

## 1. System design of the application

The application contains both front-end and back-end. Let's take a look of the overall view of the system design of the whole application.

![Image](/images/diagram.png)

When user uploads an image to the application, it first arrives at the UI and does some pre-processing to the image, and then sends to the end-back application, which is hosted on AWS Cloud.

Let's dive into the back-end world a bit more in the next session.

## 2. Back-end Services

There are a coupld of cloud services used in the back-end to achieve the goal. Let's go through them one by one.

### 2.1 AWS API Gateway

The first stop of the backend is a AWS API Gateway. 

Amazon API Gateway is an AWS service for creating, publishing, maintaining, monitoring, and securing REST, HTTP, and WebSocket APIs at any scale. 

When the request arrives at API Gateway, it will be transferred to AWS Lambda Service, which is the key service in the back-end.

### 2.2 AWS Lambda Service

AWS Lambda is an event-driven, serverless compute service for running code without having to provision or manage servers. You pay only for the compute time you consume.

You organize your code into Lambda functions. Example of Lambda function: https://github.com/zxiaoc007/llm_lambda/blob/master/app.py

In this applciation, we created these following events in Lambda function.
1. Convert image array into base64 string, and upload to S3
2. Get text and coords from image through Amazon Rekognition
3. Draw text box on original images
4. Sent text to LLM using the AWS Bedrock Runtime client and get a reply
5. Return body {reply, image_out}

### 2.3 Amazon Rekognition

One of the lambda function is to call Amazon Rekognition service to extract the text from the image that's sent to the back-end.

Amazon Rekognition is a cloud-based image and video analysis service that makes it easy to add advanced computer vision capabilities to your applications.

It is simple, easy-to-use API that can quickly analyze any image or video file that’s stored in Amazon S3

Reference: https://docs.aws.amazon.com/rekognition/latest/dg/what-is.html

### 2.4 AWS Bedrock Runtime Client

Another AWS service that's being used is Bedrock Runtime Client. This services is used to call a Large Language Model that's already hosted on AWS and is quoted based on the token used in each call.

This document has a detailed instruction about how to use AWS Bedrock Runtime client to invoke Meta Model? https://docs.aws.amazon.com/code-library/latest/ug/bedrock-runtime_example_bedrock-runtime_InvokeModel_MetaLlama3_section.html

Another topic worth digging is the price of the model:
https://aws.amazon.com/bedrock/pricing/

The LLM model that's being used in this application is meta.llama3-8b-instruct-v1:0
![Image](/images/llama.png)

The prompt that's being sent to LLM is:

'''
Does the ingredients of the product contain any potentially harmful chemicals?

The text below are from OCR of the product packaging. Answer in scientific details but don't repeat all the ingredients. 

Just discuss any potential harmful chemicals. Also please output the answer in HTML friendly compatible format (not markdown). 

And display the text in the html in this language: {language}
'''

There are 3 configurations that's assigned in the LLM model:
1. Max_token
2. temperature
3. top_p

# 

Lastly, the back-end will return a LLM reply with an image having text-detect box onto it, and everything is returned back through UI to the user.

Example:

![Image](/images/return.jpeg)
![Image](/images/example.jpeg)

Feel free to try out this application and give us any feedback!

Github code link: https://github.com/zxiaoc007/llm_lambda

Thanks for reading here!