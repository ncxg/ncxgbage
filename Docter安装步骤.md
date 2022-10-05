配置yum源

```shell
# 对于 CentOS 7
$ sudo sed -e 's|^mirrorlist=|#mirrorlist=|g' \
         -e 's|^#baseurl=http://mirror.centos.org|baseurl=https://mirrors.tuna.tsinghua.edu.cn|g' \
         -i.bak \
         /etc/yum.repos.d/CentOS-*.repo
```



1、安装docker

```shell
$ rz
docker-ce-17.03.0.ce-1.el7.centos.x86_64
docker-ce-selinux-17.03.0.ce-1.el7.centos.noarch.rpm
#自动配置selinux，让Docker正常工作。目前行业中不会对selinux设置了。
SELinux避免进程之间文件的权限访问问题，有了虚拟化，系统内部只有一个进程。

#rpm包下载地址https://download.docker.com/linux/centos/7/x86_64/stable/Packages/

$ yum -y install docker-ce-*
$ systemctl enable docker
docker要接管防火墙，去实现自定义规则，在新的内核版本允许直接被接管
网桥的流量可以经过防火墙被处理
但在3版本两边融合不到一起，低版本bug之一，设为开机自启，再重启，在开机去结合
若出现暴露端口，但是访问不到，重启下，后续升级内核可以解决掉
$ reboot

$ docker run hello-world    #检验docker是否安装成功
docker run代表从镜像运行成容器，类似拿着isO镜像去安装一个虚拟机，后面的hello-world指的是用哪个镜像运行为容器，镜像名为hello-world，不写版本号会默认补充latest，最新稳定版。
注意：最新文件版只是一个字符串，版本号。不一定代表真的最新稳定版
```

2、安装Docker Hub 加速器

浏览器搜索daocloud--Docker极速下载--Docker Hub 加速器--Linux--复制下载网址

```shell
$ curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
$ systemctl restart docker
$ cat /etc/docker/daemon.json
https://kfp63jaj.mirror.aliyuncs.com    #指定是镜像仓库的地址

填写配置文件/usr/lib/systemd/system/docker 不推荐，因为service脚本还需要去执行daemon-reload，再去重启服务。
$ docker info
有网站就证明加速器已经配上了

```

阿里云Docker官网：https：//dev.aliyun.com/serach.html

https://kfp63jaj.mirror.aliyuncs.com老师的加速器地址，可在daemon.json文件中替换直接用，修改后重启docker



3、WordPress（博客站点）环境部署

```shell
$ docker run --name db --env MYSQL_ROOT_PASSWORD=example -d mariadb
$ docker run --name MyWordPress --link db:mysql -p 8080:80 -d wordpress
$ docker exec -it db /bin/bash
$ mysql -uroot -pexample
MariaDB[(none)]>create database wordpress;
MariaDB[(none)]>grant all on *.* to 'wordpress'@'%' identified by '123456';
MariaDB[(none)]>
```

浏览器URL：IP：8080





































































































































































