---
title: How to optimized MySQL?

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

- 对查询进行优化，要尽量避免全表扫描，首先应考虑在where 及 order by 涉及的列 上建立索引
- 应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引 而进行全表扫描
- 应尽量避免在 where 子句中使用 != 或 <> 操作符，否则将引擎放弃使用索引而进行 全表扫描
- 应尽量避免在 where 子句中使用 or 来连接条件，如果一个字段有索引，一个字段没有索引，将导致引擎放弃使用索引而进行全表扫描
- Update 语句，如果只更改 1、2 个字段，不要 Update全部字段，否则频繁调用会引 起明显的性能消耗，同时带来大量日志 
- 对于多张大数据量(这里几百条就算大了)的表 JOIN，要先分页再 JOIN，否则逻 辑读会很高，性能很差。

- 熟悉业务，善用index，（where及order by，group by涉及的列上建立索引）避免使用！= ，or，in，not in操作，这样数据库无法使用索引，进行全表搜索。
- 不用select * from（因为从数据库读越多的数据，查询就会很慢）
- 使用查询缓存，并将尽量多的内存分配给MYSQL做缓存
- 尽量使用enum，而不是varchar
- 在使用索引字段作为条件时，如果该索引是复合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使用，并且应尽可能的让字段顺序与索引顺序相一致。
- 批量插入一定要用bulk_insert
- 善用group by
