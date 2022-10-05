[TOC]

#### 一、基础命令说明

```shell
# 查看当前可用的网络类型
$ docker network ls 	
		
# 网络空间名称
$ docker network create -d 类型 
	# 类型分为：
		# overlay network
		# bridge network
```

二、docker网络通讯方式

none模式：

```shell
$ rz
hub.c.163.com/public/centos:7.2-tools.tar
$ docker load -i hub.c.163.com/public/centos:7.2-tools.tar
$ docker tag hub.c.163.com/public/centos:7.2-tools.tar 163 
#若之前已有镜像则不需要进行

$ cc
$ docker images
$ docker run --name none --net none -d 163
$ docker exec -it none /bin/bash
$ ifconfig
#只有lo,没有eth0，不具备和外部通讯的能力
$ exit
```

container模式：

```shell
$ cc
$ docker run --name c1 --net none -d 163
$ docker run --name c2 --net container:c1 -d 163
通过回环接口访问，注意container:后接的是容器c1
$ docker exec -it c1 /bin/bash
$ yum -y install httpd
$ httpd
$ echo "c1" > /var/www/html/index.html
$ curl localhost     $ curl localhost/index.html  
$ exit
$ docker exec -it c2 /bin/bash
$ curl localhost
#有相同的网络栈，可以访问C1的网页

$ ifconfig
$ cd
$ touch whbdaociyiyou
$ exit
$ docker exec -it c1 /bin/bash
$ ifconfig
$ cd
$ ls
#在c1容器的家目录里看不到c2创建的whbdaociyiyou文件，说明c1和c2在container网络模式下，c2借助c1的网络可以共享网络文件，却无法共享系统文件
应用：第一个容器nginx，第二个容器php-fpm,--net container:nginx
```

host模式：

```shell
$ exit
$ cc
$ curl localhost
$ docker run --name wordpress --net host -d wordpress
$ docker exec -it wordpress /bin/bash
$ echo "qwe" > index.html
$ exit
$ curl localhost/index.html  或网页访问192.168.207.30
#容器没做网络的隔离化，用的就是物理机的。通过物理机就能访问
$ ip addr show


$ docker pull centos:centos7.9.2009
$ docker run -it --rm --net host centos:centos7.9.2009 /bin/bash
#启动容器时不需要加-p进行端口映射     
$ yum -y install net-tools
$ ifconfig
正常启一个容器就会有VETH，此时没有添加VETH
$ cd
$ ls
$ touch chenxin.txt
$ exit
$ ls
文件级别隔离的，看不到chenxin.txt
```

==$ docker run -it --rm --net host centos:centos7.9.2009 /bin/bash==



暴露端口

```shell
# -p  <ContainerPort>：将制定的容器端口映射至主机所有地址的一个动态端口
$ cc
$ docker run --name wordpress1 -p 80 -d wordpress
没有加主机端口，产生随机端口
$ docker ps
浏览器访问:192.168.207.30:32768
$ docker port wordpress1   #打印端口映射信息  更便于在脚本中截取数据
80/tcp -> 0.0.0.0:32768

# -P（大）：暴露所需要的所有端口
$ mkdir build 
$ cd build/
$ vim Dockerfile
FROM centos:centos7.9.2009
EXPOSE 80 8080 90 9090
CMD touch /root/startup.sh && tail -f /root/startup.sh  
# tail -f是为了正常运行起来，不然需要/bin/sh
$ docker build -t expose:v1 .
$ docker run --name expose -P -d expose:v1
$ docker port expose
80/tcp -> 0.0.0.0:32776
8080/tcp -> 0.0.0.0:32774
90/tcp -> 0.0.0.0:32775
9090/tcp -> 0.0.0.0:32773
#自动以EXPOSE声明的端口与随机端口去绑定

```

自定义Docker0的网桥地址

```shell
修改/etc/docker/daemon.json文件
{
    "bip": "192.168.1.5/24",
    "fixed-cidr": "192.168.1.0/24",
    "fixed-cidr-v6": "2001:db8::/64",
    "mtu": "1500",
    "default-gateway": "192.168.1.1",
    "default-gateway-v6": "2001:db8:abcd::89",
    "dns": ["192.168.1.2","192.168.1.3"]
}
```



#### 三、独立至不同的网络命名空间进行隔离

不同项目网络隔离：

项目一：www.xinxianghf.com

​     apache 

​     db       192.66.0.0/16

项目二：cloudmessage.top

​     apache -cm

​     db-cm   192.67.0.0/16

