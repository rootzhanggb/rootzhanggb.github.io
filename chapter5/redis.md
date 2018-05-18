# 5.1 redis容器化
**1.单机部署**
<li>docker启动</li>

```shell
docker run --name redis-alone -d -p 6379:6379 192.168.14.66/public/redis:4.0.9
```
<li> k8s启动 </li>

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: redis-service
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
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: redis-alone
    spec:
      hostname: redis
      containers:
      - name: redis-alone
        image: 192.168.14.66/public/redis:4.0.9
        imagePullPolicy: IfNotPresent
        # command: ["/usr/local/bin/redis-server"]
        # args: ["/redis-conf/redis.conf"]
        ports:
        - containerPort: 6379
        volumeMounts:
        - name: redis-nfs-storage
          mountPath: /data
      volumes:
      - name: redis-nfs-storage
        persistentVolumeClaim:
          claimName: redis-lv-claim
```


