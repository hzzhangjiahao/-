# Presto,Hive,GP性能对比

```
Presto在app111，app114,app115上部署了三个worker，调整了如下配置项： 
exchange.http-client.request-timeout=120s
presto完全基于网络在节点之间传输数据，如果超时时间太短，在网络拥塞时可能使集群误认为节点已挂，导致任务失败。
-Xmx15G
query.max-memory-per-node=3GB
一个查询任务在每个节点占用的最大内存，presto默认该值应小于或等于0.4*Xmx。
query.max-memory=8GB
一个查询任务占用的最大内存，即分布在各节点的内存总和，它决定了一次查询的数据量最大能有多少。
以上三个值如果太小，会导致查询终止，如果太大，超过了系统可用内存，则可能导致进程被系统kill。
```

### 测试数据

表a(id, age, sex)

表b(id, payment, job)

两张表的id一一对应，其他字段均在集合中随机取值，各生成5千万条数据。GP中表a按照age分区，表b按照job分区。

### 测试结果

select job,count(*) as count from prestotestage a inner join prestotestjob j on a.id=j.id group by job;

|                | time |
| -------------- | ---- |
| GP             | 8s   |
| hive           | 167s |
| presto on GP   | 165s |
| presto on hive | 14s  |

select age,job,payment,count(*) as count from prestotestage a inner join prestotestjob j on a.id=j.id where payment > '15k' group by age,job,payment;

|                | time   |
| -------------- | ------ |
| GP             | 37s    |
| hive           | 171s   |
| presto on GP   | 查询结果错误 |
| presto on hive | 30s    |

create table prestoresult as select age,sex,job,payment from prestotestage a inner join prestotestjob j on a.id=j.id where age>20 and age<26;

|                | time   |
| -------------- | ------ |
| GP             | 27s    |
| hive           | 151s   |
| presto on GP   | 查询结果错误 |
| presto on hive | 36s    |

#### 结果分析

presto是在PB数据规模下替代hive的交互式sql查询引擎，在本次测试中因机器内存有限，测试数据只在GB规模，即便如此，presto也比hive快了5-10倍。GP也很快，但它不是presto的替代品，这里只做参考。

#### 遗留问题

在使用1.5亿条数据测试的时候，worker节点很容易在查询开始后挂掉。通过尝试几套配置参数，发现分配给presto的可占用内存略小于系统剩余可用内存，但无法满足查询任务需求时，任务会因内存不足而终止，presto进程不会挂。而如果分配给presto的可占用内存接近系统剩余可用内存，而且查询任务会用完这些内存，那么presto进程一定会被系统kill，即使它启动时通过了内存参数的检查。

后来测试数据减少到5千万条，分配了合理的内存，进程被kill的现象就没有了。Facebook的示范文档中，给单个查询任务分配的总内存是50GB，在每个节点上分配的的内存是1GB。可以推断Facebook推荐至少部署50个worker节点，而且每个节点上的hive表不应过大，至少在一次查询中，每个节点处理的数据不超过1GB。暂时没有找到权威的信息证明这个推测，仅供参考。