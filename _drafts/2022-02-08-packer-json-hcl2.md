---
title: "Packer: From JSON to HCL2"
categories: ["Cloud"]
tags: ["cloud", "packer", "aws", "ec2"]
---

The current state of the Web encourages organizations to rely on standards and common interchange formats. Today, JSON
is one of the interchange formats that gained universal traction in the technical community.

Many technologies natively support JSON. While it became a _de facto standard_, it isn't without its shortcomings:
- It does not support comments;
- It does not support some common data types: integers, bytes, dates, chars. You can cheat using supported types
  however;
- It does not support references to other parts of the payload;
- It does not enforce data formats. Schema validators will not **enforce** a format, only validate that a payload
  follows a format.

Another shortcoming of JSON is its verbosity. Even though it is far from XML, editing JSON files can become a chore,
especially when you need to explain what you are doing and why you are doing it. Checking service boundaries or logging
is fine, however as soon as it is used for configuration then these shortcomings come to bite.

<!-- READ MORE -->

## Packer started with JSON

[HashiCorp Packer](https://www.packer.io/) is a technology that is able to build and distribute [EC2 Amazon Machine Images](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html).
AMIs are resources that provide crucial information to Amazon EC2 to create instances. If you deploy services to Amazon
EC2 then you have heard of Packer or its AWS alternative [EC2 Image Builder](https://aws.amazon.com/image-builder/).

Up until version 1.5.0 released December 19th of 2019, Packer only supported configurations written in JSON. While it
does the job, JSON configurations has to fit inside a single file. One could write configuration in another language,
say YAML, then process it to JSON before interpreting it with Packer. This process is unwieldy and not recommended,
though.

Starting from version 1.5.0 and generally availabile since version 1.7.0 released February 17th 2021, a second
configuration language for Packer that is reminiscent of the experience on other HashiCorp products like Terraform and
Consul. This language is called **HCL2**, which is short for HashiCorp Configuration Language 2.

## Packer shines anew with HCL2

Just like JSON or YAML, HCL2 is a descriptive configuration language; its role is to _describe_ the path between data
**sources** and **destinations**. For Packer, data sources include user provided variables and Cloud provider APIs, such
as AMI, VPC, Subnet, Security Group search APIs. Destinations include "source configurations" and "build
configurations", which in Packer configure the source of the generated AMI (EBS, Instance store, surrogate build).



https://learn.hashicorp.com/tutorials/packer/hcl2-upgrade