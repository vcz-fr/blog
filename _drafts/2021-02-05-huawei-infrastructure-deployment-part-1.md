---
layout: blog

title: "Deploying a complete infrastructure on the Huawei Cloud"
categories: ["Infrastructure"]
tags: ["huawei", "kubernetes"]

date: "2021-02-05"
---

## Deploying a complete infrastructure on the Huawei Cloud

*Using ELB, Kubernetes, Managed postgreSQL and Redis.*


As a Cloud Engineer at GEKKO, I’m comfortable running complex infrastructure on the AWS cloud which have datacenter all over the world. But for a particular case one of our client mandate us to replicate his infrastructure in standalone on a new cloud provider, the Huawei cloud !
I was only aware of the existence of Huawei threw phones, computers, and IT oriented products but not as a cloud provider. So I’ve decided to write a blog post about it to share my experience about running an app on Huawei using as much manages services as I could.

> This topic will be treated with 2 blog posts, one for the backbone infrastructure and the part two will handle the kubernetes + applicative part !

Here are the different components that we are going to use:
- 1 VPC
- 1 single subnet across 3 AZ
- 1 ELB
- 1 Kubernetes cluster
- 1 PostrgreSQL
- 1 Redis
- 1 Docker repository
- 1 Bastion host
- Elastic Ips

[INFRA_GLOBAL.PNG]


#### Why the Huawei cloud ?

In our case our client have a web application deployed in France on AWS, for latency purposes they’ve decided to deploy the application on a new datacenter in Asia. Their choice was oriented toward Huawei and his cloud service because it was cheaper and contain all the managed services needed by their application to run.

#### Will you use Infra As Code ?

In this blog post I will not use IaC, everything will be deployed threw the console. But there is a Terraform provider upgraded and maintained by the community. I did not use it because all the resources that I need are not disponible on the provider. The first version of the provider was released the 18 July of 2018 and it’s updated daily so maybe at the time you’ll read this post the provider will be complete and usable !

