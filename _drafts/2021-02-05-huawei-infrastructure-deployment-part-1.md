---
layout: blog

title: "Deploying a complete infrastructure on Huawei Cloud"
categories: ["Infrastructure"]
tags: ["huawei", "kubernetes"]

date: "2021-02-05"
---

## Some elements of context

As a Cloud Engineer at Gekko Part of Accenture, I am comfortable running complex infrastructure on AWS which provides
datacenters all over the world. But in one project, a customer wished to replicate their infrastructure on a different
Cloud provider, Huawei Cloud! I was only aware of the existence of Huawei hardware such as smartphones, computers and IT
oriented products but not as a cloud provider so I decided to share my experience about running an app on Huawei Cloud
using as many managed services as possible.

> This topic will be divided into two blog posts: this one for the backbone infrastructure and the next will concentrate
> on Kubernetes and the application.

Here are the components that we are going to setup:
- 1 VPC
- 1 single subnet across 3 AZs
- 1 ELB
- 1 Kubernetes cluster
- 1 PostgreSQL
- 1 Redis
- 1 Docker repository
- 1 Bastion host
- Elastic IPs

![Global infrastructure](/assets/img/posts/20210205/infra_global.png)

### Why Huawei cloud?

In this case the customer deployed a web application on AWS in France. For latency purposes they have decided to deploy
the same application on a datacenter in Asia. They were leaning towards Huawei Cloud as it is generally less expensive
and contains the managed services they needed for their application to run.

### On Infrastructure As Code

