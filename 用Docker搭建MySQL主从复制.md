环境版本:
> 
MySQL : 5.7.13
Docker : 1.12.6
CentOS : 7.2

1.分别安装两个MySQL容器.命令如下
```
docker pull mysql:5.7.13
docker run --name db1  -p 13306:3306 -e MYSQL_ROOT_PASSWORD=db1 -d mysql:5.7.13
docker run --name db2  -p 13307:3306 -e MYSQL_ROOT_PASSWORD=db2 -d mysql:5.7.13
```

2.在主库上创建一个复制账户
```
GRANT REPLICATION SLAVE ON *.* TO 'db1'@' 172.17.0.5' IDENTIFIED BY 'db1';
```
> 
复制账户为: db1
指定从库的IP必须为: 172.17.0.5
复制密码为: db1

3.修改主库的配置文件 (麻烦,应该有更方便的修改方式)

3.1先从docker拷贝配置文件到主机/root 目录:
```
docker cp db1:/etc/mysql/my.cnf /root
```

3.2在主机打开 my.cnf , 在 [mysqld] 节点最后加上
```
log-bin=mysql-bin
server-id=1
```

3.3 再把此文件上传到docker mysql 里面覆盖
```
docker cp /root/my.cnf db1:/etc/mysql/my.cnf
```

3.4 重启 mysql 的docker , 让配置生效
```
docker restart db1
```

4. 修改从库的配置文件
> 跟第三步一样, 唯一不同是修改下面的参数，然后再将文件拷贝到db2容器中
```
server-id=2
```
5. 开始备份, 在主库执行以下命令, 让主库所有表置于只读不能写的状态, 这样达到主从库数据一致性
```
FLUSH TABLES WITH READ LOCK;
```
```
mysqldump -A -uroot -h127.0.0.1 -p -P13306 --events --ignore-table=mysql.events > db1.sql
```

6. 将主库的数据库备份在从库还原
```
mysql -uroot -pdb2 -h 127.0.0.1 -P 13307 < db1.sql
```

7. 从库还原后, 释放主库的读锁, 这样主库恢复写权限
```
unlock tables;
```

8.配置从库连接主库, 在从库上执行
```
CHANGE MASTER TO
MASTER_HOST='172.17.0.4',
MASTER_PORT=3306,
MASTER_USER='db1',
MASTER_PASSWORD='db1',
MASTER_LOG_FILE='mysql-bin.000001',
MASTER_LOG_POS=154;
```
> 最后两项 MASTER_LOG_FILE 和 MASTER_LOG_POS参数
是在主库执行 SHOW MASTER STATUS; 命令可以取得的对应的字段是 File 和 Position

9. 在从库启动 slave 线程开始同步
```
START SLAVE;
```

10.在从库 查看同步状态
```
show slave status;
```

 如果看到 Slave_Io_State 字段有 :
```
Waiting for master to send event ...
```
就成功了！
