# 1.5 redis安装
**redis安装shell脚本整理**
```shell
#!/bin/bash
#单机redis
datedir="/mnt"
mkdir -p $datedir/redis &&cd $datedir/redis
wget http://download.redis.io/releases/redis-3.2.11.tar.gz
tar xzf redis-3.2.11.tar.gz
cd redis-3.2.11
make
cp src/redis-server src/redis-cli /usr/local/bin/
chmod -R a+x /usr/local/bin/*
cat >$datedir/redis/redis.conf<<eof
#bind 10.100.11.171
protected-mode no
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize yes
supervised no
pidfile $datedir/redis/redis.pid
loglevel notice
logfile "$datedir/redis/redis.log"
databases 16
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir $datedir/redis/
masterauth qwert12345
slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
#requirepass yk-2018.com
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
lua-time-limit 5000
cluster-enabled no
cluster-config-file nodes.conf
cluster-node-timeout 15000
cluster-require-full-coverage no
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
eof
#启动
nohup redis-server $datedir/redis/redis.conf &
```
