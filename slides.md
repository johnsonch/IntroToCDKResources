# Who am I

I'm good with the cloud and just ok with Typescript

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

Let's build a Lambda that writes a timestamp to a S3 bucket. This will let us see how CDK can work with multiple AWS resources.

---

## Install CDK
//TODO

---
## Setup
Setup the first project first create a directory and `cd` into it. I used `demo1`
```
cdk init app --language typescript
```
The `cdk init` takes a template and then a `--language` flag. We're using the `app` template and working in Typescript today

Then go and setup your AWS account, today we'll be using Acloud.guru

`aws configure`

^ Show the AWS account

---

## Bootstraping CDK
Now we need to bootstrap our envionrment

```
cdk bootstrap
```

^ Show the CDK stack and the resources it created

---
## Add some code
Now we can try add our code

Let's open up `lib/demo1-stack.ts` and start by adding our S3 bucket

```
const bucket = new s3.Bucket(this, 'MyBucket', {
  versioned: true,
});
```

For that to work we'll also need to import the  S3 library
```
import * as s3 from 'aws-cdk-lib/aws-s3';
```

Let's take a look at deploying this. Here is my workflow:
* `cdk diff --all`
* `cdk deploy --all`

There are some other tools like `synthesize` TODO explain the other options with `cdk`

But we don't have any errors so lets :fire: :down_right_arrow: :hole: with `cdk deploy --all`

Sweet that worked! Let's go on to a little more complex portion and we'll add the lambda function that writes to the bucket.

We're going to keep the lambda CDK code in the same file, at least for this example it will help us understand what we are doing.

Let's start by creating the Lambda, we'll use Ruby for this example.

Create a file `lib/lambda/index.rb` and add the following to the file:

```
require 'aws-sdk'

def handler(event:, context:)
  # The name of the bucket passed in by the creation of the function
  bucket = ENV['BUCKET_NAME']
  # Create a timestamp file name
  timestamp = Time.now.utc.strftime('%Y%m%dT%H%M%S%z')
  file_name = "timestamp-#{timestamp}.txt"

  # Write the timestamp file to S3
  s3 = Aws::S3::Client.new
  s3.put_object(bucket: bucket, key: file_name, body: timestamp)

  # Return a success response
  { statusCode: 200, body: 'Timestamp file written to S3' }
end
```

Now we can get the lambda function created and have it use the code we just wrote. In our `lib/demo1-stack.ts` file at the top add:
^ TODO: Explain these more
```
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as path from 'path';
import { Duration } from 'aws-cdk-lib';
```


Now below our S3 bucket lets add the following:

```
const lambdaFn = new lambda.Function(this, 'MyLambda', {
  runtime: lambda.Runtime.RUBY_2_7,
  handler: 'index.handler',
  code: lambda.Code.fromAsset(path.join(__dirname, 'lambda')),
  environment: {
    BUCKET_NAME: bucket.bucketName,
  },
  timeout: Duration.seconds(10),
});
```

Let's deploy this out, generate our diff first then deploy

Now we can try and run the funcation in the UI

Oops needs permissions to write to the bucket. This is where we learn that eventhough our code deploys fine it may not "work" as expected

Let's add `bucket.grantWrite(lambdaFn);` and then re-diff and deploy

Now we can create a test event and trigger our Lamda.

# Demo 2
## Multiple Stack Project

Next we're gonna build a stack to host a speed test.

We'll create the following:
* VPC, with subnets
* Application Load Balancer (ELB)
* ECS Fargate Cluster
* ECS Fargate Task (To run https://hub.docker.com/r/adolfintel/speedtest)
* S3 Bucket for Logging

```
mkdir demo2
cd demo2
cdk init app --language typescript
```

Remove the default stack from `bin/demo2.ts`, start with the import

```
import { Demo2Stack } from '../lib/demo2-stack';
```

and then move do the decleration

```
new Demo2Stack(app, 'Demo2Stack', {
  /* If you don't specify 'env', this stack will be environment-agnostic.
   * Account/Region-dependent features and context lookups will not work,
   * but a single synthesized template can be deployed anywhere. */

  /* Uncomment the next line to specialize this stack for the AWS Account
   * and Region that are implied by the current CLI configuration. */
  // env: { account: process.env.CDK_DEFAULT_ACCOUNT, region: process.env.CDK_DEFAULT_REGION },

  /* Uncomment the next line if you know exactly what Account and Region you
   * want to deploy the stack to. */
  // env: { account: '123456789012', region: 'us-east-1' },

  /* For more information, see https://docs.aws.amazon.com/cdk/latest/guide/environments.html */
});
```

Finally remove `lib/demo2-stack.ts` as well.

Now we'll start by adding our `lib/vpc-stack.ts` file and filling it out.

We're goig to change things up a bit here and import from `constructs` and `aws-cdk-lib`

```
import { Construct } from 'constructs';
import * as cdk from 'aws-cdk-lib';

export class VPCStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);
  }
}
```
The `scope` parameter represents the parent construct in which this stack will be defined. It is of type Construct, which is a base class for all constructs in the CDK.

We're gonna build our VPC stack and need to share information about it to other constructs we build for this project. It's a little foresight into what we are doing

I may be great with the cloud, but at times my deep explainations of specifics around languages such as javascript are not perfect!

Moving on, we'll want to add all the parts of a VPC

```
import { Construct } from 'constructs';
import * as cdk from 'aws-cdk-lib';
import * as ec2 from 'aws-cdk-lib/aws-ec2'

export class VPCStack extends cdk.Stack {
  // 👇 set a property for the vpc
  public readonly vpc: ec2.Vpc;

  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);
    this.vpc = new ec2.Vpc(
      this,
      'Vpc',
      {
        vpcName: 'Demo',
        ipAddresses: ec2.IpAddresses.cidr('192.168.0.0/16'),
        natGateways: 1,
        maxAzs: 2,
        subnetConfiguration: [
          {
            name: 'Public',
            subnetType: ec2.SubnetType.PUBLIC,
            cidrMask: 24
          },
          {
            name: 'Private',
            subnetType: ec2.SubnetType.PRIVATE_WITH_EGRESS,
            cidrMask: 24
          }
        ]
      }
    );
  }
}
```

Then we can create the "VPC Stack"

```
import { VPCStack } from '../lib/vpc-stack';

const vpcStack = new VPCStack(app, 'vpc-stack', {
  stackName: 'vpc-stack'
})
```

With that all in place we should be able to do a `cdk deploy --all` and have our stack build with a VPC!


