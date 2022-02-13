---
title: "Developer experience in AWS compute services"
categories: ["Cloud"]
tags: ["cloud", "aws", "ec2", "ecs", "lambda"]
---

Developer experience. This is one reason why we write and use abstractions, why we are comfortable with using
technologies which can hardly be called "optimized", why we write, maintain and integrate with APIs so that the services
we provide work.

The Cloud not only delivered on the promise to commoditize infrastructure so that you do not have to go through lengthy
processes to order and manage the whole stack, but it provided something else: programmability! Thanks to the many
layers of abstractions implemented by the current providers, we are able to order complex services through an API. Even
better: sometimes a development kit is provided so the integration becomes transparent, just a function call away!

<!-- READ MORE -->

As a developer, integrating with services with a function call is a game changer. Not having to provide credentials to
use those services is even better. By the way:
- [Read this](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) if you deploy to [Amazon EC2](https://aws.amazon.com/ec2/);
- [Read this](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html) if you deploy to [Amazon ECS](https://aws.amazon.com/ecs/);
- [Read this](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html) if you deploy to [AWS Lambda](https://aws.amazon.com/lambda/).

There is however one thing that 

##