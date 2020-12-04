---
layout: blog

title: "Storage in AWS: Dawn of a new era"
categories: ["Cloud"]
tags: ["repost", "gekko", "aws", "re:invent"]

date: "2020-12-03"
---

There are many different storage types in the wild and even though all of them ultimately fall back on blocks, they have
specifics that make them suitable for defined sets of use cases.

Regardless of the scale, storage poses challenging issues. For instance, you may have heard of [Brewer's theorem](
https://en.wikipedia.org/wiki/CAP_theorem), also known as the CAP theorem, which states that you will need to make
trade-offs when designing distributed data stores. Simply put, in case of a write failure on a partitioned system due to
internal network conditions, would it be better to drop the write or to attempt to process it anyway? The first would
guarantee consistency as the system becomes predictable while the second guarantees availability as everything is
processed to a certain degree. Of course this also applies to read operations.

Even at the scale of AWS, there is still room to reinvent storage. Let's have a look at three major innovations that
will cadence the next years.

## Long live eventual consistency

There is a common misconception that Amazon S3 stores files and directories. That is not true: Amazon S3 addresses
stored data as _objects_. There is no directory, consequently the storage system hierarchy is flat. In fact, we could go
further by comparing Amazon S3 to a key-value store where the key portion would be the object path, also called a "key"
and its value would be the object contents.

Objects, unlike blocks or files, are indivisible. A read or write operation affects the whole object, that means that if
you need to edit one character il a very large object, the whole object must be sent anew.

Amazon S3 is key to a large range of use cases: static web assets, low-cost storage and archival, analytics, datalakes.

https://aws.amazon.com/blogs/aws/amazon-s3-update-strong-read-after-write-consistency/
https://news.ycombinator.com/item?id=25271791
https://en.wikipedia.org/wiki/CAP_theorem#History
https://aws.amazon.com/blogs/opensource/community-collaboration-the-s3a-story/
https://aws.amazon.com/s3/consistency/

Object creation 

(Read PDF)

## IO -extra- optimized

https://aws.amazon.com/blogs/aws/now-in-preview-larger-faster-io2-ebs-volumes-with-higher-throughput/
https://aws.amazon.com/ebs/features/
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html

Uses EFA from HPC in the background
Not HPC-compatible: use instance store
Ideal for enterprise-grade databases such as SAP HANA, Oracle, Microsoft SQL Server
Sub-ms latency, a first on EBS!

4x max from io2 non BE:
Max IOPS/Volume: 256,000
Max Throughput*/Volume: 4,000 MB/s
vs.
Max IOPS*/Volume: 64,000
Max Throughput**/Volume: 1,000 MB/s

Mention io2 vs io1, recent release and game changer as well

(Read PDF)

## gp3, truly customizable

https://aws.amazon.com/blogs/aws/new-amazon-ebs-gp3-volume-lets-you-provision-performance-separate-from-capacity-and-offers-20-lower-price/
https://aws.amazon.com/about-aws/whats-new/2020/12/aws-compute-optimizer-supports-amazon-ebs-volume-recommendations/
https://cloudwiry.com/ebs-gp3-vs-gp2-pricing-comparison/

20% more cost-effective for storage compared to gp2
Strong baseline performance 3k iops and 125MBps

gp2 performance increases with volume capacity vs. gp3 storage capacity, throughput or iops can be independently
increased 4x max throughput but same max iops

-> throughput and iops are linked, for block storage iops x blocksize = throughput

compute cost to match gp3 baseline with gp2

(Read PDF)

## In a nutshell

While the perfect storage solution does not exist, this set of releases allows AWS customers to benefit from the latest
technological breakthroughs. Breakthroughs unlock new use cases and new use cases bring new challenges to solve. Our 

If all these storage game changers convinced you to hack into your infrastructure, then it is worth checking out the [AWS Blog](https://aws.amazon.com/blogs)
for more exciting announcements!