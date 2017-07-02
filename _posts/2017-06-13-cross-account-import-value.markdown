---
layout: post
title:  "Cross Account Import Value"
date:   2017-06-13 21:00:00 +1000
categories: aws
tags: aws cloudformation importvalue
---

__TL;DR__ _share CloudFormation output values between AWS accounts and regions. [Deploy](https://gist.github.com/chriscoombs/8d5717e9ab24bf8079b6562d16aa051d) a Lambda function to list the exported values and an SNS topic to share the data. Use a custom resource and `Fn::GetAtt` to import the values._

# context

In September 2016, AWS [announced](https://aws.amazon.com/blogs/aws/aws-cloudformation-update-yaml-cross-stack-references-simplified-substitution/) support for cross stack references (`Fn::ImportValue`) in CloudFormation.

> Many AWS customers use one “system” CloudFormation stack to set up their environment (VPCs, VPC subnets, security groups, IP addresses, and so forth) and several other “application” stacks to populate it (EC2 & RDS instances, message queues, and the like). Until now there was no easy way for the application stacks to reference resources created by the system stack.

However, cross stack references are [restricted](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-exports.html) to exports/imports within the same AWS account and region.

> To share information between stacks, export a stack’s output values. Other stacks that are in the __same AWS account and region__ can import the exported values.

# handler

An SNS topic provides the mechanism for sharing export data between AWS accounts and regions. The CloudFormation template below creates the SNS topic and a Lambda function to ListExports.

{% gist 8d5717e9ab24bf8079b6562d16aa051d %}

The resulting topic can then be passed to a custom resource as the service token.

{% highlight json %}
{
  "CrossAccountImportValue": {
    "Type": "AWS::CloudFormation::CustomResource",
    "Version": "1.0",
    "Properties": {
      "ServiceToken": {
        "Fn::Sub": "arn:aws:sns:REGION:ACCOUNT:cross-account-import-value"
      }
    }
  }
}
{% endhighlight %}

The custom resource will then hold all of the exports from the AWS account and region specified in the SNS topic ARN. To access these values call `Fn::GetAtt` with a reference to the custom resource and an export name.

{% highlight json %}
{
  "Fn::GetAtt": [
    "CrossAccountImportValue",
    "VPC"
  ]
}
{% endhighlight %}