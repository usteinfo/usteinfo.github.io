---
layout: fragment
title: 数据库查询优化方法
tags: [sql, 调优]
description: 本文说明一般Sql查询的优化步骤
keywords: sql, 调优
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

sql查询优化，主要关注于查询计划中，所有的表，最好不要有全表扫描，临时表，filesort等等

通过命令`explain sql`检查sql的查询计划，主要关注点有：

- join表on条件是否都使用了索引
- 条件中是否有对字段进行函数计算
- 如果在有group by 和 order by条件是否都使用了索引，如果不能同时使用索引时，当有limit语句时，优先group by使用索引，反之order by 使用索引
- 在有子查询时，验证子查询是否被强制优化的问题，可采用`show warings`显示最近执行的命令，来查询被优化的结果，可以采用hint的方式来禁止优化，常用的有取消合并优化，强制主驱动表，强制指定索引

