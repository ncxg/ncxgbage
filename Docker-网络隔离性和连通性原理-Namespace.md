### Namespace - 网络

netns 是在 linux 中提供网络虚拟化的一个项目，使用 netns 网络空间虚拟化可以在本地虚拟化出多个网络环境，目前 netns 在 lxc 容器中被用来为容器提供网络

使用 ==netns== 创建的网络空间独立于当前系统的网络空间，其中的网络设备以及 iptables 规则等都是独立的，就好像进入了另外一个网络一样 

```shell
实验规划：准备一台没有安装docker服务的机器演示Namespace：  
docker加了很多安全设置   
机器 192.168.207.40
实验目的：创建虚拟网络空间实现容器之间通过网桥进行通信

# 创建虚拟网络空间
$ ip netns add r1
$ ip netns add r2
$ ip netns ls

# 进入虚拟网络空间
$ ip netns exec r1 bash
$ ip addr show
就只有一个回环接口lo，还是down状态
$ exit

# 在物理机空间添加一对 veth 设备   虚拟网络对
$ ip link add veth1.1 type veth peer name veth1.2
$ ip link add veth2.1 type veth peer name veth2.2
$ ip addr show
#注意从物理空间往r1虚拟网络空间建立veth对


# 将其中一块网卡放入至 ns1 网络名称空间之中
$ ip link set veth1.1 netns r1
$ ip link set veth2.1 netns r2

# 进入虚拟网络空间r1
$ ip netns exec r1 bash
$ ip addr show
$ ip link set lo up    #启动虚拟网路空间回环接口
$ ping localhost
$ ip link set veth1.1 name eth0   # 更改网络名称  
$ ip addr show
$ ip link set eth0 up  # 启动网卡
$ ip addr add 100.100.100.10/24 dev eth0  # 设置网卡名称
$ ifconfig
$ exit

# 进入虚拟网络空间r2
$ ip netns exec r2 bash
$ ip link set lo up    #启动虚拟网路空间回环接口
$ ping localhost
$ ip link set veth2.1 name eth0   # 更改网络名称  
$ ip addr show
$ ip link set eth0 up  # 启动网卡
$ ip addr add 100.100.100.20/24 dev eth0  # 设置网卡名称
$ ifconfig
$ ping 10.0.0.11
#此时发现ping不同，因为网桥没设置
$ exit



# 创建 bridge
$ ip link add name br0 type bridge
$ ifconfig
$ ip link set br0 up
$ ip addr add 100.100.100.100/24 dev br0
$ ifconfig
#br0的ip和虚拟网络空间的ip处于同一网段内

$ ip addr show
$ ip link set veth1.2 up
$ ip link set veth2.2 up
$ ip addr show
$ sudo ip link set dev veth1.2  master br0  # 将设备veth1.2连接至网桥  注意是先up再连接
$ sudo ip link set dev veth2.2  master br0  # 将设备veth2.2连接至网桥

验证：
# 进入虚拟网络空间r1
$ ip netns exec r1 bash
$ ping 100.100.100.20    

Docker NS显示
为了保护网络安全，容器通常会隐藏netns虚拟网络空间
#查看隐藏虚拟空间:192.168.66.110
$ ip netns list
#list是看/var/run/netns的文件
$ docker ps -a
$ docker inspect --format='{{.State.Pid}}' myapp  
# 查看当前容器的名字空间 ID
7331
$ mkdir /var/run/netns  # 创建目录，防止未创建
$ ln -s /proc/7331/ns/net /var/run/netns/7331
# 然后，就可以通过正常的系统命令来查看或访问容器的名字空间了。
$ ip netns show
$ ip netns exec 7331 bash
$ ifconfig
$ curl localhost
$ netstat -anpt
$ exit
$ docker inspect myapp 


Docker NS显示
为了保护网络安全，容器通常会隐藏netns虚拟网络空间
$ docker ps -a
$ ip netns ls
  目前是不显示
$ docker rm -f $( docker ps -a -q )
$ docker run --name wordpress -d wordpress
$ docker inspect wordpress
  地址是172.17.0.2
$ mkdir /var/run/netns  # 创建目录，防止未创建
$ docker inspect --format='{{.State.Pid}}' wordpress  
# 查看当前容器的名字空间 ID  容器在物理空间的静态ID
7721
$ ln -s /proc/7721/ns/net /var/run/netns/7721
# 然后，就可以通过正常的系统命令来查看或访问容器的名字空间了。
$ ip netns ls
$ ip netns exec 7721 bash
 可以看到地址是172.17.0.2且MAC地址也是一样的

```

