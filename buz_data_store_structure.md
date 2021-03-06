## 业务数据存储架构选型

### 场景描述
日志类型数据如账单
1. 会不断生成，不断写入
2. 查询很久之前的记录
3. 删改单条数据

### 方案
1.单库单表
>全存一起，小业务使用，可能需要定期转存数据。

2.单库，分表
>为了提供读写性能，防止锁表，可以按日期或者记录id分表。每次查询需要计算数据在哪张表里。

3.多库，多表
>可以按日期或者记录id分库。每次查询需要计算数据在哪张个库里。如果采用一致性hash实现扩容，会导致无法找到老数据，hash混乱。


4.多库，多表+id路由表
>所有外部id通过一定规则转换成另一套id，这套id能通过id里的信息知道数据在哪个库哪个表里。方便扩容。id映射存在kv系统里。


### 核心技术分析
1. 读写性能
2. 一致性hash && 扩容
