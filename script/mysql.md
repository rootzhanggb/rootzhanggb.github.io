# mysql源码安装
<li>源码安装mysql5.6.22</li>
<li>一键脚本如下</li>
```shell
#!/bin/bash
# author zgb
# contact 312768277@qq.com
datadir="/mnt"
cd $datadir
#上传MySQL源码包mysql-5.6.38.tar.gz到/opt
wget http://dev.mysql.com/get/Downloads/MySQL-5.6/mysql-5.6.22.tar.gz
#yum方式安装相关依赖包
 yum -y install cmake bison git ncurses-devel gcc gcc-c++
#创建一个用户名为mysql的用户并加入mysql用户组
groupadd mysql
useradd -g mysql mysql -s /bin/nologin
#解压mysql-5.6.38.tar.gz，并且创建mysql安装目录和数据库文件存放目录
tar zxvf mysql-5.6.22.tar.gz
mkdir -p  $datadir/mysql  $datadir/mysql/data
cd mysql-5.6.22/
cmake -DCMAKE_INSTALL_PREFIX= $datadir/mysql -DMYSQL_UNIX_ADDR=$datadir/mysql/mysql.sock -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_ARCHIVE_STORAGE_ENGINE=1 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 -DMYSQL_DATADIR=$datadir/mysql/data -DMYSQL_TCP_PORT=3306 -DMYSQL_USER=mysql -DENABLE_DOWNLOADS=1
if [ $? -ne 0 ];then
echo -e "\033[32m编译出错了,正在处理中。。\033[0m"
rm -rf $datadir/mysql-5.6.22/CMakeCache.txt
cmake -DCMAKE_INSTALL_PREFIX= $datadir/mysql -DMYSQL_UNIX_ADDR=$datadir/mysql/mysql.sock -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_ARCHIVE_STORAGE_ENGINE=1 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 -DMYSQL_DATADIR=$datadir/mysql/data -DMYSQL_TCP_PORT=3306 -DMYSQL_USER=mysql -DENABLE_DOWNLOADS=1
fi
#如果此过程出现error,则执行命令 # rm -rf /opt/mysql-5.6.38/CMakeCache.txt
#安装相关依赖包，再重新 执行cmake
make
make install
#安装完之后，清除临时文件
make clean
#修改目录属主权限
chown -R mysql:mysql $datadir/mysql/data/
chown -R mysql:mysql $datadir/mysql/
#创建MySQL Server系统表


#此处出现报错执行命令
if [ $? -ne 0 ];then
rpm -ivh /mnt/Packages/perl-Data-Dumper-2.145-3.el7.x86_64.rpm
$datadir/mysql/scripts/mysql_install_db --user=mysql --datadir=/mnt/mysql/data
fi
#把初始化生成的 /usr/local/mysql/my.cnf 配置文件的属主数组更改为mysql
chown -R mysql:mysql /mnt/mysql
#配置启动脚本
cp /mnt/mysql/support-files/mysql.server /etc/init.d/mysql
# 启动MySQL
# /etc/init.d/mysql start
 #配置环境变量
echo "PATH=\$PATH:\$HOME/bin:\$datadir/mysql/bin:\$datadir/mysql/lib" >>/etc/profile
echo "export PATH">>/etc/profile
#变量生效
source /etc/profile
#以下操作请手动执行
#设置数据库密码
# mysql -uroot -p
# 
#密码初始化为空
#创建一个数据库用户，用于客户端访问
#use mysql;
# update user set password=password'yk-2018.com' where user="root";
# grant all privileges on *.* to root@"%" identified by ".";
# create user dev@'%' identified by 'yk-2018.com';
# grant all privileges on *.* to 'dev'@'%' identified by '123456';
#根据需要设置开机自动启动服务
# chkconfig mysql on

```