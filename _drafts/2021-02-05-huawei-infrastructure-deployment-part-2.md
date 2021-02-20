---
layout: blog

title: "Deploying a complete infrastructure on the Huawei Cloud - Part 2"
categories: ["Cloud"]
tags: ["huawei", "kubernetes"]

date: "2021-02-12"
---

**Words of notice:** This is an article has been written by [Antoine Cavaillé](https://github.com/AntoineCavaille). Many
thanks for their consent to sharing this story here.

***

Before we begin, check out the [first part of the series](https://blog.vcz.fr/cloud/2021-02-05-huawei-infrastructure-deployment-part-1)
in case you missed it. It presents the backbone, the base layer of our infrastructure. This second part concentrates on
the different components needed by our application to run on Huawei Cloud.

## Part 1: Docker Repository

To deploy our application on Kubernetes, we will be using Docker containers. Those containers will need an image to run
and we choose to store the images on a Huawei Cloud managed Docker repository called [SoftWare Repository for Container](https://www.huaweicloud.com/intl/en-us/product/swr.html),
_SWR_ for short. An SWR will store your images and provide an HTTP address to each of them so that you can pull them
from your Kubernetes cluster. To upload your newly created image, you can click on the “Upload through client” button in
the upper right part of the Console. The Console will generate temporary credentials to the SWR. You will then be able
to tag and push your image like so:

```bash
docker tag [{Image Name}:{Tag name}] swr.ap-southeast-3.myhuaweicloud.com/{Organization Name}/{Image Name}:{Tag name}
docker push swr.ap-southeast-3.myhuaweicloud.com/{Organization Name}/{Image Name}:{Tag name}
```

After that you will see your image on the Console and by clicking on it you will find a link to pull it.

![SWR](/assets/img/posts/20210205/swr.png)

## Part 2: Kubernetes Cluster

Deploying and maintaining a vanilla Kubernetes cluster is challenging, that is why Cloud providers have created managed
services to handle common tasks such as master provisioning or rolling upgrades. The implementation of the Huawei Cloud
managed Kubernetes service is noteworthy; it is a mix of Kubernetes, Rancher and Helm repository all implemented at the
same place and easy to deploy. First, let's review the [Huawei CCE](https://www.huaweicloud.com/intl/en-us/product/cce.html)
_-Cloud Container Engine-_ cluster creation.

### Cluster creation
After clicking on “Buy CCE cluster” you have a few choices to make:
- The Kubernetes version: 1.15 or 1.17 at the time of writing;
- Number of masters.

One master will be less expensive but your cluster will not be highly available. That is, if your master fails then your
cluster will be unreachable. Three masters ensures high availability.

> You can create a node to link to your Kubernetes cluster but this is optional because there is an interesting feature
> called NodePool which we will explore later in this part.

After this step you have the add-ons. It is a library of Kubernetes packages that can be deployed on your cluster
through the console. You can for example enable horizontal pod autoscaler or Node autoscaling. When all this steps are
complete, you can create your cluster and move on to node management!

### Node management

On your CCE panel go to `Resource Management > Nodepools`. This feature allow you to create a pool of node of a certain
type and attach it to your Kubernetes cluster. It’s really useful if you want to mix several type of nodes, isolate
workloads on certain machine type using **NodeAffinity** or enable node auto scaling to follow your workload! When your
cluster is created and a node pool attached to it you can start deploying your Kubernetes resources on it!

> I will not deep dive into the Kubernetes architecture of our client for obvious reasons but keep in mind that it’s a
> webapp accessible through HTTP on Kubernetes.

Let’s now review the different features the CCE cluster gives us!

### Features

First you have the **Dashboard view**, it’s really useful to have a direct view of the health of your cluster with key
metrics like CPU of your nodes or network monitoring. It is customisable and work out of the box!

![Dashboard](/assets/img/posts/20210205/dashboard.png)

Earlier I’ve told you about a **Rancher like**. If you go to the Workloads tab you’ll see all your Kubernetes internal
resources and you have the ability to create / modify them directly from the interface! It’s really useful when you
want to easily debug a deployment for example or create a test workload from scratch.You can handle Deployments,
Services, Secrets, etc!

Then you have the **charts tab**, this part allow you to integrate HELM charts directly into your cluster through the
interface. You can use Huawei sample charts like Ingress-Nginx and Elasticsearch or even upload yours which is really
cool to keep a track of your custom charts.

The last part is the **System Steward**, it was really new to me and it’s an amazing feature to debug your k8s
cluster.Basically the System Check is a real time monitoring of all your nodes, it contain key information such as
Diskpressure, Docker check, Restarted container etc! You have to take a look at it because it’s a really interesting
implementation!

![System](/assets/img/posts/20210205/system.png)

Congratulations, your application can be reached using the ip address of the load balancer and run on entirely Huawei
managed services!

## Conclusion

I’ve never used the Huawei cloud before and I can think of a few things that I want to share with you:
- The Huawei cloud is quite complete to handle a multitude of infrastructure use-case on Asia or South America;
- It’s way easier to use and deploy services through the Console than with other cloud providers;
- The services are globally cheaper than other cloud providers and you can use their coupon system to reduce even more
  your billing;
- There is a lack of public documentation even if the support is really fast but not free! So if you’re stuck, a simple
  google search of your error message will not help!
- For now the Infra As Code approach is not really mature but they are actively working on it!

During my project I used only a small amount of Huawei cloud services, take the time to discover them all it is worth it
! This project was really interesting to deploy and I will definitely follow the evolution of this cloud through years.