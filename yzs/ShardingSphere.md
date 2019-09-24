#                   ShardingSphere

# 概述

ShardingSphere是一套开源的分布式数据库中间件解决方案组成的生态圈,  它由Sharding-JDBC 、Sharding-Proxy和Sharding-Sidecar (计划中) 这三款相互独立的产品组成.  他们均提供标准化的数据分片、分布式事物和数据库治理功能, 可适用于Java同构、异构语言、云原生等各种多样化的应用场景.

ShardingSphere定位为关系型数据库中间件, 旨在充分合理地在分布式的场景下利用关系型数据库的计算和存储能力, 而并非实现一个全新的关系型数据库.  它与NoSQL和NewSQL是并存而非互斥的关系.  NoSQL和NewSQL作为新技术探索的前沿, 放眼未来, 拥抱变化 是非常值得推荐的. 反之也可以用另一种思路看待问题, 放眼未来, 关注不变的东西, 进而抓住事物的本质.  关系型数据库当今依然有巨大的市场, 是各个公司核心业务的基石, 未来也难以撼动, 我们目前阶段更加关注在原有基础上的增量, 而非颠覆.

ShardingSphere目前已经进入[Apache孵化器](http://incubator.apache.org/projects/shardingsphere.html)

![img](https://shardingsphere.apache.org/document/current/img/shardingsphere-scope_cn.png)



## 简介

### Sharding-JDBC

定位为轻量级Java框架, 在Java和JDBC层提供额外的服务.  它使用客户端直连数据库, 以jar包形式提供服务, 无需额外部署和依赖, 可理解为增强型的JDBC驱动,  完全兼容JDBC和各种ORM框架.

- 适用于任何基于Java的ORM框架，如：JPA, Hibernate, Mybatis, Spring JDBC Template或直接使用JDBC。
- 基于任何第三方的数据库连接池，如：DBCP, C3P0, BoneCP, Druid, HikariCP等。
- 支持任意实现JDBC规范的数据库。目前支持MySQL，Oracle，SQLServer和PostgreSQL。

![img](https://shardingsphere.apache.org/document/current/img/sharding-jdbc-brief.png)

### Sharding-Proxy

定位为透明化的数据库代理端, 提供封装了数据库二进制协议的服务端版本, 对于完成对异构语言的支持.  目前首先提供MySQL版本,  它可以支持任何兼容MySQL协议的访问客户端(如: MySQL Command Client, MySQL Workbench等) 操作数据, 对DBA更加友好.

- 向应用程序完全透明，可直接当做MySQL使用。
- 适用于任何兼容MySQL协议的客户端。

![Sharding-Proxy Architecture](https://shardingsphere.apache.org/document/current/img/sharding-proxy-brief_v2.png)

### Sharding-Sidecar（TODO）

定位为Kubernetes的云原生数据库代理，以Sidecar的形式代理所有对数据库的访问。 通过无中心、零侵入的方案提供与数据库交互的的啮合层，即Database Mesh，又可称数据网格。

Database Mesh的关注重点在于如何将分布式的数据访问应用与数据库有机串联起来，它更加关注的是交互，是将杂乱无章的应用与数据库之间的交互有效的梳理。使用Database Mesh，访问数据库的应用和数据库终将形成一个巨大的网格体系，应用和数据库只需在网格体系中对号入座即可，它们都是被啮合层所治理的对象。

![Sharding-Sidecar Architecture](https://shardingsphere.apache.org/document/current/img/sharding-sidecar-brief_v2.png)

|            | *Sharding-JDBC* | *Sharding-Proxy* | *Sharding-Sidecar* |
| :--------- | :-------------: | :--------------: | :----------------: |
| 数据库     |      任意       |      MySQL       |       MySQL        |
| 连接消耗数 |       高        |        低        |         高         |
| 异构语言   |     仅Java      |       任意       |        任意        |
| 性能       |     损耗低      |     损耗略高     |       损耗低       |
| 无中心化   |       是        |        否        |         是         |
| 静态入口   |       无        |        有        |         无         |

### 混合架构

Sharding-JDBC采用无中心化架构，适用于Java开发的高性能的轻量级

[^OLTP是传统的关系型数据库的主要应用，主要是基本的、日常的事务处理，记录即时的增、删、改、查]: 

应用；Sharding-Proxy提供静态入口以及异构语言的支持，适用于OLAP应用以及对分片数据库进行管理和运维的场景。

ShardingSphere是多接入端共同组成的生态圈。 通过混合使用Sharding-JDBC和Sharding-Proxy，并采用同一注册中心统一配置分片策略，能够灵活的搭建适用于各种场景的应用系统，架构师可以更加自由的调整适合于当前业务的最佳系统架构。

![ShardingSphere Hybrid Architecture](https://shardingsphere.apache.org/document/current/img/shardingsphere-hybrid.png)

### 功能列表

#### 数据分片

- 分库 & 分表
- 读写分离
- 分片策略定制化
- 无中心化分布式主键

#### 分布式事务

- 标准化事务接口
- XA强一致事务
- 柔性事务

#### 数据库治理

- 配置动态化
- 编排 & 治理
- 数据脱敏
- 可视化链路追踪
- 弹性伸缩(规划中)

### 项目状态

![Status](https://shardingsphere.apache.org/document/current/img/shardingsphere-status_cn.png)

# 概念&功能

## 数据分片

### 背景

传统的将数据集中存储至单一数据节点的解决方案，在性能、可用性和运维成本这三方面已经难于满足互联网的海量数据场景。

从性能方面来说，由于关系型数据库大多采用B+树类型的索引，在数据量超过阈值的情况下，索引深度的增加也将使得磁盘访问的IO次数增加，进而导致查询性能的下降；同时，高并发访问请求也使得集中式数据库成为系统的最大瓶颈。

从可用性的方面来讲，服务化的无状态性，能够达到较小成本的随意扩容，这必然导致系统的最终压力都落在数据库之上。而单一的数据节点，或者简单的主从架构，已经越来越难以承担。数据库的可用性，已成为整个系统的关键。

从运维成本方面考虑，当一个数据库实例中的数据达到阈值以上，对于DBA的运维压力就会增大。数据备份和恢复的时间成本都将随着数据量的大小而愈发不可控。一般来讲，单一数据库实例的数据的阈值在1TB之内，是比较合理的范围。

在传统的关系型数据库无法满足互联网场景需要的情况下，将数据存储至原生支持分布式的NoSQL的尝试越来越多。 但NoSQL对SQL的不兼容性以及生态圈的不完善，使得它们在与关系型数据库的博弈中始终无法完成致命一击，而关系型数据库的地位却依然不可撼动。

数据分片指按照某个维度将存放在单一数据库中的数据分散地存放至多个数据库或表中以达到提升性能瓶颈以及可用性的效果。 数据分片的有效手段是对关系型数据库进行分库和分表。分库和分表均可以有效的避免由数据量超过可承受阈值而产生的查询瓶颈。 除此之外，分库还能够用于有效的分散对数据库单点的访问量；分表虽然无法缓解数据库压力，但却能够提供尽量将分布式事务转化为本地事务的可能，一旦涉及到跨库的更新操作，分布式事务往往会使问题变得复杂。 使用多主多从的分片方式，可以有效的避免数据单点，从而提升数据架构的可用性。

通过分库和分表进行数据的拆分来使得各个表的数据量保持在阈值以下，以及对流量进行疏导应对高访问量，是应对高并发和海量数据系统的有效手段。 数据分片的拆分方式又分为垂直分片和水平分片。

#### 垂直分片

按照业务拆分的方式称为垂直分片，又称为纵向拆分，它的核心理念是专库专用。 在拆分之前，一个数据库由多个数据表构成，每个表对应着不同的业务。而拆分之后，则是按照业务将表进行归类，分布到不同的数据库中，从而将压力分散至不同的数据库。 下图展示了根据业务需要，将用户表和订单表垂直分片到不同的数据库的方案。

![åç´åç](https://shardingsphere.apache.org/document/current/img/sharding/vertical_sharding.png)

垂直分片往往需要对架构和设计进行调整。通常来讲，是来不及应对互联网业务需求快速变化的；而且，它也并无法真正的解决单点瓶颈。 垂直拆分可以缓解数据量和访问量带来的问题，但无法根治。如果垂直拆分之后，表中的数据量依然超过单节点所能承载的阈值，则需要水平分片来进一步处理。

#### 水平分片

水平分片又称为横向拆分。 相对于垂直分片，它不再将数据根据业务逻辑分类，而是通过某个字段（或某几个字段），根据某种规则将数据分散至多个库或表中，每个分片仅包含数据的一部分。 例如：根据主键分片，偶数主键的记录放入0库（或表），奇数主键的记录放入1库（或表），如下图所示。

![æ°´å¹³åç](https://shardingsphere.apache.org/document/current/img/sharding/horizontal_sharding.png)

水平分片从理论上突破了单机数据量处理的瓶颈，并且扩展相对自由，是分库分表的标准解决方案。

### 挑战

虽然数据分片解决了性能、可用性以及单点备份恢复等问题，但分布式的架构在获得了收益的同时，也引入了新的问题。

面对如此散乱的分库分表之后的数据，应用开发工程师和数据库管理员对数据库的操作变得异常繁重就是其中的重要挑战之一。他们需要知道数据需要从哪个具体的数据库的分表中获取。

另一个挑战则是，能够正确的运行在单节点数据库中的SQL，在分片之后的数据库中并不一定能够正确运行。例如，分表导致表名称的修改，或者分页、排序、聚合分组等操作的不正确处理。

跨库事务也是分布式的数据库集群要面对的棘手事情。 合理采用分表，可以在降低单表数据量的情况下，尽量使用本地事务，善于使用同库不同表可有效避免分布式事务带来的麻烦。 在不能避免跨库事务的场景，有些业务仍然需要保持事务的一致性。 而基于XA的分布式事务由于在并发度高的场景中性能无法满足需要，并未被互联网巨头大规模使用，他们大多采用最终一致性的柔性事务代替强一致事务。

### 目标

**尽量透明化分库分表所带来的影响，让使用方尽量像使用一个数据库一样使用水平分片之后的数据库集群，是ShardingSphere数据分片模块的主要设计目标。**

### 核心概念

本小节主要介绍数据分片的核心概念，主要包括：

- SQL核心概念
- 分片核心概念
- 配置核心概念

#### SQL

##### 逻辑表

水平拆分的数据库（表）的相同逻辑和数据结构表的总称。例：订单数据根据主键尾数拆分为10张表，分别是`t_order_0`到`t_order_9`，他们的逻辑表名为`t_order`。

##### 真实表

在分片的数据库中真实存在的物理表。即上个示例中的`t_order_0`到`t_order_9`。

##### 数据节点

数据分片的最小单元。由数据源名称和数据表组成，例：`ds_0.t_order_0`。

##### 绑定表

指分片规则一致的主表和子表。例如：`t_order`表和`t_order_item`表，均按照`order_id`分片，则此两张表互为绑定表关系。绑定表之间的多表关联查询不会出现笛卡尔积关联，关联查询效率将大大提升。举例说明，如果SQL为：

```sql
SELECT i.* FROM t_order o JOIN t_order_item i ON o.order_id=i.order_id WHERE o.order_id in (10, 11);
```

在不配置绑定表关系时，假设分片键`order_id`将数值10路由至第0片，将数值11路由至第1片，那么路由后的SQL应该为4条，它们呈现为笛卡尔积：

```
SELECT i.* FROM t_order_0 o JOIN t_order_item_0 i ON o.order_id=i.order_id WHERE o.order_id in (10, 11);

SELECT i.* FROM t_order_0 o JOIN t_order_item_1 i ON o.order_id=i.order_id WHERE o.order_id in (10, 11);

SELECT i.* FROM t_order_1 o JOIN t_order_item_0 i ON o.order_id=i.order_id WHERE o.order_id in (10, 11);

SELECT i.* FROM t_order_1 o JOIN t_order_item_1 i ON o.order_id=i.order_id WHERE o.order_id in (10, 11);
```

在配置绑定表关系后，路由的SQL应该为2条：

```
SELECT i.* FROM t_order_0 o JOIN t_order_item_0 i ON o.order_id=i.order_id WHERE o.order_id in (10, 11);

SELECT i.* FROM t_order_1 o JOIN t_order_item_1 i ON o.order_id=i.order_id WHERE o.order_id in (10, 11);
```

其中`t_order`在FROM的最左侧，ShardingSphere将会以它作为整个绑定表的主表。 所有路由计算将会只使用主表的策略，那么`t_order_item`表的分片计算将会使用`t_order`的条件。故绑定表之间的分区键要完全相同。

##### 广播表

指所有的分片数据源中都存在的表，表结构和表中的数据在每个数据库中均完全一致。适用于数据量不大且需要与海量数据的表进行关联查询的场景，例如：字典表。

#### 分片

##### 分片键

用于分片的数据库字段，是将数据库(表)水平拆分的关键字段。例：将订单表中的订单主键的尾数取模分片，则订单主键为分片字段。 SQL中如果无分片字段，将执行全路由，性能较差。 除了对单分片字段的支持，ShardingSphere也支持根据多个字段进行分片。

##### 分片算法

通过分片算法将数据分片，支持通过`=`、`BETWEEN`和`IN`分片。分片算法需要应用方开发者自行实现，可实现的灵活度非常高。

目前提供4种分片算法。由于分片算法和业务实现紧密相关，因此并未提供内置分片算法，而是通过分片策略将各种场景提炼出来，提供更高层级的抽象，并提供接口让应用开发者自行实现分片算法。

###### 精确分片算法

对应PerciseShardingAlgorithm, 用于处理使用单一键作为分片键的 = 与 IN 进行分片的场景. 需要配合StandardShardingStrategy使用.

###### 范围分片算法

对应RangeShardingAlgorithm, 用于处理使用单一键作为分片键的 BETWEEN AND 进行分片的场景.  需要配合StandardShardingStrategy使用.

###### 复合分片算法

对应ComplexKeysShardingAlgorithm, 用于处理使用多键作为分片键进行分片的场景.  包含多个分片键的逻辑较复杂,  需要应用开发者自行处理其中的复杂度. 需要配合ComplexShardingStrategy使用.

###### Hint分片算法

对应HintShardingAlgorithm, 用于处理使用Hint行分片的场景.  需要配合HintShardingStrategy使用.

##### 分片策略

包含分片键和分片算法,  由于分片算法的独立性, 将其独立抽离. 真正可用于分片操作的是 分片键+分片算法,  也就是分片策略.  目前提供5中分片策略.

###### 标准分片策略

对应StandardShardingStrategy.  提供对SQL语句中的 = , IN 和 BETWEEN AND 进行分片操作支持. StandardShardingStrategy只支持单分片键. 提供PerciseShardingAlgorithm和RangeShardingAlgorithm两个分片算法.  PerciseShardingAlgorithm是必选的,  用于处理 = 和 IN 的分片.  RangeShardingAlgorithm 是可选的.  用于处理 BETWEEN AND 分片, 如果不配置 RangeShardingAlgorithm , SQL中的BETWEEN AND 将安装全库路由处理.

###### 复合分片策略

对应ComplexShardingShrategy. 复合分片策略。提供对SQL语句中的=, IN和BETWEEN AND的分片操作支持。ComplexShardingStrategy支持多分片键，由于多分片键之间的关系复杂，因此并未进行过多的封装，而是直接将分片键值组合以及分片操作符透传至分片算法，完全由应用开发者实现，提供最大的灵活度。

###### 行表达式分片策略

对应InlineShardingStrategy。使用Groovy的表达式，提供对SQL语句中的=和IN的分片操作支持，只支持单分片键。对于简单的分片算法，可以通过简单的配置使用，从而避免繁琐的Java代码开发，如: `t_user_$->{u_id % 8}` 表示t_user表根据u_id模8，而分成8张表，表名称为`t_user_0`到`t_user_7`。

###### Hint分片策略

对应HintShardingStrategy.  通过Hint而非SQL解析的分片方式.

###### 不分片策略

对应NoneShardingStrategy, 对应不分片的策略.

##### SQL Hint

对于分片字段非SQL决定，而由其他外置条件决定的场景，可使用SQL Hint灵活的注入分片字段。例：内部系统，按照员工登录主键分库，而数据库中并无此字段。SQL Hint支持通过Java API和SQL注释(待实现)两种方式使用。

#### 配置

##### 分片规则

分片规则配置的总入口。包含数据源配置、表配置、绑定表配置以及读写分离配置等。

##### 数据源配置

真实数据源列表

##### 表配置

逻辑表名称, 数据节点与分表规则的配置.

##### 数据节点配置

用于配置 逻辑表与真实表的映射关系.  可分为均匀分布和自定义分布两种形式.

- 均匀分布

指数据表在每个数据源内呈现均匀分布的态势,  例如:

```
db0
├── t_order0 
└── t_order1 
db1
├── t_order0 
└── t_order1
```

那么数据节点配置如下:

```
db0.t_order0, db0.t_order1, db1.t_order0, db1.t_order1
```

-    自定义分布

指数据表呈现有特定规则的分布, 例如:

```
db0
  ├── t_order0 
  └── t_order1 
db1
  ├── t_order2
  ├── t_order3
  └── t_order4
```

那么数据节点的配置如下：

```
db0.t_order0, db0.t_order1, db1.t_order2, db1.t_order3, db1.t_order4
```

##### 分片策略配置

对于分片策略存有数据源分片策略和表分片策略两种维度。

-    数据源分片策略

对应于DatabaseShardingStrategy. 用于配置数据被分配的目标数据源.

- 表分配策略

对应于TableShardingStrategy.  用于配置数据被分配的目标表.  该目标表存在与该数据的目标数据源内。故表分片策略是依赖与数据源分片策略的结果的。

两种策略的API完全相同。

##### 自增主键生成策略

通过在客户端生成自增主键替换以数据库原生自增主键的方式，做到分布式主键无重复。

### 内核剖析

ShardingSphere的3个产品的数据分片主要流程是完全一致的。 核心由

`SQL解析 => 执行器优化 => SQL路由 => SQL改写 => SQL执行 => 结果归并`

的流程组成。

![åçæ¶æå¾](https://shardingsphere.apache.org/document/current/img/sharding/sharding_architecture_cn.png)

- SQL解析

分为词法解析和语法解析。 先通过词法解析器将SQL拆分为一个个不可再分的单词。再使用语法解析器对SQL进行理解，并最终提炼出解析上下文。 解析上下文包括表、选择项、排序项、分组项、聚合函数、分页信息、查询条件以及可能需要修改的占位符的标记。

执行器优化

合并和优化分片条件，如OR等。

- SQL路由

根据解析上下文匹配用户配置的分片策略，并生成路由路径。目前支持分片路由和广播路由。

- SQL改写

将SQL改写为在真实数据库中可以正确执行的语句。SQL改写分为正确性改写和优化改写。

- SQL执行

同步多线程执行器异步执行.

- 结果归并

将多个执行结果集归并以便于通过统一的JDBC接口输出。结果归并包括流式归并、内存归并和使用装饰者模式的追加归并这几种方式。

#### 解析引擎

相对于其他编程语言，SQL是比较简单的。 不过，它依然是一门完善的编程语言，因此对SQL的语法进行解析，与解析其他编程语言（如：Java语言、C语言、Go语言等）并无本质区别。

##### 抽象语法树

解析过程分为词法解析和语法解析。 词法解析器用于将SQL拆解为不可再分的原子符号，称为Token。并根据不同数据库方言所提供的字典，将其归类为关键字，表达式，字面量和操作符。 再使用语法解析器将SQL转换为抽象语法树。

例如, 以下 SQL:

```sql
SELECT id, name FROM t_user WHERE status = 'ACTIVE' AND age > 18
```

解析之后的为抽象语法树见下图。

![SQLæ½è±¡è¯­æ³æ ](https://shardingsphere.apache.org/document/current/img/sharding/sql_ast.png)

为了便于理解，抽象语法树中的关键字的Token用绿色表示，变量的Token用红色表示，灰色表示需要进一步拆分。

最后，通过对抽象语法树的遍历去提炼分片所需的上下文，并标记有可能需要改写的位置。 供分片使用的解析上下文包含查询选择项（Select Items）、表信息（Table）、分片条件（Sharding Condition）、自增主键信息（Auto increment Primary Key）、排序信息（Order By）、分组信息（Group By）以及分页信息（Limit、Rownum、Top）。 SQL的一次解析过程是不可逆的，一个个Token按SQL原本的顺序依次进行解析，性能很高。 考虑到各种数据库SQL方言的异同，在解析模块提供了各类数据库的SQL方言字典。

##### SQL解析引擎

SQL解析作为分库分表类产品的核心，其性能和兼容性是最重要的衡量指标。 ShardingSphere的SQL解析器经历了3代产品的更新迭代。

第一代SQL解析器为了追求性能与快速实现，在1.4.x之前的版本使用Druid作为SQL解析器。经实际测试，它的性能远超其它解析器。

第二代SQL解析器从1.5.x版本开始，ShardingSphere采用完全自研的SQL解析引擎。 由于目的不同，ShardingSphere并不需要将SQL转为一颗完全的抽象语法树，也无需通过访问器模式进行二次遍历。它采用对SQL`半理解`的方式，仅提炼数据分片需要关注的上下文，因此SQL解析的性能和兼容性得到了进一步的提高。

第三代SQL解析器则从3.0.x版本开始，ShardingSphere尝试使用ANTLR作为SQL解析的引擎，并计划根据`DDL -> TCL -> DAL –> DCL -> DML –>DQL`这个顺序，依次替换原有的解析引擎，目前仍处于替换迭代中。 使用ANTLR的原因是希望ShardingSphere的解析引擎能够更好的对SQL进行兼容。对于复杂的表达式、递归、子查询等语句，虽然ShardingSphere的分片核心并不关注，但是会影响对于SQL理解的友好度。 经过实例测试，ANTLR解析SQL的性能比自研的SQL解析引擎慢3-10倍左右。为了弥补这一差距，ShardingSphere将使用`PreparedStatement`的SQL解析的语法树放入缓存。 因此建议采用`PreparedStatement`这种SQL预编译的方式提升性能。

第三代SQL解析引擎的整体结构划分如下图所示。

![è§£æå¼æç»æ](https://shardingsphere.apache.org/document/current/img/sharding/parsing_architecture_cn.png)

#### 路由引擎

根据解析上下文匹配数据库和表的分片策略，并生成路由路径。 对于携带分片键的SQL，根据分片键的不同可以划分为单片路由(分片键的操作符是等号)、多片路由(分片键的操作符是IN)和范围路由(分片键的操作符是BETWEEN)。 不携带分片键的SQL则采用广播路由。

分片策略通常可以采用由数据库内置或由用户方配置。 数据库内置的方案较为简单，内置的分片策略大致可分为尾数取模、哈希、范围、标签、时间等。 由用户方配置的分片策略则更加灵活，可以根据使用方需求定制复合分片策略。 如果配合数据自动迁移来使用，可以做到无需用户关注分片策略，自动由数据库中间层分片和平衡数据即可，进而做到使分布式数据库具有的弹性伸缩的能力。 在ShardingSphere的线路规划中，弹性伸缩将于4.x开启。

