# 5.1 redis容器化
**1.单机部署**
<li>docker启动</li>

```shell
docker run --name redis-alone -d -p 6379:6379 192.168.14.66/public/redis:4.0.9
```
<li> k8s启动 </li>
--configmap

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: ykfw-redis-configmap
  namespace: ykfw-public
data:
  redis.conf: |
    #bind 10.100.11.171
    protected-mode no
    port 6379
    tcp-backlog 511
    timeout 0
    tcp-keepalive 300
    daemonize yes
    supervised no
    pidfile /mnt/redis/redis.pid
    loglevel notice
    logfile "/mnt/redis/redis.log"
    databases 16
    save 900 1
    save 300 10
    save 60 10000
    stop-writes-on-bgsave-error yes
    rdbcompression yes
    rdbchecksum yes
    dbfilename dump.rdb
    dir /mnt/redis/
    masterauth qwert12345
    slave-serve-stale-data yes
    slave-read-only yes
    repl-diskless-sync no
    repl-diskless-sync-delay 5
    repl-disable-tcp-nodelay no
    slave-priority 100
    requirepass yk@2018
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
```
--deployment

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: redis-service
  namespace: ykfw-public
  labels:
    app: redis-alone
spec:
  type: NodePort
  ports:
    - port: 6379
      nodePort: 6379
  selector:
    app: redis-alone
#   clusterIP: None
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: redis-lv-claim
  namespace: ykfw-public
  labels:
    app: redis-alone
  annotations:
    volume.beta.kubernetes.io/storage-class: "managed-nfs-storage"
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 2Gi
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: redis-alone
  namespace: ykfw-public
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: redis-alone
    spec:
      hostname: redis
      imagePullSecrets: 
      - name: harbor-default
      nodeSelector: 
         physical.group: app-share
      containers:
      - name: redis-alone
        image: 192.168.14.66/public/redis:3.2.11
        imagePullPolicy: IfNotPresent
        # command: ["/usr/local/bin/redis-server"]
        # args: ["/redis-conf/redis.conf"]
        resources: # 资源限制和请求
          limits: # 容器使用的最大资源限制
            cpu: 1048m
            memory: 1Gi
        requests: # 容器启动时资源请求
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 6379
        env:
        - name: TZ
          value: "Asia/Shanghai"
        volumeMounts:
        - name: redis-nfs-storage
          mountPath: /mnt/redis
        - mountPath: /mnt/config # 项目配置文件目录
          name: config
          readOnly: false
      volumes:
      - name: config
        configMap: 
            name: ykfw-redis-configmap
            items: 
            - key: redis.conf
              path: redis.conf
      - name: redis-nfs-storage
        persistentVolumeClaim:
          claimName: redis-lv-claim

```