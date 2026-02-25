
#### **3、Redis哨兵模式高可用-故障转移**


#### **1、取配置好的哨兵模式**

```mysql
root@~/myRedis#ps -ef | grep redis
root     23028     1  0 Nov24 ?        00:00:32 /usr/local/redis-5.0.10/src/redis-server *:6479
root     21626     1  0 Nov24 ?        00:00:36 /usr/local/redis-5.0.10/src/redis-server *:6480
root     21634     1  0 Nov24 ?        00:00:33 /usr/local/redis-5.0.10/src/redis-server *:6481
root     22830     1  0 Nov24 ?        00:01:14 /usr/local/redis-5.0.10/src/redis-sentinel *:26479 [sentinel]
root     22835     1  0 Nov24 ?        00:01:14 /usr/local/redis-5.0.10/src/redis-sentinel *:26480 [sentinel]
root     22846     1  0 Nov24 ?        00:01:14 /usr/local/redis-5.0.10/src/redis-sentinel *:26481 [sentinel]
```
PS: 6480主，6479，6481为从服务器

#### **2、人为故障，停止主服务器-----查看哨兵Sentinel日志文件（，kill -9 21626 ）**

```mysql
#日志文件配置在哨兵节点的配置文件中
root@/usr/local/redis-5.0.10/conf#ls -l | grep sentinel
-rw-r--r-- 1 root root 2489 Nov 25 14:48 redis_6479_sentinel.conf
-rw-r--r-- 1 root root 2489 Nov 25 14:48 redis_6480_sentinel.conf
-rw-r--r-- 1 root root 2489 Nov 25 14:48 redis_6481_sentinel.conf
root@/usr/local/redis-5.0.10/conf#less redis_6481_sentinel.conf
port 26481
daemonize yes
logfile "26481.log"
dir "/root/myRedis/master"
...
#查看从节点6479的哨兵日志
root@~/myRedis/master#less 26479.log
22830:X 25 Nov 2020 14:48:23.954 # +sdown master mymaster 127.0.0.1 6480  ####主观发现master主服务器不可用 【主观下线】 
22830:X 25 Nov 2020 14:48:24.013 # +odown master mymaster 127.0.0.1 6480 #quorum 3/2 ####投票后有两个sentinel发现master不能用 【客观下线】
22830:X 25 Nov 2020 14:48:24.013 # +new-epoch 2     ####当前配置版本被更新
22830:X 25 Nov 2020 14:48:24.013 # +try-failover master mymaster 127.0.0.1 6480  ####达到故障转移failover条件，正等待其他sentinel的选举
22830:X 25 Nov 2020 14:48:24.015 # +vote-for-leader d4a37020c85b66709927dc62962a9c219e6159d0 2   ####进行投票选举slave服务器 票数：2
22830:X 25 Nov 2020 14:48:24.017 # 0fd3a90d2eccd83bb5332a70c064219e61190fbc voted for d4a37020c85b66709927dc62962a9c219e6159d0 2
22830:X 25 Nov 2020 14:48:24.018 # a41a277fe42c95c698610a6646fdaab26ef7e09c voted for d4a37020c85b66709927dc62962a9c219e6159d0 2
22830:X 25 Nov 2020 14:48:24.086 # +elected-leader master mymaster 127.0.0.1 6480
22830:X 25 Nov 2020 14:48:24.086 # +failover-state-select-slave master mymaster 127.0.0.1 6480
22830:X 25 Nov 2020 14:48:24.145 # +selected-slave slave 127.0.0.1:6479 127.0.0.1 6479 @ mymaster 127.0.0.1 6480 ####选择一个slave当选新的master 6479
22830:X 25 Nov 2020 14:48:24.145 * +failover-state-send-slaveof-noone slave 127.0.0.1:6479 127.0.0.1 6479 @ mymaster 127.0.0.1 6480 ####把选举出来的slave进行身份master切换
22830:X 25 Nov 2020 14:48:24.228 * +failover-state-wait-promotion slave 127.0.0.1:6479 127.0.0.1 6479 @ mymaster 127.0.0.1 6480
22830:X 25 Nov 2020 14:48:25.095 # +promoted-slave slave 127.0.0.1:6479 127.0.0.1 6479 @ mymaster 127.0.0.1 6480
22830:X 25 Nov 2020 14:48:25.095 # +failover-state-reconf-slaves master mymaster 127.0.0.1 6480 ####把故障转移failover改变reconf-slaves 把6480配置成从服务器
22830:X 25 Nov 2020 14:48:25.166 * +slave-reconf-sent slave 127.0.0.1:6481 127.0.0.1 6481 @ mymaster 127.0.0.1 6480 ####sentinel发送slaveof命令把6481重新同步6479master（因为还没真正切换，所以日志还是6480端口）
22830:X 25 Nov 2020 14:48:25.904 * +slave-reconf-inprog slave 127.0.0.1:6481 127.0.0.1 6481 @ mymaster 127.0.0.1 6480   ####slave被重新配置为另外一个master的slave，但数据还未发生
22830:X 25 Nov 2020 14:48:26.235 # -odown master mymaster 127.0.0.1 6480    ####离开不可用的master
22830:X 25 Nov 2020 14:48:26.962 * +slave-reconf-done slave 127.0.0.1:6481 127.0.0.1 6481 @ mymaster 127.0.0.1 6480 ####与master进行数据同步
22830:X 25 Nov 2020 14:48:27.026 # +failover-end master mymaster 127.0.0.1 6480    ####故障转移完成
22830:X 25 Nov 2020 14:48:27.026 # +switch-master mymaster 127.0.0.1 6480 127.0.0.1 6479 ####master地址发生改变
22830:X 25 Nov 2020 14:48:27.026 * +slave slave 127.0.0.1:6481 127.0.0.1 6481 @ mymaster 127.0.0.1 6479    ####检测slave并添加到slave列表【完成】
22830:X 25 Nov 2020 14:48:27.026 * +slave slave 127.0.0.1:6480 127.0.0.1 6480 @ mymaster 127.0.0.1 6479
22830:X 25 Nov 2020 14:48:57.048 # +sdown slave 127.0.0.1:6480 127.0.0.1 6480 @ mymaster 127.0.0.1 6479
#查看从节点6481的哨兵日志
root@~/myRedis/master#less 26479.log
22846:X 25 Nov 2020 14:48:23.939 # +sdown master mymaster 127.0.0.1 6480   ####主观下线
22846:X 25 Nov 2020 14:48:24.017 # +new-epoch 2
22846:X 25 Nov 2020 14:48:24.017 # +vote-for-leader d4a37020c85b66709927dc62962a9c219e6159d0 2 ####进行投票选举slave服务器
22846:X 25 Nov 2020 14:48:25.033 # +odown master mymaster 127.0.0.1 6480 #quorum 3/2  ####客观下线
22846:X 25 Nov 2020 14:48:25.033 # Next failover delay: I will not start a failover before Wed Nov 25 14:48:54 2020  ####下一次故障转移延迟
22846:X 25 Nov 2020 14:48:25.167 # +config-update-from sentinel d4a37020c85b66709927dc62962a9c219e6159d0 127.0.0.1 26479 @ mymaster 127.0.0.1 6480   ####哨兵节点配置更新
22846:X 25 Nov 2020 14:48:25.167 # +switch-master mymaster 127.0.0.1 6480 127.0.0.1 6479   ####切换主服务器
22846:X 25 Nov 2020 14:48:25.167 * +slave slave 127.0.0.1:6481 127.0.0.1 6481 @ mymaster 127.0.0.1 6479
22846:X 25 Nov 2020 14:48:25.167 * +slave slave 127.0.0.1:6480 127.0.0.1 6480 @ mymaster 127.0.0.1 6479
22846:X 25 Nov 2020 14:48:55.186 # +sdown slave 127.0.0.1:6480 127.0.0.1 6480 @ mymaster 127.0.0.1 6479
#查看主节点6480的哨兵日志
root@~/myRedis/master#less 26480.log
22835:X 25 Nov 2020 14:48:23.947 # +sdown master mymaster 127.0.0.1 6480
22835:X 25 Nov 2020 14:48:24.016 # +new-epoch 2
22835:X 25 Nov 2020 14:48:24.017 # +vote-for-leader d4a37020c85b66709927dc62962a9c219e6159d0 2 ####进行投票选举slave服务器 票数：2
22835:X 25 Nov 2020 14:48:24.025 # +odown master mymaster 127.0.0.1 6480 #quorum 2/2
22835:X 25 Nov 2020 14:48:24.025 # Next failover delay: I will not start a failover before Wed Nov 25 14:48:54 2020
22835:X 25 Nov 2020 14:48:25.166 # +config-update-from sentinel d4a37020c85b66709927dc62962a9c219e6159d0 127.0.0.1 26479 @ mymaster 127.0.0.1 6480
22835:X 25 Nov 2020 14:48:25.166 # +switch-master mymaster 127.0.0.1 6480 127.0.0.1 6479
22835:X 25 Nov 2020 14:48:25.166 * +slave slave 127.0.0.1:6481 127.0.0.1 6481 @ mymaster 127.0.0.1 6479
22835:X 25 Nov 2020 14:48:25.166 * +slave slave 127.0.0.1:6480 127.0.0.1 6480 @ mymaster 127.0.0.1 6479
22835:X 25 Nov 2020 14:48:55.191 # +sdown slave 127.0.0.1:6480 127.0.0.1 6480 @ mymaster 127.0.0.1 6479
```

