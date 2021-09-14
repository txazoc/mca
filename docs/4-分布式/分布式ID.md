### 分布式ID

> 分布式ID，分布式下的全局唯一ID

**分布式ID特性**

* 全局唯一性，`必须`
* 趋势递增
* 时间相关
* 分布式ID生成保证高性能、高可用

#### UUID

> UUID，128位二进制，`32位十六进制`

UUID示例，`550e8400-e29b-41d4-a716-446655440000`

**UUID编码规则**

* 1~8位采用系统时间，在系统时间上精确到毫秒级保证时间上的唯一性
* 9~16位采用底层的IP地址，保证服务器集群中的唯一性
* 17~24位采用当前对象的HashCode值，保证一个对象上的唯一性
* 25~32位采用调用方法的一个随机数，保证一个对象上毫秒级的唯一性

**缺点**

* 无序
* 无序，长度过大，对MySQL存储和查询不友好

#### 基于数据库集群的自增id

> 多个数据库实例，每个数据库实例的自增起始id不同，自增步长一致

* 不同的`auto_increment_offset`，相同的`auto_increment_increment`
* `table_1`: 自增起始id = n，自增步长 = n
* `table_2`: 自增起始id = n+1，自增步长 = n
* ...
* `table_n`: 自增起始id = n + (n-1)，自增步长 = n

**缺点**

* 数据库存在写入性能瓶颈
* 扩容时，必须人工修改自增起始id和自增步长

#### 数据库号段模式

> 按号段进行分配，例如`[n, n + 1000)`，当下分布式ID生成器的主流实现方式之一

**表结构设计**

* biz_type: 业务类型，`int(11)`
* max_id: 已被分配号段的最大值，`bigint(20)`
* step: 每次分配号段的长度，`int(11)`
* version: 乐观锁版本号，`bigint(20)`

```sql
select max_id, version from table where biz_tag = ?;
update table set max_id = #{max_id} + step, version = version + 1
where biz_tag = ? and version = #{version}
```

**进阶**

* 预分配加载: 当前号段下发超过一定比例例如50%时，拉取下一个号段，双号段缓存
* MySQL主从: 半同步复制、全同步复制


[<< 上一篇: Markdown语法示例](3-其它/Markdown语法示例.md)

[>> 下一篇: 分布式事务](4-分布式/分布式事务.md)
