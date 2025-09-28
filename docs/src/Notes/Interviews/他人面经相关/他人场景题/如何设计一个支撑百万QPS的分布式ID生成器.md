### 来自拼多多提前批一面

#### 如何设计一个支撑百万QPS的分布式ID生成器？

首先需要**限定场景**，比如这里限定的场景是：
唯一性：全局唯一（跨机房/跨服务）。有序性：通常希望时间有序（便于分库分表、索引局部性），但不一定强要求。
高可用、低延迟：单 ID 生成延迟需尽可能小（微秒~毫秒级）。
高吞吐：目标 1,000,000 QPS（并发/每秒生成 ID）。
可扩展：节点水平扩容以支撑更大 QPS。
幂等/无冲突：ID 不重复，遇故障能快速恢复。
（可选）64-bit 紧凑表示以减少存储索引膨胀。

**方案设计**：
Segment（中心分配） + 本地高性能生成器（Snowflake-like 或本地序列缓存）

Snowflake: Twitter（X）开源的一种 分布式唯一 ID 生成算法，把时间、机器标识、局部序列合并成一个 64 位（long）整数，从而满足全局唯一并能按时间排序的需求。其经典位划分是：1 位符号，41 位时间戳，10 位机器标识，12 位序列号（同一毫秒内的自增序号，这样也能保证每个节点每毫秒生成的id数量是固定的，多的需要等待到下一毫秒继续生成）。用于保证ID的单调性和事件可解析性。

**面向百万QPS的ID生成架构**
架构组件：
1. Id Distributor（中央分配器）用DB或Redis持久化原子分配的号段，每次请求都会将max_id加上step后返回给请求者
2. 各服务节点从Distributor中拉取一段，在本地内存中以AtomicLong或更高性能分配方式发放ID，节点在剩余数量小于阈值时异步获取下一段来确保不断生成新的请求。
3. 节点使用时间+序列号基于Snowflake算法在节点内部生成有序的ID
4. 实时监控本地剩余的段、请求延迟、Distributor分配慢或失败等问题，达到阈值后触发告警和自动降级

**中央分配的实现方式**：
基于MySQL表的原子更新：
```sql
CREATE TABLE id_segment (
  biz_tag VARCHAR(64) PRIMARY KEY,
  max_id BIGINT NOT NULL,
  step INT NOT NULL,
  updated_at TIMESTAMP
);

SELECT max_id FROM id_segment WHERE biz_tag='order' FOR UPDATE;
newMax = max_id + step; 
UPDATE id_segment SET max_id = newMax WHERE biz_tag='order';
```

基于Redis的**INCRBY**
Redis 的 INCRBY 是原子的且非常快，但要注意 Redis 持久化/故障恢复造成的“ID 丢失 / 重复”风险

**容量计算**
假设目标1000000QPS（即 1000000IDs/s）：
单台机器理论上线是 4,096,000 IDs/s，实际生产中考虑到GC/线程/OS抖动保守估计为 200k~1M QPS

因此更稳妥的做法是通过segment分段+本地生成将请求分发到多节点计算，每台节点在应用层实现异步预取段，这样中心压力减小而整体可扩展性达到百万QPS