# 定义

- runID：服务器运行的 ID。
- Offset：主服务器的**复制偏移量**和从服务器**复制的偏移量**。
- Replication backlog：主服务器的**复制积压缓冲区**。

在 Redis 2.8 之后，使用 Psync 命令代替 Sync 命令来执行复制的同步操作。

Psync 命令具有**完整重同步和部分重同步**两种模式：

- 完整同步用于处理初次复制情况：

  完整重同步的执行步骤和 Sync 命令执行步骤一致，都是通过让**主服务器创建并发送 RDB 文件，以及向从服务器发送保存在缓冲区的写命令来进行同步。**

- 部分重同步是用于处理断线后重复制情况：

  当**从服务器在断线后重新连接主服务器时**，主服务可以将主从服务器连接断开期间执行的写命令发送给从服务器，从服务器**只要接收并执行这些写命令**，就可以将数据库更新至主服务器当前所处的状态。

## 完整重同步

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/Redis%E5%AE%8C%E6%95%B4%E5%90%8C%E6%AD%A5.jpg)

- **Slave 发送 Psync 给 Master**，由于是第一次发送，不带上 runID 和 Offset。

- Master 接收到请求，**发送 Master 的 runID 和 Offse**t 给从节点。

- Master 生成保存 RDB 文件。

- Master **发送 RDB 文件**给 Slave。

- 在发送 RDB 这个操作的同时，写操作会复制到缓冲区 Replication Backlog Buffer 中，并从 Buffer 区发送到 Slave。

- Slave **将 RDB 文件的数据装载，并更新自身数据**。

## 部分重同步

![](https://raw.githubusercontent.com/wuqifan1098/picBed/master/Redis%E9%83%A8%E5%88%86%E9%87%8D%E5%90%8C%E6%AD%A5.jpg)

- 网络发生错误，Master 和 Slave 失去连接。
- Master 依然**向 Buffer 缓冲区写入数据。**
- Slave 重新连接上 Master。
- Slave 向 Master **发送自己目前的 runID 和 Offset。**
- Master 会**判断 Slave 发送给自己的 Offset 是否存在 Buffer 队列中。**
- 如果存在，则发送 Continue 给 Slave;如果不存在，意味着可能错误了太多的数据，缓冲区已经被清空，这个时候就需要重新进行全量的复制。
- Master 发送**从 Offset 偏移后的缓冲区数据给 Slave。**
- Slave 获取数据更新自身数据。