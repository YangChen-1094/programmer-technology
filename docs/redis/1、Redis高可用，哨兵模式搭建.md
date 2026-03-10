
#### **1、Redis高可用，哨兵模式搭建**


#### **1、下载安装redis**

```mysql
wget https://download.redis.io/releases/redis-5.0.10.tar.gz
tar zxvf redis-5.0.10.tar.gz
mv redis-5.0.10/ /usr/local/        #移动
cd /usr/local/redis-5.0.10/src
make MALLOC=libc
make install
```

#### **2、配置主从复制**

**<span style='color:red'>2.1、配置主从服务</span>**
master的redis.conf文件(其余是默认设置)
```mysql
port 6479
daemonize yes
# 这个文件夹要改成自己的目录
dir "/root/myRedis/master"
```
slaver1 redis.conf文件
```mysql
port 6480
# 主服务器端口为6379
slaveof 127.0.0.1 6479
dir "/root/myRedis/slaver1"
```
slaver2 redis.conf文件
```mysql
port 6481
# 主服务器端口为6379
slaveof 127.0.0.1 6479
dir "/root/myRedis/slaver2"
```
**<span style='color:red'>2.2、启动主从redis服务：</span>**
```mysql
/usr/local/redis-5.0.10/src/redis-server /usr/local/redis-5.0.10/conf/redis_6479.conf
/usr/local/redis-5.0.10/src/redis-server /usr/local/redis-5.0.10/conf/redis_6480.conf
/usr/local/redis-5.0.10/src/redis-server /usr/local/redis-5.0.10/conf/redis_6481.conf
ps -ef | grep redis
root     23028     1  0 17:06 ?        00:00:01 /usr/local/redis-5.0.10/src/redis-server *:6479
root     21626     1  0 16:34 ?        00:00:02 /usr/local/redis-5.0.10/src/redis-server *:6480
root     21634     1  0 16:34 ?        00:00:02 /usr/local/redis-5.0.10/src/redis-server *:6481
```
**<span style='color:red'>2.3、连接客户端查看主从信息：</span>**
查看redis主服务：
```mysql
/usr/local/redis-5.0.10/src/redis-cli -p 6479
127.0.0.1:6479> info Replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6480,state=online,offset=30253,lag=0
slave1:ip=127.0.0.1,port=6481,state=online,offset=30120,lag=0
master_replid:816cdb4e1e03abb4b6d38ba4c650ad2b1309ce41
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:30253
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:30253
```
查看redis从服务：
```mysql
root@/usr/local/redis-5.0.10/conf#/usr/local/redis-5.0.10/src/redis-cli -p 6480
127.0.0.1:6480> info Replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6479
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_repl_offset:32395
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:816cdb4e1e03abb4b6d38ba4c650ad2b1309ce41
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:32395
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:32395
root@/usr/local/redis-5.0.10/conf#/usr/local/redis-5.0.10/src/redis-cli -p 6481
127.0.0.1:6481> info Replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6479
master_link_status:up
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_repl_offset:33872
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:816cdb4e1e03abb4b6d38ba4c650ad2b1309ce41
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:33872
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:15
repl_backlog_histlen:33858
```

#### **3、配置哨兵模式**

