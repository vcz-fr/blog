---
layout: blog

title: "Storage in AWS: Dawn of a new era"
categories: ["Cloud"]
tags: ["repost", "gekko", "aws", "re:invent"]

date: "2020-12-13"
---

**Words of notice:** This is an article I wrote for [Gekko](https://www.gekko.fr/). Many thanks for their consent to
sharing this story here.

***

There are many different storage types in the wild and they each have specifics that make them suitable for defined sets
of use cases.

Regardless of the scale, storage poses challenging issues. For instance [Brewer's theorem](https://en.wikipedia.org/wiki/CAP_theorem){:rel="nofollow"},
also known as the CAP theorem, states that designing distributed data stores requires trade-offs. Simply put, in case of
a write failure on a storage system due to internal network conditions, would it be preferable to drop the operation or
to attempt to process it anyway? The first would guarantee consistency as the system becomes predictable while the
second guarantees availability as everything is processed to a certain degree. Of course this also applies to read
operations.

Even at the scale of AWS, there is still room to reinvent storage. Let's have a look at three major innovations that
will cadence the next years.

## Long live eventual consistency

There is a common misconception that [Amazon S3](https://aws.amazon.com/s3){:rel="nofollow"} stores files and
directories. That is not true: Amazon S3 addresses stored data as _objects_. There is no directory, consequently the
storage system hierarchy is flat. In fact, we could compare Amazon S3 to a key-value datastore where the key portion
would be the object "prefix", i.e. its path and its value would be the object contents.

On the subject of objects: unlike files, they are indivisible. If one needs to edit one character in a very large
object, the whole object must be replaced.

Amazon S3 is key to a large range of use cases: static web assets, low-cost storage and archival, analytics, datalakes,
etc. Still, because of the highly distributed nature of the service, some cases required more effort as some behaviors
were not completely predictable. In fact, some operations patterns were treated as "eventually consistent", which means
that they would _eventually_ reach the desired state.

This known unknown has been a major pain point, especially for projects heavily relying on Amazon S3's capabilities.
Here are four example scenarios that sum up the **old** consistency model:

![Old consistency model scenarios](/assets/img/posts/20201213/s3-old-consistency.png)  
_Old consistency model scenarios_

_Scenario 1_ is expected: a new object can be retrieved immediately after its creation. What if it is queried first,
like in _scenario 2_? Then Amazon S3 may have responded with an error or with the new object's content.  
_Scenario 3_ happens when an object is updated multiple times in quick successions. Immediately requesting the updated
object may have returned one of the previous versions and would eventually return the last.  
Finally, _scenario 4_ was similar to _scenario 3_ in the sense that a deletion followed by a query may have returned the
object in its previous state, followed by the updated version a few moments later.

Projects sitting on top of S3 had to create a layer to absorb these sorts of behaviors and expose a more predictable
interface. This involved thousand of lines of code using other adjacent services to store states and metadata, like [S3Guard](https://blog.cloudera.com/introducing-s3guard-s3-consistency-for-apache-hadoop/){:rel="nofollow"}.

For this reason, the teams behind Amazon S3 decided to tackle this issue. The challenge comes from how widespread this
service is. Customers expect their projects to behave "identically but better" after such an update. One of the ways to
achieve this goal was to get closer to communities developing critical tools around S3 like [S3A](https://archive.cloudera.com/cdh6/6.0.0/docs/hadoop-3.0.0-cdh6.0.0/hadoop-aws/tools/hadoop-aws/index.html#Introducing_the_Hadoop_S3A_client.){:rel="nofollow"},
a connector between Hadoop and Amazon S3. Both teams collaborated to make this connector ready in preparation for a
consistency model change, as stated in the related [blog post from AWS](https://aws.amazon.com/blogs/opensource/community-collaboration-the-s3a-story/){:rel="nofollow"}.


![New consistency model scenarios](/assets/img/posts/20201213/s3-new-consistency.png)  
_New consistency model scenarios_

Situations where the result was indeterminate in the old consistency model now have a response. This new model
implements "strong consistency", which means that reading an object just after it has been updated returns its new
contents! _Scenarios 2, 3 and 4_ demonstrate that updates to the object are taken into account when chaining Write and
Read requests, even when they are preceded by a Read like _scenario 2_. But is this the whole story? Applications
sometimes exchange asynchronously and concurrently with Amazon S3. How does this update affects these cases?


![Concurrent S3 updates](/assets/img/posts/20201213/s3-concurrency.png)  
_Concurrent S3 updates_

Two administrators are browsing a back-office application and update the same content. A is subject to bad network
conditions while B benefits from a fast and reliable connection. There is no way to predict what the Read operations
return; even though network conditions differ, both requests have reached Amazon S3 at roughly the same moment and
Amazon S3 applies a last-writer-wins strategy on content updates as stated [in the documentation](https://docs.aws.amazon.com/AmazonS3/latest/dev/Introduction.html#ApplicationConcurrency){:rel="nofollow"}.

The zoomed in timeline explores three scenarios where A goes first. The same scenarios apply if B goes first. Whether A
overlaps B entirely (Blue), partially (Green) or not at all (Red) is not visible by the application. This can however
influence the first Read response received by B: if A has not completed yet, B _may_ receive its own version due to the
strong consistency model. However, we know the other two Read responses will be identical since both Writes will be
processed by that point. In other words: in the eventuality of concurrent updates of the same object, the only way to
find out which version has been persisted is to perform a Read after all Writes have completed. Consequently, strong
consistency also adds some degree of predictability to concurrent scenarios.

If you wish to learn more about this change and how it may affect you, [read the announcement](https://aws.amazon.com/blogs/aws/amazon-s3-update-strong-read-after-write-consistency/){:rel="nofollow"}
or the [Strong consistency](https://aws.amazon.com/s3/consistency/){:rel="nofollow"} feature page for Amazon S3.

## Intermission: EBS storage and Shuffle Sharding

AWS uses a special type of sharding called "Shuffle sharding" to virtually isolate each customer while colocating their
resources. The most visible example, the one mentioned in the [Amazon Builders' Library](https://aws.amazon.com/builders-library/workload-isolation-using-shuffle-sharding/){:rel="nofollow"},
concerns Route 53 and how each DNS zone gets assigned four of the 2048 virtual shards created by AWS for the service.
There are 700B+ possible combinations of those shards and the number of combinations grows exponentially with the total
number of virtual shards and the number assigned to each DNS zone. What is this for? To isolate customers from each
other in case one receives an attack that knocks down their infrastructure. Colocating a group of customers on the same
physical infrastructure would cause downtime for all of them if one misbehaves.


![Shuffle sharding for EBS](/assets/img/posts/20201213/ebs-shuffle-sharding.png)  
_Shuffle sharding for EBS_

This colored diagram shows an EC2 instance with an EBS volume attached and how the actual data in the EBS is stored in
different shards. For simplicity, not all of them are included and there is no mention of encryption here. In this
model, the individual block can be tracked and assigned to a random combination of disks. All of them would need to fail
**at the same time** for data to be lost.

If multiple customers were assigned the same hardware and one overused their share the others would be impacted. By
assigning random disks on volume creation, this creates a layer of isolation but hosting workloads requires disks with
enough free space and changing disks whenever a resize becomes impossible. Not an optimal solution at scale.

Since these disks are used in block storage situations, it is possible to split the volume into blocks that may be
larger than the ones an EC2 instance uses and assign them random shards, which would map to disk portions. This solves
the free space and resize issues, provided the status of the shards and the location of each block in the system are
tracked. This design can be incremented upon with encryption and access control! What happens when one physical disk
fails, though?


![KO disk in EBS backend](/assets/img/posts/20201213/ebs-shuffle-sharding-ko.png)  
_KO disk in EBS backend_

One disk failed. The service endpoint keeps a note on the status of the related shards and trivially redirects requests
to healthy shards. Nevertheless, if all the shards connected to the same volume are suddenly worn out this could become
a major outage, except that... each volume has quotas and burst capacity and the service endpoint evenly balances the
load between multiple shards so that the wear is predictable and disk usage is optimal.

Speaking of quotas, IOPS optimized and general purpose Amazon EBS volumes have received major updates too! Read on to
learn more about that.

## IO -extra- optimized

A few years ago, a database administrator that knew my passion for extreme performance sent me a research article and an
advertisement on the topic of large relational databases. The advertisement contained a synthetic benchmark mentioning
millions of database transactions per second in a single piece of hardware.

These kinds of workloads are not a clear cut for the Cloud: dedicated specialized bare-metal hardware with
highest-performance everything to fully benefit from that hardware. With no obvious way to do this, it could be wise to
provide these features in a more generic fashion, like high-speed networking and storage capabilities with high-end
instances.


![The new EBS SSD line-up with a focus on io2](/assets/img/posts/20201213/no-r5b-focus.png)  
_The new EBS SSD line-up with a focus on io2_

With the release of Amazon EBS io2 volumes [earlier this year](https://aws.amazon.com/blogs/aws/new-ebs-volume-type-io2-more-iops-gib-higher-durability/){:rel="nofollow"},
building durable high-end storage systems became more cost-effective than ever. With a few asterisks: at the time of
writing, an io2 volume is still limited to 64k IOPS and 1000 MB/s throughput and no support for [EBS Multi-Attach](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volumes-multi.html){:rel="nofollow"},
released [earlier this year too](https://aws.amazon.com/blogs/aws/new-multi-attach-for-provisioned-iops-io1-amazon-ebs-volumes/){:rel="nofollow"}.
Moreover, the maximum 260k IOPS of the R5b instance family, released [in December of 2020](https://aws.amazon.com/about-aws/whats-new/2020/12/introducing-new-amazon-ec2-r5b-instances-featuring-60-gbps-of-ebs-bandwidth-and-260K-iops/){:rel="nofollow"},
cannot be reached. This generation increases the durability by one nine and the IOPS per GiB, which avoids storage
overprovisioning to reach the maximum performance **for a volume**.

Most of these issues have been resolved with a new Amazon EBS volume offering: io2 Block Express! Currently in Preview,
these volumes obsolete multiple high-performance EBS volumes attachments on the same Amazon EC2 instance in order to
reach the maximum performance **for an Amazon EC2 instance**, even for the R5b family! What's more is that thanks to the [Elastic Fabric Adapter](https://aws.amazon.com/hpc/efa/){:rel="nofollow"},
which is the network brick allowing HPC workloads to run on AWS, a volume can reach 4x the IOPS, 4x the throughput and
4x the storage size with sub-millisecond latencies and low jitter.

Remember the cost-effectiveness argument of io2? io2 Block Express goes beyond by doubling the IOPS per GiB and
decreasing the provisioned IOPS cost from 64k to 256k IOPS. io2 Block Express is therefore even more cost-effective and
reduces complexity.

If you want to learn more about io2 Block Express, then the [announcement blog post](https://aws.amazon.com/blogs/aws/now-in-preview-larger-faster-io2-ebs-volumes-with-higher-throughput/){:rel="nofollow"}
is what you need. There are a few caveats to account for since this feature is currently in Preview.


## gp3, truly customizable

As the name might suggest, general purpose volumes are the middle-ground in AWS. Any new feature will have far reaching
consequences on the Cloud community, let alone a new volume generation! This [newly announced](https://aws.amazon.com/blogs/aws/new-amazon-ebs-gp3-volume-lets-you-provision-performance-separate-from-capacity-and-offers-20-lower-price/){:rel="nofollow"}
volume type is gp3.

gp3 changes habits: there is no 100 IOPS baseline then 3 IOPS per GiB starting from 33.33 GiB with the ability to burst
to 3k IOPS for extended periods of time. 3k IOPS is the new baseline as is 125 MB/s throughput. Additional IOPS and
throughput can be provisioned independently. This comes with a 20% storage cost decrease compared to gp2, which makes
gp3 appealing to replace gp2 volumes and some io1 and io2 volumes too!

The legitimacy of increasing IOPS and throughput independently come from multiple reasons:
- There is a formula linking throughput to IOPS: `IOPS * average IO size = throughput`;
- IOPS counted towards the quota may differ from IOPS submitted by the OS. Amazon EBS merges contiguous operations into
  one and splits random operations [as per the documentation](https://aws.amazon.com/premiumsupport/knowledge-center/ebs-calculate-optimal-io-size/){:rel="nofollow"}.

Because the IO size is optimized by AWS, it is possible to reach the throughput limit before the IOPS quota.

On the subject of gp2 vs gp3, Cloudwiry wrote [a blog post](https://cloudwiry.com/ebs-gp3-vs-gp2-pricing-comparison/){:rel="nofollow"}
with cost comparisons between those volume types and they came to the conclusion that the upgrade is worth it in every
situation.

Finally, to optimize EBS usage, [AWS Compute Optimizer](https://aws.amazon.com/compute-optimizer/){:rel="nofollow"} now
supports [EBS volume recommendations](https://aws.amazon.com/about-aws/whats-new/2020/12/aws-compute-optimizer-supports-amazon-ebs-volume-recommendations/){:rel="nofollow"}!
Modifying the go-to volume type for the EC2 instances seems like the best time to upgrade the price-performance ratio of
an infrastructure.


## In a nutshell

While the perfect storage solution does not exist, this set of releases allows AWS customers to benefit from the latest
technological breakthroughs. Breakthroughs unlock new use cases and new use cases bring new exciting challenges to
solve.

If all these storage game changers convinced you to hack into your infrastructure, then it is worth checking out the [AWS Blog](https://aws.amazon.com/blogs){:rel="nofollow"}
for more exciting announcements!
