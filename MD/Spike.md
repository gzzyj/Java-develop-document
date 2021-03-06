# 设计一个秒杀系统

主要做到以下两点:

- 尽量将请求过滤在上游。
- 尽可能的利用缓存(大多数场景下都是**查多于写**)。

常用的系统分层结构:

![](https://ww4.sinaimg.cn/large/006tNc79ly1fmjw06nz2zj306f0fejrh.jpg)

针对于浏览器端，可以使用 JS 进行请求过滤，比如五秒钟之类只能点一次抢购按钮，五秒钟只能允许请求一次后端服务。(APP 同理)

这样其实就可以过滤掉大部分普通用户。

但是防不住直接抓包循环调用。这种情况可以最简单的处理:在`Web层`通过限制一个 UID 五秒之类的请求服务层的次数(可利用 Redis 实现)。

但如果是真的有 10W 个不同的 UID 来请求，比如黑客抓肉鸡的方式。

这种情况可以在`服务层` 针对于写请求使用请求队列，再通过限流算法([限流算法](https://github.com/crossoverJie/Java-Interview/blob/master/MD/Limiting.md))每秒钟放一部分请求到队列。

对于读请求则尽量使用缓存，可以提前将数据准备好，不管是 `Redis` 还是其他缓存中间件效率都是非常高的。

> ps : 刷新缓存情况，比如库存扣除成功这种情况不用马上刷新缓存，如果库存扣到了 0 再刷新缓存。因为大多数用户都只关心是否有货，并不关心现在还剩余多少。

## 总结

- 如果流量巨大，导致各个层的压力都很大可以适当的加机器横向扩容。如果加不了机器那就只有放弃流量直接返回失败。快速失败非常重要，至少可以保证系统的可用性。
- 业务分批执行：对于下单、付款等操作可以异步执行提高吞吐率。
- 主要目的就是尽量少的请求直接访问到 `DB`。
