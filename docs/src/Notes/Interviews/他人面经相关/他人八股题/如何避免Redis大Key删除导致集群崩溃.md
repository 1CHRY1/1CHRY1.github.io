### 来自拼多多提前批一面

#### Redis大Key删除导致集群崩溃，如何避免？

Redis删除大包含大量元素的key（如百万字段）的时候，使用同步DEL命令可能会阻塞主线程数秒到十几秒，期间所有请求延迟或超时，严重时会触发主从切换或集群故障。

避免集群崩溃策略：
1. 异步删除：使用异步删除命令 UNLINK，使其在后台线程执行而不阻塞主线程
2. 分批删除：可以使用SCAN家族命令分片遍历，再用小批量HDEL(每批取500字段)，SREM，LTRIM（分批截断尾部）等命令逐步删除元素。
3. 业务低峰期操作：凌晨或夜间执行删除任务
4. 预防/治理：定期扫描集群中的大Key预警和告警
5. 在设计上可以拆分数据防止一个key存储过多数据，也可以为每个Key设置合理过期时间，结合LRU控制内存使用

```java
void deleteLargeHashKey(RedisClient redis, String key, int batchSize) {
    String cursor = "0";
    do {
        ScanResult<Map.Entry<String, String>> res = redis.hscan(key, cursor, batchSize);
        cursor = res.getCursor();
        for (Map.Entry<String,?> entry : res.getResult()) {
            redis.hdel(key, entry.getKey());
        }
    } while (!"0".equals(cursor));
    // 最后删除空 hash key
    redis.unlink(key); // 后台删除空 hash 对象
}
```

