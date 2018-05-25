# vsftpd安装
```shell
#!/bin/bash
# author zgb
# contact 312768277@qq.com
# date 2018-04-25 10:43
yum install vsftpd pam pam-* db4 db4-* -y
chkconfig vsftpd on
#创建虚拟用户
echo  "yk">>/etc/vsftpd/vuser.txt
echo  "yk-2018.com">>/etc/vsftpd/vuser.txt
#将虚拟用户保存到DB4中
db_load -T -t hash -f /etc/vsftpd/vuser.txt /etc/vsftpd/vuser.db
#将原来的配置文件vsftpd全部注释掉并添加下面内容
sed -ir 's/^/#/g' /etc/pam.d/vsftpd 
echo -n '
auth    required   /lib64/security/pam_userdb.so   db=/etc/vsftpd/vuser
account required   /lib64/security/pam_userdb.so   db=/etc/vsftpd/vuser
' >> /etc/pam.d/vsftpd 
# 创建一个vsftpd服务的用户vsftpd，你也可以使用-d来指定他的家目录
groupadd vsftpd && useradd -g vsftpd -d /home/vsftpd -s /sbin/nologin vsftpd
# 更改 vsftpd的配置文件
cp /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.bak 
awk '! /^(#|$)/' /etc/vsftpd/vsftpd.conf.bak
rm -rf /etc/vsftpd/vsftpd.conf
cat >/etc/vsftpd/vsftpd.conf<<vseof
#不允许匿名访问
anonymous_enable=NO
#设定本地用户可以访问。注意：主要是为虚拟宿主用户，如果该项目设定为NO那么所有虚拟用户将无法访问
local_enable=YES
#允许写操作
write_enable=YES
#创建或上传后文件的权限掩码
local_umask=022
#禁止匿名用户上传
anon_upload_enable=NO
#禁止匿名用户创建目录
anon_mkdir_write_enable=NO
#进入目录时可以显示一些设定的信息，可以通过message_file=.message来设置
dirmessage_enable=YES
#开启日志
xferlog_enable=YES
#主动连接的端口号
connect_from_port_20=YES
#设定禁止上传文件更改宿主
chown_uploads=NO
#日志路径，记得自己创建一下并且对这个文件进行chown  vsftpd.vsftpd /var/log/vsftpd.log
xferlog_file=/var/log/vsftpd.log
#就是格式话日志格式的，你懂得。使用wu ftp软件时设置yes就行
xferlog_std_format=YES
#因为我们把vsftpd的shell设置为nobody 了，所以 这个地方写vsftpd就可以啦！当然或者可以写成系统内的nobody
nopriv_user=vsftpd
#设定支持异步传输功能
async_abor_enable=YES
#设定支持ASCII模式的上传
ascii_upload_enable=YES
#设定支持ASCII模式的上传
ascii_download_enable=YES
#登陆欢迎语
ftpd_banner=Welcome to zgb FTP service.
#限定在自己的目录内，不让他出去，就比如如果设置成NO，那么当你登陆到ftp的时候，可以访问服务器的其他一些有权限目录。设置为YES后即，锁定你的目录了
chroot_list_enable=YES
#待会要把用户写到这个里面，写到这里的用户乖乖的呆在家目录下吧
chroot_list_file=/etc/vsftpd/chroot_list
#以standalone方式来启动
listen=YES
#/etc/pam.d/下的vsftpd文件
pam_service_name=vsftpd
#在/etc/vsftpd/user_list中的用户将不得使用FTP
userlist_enable=YES
#支援 TCP Wrappers 的防火墙机制
tcp_wrappers=YES
#启用虚拟用户功能
guest_enable=YES
guest_username=vsftpd
#虚拟用户的权限符合他们的宿主用户
virtual_use_local_privs=YES
#虚拟用户个人vsftpd的配置文件存放路径。vsftpd_config是文件夹啊。注意：配置文件名必须和虚拟用户名相同
user_config_dir=/etc/vsftpd/vuser_conf

# 设置被动模式
allow_writeable_chroot=YES
#connect_from_port_10021=YES
pasv_min_port=8888
pasv_max_port=8899
# 禁用DNS反向解析即可解决文件
reverse_lookup_enable=NO
vseof
# 创建一下保存虚拟用户配置文件的目录
mkdir /etc/vsftpd/vuser_conf/ 
# 创建vsftp日志文件
touch /var/log/vsftpd.log 
chmod 600 /var/log/vsftpd.log 
chown vsftpd.vsftpd /var/log/vsftpd.log
# 创建要将哪些用户固定在家目录的配置文件
touch /etc/vsftpd/chroot_list
echo -n 'yk
vsftpd'>>/etc/vsftpd/chroot_list
echo vsftpd > /etc/vsftpd/chroot_list
# 将需要固定用户目录的用户名字写进去即可。
cd /etc/vsftpd/vuser_conf/ 
cat > yk<< EOF #起用虚拟用户,centos下yes必须为小写字母
local_root=/home/vsftpd/yk
write_enable=yes
anon_umask=022
anon_world_readable_only=NO
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
EOF
mkdir -p /home/vsftpd/yk
service vsftpd start

```
