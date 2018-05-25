# 5.2 rabbitmq容器化
**docker部署**
```shell
docker run -d --hostname my-rabbit --name some-rabbit -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin  -p 8080:15672 -p 5672:5672 rabbitmq:3-management
```
**k8s部署**
```yaml
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: rabbitmq-lv-claim
  namespace: ykfw-public
  labels:
    app: rabbitmq-alone
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
  name: rabbitmq-alone
  namespace: ykfw-public
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: rabbitmq-alone
    spec:
      hostname: rabbitmq
      imagePullSecrets: 
      - name: harbor-default
      nodeSelector: 
         physical.group: app-share
      containers:
      - name: rabbitmq-alone
        image: 192.168.14.66/public/rabbitmq:3-management
        imagePullPolicy: IfNotPresent
        # command: ["/usr/local/bin/rabbitmq-server"]
        # args: ["/rabbitmq-conf/rabbitmq.conf"]
        resources: # 资源限制和请求
          limits: # 容器使用的最大资源限制
            cpu: 1048m
            memory: 1Gi
        requests: # 容器启动时资源请求
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 5672
          name: tcp
        - containerPort: 15672
          name: http
        env:
        - name: RABBITMQ_DEFAULT_USER
          value: admin
        - name: RABBITMQ_DEFAULT_PASS
          value: admin
        - name: TZ
          value: "Asia/Shanghai"
        volumeMounts:
        - name: rabbitmq-nfs-storage
          mountPath: /var/lib/rabbitmq/
      volumes:
      - name: rabbitmq-nfs-storage
        persistentVolumeClaim:
          claimName: rabbitmq-lv-claim
---
apiVersion: v1
kind: Service
metadata:
  name: rabbitmq-service
  namespace: ykfw-public
  labels:
    app: rabbitmq-alone
spec:
  type: NodePort
  ports:
  - port: 15672
    name: http
    nodePort: 15672
  - port: 5672
    name: tcp
    nodePort: 5672
  selector:
    app: rabbitmq-alone
#   clusterIP: None

```