#### **3、查看新的主节点信息**

```mysql
root@~#/usr/local/redis-5.0.10/src/redis-cli -p 6479
127.0.0.1:6479> info Replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6481,state=online,offset=16236519,lag=1
master_replid:7a33bb4fa29bf18edc63c5853d05a1919c229767
master_replid2:89854298ef90868f88e1282dcf32b54005e211df
master_repl_offset:16236785
second_repl_offset:15439505
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:15188210
repl_backlog_histlen:1048576
```

#### **4、启动之前挂掉的主服务（自动变为从服务）**

```mysql
####启动服务
root@/usr/local/redis-5.0.10/conf#/usr/local/redis-5.0.10/src/redis-server /usr/local/redis-5.0.10/conf/redis_6480.conf
13496:C 25 Nov 2020 15:59:28.782 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
13496:C 25 Nov 2020 15:59:28.782 # Redis version=5.0.10, bits=64, commit=00000000, modified=0, pid=13496, just started
13496:C 25 Nov 2020 15:59:28.782 # Configuration loaded
###启动后再去新的主服务查看主从信息
root@/usr/local/redis-5.0.10/conf#/usr/local/redis-5.0.10/src/redis-cli -p 6479
127.0.0.1:6479> info Replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6481,state=online,offset=16290785,lag=0
slave1:ip=127.0.0.1,port=6480,state=online,offset=16290785,lag=0
master_replid:7a33bb4fa29bf18edc63c5853d05a1919c229767
master_replid2:89854298ef90868f88e1282dcf32b54005e211df
master_repl_offset:16290785
second_repl_offset:15439505
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:15242210
repl_backlog_histlen:1048576
```