[Huawei Provider Link](https://github.com/huaweicloud/terraform-provider-huaweicloud)

#### Here we go !

It’s time to build our app step by step. To begin with we need a landing zone on our Huawei account to create the several resources needed by our application.We will start by creating a VPC with a single subnet across AZ. Since our VPC is private and not accessible from the internet we will create a bastion and associate it with a private key and an Elastic IP.

You can find the Huawei console [HERE](https://auth.huaweicloud.com/)

### Part I: The landing zone

First we need to create our VPC, it will be the base of your infrastructure.

##### VPC and Subnets
You can create one under `Virtual Private Cloud > Buy VPC` from the Huawei Console.
Select a Region, a name for your VPC and a CIDR block (you can use the recommended VPCs).

> Hint: A /16 at the end of your CIDR block will give you a total of 65,536 IP adresses that can be used by your different services.

Then on Advanced Settings create your default subnet.

> We choose the default /24 CIDR block allowing us to use 256 IP adresses, it will be sufficient for our newly created cluster but keep in mind that it can be a good practice to split our different services on separate subnets. Allowing us a better approach to handle security and reliability on our devices in the future.

[VPC.PNG]

##### NAT Gateway

Our infrastructure need a limited access to the outside world to download packages, etc. For that a tool called NAT gateway is available on the VPC panel.You only need to create a public NAT Gateway and attach it to your vpc private subnet. When this step is complete your kubernetes cluster will be connected to the internet for specific actions !

Now that we have our first layer up and running we need to have the ability to access it !
For that we will use a [Bastion host](https://en.wikipedia.org/wiki/Bastion_host).

> Why ? Because our VPC is private, it means that it’s impossible to reach our subnet from the internet, that’s a good way to secure our resources that don’t need to have an access to the outside world. A bastion is an instance that will allow us to enter our private network to connect/maintain/secure the infrastructure.


Lets go to the ECS panel to launch our instance.

##### Bastion

A Bastion is a simple server on our private subnet with an inbound gate to the internet (Like a Public IP). With this host you can access your private VPC to handle run operations for example.

I will help you understand the different topic needed to create your instance (They will be used on other resources later in this post so it’s good to understand them).

_Billing mode:_
Yearly/Monthly - it’s a way to reserve an instance for a given period of time, then you will not pay as the hour (the most known way to bill your instance in the cloud) but as a fixed period of time (Monthly or Yearly), it is the cheaper way if you have predictions about the compute capacity you’ll need on a certain amount of time.
On-Demand - This is the default cloud billing method, launch an instance and pay every hour. This option is the best if you need instances for short amount of time or if you do not know the compute capacity you’ll need. It’s the method we will choose for our Bastion since it’s for a test.
Spot price - This one is a little bit more complex. Cloud provider have always instances that are ready to run but not used by their customers. So they’ve created the spot instances to use this sleeping compute capacity. The billing is based on bets. Lets say an instance cost 0.5$ an hour. You can choose to pay 0.2$ for this instance, if no one is using it, then it’s yours. But if someone bet 0.3$ on the same instance or if no more compute capacity is available then you’ll lose your device. This type of billing is useful for non-critical jobs for example.

_Region:_
This is the different geographical places available to run your instances, since we choose to deploy our different	services in Singapore we will choose AP-Singapore.

_AZ (Availibility Zone):_
This is a subdivision of your region. Basically for reliability purposes, a cloud region is composed of différents datacenter placed in the same area. A datacenter = an Availibility Zone.
For now we will choose Random since we do not care on which datacenter our Bastion is spawned since it’s in Singapore.

_CPU Architecture:_
This depend of the cloud provider you are using, here on Huawei you have the choice between classical Intel instructions set x86 and the Kunpeng which run ARM instructions set.
We will stick with a classical x86 cpu for now

_Flavor type:_
This is the part were you’ll chose your “server”. Basically select the best ratio CPU/RAM/Price for your application.You’ll have the choice to select Processor based instances, memory based instances, balanced instances, etc depending of your needs and the price associated with them.For us it will be a classical balanced instances (t3.medium with 1CPU and 2Gb of RAM) since it’ll be used only to jump on other resources or to make some maintenance.

_Security Group:_
Security groups are composed of inbound and outbound rules to secure your instance.
By default no one can talk to your instance, you’ll have to open flow so you can access it.For us we will access it threw SSH so we need to have an open port 22 in inbound.It’s the case with the default security group.

_EIP:_
This part permit us to attach an internet facing static ip address to our instance.It’s a mandatory part for us since we want to access this instance directly from the internet. We will stick with the default lowest values here.

_ECS Name:_
The name of your instance on the Huawei console

Know that all the topic are understand we can hit create !

[BASTION.JPG] 

### Part 2: The Database (PostgreSQL)

To run, our application need a relational database.
Like I said at the beginning of this blog post we want to use as many managed services as possible.Huawei let us use a service called RDS (Relational Database Service) to store our database on the cloud ! By clicking the RDS service then “Buy db instance”, You can create a new managed Postgresql.
For our use case we will use a PostgreSQL 11 in Singapore and use the Pay-per-use billing mode.The RDS service is quite customisable, you can choose 3 types of db (MySql, Postgresql and Microsoft SQL Server), Use a Primary/Standby infrastructure if you need highly available databases, select your instance type and it’s storage, the AZ, etc.
When you RDS is up and running click on it from the console to familiarise yourself with the interface. As you can see you can create Read Replicas to reduce the load on your master, you can failover your master by using the Primary/Standby button, dynamically scale your storage and see all the informations that you need to connect to your instance.

[RDS.JPG]

### Part 4: Redis

In addition with the relational DB we need a data structure store.
For that you can use the Huawei DCS (Distributed Cache Service). For us it will be a Redis instance. 
Here again when you click the “Buy DCS instance” button you’ll see plenty of flags available to customise.
Here are the main ones:
- Cache engine: Redis or Memcached
- Instance type will allow you to enable active/passive cache for Backup, Failover and/or Persistence or Redis Cluster to ingest huge load on your Data-Structure storage.
- You can also select the number of replicas or the Sharding of your nodes if needed !

When your cluster is up and running you can click on it and select Performance Monitoring, this view is quite useful to have a better understanding of what is happening on your Redis.

[REDIS.JPG]


### Part 5: Load Balancer

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

