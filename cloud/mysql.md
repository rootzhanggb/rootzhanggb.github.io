# mysql容器化
**docker部署**
```shell
docker run --name mysql-tz -d -p 3306:3306 -v /home/harbor/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -e TZ='Asia/Shanghai' 192.168.14.66/public/mysql:5.6.35
```

