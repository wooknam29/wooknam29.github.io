---
title: "starrocks"
date: 2024-02-20T16:25:00+08:00
# draft: true
---

## 特性和使用场景
### 特性
1. StarRocks 采用 MPP (Massively Parallel Processing) 大规模并行执行框架  
- MPP 计算模型实现：每个 task 被绑定到持有该数据的 Executor 上，由于 SR 表的分组分桶特性，数据会按 key 分组存储到节点上，相当于每个节点都拥有部分数据，那么使得单个查询请求可以充分利用所有执行节点的资源。  
优点：充分利用所有执行节点资源  
缺点：容易由于单节点速度形成短板效应

- MapReduce 计算模型实现：每个 task 会被随机分配到空闲的 Executor 上，但是该节点可能并不持有数据，所以需要经过数据的 Shuffle （移动到空闲的计算节点）才能计算。    
优点：推测执行可以避免单节点执行过慢问题  
缺点：数据移动，落盘，资源申请等

2. 全面向量化执行引擎  
SIMD、批量按列执行

3. 定制化的CBO优化器  
估算每个算子的执行代价，选择代价最低的一条查询路径作为最终的物理查询计划。  
该优化器内部实现了CTEs复用、相关子查询重写、Lateral Joins, Join Reordering、Join 分布式执行策略选择、低基数优化。

4. 智能的物化加速视图  
自动刷新无需外部维护调度任务，组织灵活，且能够在查询时进行 SQL 改写，最大程度使用物化结果集。

5. 全面的湖仓分析能力  
可以作为计算引擎，分析数据湖中的数据。

### 使用场景
OLAP多维分析、实时&统一数据查询。

## 实践
- 提升链路流转效益：  
消费实时数据用Routine Load方式确保实现Exactly-Once语义，保证数据不丢不重。  
离线转存数据通过Broker Load和Spark Load等方式导入，且都保证了导入的原子性。

- 湖仓融合:  
StarRocks 提供两种类型 Catalog：internal catalog 和 external catalog，其中前者可以查询 SR 内表，后者可以访问数据湖表如 iceberg 表，以及传统数仓表如 hive 和 thive 表。  

- 极速查询性能  
自助查询场景：利用colocated groups特性加速了join的效率；通过bitmap字段的操作，加速计算count和画像表关联的效率。

### 为什么starrocks如此的快？   
1. 向量化执行引擎：列式地组织和处理数据；在内存中存储和组织数据，列式的计算sql算子，能够充分利用CPU缓存，减少虚拟函数调用和分支判断；SIMD；编码数据的处理技术，省去解码时间。  
2. Cost-based Optimizer:使得多表join查询性能更好，尤其是复杂多表join查询。  
3. bitmap indexing:位图索引。快速过滤。  
4. 低基数字段的全局字典：2.0版给低基数的字段构建了全局字典，使得从string型转到了int型。2.0优化了字典编码、表达式计算、运算符计算去极大地提升count distinct和group by的性能。用户在创建表的时候不需要重新导入数据并定义数据类型，相反，starrocks自动地在前序查询结束后给低基数字段的构造了字典，这种查询加速对用户是透明的。  
5. 晚物化：晚物化通过延迟load数据来减少数据的扫描量。先load year colunm，然后在该column做预先地过滤，之后获得这些row的number。再用这些row的number获取到数据。减少了数据行的扫描量。  
6. starrocks作为数据湖：可以作为一个查询引擎查询数据湖的数据。导入starrocks，可以有很多特性：分区、分桶（修剪数据减少数据的扫描量）、排序（提高数据的区域性）、索引（加速查询）、colocated groups（本地化join，在tableA 和 tableB 建表时，在 PROPERTIES 中指定属性 "colocate_with" = "group_name"，那么同一个 group_name 中的表数据具有一致的分桶键且数据分布在相同的一组BE 节点上，当 Join 列为分桶键时，计算节点只需做本地 Join，从而减少数据在节点间的传输耗时，减少shuffle操作）、Local joins（加速表关联）