```shell
$ docker network ls   #查看当前可用的网络类型
$ cc
-------------------------------
$ docker-compose -f /usr/local/test/docker-compose.yaml up -d
#会显示出创建test_default网卡的步骤
-------------------------------

$ docker network create -d bridge --subnet "192.66.0.0/16" --gateway "192.66.0.1" xinxianghf

$ docker network create -d bridge --subnet "192.67.0.0/16" --gateway "192.67.0.1" cloudmessage
$ docker network ls

$ docker run --name apache --net xinxianghf -d 163
$ docker run --name db --net xinxianghf -d 163

$ docker run --name apache-cm --net cloudmessage -d 163
$ docker run --name db-cm --net cloudmessage -d 163

$ docker ps -a
$ docker inspect apache-cm
#IP地址是192.67.0.2
$ docker exec -it db-cm ping 192.67.0.2
#同项目能访问，同一个广播域，容器之间可以通信
$ docker exec -it db ping 192.67.0.2
#不同项目无法ping通，不同广播域因为网桥分割导致无法通信，达到了隔离不同项目网络的目的，避免因一个项目受到网络攻击而牵连到其他项目

$ docker run --name test -d 163
$ docker exec -it test ping 192.67.0.2
#无法ping通，在同一个网络栈下，但是各有各的门，除非路由记录转发

```



#### 四、使用 Linux 桥接器进行主机间的通讯        

**脚本创建**                

```shell
$ exit
$ cc
$ docker images
$ docker load -i hub.c.163.com/public/centos:7.2-tools.tar
#默认安装好了ssh了
$ docker run --name ssh -p 2222:22 -d hub.c.163.com/public/centos:7.2-tools
$ docker exec -it ssh /bin/bash
$ passwd
密码：123456
密码：123456
$ yum -y install httpd

#在Xshell中建立一个新的终端
名称：自定义
主机：192.168.207.30
端口：2222
尝试登录一下：192.168.207.30
$ touch chenxin   #进入端口映射的容器创建文件

返回原来的终端端口
$ cd
$ ls
#发现新建终端进入容器创建的文件可以查询的到
$ exit
$ docker ps -a
$ docker commit ssh ssh:v1   #将新创建的ssh容器封装成镜像，配上ip的话就可是一个物理容器了  
#启动命令会继承下来，所以是有启动命令的  容器叫ssh，镜像名叫ssh：v1
$ cc


$ route -n
$ vim /etc/sysconfig/network-scripts/ifcfg-ens34
ONBOOT=no       #关闭一张网卡，避免网络冲突
$ systemctl restart network
$ mkdir /usr/local/script     #养成把脚本放到固定位置的习惯
$ cd /usr/local/script
$ vim bridgeinit.sh    #给docker配置ip
#!/bin/bash
ip addr del dev ens33 192.168.207.30/24
ip link add link ens33 dev br0 type macvlan mode bridge
ip addr add 192.168.207.30/24 dev br0
ip link set dev br0 up
#ip route add default via 192.168.66.1 dev br0  #由于使用的是仅主机模式，没有网关，所以把此行命令注释掉
# macvlan 三层交换，网桥模式，给br0添加IP地址，就是第一行刚刚在ens33删除的。因为br0绑定到网络栈，网络栈具备信息的确认权，所以地址是基于br0去写的，而不是ens33，只不过br0的实际工作者是ens33，垂帘听政
#不同版本下脚本内的命令调用关系会有区别，那怕是7.2和7.4都有所区别，但是通过脚本就没问题。若想持久化可以加入开机自启

$ chmod a+x bridgeinit.sh
$ ./bridgeinit.sh
$ ifconfig
#此时ens33没有ip地址，成为网桥br0的依附设备，负责支持网桥对外传递数据报文
br0的ip地址为192.168.207.30
$ cd
$ rz
pipework-master.zip
#注意若此时物理机没有网关上不了网，无法安装unzip工具，所以可以提前安好unzip工具或者将pipework-master.zip包在其他机器上解压后再上传过来
$ unzip pipework-master.zip
$ mv pipework-master/pipework /usr/local/bin/
#脚本
$ chmod a+x /usr/local/bin/pipework
$ cc
$ docker run --name ssh --net none -d ssh:v1
#既然要分配地址的话，分配VETH，本身就不需要VETH了
#没有加端口映射，网络是none模式
$ pipework br0 ssh 192.168.207.200/24   #赋予网桥IP地址

windows中的CMD
ping 192.168.207.200

新开启一个ssh终端：
$ ssh 192.168.207.200
密码：123456
$ ls
chenxin   #查看到此文件，说明利用ssh:v1镜像创建的容器，配上ip以后就可以成为一个独立访问的物理容器了
$ httpd
$ ls /root/ > /var/www/html/index.html
网页访问192.168.207.200

采用这种方案把容器当传统虚拟机用，代价低启动快。   --分配独立地址--
容器的启动时间等同于经常启动时间
```

**设置容器地址分配**

```shell
$ pipework br0 test 192.168.66.200/24@192.168.66.1
```

从github.com网站上下载pipework安装包

