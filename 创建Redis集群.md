#### 1. 先安装ruby环境依赖
```
yum install ruby rubygem
```
#### 2. 安装redis集群管理器
```
# gem install redis
Successfully installed redis-3.2.2
1 gem installed
Installing ri documentation for redis-3.2.2...
Installing RDoc documentation for redis-3.2.2...
```
#### 3. 将源码包中的Redis 集群工具拷贝到Redis serevr 安装位置
```
cp /src/redis-trib.rb /usr/local/bin/redis-trib
```
#### 4. 在每个 Redis 配置文件中添加
```
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
```

#### 5. 启动所有 Redis server 服务
#### 6. 查看Redis 服务进程
```
# ps -aux | grep redis-server
Warning: bad syntax, perhaps a bogus '-'? See /usr/share/doc/procps-3.2.8/FAQ
root      8986  0.0  0.3  40608  7356 ?        Ssl  19:16   0:00 /usr/local/redis/bin/redis-server *:6379 [cluster]    
root      8996  0.0  0.3 137440  7368 ?        Ssl  19:17   0:00 redis-server *:6380 [cluster]    
root      9027  0.0  0.3 137440  7584 ?        Ssl  19:23   0:00 redis-server *:6381 [cluster]    
root      9065  0.0  0.3 137440  7520 ?        Ssl  19:29   0:00 redis-server *:6382 [cluster]
root      9069  0.0  0.3 137440  7520 ?        Ssl  19:29   0:00 redis-server *:6383 [cluster]
root      9073  0.0  0.3 137440  7520 ?        Ssl  19:29   0:00 redis-server *:6384 [cluster]
root      9079  0.0  0.0 103320   860 pts/0    S+   19:29   0:00 grep redis
```
#### 7. 创建集群
```
# redis-trib create --replicas 1 192.168.2.105:6379 192.168.2.105:6380 192.168.2.105:6381 192.168.2.105:6382 192.168.2.105:6383 192.168.2.105:6384
```
#### 8. 通过Redis client 进入某个Redis集群节点
```
# redis-cli -c -h 192.168.2.105 -p 6382
```
#### 9. 查看集群的状态
```
192.168.2.105:6383> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:1
cluster_stats_messages_sent:353048
cluster_stats_messages_received:353048
```
#### 10. 查看集群中各节点的状态
```
192.168.2.105:6383> cluster nodes
a53adc9730f8d655266b0daeeb9f5d2c594d2630 192.168.2.105:6382 myself,slave b510459e932c2fad907277e6c44362da2514befb 0 0 4 connected
1862087294efabd66d2ea5c972746c9dacc2119c 192.168.2.105:6383 slave 999596b3e6b7508a6f33c808a5f27d494726bf07 0 1449655176281 5 connected
573bc078efa51518ebf97b80e28d9f68f137eb94 192.168.2.105:6384 slave 0b2d2b8137064bc2d0a78b5ce1c899e9c8c33bbe 0 1449655175280 6 connected
b510459e932c2fad907277e6c44362da2514befb 192.168.2.105:6379 master - 0 1449655176280 1 connected 0-5460
999596b3e6b7508a6f33c808a5f27d494726bf07 192.168.2.105:6380 master - 0 1449655176780 2 connected 5461-10922
0b2d2b8137064bc2d0a78b5ce1c899e9c8c33bbe 192.168.2.105:6381 master - 0 1449655177281 3 connected 10923-16383
```
