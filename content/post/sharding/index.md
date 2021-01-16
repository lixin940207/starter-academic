---
title: Challenges of database sharding

summary: When it comes to sharding MySQL clusters, significant challenges can be faced. Take a look at some of them, how to address them, and why they're worth overcoming.


# Link this post with a project
projects: []

# Date published
date: "2016-04-20T00:00:00Z"

# Date updated
lastmod: "2020-12-13T00:00:00Z"

# Is this an unpublished draft?
draft: false

# Show this page in the Featured widget?
featured: false

authors:
- admin

tags:
- scalability
- database

categories:
- system design
---
Database Sharding
- Database sharding is the process of making partitions of data in a database, such that the data is divided into various smaller distinct shards.
- Each shard could be a table or part of a table.
- There are horizontal sharding(水平分) and vertical sharding(竖直分)
- Advantages of sharding：
    + High availability: shards are independent one shard lost, other shards still operate
	+ Faster queries: smaller amount of data to query each time
	+ Write in parallel
	+ Don't need master slave replication, And as load increases the cost of replication increases. Replication costs in CPU, network bandwidth, and disk IO. The slaves fall behind and have stale data.
- Problems of sharding
	+ Rebalancing data: a shard outgrows the storage and need to be split; moving data from shard to shard required a lot of downtime.
	+ Join: cannot join in the shard level, must make join in the application level
	+ How to partition? 怎么选partition key是很难的
	+ Implement shard on your own

