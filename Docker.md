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





## 查看私有仓库

```bash
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