In this blog post I will not use Infrastructure As Code _-IaC-_, everything will be deployed through the web interface,
called the Console. Nevertheless, there is a [community Terraform provider](https://github.com/huaweicloud/terraform-provider-huaweicloud)
for Huawei Cloud. All the resources I need were not available on the provider during the course of the project. The
first version of the provider was released the 18th of July, 2018 and it is updated often therefore it is possible the provider could be complete and usable for this project now!

### Here we go!

It is time to build our application step by step. First, we need a Landing zone on our Huawei Cloud account to create
the resources needed by our application. We will start by creating a [Virtual Private Cloud](https://www.huaweicloud.com/en-us/product/vpc.html)
(VPC) with a single subnet in one AZ. Since our VPC is not accessible from the internet we will create a
bastion [Elastic Cloud Server](https://www.huaweicloud.com/en-us/product/ecs.html) (ECS) instance and associate it with
a private key and an [Elastic IP](https://www.huaweicloud.com/en-us/product/eip.html).

You can find the Huawei console [here](https://auth.huaweicloud.com/).

## Part I: The landing zone

Let's start with the VPC, the base of our infrastructure.

### VPC and Subnets

You can create one under `Virtual Private Cloud > Buy VPC` in the Huawei Console.
Select a Region, a name for the VPC and a CIDR block. You can use the recommended VPCs to get started faster.

> Hint: A /16 at the end of a CIDR block translates to 256 * 256 = 65,536 IP addresses.

![VPC](/assets/img/posts/20210205/vpc.png)

Go to `Advanced Settings` to create a default subnet.

> We choose the default /24 CIDR block allowing us to use 256 IP addresses, which will be sufficient for our future
> cluster. Keep in mind that it is good practice to split different services onto separate subnets; that is a standard
> approach to handle security and reliability on Cloud infrastructures.

![Subnets](/assets/img/posts/20210205/subnets.png)

### NAT Gateway

Our infrastructure needs a limited access to the outside world to update itself, etc. There is a service for that in the
VPC panel: [NAT Gateway](https://www.huaweicloud.com/en-us/product/nat.html), of the Public kind in this case. You only
need to create a Public NAT Gateway and attach it to your private subnet. When this step is complete your Kubernetes
cluster may be connected to the Internet!

Now that we have our first layer up and running we need to have the ability to access it and for that we will use a [Bastion host](https://en.wikipedia.org/wiki/Bastion_host).

> Because our VPC is private, it is impossible to directly reach our subnet from the Internet. A bastion is an instance that will allow us to access our private network to operate on the infrastructure.

Lets go to the ECS panel to launch our instance.

### Bastion

A Bastion is a simple server on our private subnet with an inbound gate to the Internet like a Public IP. With this
host, you can access your private VPC.

I will help you understand the different steps needed to create your instance. These are common to many resources so it
could be relevant to understand them sooner.

_Billing mode:_

Yearly or Monthly - It is a way to reserve an instance for a given period of time. This means you will
not pay hourly rates as it is common with most Cloud providers but monthly or yearly rates. This is the less expensive
way if you can predict the capacity you will need.

On-Demand - This is the default billing mode. Launch an instance and pay by the hour. This option is the best if you
need instances for short amounts of time or if you cannot reliably predict the compute capacity you will need. It is the
mode we will choose for our Bastion host since it will be shut down after our tests.

Spot price - Cloud providers always have some leftover compute capacity thus they have created the Spot instances mode
for that purpose. The price is variable and based on current demand. Let's say an instance costs a baseline of $0.50 per
hour. You could choose to pay $0.20 for this instance and if no one is using it, then it is yours. But if anyone places
$0.30 on that same instance or if no more compute capacity is available then it will terminate. This type of
billing is useful for preemptible and non-critical jobs for instance, that is jobs that you can interrupt at any time and that are not time-constrained.

_Region:_

This represents the available geographical locations for your instances. In our case, we will choose AP-Singapore.

_AZ (Availability Zone):_

An AZ is a subdivision of a Region. For reliability purposes a cloud region is composed of different datacenters placed
in the same general area. A datacenter corresponds to one AZ. For now we will choose `Random` since the location of the Bastion is not important here.

_CPU Architecture:_

x86 is the historical platform for server computing. More and more workloads tend to natively support or to migrate to
the more efficient ARM instruction sets. Huawei Cloud gives the choice between Intel x86 and home-made [Kunpeng](https://e.huawei.com/en/products/servers/kunpeng)
ARM processors. We will stick with Intel's x86.

_Flavor type:_

This is where you customize your instance. Select the optimal CPU capacity, RAM capacity and Price for your application.You will have the choice between compute-optimized instances, memory-optimized instances, general purpose instances, etc. For us, a general purpose T6 with 1 vCPU and 2GB of RAM instance will do since it will be used to access other resources and for maintenance.

_Security Group:_

Security groups contain inbound and outbound stateful network rules to secure your instance. This implies that any request which passes though your security group will be checked once and only once and if any rule matches the request, it and its response may be allowed to pass through. Security groups may be attached to one or multiple instances.

By default, no one is allowed to talk to your instance; you will need to grant inbound accesses. For SSH, add an inbound rule opening port 22 to your IP. That is roughly the configuration of the default security group.

_EIP:_

You can attach static IP addresses to your instances to make them addressable from the Internet. Use the lowest values for your Bastion host.

_ECS Name:_

The name of your instance on the Huawei Cloud Console.

Know that all these steps are handled, hit create!

![Bastion](/assets/img/posts/20210205/bastion.png)

## Part 2: The Database (PostgreSQL)

To run, our application need a relational database.
Like I said at the beginning of this blog post we want to use as many managed services as possible.Huawei let us use a service called RDS (Relational Database Service) to store our database on the cloud ! By clicking the RDS service then “Buy db instance”, You can create a new managed Postgresql.
For our use case we will use a PostgreSQL 11 in Singapore and use the Pay-per-use billing mode.The RDS service is quite customisable, you can choose 3 types of db (MySql, Postgresql and Microsoft SQL Server), Use a Primary/Standby infrastructure if you need highly available databases, select your instance type and it’s storage, the AZ, etc.
When you RDS is up and running click on it from the console to familiarise yourself with the interface. As you can see you can create Read Replicas to reduce the load on your master, you can failover your master by using the Primary/Standby button, dynamically scale your storage and see all the informations that you need to connect to your instance.

![rds](/assets/img/posts/20210205/rds.png)

## Part 4: Redis

In addition with the relational DB we need a data structure store.
For that you can use the Huawei DCS (Distributed Cache Service). For us it will be a Redis instance. 
Here again when you click the “Buy DCS instance” button you’ll see plenty of flags available to customise.
Here are the main ones:
- Cache engine: Redis or Memcached
- Instance type will allow you to enable active/passive cache for Backup, Failover and/or Persistence or Redis Cluster to ingest huge load on your Data-Structure storage.
- You can also select the number of replicas or the Sharding of your nodes if needed !

When your cluster is up and running you can click on it and select Performance Monitoring, this view is quite useful to have a better understanding of what is happening on your Redis.

![Redis](/assets/img/posts/20210205/redis.png)


## Part 5: Load Balancer

Soon we will create our Kubernetes cluster. That means our application will be running on the cloud and we will to make it available to our clients.
The best way to “open” our platform (which run on a private network if you remember) is to link it to a Load Balancer. This element will make a bridge between the external world and our infrastructure !

You have 2 types of Huawei LB:
- Dedicated: Ensure that your traffic goes threw dedicated resources only (better performance but higher pricing), more customisable, possibilities to do cross AZ deployment
- Shared: Deployed in cluster mode with backend instances shared with other LB, free of charges.

We will go for a Shared LB. On the creation page you can see several options like the IP address you want to bind it to, the billing mode you want (Bandwith for stable heavy traffic or Traffic for light or spiked traffic) and the Bandwidth size (5mbit/s to 2gbit/s for Bandwith mode or 1mbit/s to 300mbit/s for Traffic mode).

In our case we want users to access our resources deployed on kubernetes. For that we have multiple solutions to link our LB to our workload.

- The first one is to create a Service configuration directly from the CCE panel (under Resource Managment > Network > Services), during the Service configuration we can choose an existing load balancer. It’s the easier way to do it because your kubernetes resources will automatically create listeners on your load balancer and deploy the corespondent resources inside your k8s cluster. The negative part of that is you need to deploy your k8s workloads threw the console and not with yml based file or helm charts and it’s not the best practice !

- The second one is only threw the CLI and need manual actions but it allow you to link any workloads to your existing Load balancer. For that you have to manually edit the nginx-ingress-controller service (if you use the ingress-nginx routing component for example) and fill the kubernetes.io/elb.* annotations with the informations of your previously created Load balancer.

- In the future a third method could be available with custom automatic annotations like service.beta.kubernetes.io/aws-load-balancer for AWS but it has not been released yet !

Congratulations, the skeleton of your application is ready to onboard it's first kubernetes cluster and run on entirely Huawei managed services !

In the next article i'll present you the managed solution to run your application on a Huawei Kubernetes cluster !

