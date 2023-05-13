# Who am I

---

# What is CDK

From Chat GPT:
"CDK stands for AWS Cloud Development Kit. It is an open-source software development framework that enables developers to define cloud infrastructure using familiar programming languages such as TypeScript, Python, Java, and C#. With CDK, you can define your infrastructure as code (IaC) using these high-level programming languages, which are then translated into AWS CloudFormation templates.

This approach allows developers to use the same tools, practices, and workflows they use for application code to define and provision infrastructure, which results in a more efficient and consistent way to manage infrastructure. Additionally, CDK provides a set of high-level constructs that abstract away the low-level details of AWS services, allowing developers to focus on the application's business logic."

---

# Why or why not CDK
* Difference between raw cloudformation and CDK
* Difference between terraform and CDK
* Difference between other and CDK

---

# Prep for demos

* Acloud.guru (my personal favorite)
* https://github.com/rebuy-de/aws-nuke
* Local Stack

^ A Safeway to work in the cloud is always a good idea. Here are some options, they all have different strengths and weaknesses

---

# Tools

## Local Stack
https://docs.localstack.cloud/getting-started/installation/
```
python3 -m pip install localstack
```
After installing run `localstack start` to start your environment

Then we need a wrapper to use CDK with loacal stack `npm install -g aws-cdk-local aws-cdk` from https://github.com/localstack/aws-cdk-local

## VS Code
* Not my favorite thing in the world, but makes working with typescript eaiser. I do have VIM setup that works but it took a bit

---

# Demo 1
## Single Stack Project

---

## Install CDK
//TODO

---
## Setup
Setup the first project first create a directory and `cd` into it. I used `demo1`
```
cdk init app --language typescript
```
Then go and setup your AWS account, today we'll be using Acloud.guru

`aws configure`

^ Show the AWS account

---

## Bootstraping CDK
Now we need to bootstrap our envionrment

```
cdk bootstrap
```

** Show the CDK stack and the resources it created

Now we can try add our code



My work flow
* `cdk diff --all`
* `cdk deploy --all`

There are some other tools like `synthesize` TODO explain the other options with `cdk`

# Demo 2
## Multiple Stack Project
