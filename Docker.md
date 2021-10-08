# Docker

## 中文手册

> [中文手册](https://www.docker.org.cn/docker/docker-206.html)

## docker run

### --network

#### host

```markdown
Docker使用了Linux的Namespaces技术来进行资源隔离，如PID Namespace隔离进程，Mount Namespace隔离文件系统，Network Namespace隔离网络等。一个Network Namespace提供了一份独立的网络环境，包括网卡、路由、Iptable规则等都与其他的Network Namespace隔离。

host模式类似于Vmware的桥接模式，与宿主机在同一个网络中，但没有独立IP地址。一个Docker容器一般会分配一个独立的Network Namespace。但如果启动容器的时候使用host模式，那么这个容器将不会获得一个独立的Network Namespace，而是和宿主机共用一个Network Namespace。容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口。
```

### -- rm

```bash
# 退出容器后就删除该容器，常用于临时调试
docker run --rm -it registry.ustack.com/centos/ustack-base:7.aarch64 bash
```

### docker cp

> **docker cp :**用于容器与主机之间的数据拷贝。

```bash
docker cp /www/runoob 96f7f14e99ab:/www/
```

### docker ps

```bash
[root@centos7-aarch64-rocky-binary ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
b3120d68f2d9        d6a36f04320f        "/bin/sh -c 'yum -..."   About an hour ago   Up About an hour                        awesome_darwin
```

`--no-trunc`  显示完整信息

```bash
[root@centos7-aarch64-rocky-binary ~]# docker ps -a --no-trunc
CONTAINER ID                                                       IMAGE                                                                     COMMAND                                                                                                 CREATED             STATUS              PORTS               NAMES
b3120d68f2d9835c425e0a400db3706f382cc17c03f1a304cc2379ce05fb3b09   sha256:d6a36f04320f5144153f365a37bae3f8943c441f27380432f1f4ea1c6f6e6aff   "/bin/sh -c 'yum -y install openstack-neutron-bgp-dragent && yum clean all && rm -rf /var/cache/yum'"   2 hours ago         Up 2 hours                              awesome_darwin
```



## 查看私有仓库

```bash
# registry默认端口在5000
curl http://localhost:5000/v2/_catalog
```

## 将私有仓库添加到docker

```bash
# 修改 /etc/docker/daemon.json
{
    "insecure-registries":["$seed_ip:5000"]
}
```

## docker查看容器卷

```bash
docker volume ls # 查看所有容器卷
docker volume create edc-nginx-vol # 创建一个自定义容器卷
docker volume inspect edc-nginx-vol # 查看指定容器卷详情信息
docker volume rm  <volume_name>  # 删除卷
```

## registry api分页

```bash
curl 192.168.121.4:4000/v2/_catalog?n=200 | grep zaqar    # 给定参数n返回结果
```

[registry分页参数](https://docs.docker.com/registry/spec/api/#pagination)

## Dockerfile

> [Dockerfile参考1](https://www.docker.org.cn/dockerppt/114.html)
>
> [Dockerfile参考2](https://docker-practice.github.io/zh-cn/image/build.html)

### dockerfile构建镜像

```bash
docker build -t <images_name>:<tag> .
# -t 指定镜像名和标签名
# 默认找Dockerfile这个文件名进行构建
```

### FROM

```dockerfile
# 基础镜像，后续命令都是基于这个镜像进行
FROM registry.ustack.com/dockerhub-proxy/library/alpine:3.12.0
```

### RUN

### ENV

### WORKDIR

该指令可以来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录，如该目录不存在，WORKDIR 会帮你建立目录。

```dockerfile
WORKDIR /root
```



# Kolla

> [kolla-build](https://docs.openstack.org/kolla/latest/admin/image-building.html#packages-customisation)

