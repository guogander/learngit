# Linux相关需求解决办法
## 安装VM-tools失败

```shell
apt-get install open-vm-tools
apt-get install open-vm-tools-desktop
```

## 启用root权限以及切换root用户

```shell
 sudo passwd root
 su
 # Amazon Linux 2以及某些版本Linux
 sudo passwd
```

## 显示隐藏的目录

```shell
Ctrl+H
```

## 打开终端

```shell
ctrl+alt+t
```

## 解压缩

```bash
# 压缩
tar -zcvf 文件名  
# 解压缩
tar -zxvf 文件名.tar.gz 

# 压缩etc目录到/data路径，压缩名为etc.tar.gz
tar -zcvf /data/etc.tar.gz /etc # 快

tar -Jcvf /data/etc.tar.xz /etc # 慢 压缩比高
```

## 运行sh脚本

> cd到目标文件，然后执行 ./*.sh

## 打开root权限的文件管理 

```shell
sudo nautilus
```

## 设置静态IP

### Ubuntu

```yaml
# Ubuntu 18.04
vim /etc/netplan/XX-installer-config.yaml
# 打开yaml文件，修改文件内容：
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
  ethernets: 
    ens33:    
      addresses: [192.168.201.131/24]  # 修改静态ip地址
      gateway4: 192.168.201.2    # 网关
      nameservers: 
          addresses: [192.168.201.2]
```

重启应用：

```shell
sudo netplan apply
```

### CentOS

```shell
# CentOS7.9
vim /etc/sysconfig/network-scripts/ifcfg-ens33
# 修改内容
BOOTPROTO=static
ONBOOT=yes
IPADDR=192.168.201.122 # 静态IP地址
NETMASK=255.255.255.0
GATEWAY=192.168.201.2
DNS1=192.168.201.2
```

重启网络：

```shell
systemctl restart network
```

## 解决虚拟机没网

```shell
# ubuntu18.04
sudo service network-manager stop
sudo rm /var/lib/NetworkManager/NetworkManager.state
sudo service network-manager start
```

```shell
# CentOS7.9
# 没网报错 Failed to start LSB: Bring up/down
systemctl stop NetworkManager
systemctl disable NetworkManager
systemctl start network.service
```



## SSH

```shell
# ubuntu18.04
apt-get install openssh-server
```

打开 sshd_config 配置文件,远程连接修改

```shell
vim /etc/ssh/sshd_config 
# 此行取消注释，并改为yes
#PermitRootLogin prohibit-password
PermitRootLogin yes



/etc/init.d/ssh restart #重启SSH服务  
（/etc/init.d/ssh stop #关闭SSH服务）
查看ssh服务：ps -e | grep ssh
```

>cmd ssh远程连接虚拟机，错误信息：
>
>ECDSA host key for 192.168.201.146 has changed and you have requested strict checking.
>
>解决方法：命令  ssh-keygen -R "你的远程服务器ip地址"   清除缓存秘钥

## 删除文件夹所有的文件

```shell
rm -f '文件夹'
# 慎用
rm -vf '文件夹'
```

## 查看端口占用

```shell
ss -ntl
netstat -tunlp | grep 端口号
```

## 进程

```bash
# 杀死进程
kill -9 pid
# 0信号查看进程是否存在
killall -0 nginx
```

## 查看是否安装包

```bash
rpm -qi '包名'
```





## 防火墙&SELinux

### CentOS

```shell
# 关闭防火墙
# CentOS6.7
service iptables stop
chkconfig iptables off
# CentOS7.9
systemctl stop firewalld
systemctl disable firewalld
# 关闭selinux
vim /etc/selinux/config
# 修改内容为
SELINUX=disable
```

### Ubuntu

```shell
systemctl stop firewalld
systemctl disable firewalld
```



## Windows本地文件传输到Linux

```shell
# scp命令
scp E:\IDM下载文件\jdk-16.0.1_linux-aarch64_bin.tar.gz root@192.168.201.250:/opt/data
```

## Linux 命令行向文件添加内容

```shell
 # eg：
 echo 192.168.1.21 master >> /etc/hosts
```

