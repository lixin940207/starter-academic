---
title: Scalability
subtitle: clones, database, cache, asynchronism

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
- cache
- asynchronism

categories:
- system design
---

#### Scalability 1: horizontal scaling
- Add load balancer in front of servers
- Sessions need to be stored in a centralized data store which is accessible to all your application servers(can be cache like Redis). Or in AWS, we can enable stickiness in its own LB
- How can you make sure that a code change is sent to all your servers without one server still serving old code?
    + Tool: Capistrano(automates the process of making a new version of an application available on one or more web servers)
	+ We'd better make code compatible between version
	+ In AWS, several different development type: all at once, rolling, rolling with additional, mutable etc.
	
#### Scalability 2: Database(如果你的system越来越慢最后break down，说明是mysql的问题)
- Scale mysql: master-slave replication (read from slaves, write to master), upgrade your master server by adding RAM. 问题是需要自己考虑sharding, sql tuning, 之后的每一个new action都会很贵和time consuming
- Switch to nosql: 
	+ nosql 不支持join
	+ join操作可以在application code中完成
	+ 再加入cache

#### Scalability 3: Cache
- Buffering layer between application and data storage
- Store in ram
- Ex. Redis one hundred thousand times/s
- 2 patterns of cache:
	+ Cache database queris
		* 数据更新时，需要删除cache
	+ Cache objects
		* Store class, instances in cache
- Asynchronous processing
- Redis: extra database-features, like persistence and the built-in data structures like lists and sets. With Redis and a clever key there may be a chance that you even can get completely rid of a database.
- just need to cache, take Memcached, because it scales like a charm.

#### Scalability 4: Asynchronism
- Async 1:
	+ Do the time-consuming work in advance and serving the finished work with a low request time
	+ 用在：turn dynamic content into static content, everytime pages of websites change, may involve in massive framewok, these can be done on a regular basis to senf these pre-rendered HTML pages to AWS S3 or CDN. The webiste would be super scalable
- Async 2:
	+ Client send a request, request is sent to Job queue, and client receive a signal said request received
	+ a bunch of workers receive new job from the queue, and do the work separately, once finished send a signal: job done.
	+ Ex. rabbitMQ
