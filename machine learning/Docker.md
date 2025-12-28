## `docker compose down`

- **停止**当前 compose 项目里的所有服务容器
- 并且**删除这些容器**
- 一般还会删除 compose 创建的**默认网络**

注意：

- **默认不会删除命名卷（volumes）**，所以数据库/Kafka 的数据通常还在。
- 如果你加 `--volumes`，才会把命名卷也删掉（⚠️ 数据会没）。

一句话：**停 + 删容器（默认保留数据卷）**。

## docker compose up -d
根据 docker-compose.yml 创建并启动服务
-d = detached，表示后台运行（命令执行完就返回到终端，不一直打印日志）
一句话：按配置启动整套服务，并放到后台跑。

