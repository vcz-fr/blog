---
layout: blog

title: "Deploying a complete infrastructure on the Huawei Cloud - Part 2"
categories: ["Infrastructure"]
tags: ["huawei", "kubernetes"]

date: "2021-02-05"
---

## Deploying a complete infrastructure on the Huawei Cloud - Part 2

*Using ELB, Kubernetes, Managed postgreSQL and Redis.*


If you miss the backbone infrastructure setup it's [HERE](Part1_link)

Our main goal today is to understand the different components needed by our applicative part to run on Huawei.

### Part 1: Docker Repository

To deploy our applicative stack on Kubernetes we will be using Docker containers.
Those containers will need an image to run and we choose to store the images on a Huawei managed repository called **SWR (SoftWare Repository)**.
A SWR will store your images and attach an HTTP link to each of them so you can pull the image directly from your Kubernetes cluster.
To upload your newly created image you can click on the “Upload threw client” button in the upper right part of the console. The console will give you a temporary login to the SWR. After that you’ll only need to tag and push your image like this:
- `docker tag [{Image Name}:{Tag name}] swr.ap-southeast-3.myhuaweicloud.com/{Organization Name}/{Image Name}:{Tag name}`
<br>

- `docker push swr.ap-southeast-3.myhuaweicloud.com/{Organization Name}/{Image Name}:{Tag name}`

After that you’ll see your image on the console and by clicking on it you’ll find a link to pull your newly uploaded image !

[SWR.JPG]

### Part 2: The Kubernetes Cluster

Deploy and maintain a vanilla kubernetes cluster can be really painful, that’s why cloud providers have created managed solution to handle things like master provisioning or rolling upgrade.The implementation of the Huawei managed kubernetes is really appreciable, it’s a mix of kubernetes, Rancher and Helm repository all implemented in one place and really easy to deploy. First lets review the Huawei CCE (Cloud Container Engine) cluster creation !

#### Cluster creation:
After clicking on the “Buy CCE cluster” you have several step to choose.
Right now 2 kubernetes versions are available (1.15 and 1.17), you can select the number of masters, 1 master will be cheaper but not highly available (if your master die your cluster will be unreachable) and 3 masters assure you a always available cluster.
> You can create a Node to link to your kubernetes cluster but this is optional because there is an interesting feature called NodePool that I’ll show you later!

After this step you have the add-ons. It’s a library of kubernetes packages that can be deployed on your cluster threw the console. You can for example enable horizontal pod autoscaler or Node autoscaling!
When all this steps are filled you can create your cluster and go to the node management part !

#### Node management

On your CCE panel go to `Resource Management > Nodepools`.
This feature allow you to create a pool of node of a certain type and attach it to your kubernetes cluster. It’s really useful if you want to mix several type of nodes, isolate workloads on certain machine type using **NodeAffinity** or enable node auto scaling to follow your workload !
When your cluster is created and a node pool attached to it you can start deploying your kubernetes resources on it !

> I will not deep dive into the kubernetes architecture of our client for obvious reasons but keep in mind that it’s a webapp accessible threw http on kubernetes.

Let’s now review the different features the CCE cluster gives us !

#### Features

First you have the **Dashboard view**, it’s really useful to have a direct view of the health of your cluster with key metrics like CPU of your nodes or network monitoring. It is customisable and work out of the box !

[DASHBOARD.JPG]

Earlier I’ve told you about a **Rancher like**. If you go to the Workloads tab you’ll see all your kubernetes internal resources and you have the ability to create / modify them directly from the interface ! It’s really useful when you want to easily debug a deployment for example or create a test workload from scratch.You can handle Deployments, Services, Secrets, etc !

Then you have the **charts tab**, this part allow you to integrate HELM charts directly into your cluster threw the interface. You can use Huawei sample charts like Ingress-Nginx and Elasticsearch or even upload yours which is really cool to keep a track of your custom charts.

The last part is the **System Steward**, it was really new to me and it’s an amazing feature to debug your k8s cluster.Basically the System Check is a real time monitoring of all your nodes, it contain key information such as Diskpressure, Docker check, Restarted container etc !
You have to take a look at it because it’s a really interesting implementation !

[SYSTEM.JPG]
