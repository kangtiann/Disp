# disp 架构

![disp 架构](./res/disp-arch.png)

- Apache Arrow 作为数据处理单元 `DataFrame`
- 每个节点有多个 Actor (可大于CPU核数, 考虑到 IO 阻塞)
- Actor 平等且能力对等
- LIST driven process
- Actor 之间无法相互通信, 除了响应 ping, 以及 kill
- Actor 同一时间只做一件事
- ETCD 作为 `Share Store`, 原则是可以用 Redis 替代
- ETCD 不是 Master
- Actor 的世界，只有 `DataFrame` 和 `ETCD`

## 计算模型

- `Job` 直接提交到 `ETCD`
- `Actor` 抢占式处理 `Job`
- `Actor` 如果可以直接处理完成 `Job`, 将结果写回 `ETCD`, 将任务标记为已解决
- `Actor` 如果不能单独处理任务, 需要将任务拆解为一组子 `Job` 列表
- `Actor` 如果将所有子 `Job` 处理完成，则完成整个 `Job` 的处理
- `Job` 具有依赖关系，DAG 关系需要被引入
- `Actor` 需要定时汇报心跳，如果无心跳，其他 `Actor` 将会抢占 `Job`
- `Job` 需要具有幂等性，即可以重复执行

总之，计算模型像是 分布式 LISP 解析器.
