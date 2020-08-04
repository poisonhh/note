# Linux (Centos7)

## 网络配置

### ifconfig

net-tools 安装

```shell
sudo yum install net-tools
```

### 常用命令

1. ip addr/ifconfig
2. systemctl restart/start/stop NetworkManager
3. systemctl restart/start/stop network

### 配置静态IP

1. ifconfig 查看网卡设备名称
2. 进入到默认文件路径

```shell
cd /etc/sysconfig/network-scripts/
```

3. 修改ifcfg-enp0s3 文件 

```shell
vi ifcfg-enp0s3
```

4. 文件内容修改

   其中关键内容：

   1. TYPE="Ethernet"     # 网络类型为以太网
   2. BOOTPROTO="static"  # 配置成静态ip
   3. NAME="eno16777736"  # 网卡设备名，设备名一定要跟文件名一致
   4. DEVICE="eno16777736" # 网卡设备名，设备名一定要跟文件名一致
   5. ONBOOT="yes"        # 该网卡是否随网络服务启动
   6. IPADDR="192.168.1.103"  #主机ip地址
   7. GATEWAY="192.168.1.1"   # 网关
   8. NETMASK="255.255.255.0"   # 子网掩码
   9. DNS1="218.85.152.99"    #DNS2也可以配置成8.8.8.8

```shell
BOOTPROTO=static
ONBOOT=yes
IPADDR=192.168.1.112
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS=8.8.8.8
```

5. 重启网络

```shell
systemctl restart NetworkManager
systemctl restart network
```

### 配置DNS域名服务器

1. 添加DNS

```shell
vi /etc/resolv.conf #修改配置文件
nameserver 8.8.8.8
```

2. 重启网络环境

```shell
systemctl restart NetworkManager
systemctl restart network
```

3. 解析百度地址

```shell
yum -y install bind-utils #nslookup
nslookup baidu.com
```

## Docker

### Centos7下卸载docker

1. 首先停止docker运行：systemctl stop docker
2. 搜索已经安装的docker安装包 yum list installed | grep docker 和 rpm -qa | grep docker
3. 删除搜索出来的安装包 yum -y remove xxx
4. 删除docker 镜像：rm -rf /var/lib/docker

### Centos7安装docker

1. 查看linux发行版、内核

```shell
[root@localhost ~]# cat /etc/redhat-release #查看版本号
CentOS Linux release 7.8.2003 (Core)
[root@localhost ~]# uname -r #查看linux内核
3.10.0-123.el7.x86_64
```

2. 替换阿里云yum源

```shell
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo #下载阿里yum源2 
yum makecache #生成仓库缓存
```

3. 安装docker

```she
yum install docker -y
```

4. 启动docker

```shell
systemctl start docker #启动docker
systemctl enable docker #开机启动docker
systemctl status docker #查看docker状态
```

5. 查看docker版本

```shell
docker version
```

6. DaoCloud加速器是广受欢迎的Docker工具，解决了国内用户Docker Hub缓慢的问题。

   DaoCloud加速器结合国内的CDN服务与协议层优化，成倍的提升了下载速度。

   使用前先确保Docker版本在1.8或更高版本，否则无法使用加速。

```shell
cat /etc/docker/daemon.json
{
    "registry-mirrors": [
        "http://95822026.m.daocloud.io"
    ],
    "insecure-registries": []
}
或者用这条命令
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://95822026.m.daocloud.io

#事后重启docker
systemctl restart docker
```

7. 公用镜像搜索、下载

```shell
docker search nginx #就找第一个，下载最多的，官方镜像
docker pull nginx #下载nginx镜像
docker images #查看有哪些镜像
```

8. 启动nginx镜像

```shell
docker run -p 8000:80 --name mynginx -d nginx   
#-p指定服务器8000端口,映射容器80 web端口，容器名为mynginx -d 守护进程模式启动（因为容器必须有进程在运行，否则结束就挂）
docker ps #查看目前工作的容器
docker ps -a #查看所有运行过的容器
```

9. 此时可以用服务器ip地址，在浏览器访问，默认80端口不用写，即可访问到 welcome nginx
10. 可用exec命令进入容器系统

```shell
docker exec -it container_ID /bin/bash
```

### Docker操作

- 容器自启动设置

```shell
docker run --restart=always  # 设置每次docker服务重启后容器也自动重启
docker update --restart=always <CONTAINER ID>  # 如果已经启动了则可以使用此命令
```

- 容器重启

```shell
docker restart imageid
```

