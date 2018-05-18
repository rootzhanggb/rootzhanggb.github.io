# 5.3 mongodb容器化
# 5.2 rabbitmq容器化
**docker部署**
```shell
mkdir -p /mnt/mongo &&\
docker run -d --name some-mongo -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=admin \
-v /mnt/mongo:/data/db  -p 27017:27017 mongo:3.2.20-jessie
```
**k8s部署**
```yaml

```