**<span style='color:red'>3.1配置主服务的哨兵</span>**
```mysql
vim redis_6479_sentinel.conf
port 26479
daemonize yes
logfile "26479.log"
dir "/root/myRedis/master"
sentinel monitor mymaster 127.0.0.1 6479 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 15000
```
**<span style='color:red'>3.2配置从服务的哨兵</span>**
```mysql
vim redis_6480_sentinel.conf
port 26480
daemonize yes
logfile "26480.log"
dir "/root/myRedis/master"
 
#sentinel monitor <master-name> <ip> <redis-port> <quorum>
sentinel monitor mymaster 127.0.0.1 6479 2       
##告诉sentinel去监听地址为ip:port的一个master，这里的master-name可以自定义，quorum是一个数字，指明当有多少个sentinel认为一个master失效时，master才算真正失效
#sentinel auth-pass <master-name> <password>
#设置连接master和slave时的密码，注意的是sentinel不能分别为master和slave设置不同的密码，因此master和slave的密码应该设置相同。 
sentinel down-after-milliseconds mymaster 30000    ##需要多少失效时间，一个master才会被这个sentinel主观地认为是不可用的
sentinel parallel-syncs mymaster 1
#sentinel parallel-syncs <master-name> <numslaves>
#这个配置项指定了在发生failover主备切换时最多可以有多少个slave同时对新的master进行同步，这个数字越小，完成failover所需的时间就越长，但是如果这个数字越大，就意味着越 多的slave因为replication而不可用。可以通过将这个值设为1 来保证每次只有一个slave 处于不能处理命令请求的状态。
sentinel failover-timeout mymaster 15000
#sentinel failover-timeout <master-name> <milliseconds>
#failover-timeout 可以用在以下这些方面：    
#1. 同一个sentinel对同一个master两次failover之间的间隔时间。  
#2. 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那里同步数据时。   
#3. 当想要取消一个正在进行的failover所需要的时间。   
#4. 当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来了。
vim redis_6481_sentinel.conf
port 26481
daemonize yes
logfile "26481.log"
dir "/root/myRedis/master"
sentinel monitor mymaster 127.0.0.1 6479 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 15000
```
**<span style='color:red'>3.3、启动哨兵服务器</span>**
```mysql
/usr/local/redis-5.0.10/src/redis-sentinel /usr/local/redis-5.0.10/conf/redis_6479_sentinel.conf
/usr/local/redis-5.0.10/src/redis-sentinel /usr/local/redis-5.0.10/conf/redis_6480_sentinel.conf
/usr/local/redis-5.0.10/src/redis-sentinel /usr/local/redis-5.0.10/conf/redis_6481_sentinel.conf
root@/usr/local/redis-5.0.10#ps -ef | grep redis                          
root     23028     1  0 17:06 ?        00:00:01 /usr/local/redis-5.0.10/src/redis-server *:6479
root     21626     1  0 16:34 ?        00:00:02 /usr/local/redis-5.0.10/src/redis-server *:6480
root     21634     1  0 16:34 ?        00:00:02 /usr/local/redis-5.0.10/src/redis-server *:6481
root     22830     1  0 17:02 ?        00:00:03 /usr/local/redis-5.0.10/src/redis-sentinel *:26479 [sentinel]
root     22835     1  0 17:02 ?        00:00:03 /usr/local/redis-5.0.10/src/redis-sentinel *:26480 [sentinel]
root     22846     1  0 17:02 ?        00:00:03 /usr/local/redis-5.0.10/src/redis-sentinel *:26481 [sentinel]
```
查看表示启动成功

#### **4、让主redis服务挂掉【failover主备切换】**

**<span style='color:red'>4.1、杀死redis的主服务进程</span>**
```mysql
kill -9 23028
并连接到其中一台从服务器查看主从信息，如下：【当前6480端口已经切换为主服务】
root@/usr/local/redis-5.0.10/conf#/usr/local/redis-5.0.10/src/redis-cli -p 6480
127.0.0.1:6480> info Replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6481,state=online,offset=51985,lag=1
master_replid:89854298ef90868f88e1282dcf32b54005e211df
master_repl_offset:51985
second_repl_offset:38955
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:51985
```
**<span style='color:red'>4.2、重新启动6479端口的redis服务</span>**
```mysql
/usr/local/redis-5.0.10/src/redis-server /usr/local/redis-5.0.10/conf/redis_6479.conf
```
**<span style='color:red'>4.3、连接到主服务器查看，【6479自动切换成从服务器】</span>**
```mysql
root@/usr/local/redis-5.0.10/conf#/usr/local/redis-5.0.10/src/redis-cli -p 6480
127.0.0.1:6480> info Replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6481,state=online,offset=51985,lag=1
slave1:ip=127.0.0.1,port=6479,state=online,offset=51985,lag=1
master_replid:89854298ef90868f88e1282dcf32b54005e211df
master_replid2:816cdb4e1e03abb4b6d38ba4c650ad2b1309ce41
master_repl_offset:51985
second_repl_offset:38955
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:51985
```
配置文件在gitbub
```mysql
https://github.com/YangChen-1094/redis_sentinel/
```