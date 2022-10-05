## 安装 Docker 软件

```shell
yum install -y yum-utils device-mapper-persistent-data lvm2

yum-config-manager \
  --add-repo \
  http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
 
yum install -y docker-ce

## 创建 /etc/docker 目录
mkdir /etc/docker

# 配置 daemon.
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "registry-mirrors": ["https://kfp63jaj.mirror.aliyuncs.com"],
  "insecure-registries": ["harbor.hongfu.com"]
}
EOF
mkdir -p /etc/systemd/system/docker.service.d

# 重启docker服务
systemctl enable docker && reboot
```

