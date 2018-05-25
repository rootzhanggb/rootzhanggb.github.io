# mongodb安装
**源码安装**
```shell
#！/bin/bash
set -x
#创建MongoDB程序,数据，日志存放目录
mkdir -p /mnt/mongodb /mnt/mongodb/data /mnt/mongodb/log
# 下载源码并解压
cd /mnt/
if [ ! -f "mongodb-linux-x86_64-3.2.7.tgz" ];then
curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.2.7.tgz
fi
tar -zxvf mongodb-linux-x86_64-3.2.7.tgz -C /mnt/mongodb/
cd /mnt/mongodb/
mv mongodb-linux-x86_64-3.2.7/* .
rm -rf mongodb-linux-x86_64-3.2.7
#设置全局变量
echo "export PATH=\$PATH:/mnt/mongodb/bin">>/etc/profile.d/mongodb.sh
source /etc/profile
#配置文件设置
cat >/mnt/mongodb/mongodb.conf<<EOF
#数据文件存放目录
dbpath = /mnt/mongodb/data
#日志文件存放目录
logpath = /mnt/mongodb/log/mongodb.log
#端口 port = 27017
#以守护程序的方式启用，即在后台运行
fork = true
nohttpinterface = true
EOF
cat >sysconf<<EOF
if test -f /sys/kernel/mm/transparent_hugepage/enabled; then
  echo never > /sys/kernel/mm/transparent_hugepage/enabled
fi
if test -f /sys/kernel/mm/transparent_hugepage/defrag; then
  echo never > /sys/kernel/mm/transparent_hugepage/defrag
fi
ulimit -u 65535
EOF
cat sysconf|while read line;do echo ${line} >>/etc/rc.local;done
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag
#拷贝文件
cp -r /mnt/mongodb/bin/mongod /mnt/mongodb/bin/mongo /usr/local/bin/
#服务化
cat > /usr/lib/systemd/system/mongodb.service << EOF
[Unit]
Description=mongodb
After=network.target
[Service]
Type=forking
#PIDFile=/mnt/mongodb/mongod.lock
ExecStart=/usr/local/bin/mongod -f /mnt/mongodb/mongodb.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true
[Install]
WantedBy=multi-user.target
EOF
# 设置服务
systemctl enable mongodb.service
# 启动
systemctl start mongodb.service
```
