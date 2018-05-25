# docker安装

**在线安装docker**

```shell
# 导入 yum 源
# 安装 yum-config-manager
yum -y install yum-utils
# 导入
yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo
# 更新 repo
yum makecache
# 查看yum 版本
yum list docker-ce.x86_64 --showduplicates |sort -r
# 安装指定版本 docker-ce 17.03 被 docker-ce-selinux 依赖, 不能直接yum 安装 docker-ce-selinux
wget https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.1.ce-1.el7.centos.noarch.rpm
rpm -ivh docker-ce-selinux-17.03.1.ce-1.el7.centos.noarch.rpm
yum -y install docker-ce-17.03.1.ce
# 查看安装
docker version
Client:
Version: 17.03.1-ce
API version: 1.27
Go version: go1.7.5
Git commit: f5ec1e2
Built: Tue Jun 27 02:21:36 2017
OS/Arch: linux/amd64

```
**离线安装**
