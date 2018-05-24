# 5.3 mongodb容器化
# 5.2 rabbitmq容器化
**docker部署**
```shell
mkdir -p /mnt/mongo/db &&\
docker run -p 27017:27017 -v /mnt/mongo/db:/data/db -d mongo:3.2 mongo:3.2.20-jessie
```
命令说明：

-p 27017:27017 :将容器的27017 端口映射到主机的27017 端口

-v $PWD/db:/data/db :将主机中当前目录下的db挂载到容器的/data/db，作为mongo数据存储目录

**k8s部署**
```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
  namespace: ykfw-public
  labels:
    app: mongodb-alone
spec:
  type: NodePort
  ports:
    - port: 27017
      nodePort: 27017
  selector:
    app: mongodb-alone
#   clusterIP: None
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mongodb-lv-claim
  namespace: ykfw-public
  labels:
    app: mongodb-alone
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
  name: mongodb-alone
  namespace: ykfw-public
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: mongodb-alone
    spec:
      hostname: mongodb
      imagePullSecrets: 
      - name: harbor-default
      nodeSelector: 
         physical.group: app-share
      containers:
      - name: mongodb-alone
        image: 192.168.14.66/public/mongo:3.2.20-jessie
        imagePullPolicy: IfNotPresent
        # command: ["/usr/local/bin/mongodb-server"]
        # args: ["/mongodb-conf/mongodb.conf"]
        env:
        - name: TZ
          value: "Asia/Shanghai"
        resources: # 资源限制和请求
          limits: # 容器使用的最大资源限制
            cpu: 1048m
            memory: 1Gi
        requests: # 容器启动时资源请求
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 27017
        volumeMounts:
        - name: mongodb-nfs-storage
          mountPath: /data/db
      volumes:
      - name: mongodb-nfs-storage
        persistentVolumeClaim:
          claimName: mongodb-lv-claim

```


