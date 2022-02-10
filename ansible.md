# Ansible

## 安装

```shell
yum install epel-release -y
yum install ansible –y
```

ansible命令格式：

```bash
# -k 没配置免密时输入口令 所有主机口令一样才可以
ansible hosts文件里面的ip或分组名 -m 模块名 -a "参数" 
```

配置文件：

- /etc/ansible/ansible.cfg

  ```yaml
  # 一般保持默认
  host_key_checking = False # 检查对应服务器的host_key，建议取消注释
  log_path = /var/log/ansible.log # 日志存放路径，建议取消注释
  ```

基本操作：

```bash
ansible all --list-hosts # 列出主机清单
ansible all -m ping -u user -k # -u 远程执行命令的用户，使用对方的user用户连接

ansible 192.168.201.123 -m command -a 'ls /root' -u g -k -b -K  # -u 用户 -b 使用root(默认)执行 -K root口令
[root@ansible ansible]# ansible 192.168.201.123 -m command -a 'ls /root' -u g -k -b -K
SSH password: 
BECOME password[defaults to SSH password]: 
192.168.201.123 | CHANGED | rc=0 >>
anaconda-ks.cfg
initial-setup-ks.cfg
```

## 内置变量

[内置变量](https://blog.csdn.net/qq_35887546/article/details/105177305)

## 1、Ansible相关工具

hosts:要执行任务的主机

remote_user:可用于Host和task中，可指定其通过sudo方式在远程主机执行任务

tasks:任务列表 格式：module:arguments  ; shell和command模块后面跟命令而非key=values

**运行playbook常见选项**

> --check ：只检测可能会发生的改变，但不真正执行操作
>
> --list-hosts ：列出运行任务的主机
>
> --limit 主机列表      ：只针对主机列表中的主机执行
>
> -v显示过程 -vv -vvv更详细

### ansible-doc

> 查看模块使用方法

```bash
# 语法：ansible-doc  <模块名>
ansible-doc -s ping
```

### ansible-vault

```shell
# 加密playbook工具

# 加密
ansible-vault encrypt hello.yaml 

[root@ansible ansible]# ansible-vault encrypt hello.yaml 
New Vault password: 
Confirm New Vault password: 
Encryption successful
[root@ansible ansible]# cat hello.yaml 
$ANSIBLE_VAULT;1.1;AES256
38636133636166346332633633333864366661626262343536366633313731636539373562663933
3736616537636531616638356465626366366363323932650a626462656334386161663735323733
62633533336530376231326139373333356137373763616133356262303536653235323238633430
3435636332393739360a663665616439666237366330363037633765333037656535643232653863
63633838613532323466386533306233343831393761393962326665646339333861366436313837
38336264613233306566663035636535643332373630333532656338643466633132306236653030
39313833393331356237626134323333383734663265666162373463633833316261313234346361
66393962623239643561353462323561633237356266333030396462343531613464653531643162
3832
# 解密
ansible-vault decrypt hello.yaml 

[root@ansible ansible]# ansible-vault decrypt hello.yaml 
Vault password: 
Decryption successful

# 加密的情况下进行查看编辑
# 查看
ansible-vault view hello.yaml 
# 编辑
ansible-vault edit hello.yaml 

[root@ansible ansible]# ansible-vault view hello.yaml 
Vault password: 
---
- hosts: web
  remote_user: root
  
  tasks: 
    - name: hello
      command: hostname
      
# 修改口令
ansible-vault rekey hello.yaml

[root@ansible ansible]# ansible-vault rekey hello.yaml
Vault password: 
New Vault password: 
Confirm New Vault password: 
Rekey successful
```

### ansible-console

> 交互界面，进入ansible，直接输入命令以及参数

```shell
g@web (1)[f:5]$ hostname name=node1 # 修改主机名命令
```

### ansible-galaxy

> 可以下载角色

## 2、Ansible模块

### ping

```bash
# 测试主机连通性
[root@ansible ansible]# ansible 192.168.201.123 -m ping
```

### command

默认模块 ，不加-m默认使用command，可以在ansible.cfg文件修改默认模块，局限大，很多命令不能执行

```bash
ansible web -m command -a 'systemctl status network'   # 使用单引号
# 查看用户
ansible web -a 'getent passwd nginx'
# 查看组
ansible web -a 'getent group nginx'
```

### shell

类似command，执行的命令比command多，但也有些许不能执行

### script

> 运行脚本模块

```bash
# 在ansible新建脚本并添加权限，可将此脚本文件在远程执行
ansible all -m script -a '/root/ansible/hostname.sh'
```



### cron

> 计划任务 周期进行任务  支持时间：minute,hour,day,month,weekday

```bash
# 查看计划任务
crontab -e
# 周期性广播
ansible all -m cron -a 'minute=* weekday=1,3,5 job="/usr/bin/wall FBI warning" name=warningcron'
# 禁用 disabled=true/yes (false/no 启用)
ansible all -m cron -a 'disabled=true job="/usr/bin/wall FBI warning" name=warningcron'
# 删除计划任务
ansible all -m cron -a 'job="/usr/bin/wall FBI warning" name=warningcron state=absent'
```



**时钟同步**

国内阿里时钟源`time1.aliyun.com`

国际微软时钟源`time.windows.com`

使用命令测试服务器可用性：

```shell
ntpdate time1.aliyun.com
```

```shell
# 同步时钟命令
# name 此次计划任务名称；job 计划任务后面要执行的命令；后面就是任务周期性进行的时钟配置；minute 分 hour 时  (每小时同步时间)
ansible 192.168.201.123 -m cron -a 'name="test cron1" job="ntpdate time1.aliyun.com" minute=0 hour=*/'
```

```shell
# 查看计划任务 被管理主机查看
[root@ansible2 g]# crontab -l
#Ansible: test cron1
0 */1 * * * ntpdate time1.aliyun.com
```

### copy

同步文件，仅用于把本地文件copy到远程主机

```shell
# src 源文件  dest 目标文件
ansible 192.168.201.123 -m copy -a "src=/etc/hosts dest=/etc/hosts"
# /etc/sysconfig/selinux是指向/etc/selinux/config文件的软链接，替换文件要替换真实的文件；backup表示将被替换的文件做备份
ansible all -m copy -a 'src=/root/ansible/selinux dest=/etc/selinux/config backup=yes'
```

### fetch

> 抓取远程服务器文件到ansible本机

```bash
# src 源文件，必须是文件而不是目录；dest 存放文件目录 存放路径一般为：<dest>/<ip>/<src>  下例为：/data/192.168.201.123/var/log/messages
ansible all -m fetch -a "src=/var/log/messages dest=/data"
```

### file

> 管理文件和文件属性

```bash
# 新建空文件
ansible all -m file -a 'name=/data/f3 state=touch'
# 删除文件
ansible all -m file -a 'name=/data/f3 state=absent'
# 新建文件夹
ansible all -m file -a 'name=/data/dir1 state=directory'
# 删文件夹
ansible all -m file -a 'name=/data/dir1 state=absent'
# 创建软链接
ansible all -m file -a 'src=/etc/fstab name=/data/fstab.link state=link'
# 删除软链接
ansible all -m file -a 'name=/data/fstab.link state=absent'
```

### yum

> 软件安装管理

```bash
# 安装包
ansible all -m yum -a 'name=httpd,nginx,appche state=latest'
# 删除包
ansible all -m yum -a 'name=httpd state=absent'
# 列出安装的包
ansible all -m yum -a 'list=installed'
# 单独安装rpm包, disable_gpg_check=yes忽略key的检查
ansible <ip> -m yum -a 'name=/root/xxxx.rpm disable_gpg_check=yes'

# playbook
- name: install package
  yum: name=httpd state=present
- name: start service
  service: name=httpd state=started enabled=yes
```

### service

> 管理服务

```yaml
# 启动服务并开机自启
ansible all -m service -a 'name=httpd state=started enabled=yes'

# playbook
- name: install package
  yum: name=httpd state=present
- name: start service
  service: name=httpd state=started enabled=yes
```

### user

> 用户管理

```bash
# 创建nginx账户
ansible all -m user -a 'name=nginx shell=/sbin/nologin system=yes home=/var/nginx groups=root,bin uid=80 comment="nginx service"'
# 删除nginx账户
ansible all -m user -a 'name=nginx state=absent remove=yes'
```

### group

> 组的管理

```bash
# 创建nginx组
ansible all -m group -a 'name=nginx system=yes gid=80'
# 删除组
ansible all -m group -a 'name=nginx state=absent'
```

### unarchive

> 解压缩

```bash
# copy参数：copy=yes将ansible主机上的压缩包传到目标主机并解压至特定目录；copy=no将远程主机上的某个压缩包解压到指定路径

# 将本地主机文件传到目标主机并解压缩，并更改文件所属用户为g
ansible web -m unarchive -a 'src=/data/etc.tar.gz dest=/data/ owner=g copy=yes'
# 在远程主机解压压缩包并更改文件所属为g，修改权限为700
ansible web -m unarchive -a 'src=/data/etc.tar.xz dest=/opt/ owner=g copy=no mode=700'
```

### archive

> 压缩

```bash
# 将目标主机的path路径压缩到dest，指定格式为bz2 ，所属用户g，权限0600
ansible web -m archive -a 'path=/var/log/ dest=/data/log.tar.bz2 format=bz2 owner=g mode=0600'
```

### hostname

> 修改主机名

```bash
ansible 192.168.201.123 -m hostname -a 'name=node02'
```

### lineinfile

> 替换文件内容

```bash
# 将<path>路径的文件的<regexp>匹配的一行替换为<line>里面的内容;   regexp="^SELINUX=" 匹配'SELINUX='开头的行
ansible web -m lineinfile -a 'path=/etc/selinux/config regexp="^SELINUX=" line="SELINUX=enforcing"'
```

### replace

> 和lineinfile类似，使用正则表达式匹配内容并替换

```bash
# 找到UUID开头的行，在前面添加#注释
ansible web -m replace -a "path=/etc/fstab regexp='^#(UUID.*)' replace='#\1'"
# 匹配#开头的所有内容，去掉#号  \1就是括号中的内容，匹配到的#后面的内容
ansible web -m replace -a "path=/etc/fstab regexp='^#(.*)' replace='\1'"
```

### setup

> 收集远程主机系统信息   默认收集fact信息，主机较多的话会慢，可以使用`gather_facts: no`来禁止Ansible收集facts信息

```bash
# setup收集信息部分过滤字段
ansible_python_version  # python版本
ansible_memtotal_mb # 内存 
ansible_default_ipv4 # ip地址
ansible_distribution_version # centos版本
ansible_processor_vcpus # 内核个数

# 例
ansible web -m setup -a 'filter=ansible_python_version'
[root@ansible ~]# ansible web -m setup -a 'filter=ansible_python_version'
192.168.201.123 | SUCCESS => {
    "ansible_facts": {
        "ansible_python_version": "2.7.5", 
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false
}
```

### register

> 注册变量，将输出的信息赋给注册的变量

```yaml
# 将shell命令"echo test2_string"的返回值赋给变量shellreturn
tasks:
  - shell: "echo test2_string"
    register: shellreturn
```



## 3、Playbook组件

> 剧本 编排任务  
>
> -C 检查剧本  不实际执行

简单例子：

```yaml
# vim hello.yaml
# 编辑内容如下：
---
- hosts: web # 要执行的主机
  remote_user: root  # 远程主机执行的身份

  tasks:  # 事务，执行命令
    - name: hello  # 说明
      command: hostname  # 执行hostname命令
      
# 参考      
---
- hosts: web
  remote_user: root

  tasks: 
    - name: create new file
      file: name=/opt/newfile state=touch
    - name: create new user
      user: name=test2 system=yes shell=/sbin/nologin
    - name: install package
      yum: name=httpd
    - name: copy conf
      copy: src=/var/www/html/index.html dest=/var/www/html
    - name: start service
      service: name=httpd state=started enabled=yes

```
- Hosts 执行的远程主机列表
- Tasks 任务集
- Variables 内置变量或自定义变量在playbook中调用
- Templates 模板，可替换模板文件中的变量并实现一些简单逻辑的文件
- Handlers和notify结合使用，由特定条件出发的操作，满足条件才执行，否则不执行
- Tags 标签 指定某条任务执行，用于选择运行playbook中的部分代码。ansible具有幂等性，会自动跳过没有变化的部分，既是如此，有些代码为测试其确实没有发生变化的时间依然会非常的长。此时，如果确信其没有变化，就可以通过tags跳过这些代码片段。

### hosts组件

> hosts中指定的主机必须在主机清单中存在

```yaml
# 例 ":"表示或
- hosts: web:all
```

### remote_user组件

> 可用于Host和task中，可指定其通过sudo方式在远程主机执行任务

```yaml
- hosts: web
  remote_user: root
  
  tasks: 
    - name: test connection
    ping: 
    remote_user: g  # 也可指定此用户
    sudo: yes    # sudo 默认root用户
    sudo_user: g   # sudo 为g
```

### task列表和action组件

> play的主体部分是task list，task list中有一个或多个task,各个task按次序逐个在hosts中指定的所有主机上执行，即在所有主机上完成第一个task后，再开始第二个task
>
> task的目的是使用指定的参数执行模块，而在模块参数中可以使用变量。模块执行是幂等的，这意味着多次执行是安全的，因为其结果均一致
>
> 每个task都应该有其name，用于playbook的执行结果输出，建议其内容能清晰地描述任务执行步骤。如果未提供name，则action的结果将用于输出

**task两种格式**：

(1)action: module arguments

(2)module: arguments      建议使用

> shell和command模块后面跟命令而不是键值对

```yaml
---
- hosts: websrvs
  remote_user: root
  tasks:
    - name: insta1l httpd
      yum: name=httpd
    - name: start httpd
      service: name=httpd state=started enabled=yes
```

## 4、Playbook命令

### 循环

#### loop

> loop等价于with_list，从名字上可以知道它是遍历数组（列表）的，所以在loop指令中，每个元素都以列表的方式去定义。列表有多少个元素，就循环执行file模块多少次，每轮循环中，都会将本次迭代的列表元素保存在控制变量 item中。

```yaml
---
- name: play1
  hosts: all
  remote_user: root
  gather_facts: false
  tasks:
   - name: create file /tmp/test1.txt
     file: path={{item}} state=touch
     loop:
       - /tmp/test1.txt
       - /tmp/test2.txt
```



格式：

```bash
ansible-playbook <filename.yaml> ... [options]
```

常见选项：

```bash
-C --check   # 只检测可能会发生的改变，但不真正执行操作
--list-hosts # 列出运行任务的主机
--list-tags  # 列出tag
--list-tasks # 列出task
--limit <主机列表> # 只针对主机列表中的主机执行
-v -vv -vvv   # 显示过程，v越多越详细
```

### playbook创建mysql用户和组

```yaml
gather_facts: no
---
#create mysql user and group
- hosts: web
  remote_user: root
  gather_facts: no  # 不收集主机信息

  tasks:
    - {name: create group, group: name=mysql system=yes gid=306}
    - name: create user
      user: name=mysql shell=/sbin/nologin system=yes group=mysql uid=306 home=/data/mysql create_home=no
```

查询：

```bash
[root@ansible ansible]# ansible web -a 'getent passwd mysql'
192.168.201.123 | CHANGED | rc=0 >>
mysql:x:306:306::/data/mysql:/sbin/nologin
```

### playbook安装nginx

```yaml
vim install_nginx.yml
---
# install nginx
- hosts: web
  remote_user: root
  gather_facts: no
  tasks:
    - name: create group nginx
      group: name=nginx state=present
    - name: create user nginx
      user: name=nginx system=yes group=nginx
    - name: install nginx
      yum: name=nginx state=present
    - name: html pages  # 修改默认的页面
      copy: src=files/index.html dest=/usr/share/nginx/html/index.html
    - name: start nginx
      service: name=nginx state=started enabled=yes   
```

### playbook安装和卸载httpd

```yaml
# 安装httpd
vim install_httpd.yml
--- 
# install httpd
- hosts: web
  remote_user: root
  gather_facts: no
  
  tasks:
    - name: install httpd
      yum: name=httpd state=present
    - name: install configre file
      copy: src=files/httpd.conf dest=/etc/httpd/conf/
    - name: web html
      copy: src=files/index.html dest=/var/www/html/
    - name: start service
      service: name=httpd state=started enabled=yes

# 卸载httpd
vim remove_httpd.yml
# remove httpd
---
- host: web
  remote_user: root
  
  tasks: 
    - name: remove httpd package
      yum: name=httpd state=absent
    - name: remove apache user
      user: name=apache state=absent
    - name: remove data file
      file: name=/etc/httpd state=absent
    - name: remove web html
      file: name=/var/www/html/ state=absent
```

### playbook安装mysql

```yaml
# 文件准备 下载地址：https://downloads.mysql.com/archives/community/
# 目标主机创建目录/ data/mysql
[root@ansible ~]#ls -l /data/ansible/files/mysql-5.6.46-linux-glibc2.12-x86_64.tar.gz 
-rw-r--r-- 1 root root 403177622 Dec  4 13:05 /data/ansible/files/mysql-5.6.46-linux-glibc2.12-x86_64.tar.gz

[root@ansible ~]#cat /data/ansible/files/my.cnf 
[mysqld]
socket=/tmp/mysql.sock
user=mysql
symbolic-links=0
datadir=/data/mysql
innodb_file_per_table=1
log-bin
pid-file=/data/mysql/mysqld.pid

[client]
port=3306
socket=/tmp/mysql.sock

[mysqld_safe]
log-error=/var/log/mysqld.log

[root@ansible ~]#cat /data/ansible/files/secure_mysql.sh 
#!/bin/bash
/usr/local/mysql/bin/mysql_secure_installation <<EOF

y
password
password
y
y
y
y
EOF

[root@ansible ~]#tree /data/ansible/files/
/data/ansible/files/
├── my.cnf
├── mysql-5.6.46-linux-glibc2.12-x86_64.tar.gz
└── secure_mysql.sh

0 directories, 3 files

[root@ansible ~]#cat /data/ansible/install_mysql.yml
---
# install mysql-5.6.46-linux-glibc2.12-x86_64.tar.gz
- hosts: dbsrvs
  remote_user: root
  gather_facts: no

  tasks:
    - name: install packages
      yum: name=libaio,perl-Data-Dumper,perl-Getopt-Long
    - name: create mysql group
      group: name=mysql gid=306 
    - name: create mysql user
      user: name=mysql uid=306 group=mysql shell=/sbin/nologin system=yes create_home=no home=/data/mysql
    - name: copy tar to remote host and file mode 
      unarchive: src=/data/ansible/files/mysql-5.6.46-linux-glibc2.12-x86_64.tar.gz dest=/usr/local/ owner=root group=root 
    - name: create linkfile  /usr/local/mysql 
      file: src=/usr/local/mysql-5.6.46-linux-glibc2.12-x86_64 dest=/usr/local/mysql state=link
    - name: data dir
      shell: chdir=/usr/local/mysql/  ./scripts/mysql_install_db --datadir=/data/mysql --user=mysql
      tags: data
    - name: config my.cnf
      copy: src=/data/ansible/files/my.cnf  dest=/etc/my.cnf 
    - name: service script
      shell: /bin/cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
    - name: enable service
      shell: /etc/init.d/mysqld start;chkconfig --add mysqld;chkconfig mysqld on  
      tags: service
    - name: PATH variable
      copy: content='PATH=/usr/local/mysql/bin:$PATH' dest=/etc/profile.d/mysql.sh
    - name: secure script
      script: /data/ansible/files/secure_mysql.sh
      tags: script
```

## 5、Playbook中使用handlers和notify

### handlers和notify的使用



```yaml
vim notify_httpd.yml

---
- hosts: web
  remote_user: root
  gather_facts: no
  tasks: 
    - name: install httpd
      yum: name=httpd state=present
    - name: install configre file
      copy: src=files/httpd.conf dest=/etc/httpd/conf/
      notify: restart httpd
      tags: conf
    - name: ensure apache is running
      service: name=httpd state=started enabled=yes
      tags: service
      
    
  handlers: 
    - name: restart httpd
      service: name=httpd state=restarted
```

## 6、Playbook Tags组件

> 标签，在playbook文件中，可以利用tags组件，为特定task指定标签，当在执行playbook的时候，可以只执行特定tags的task，而非整个playbook文件
>
> --list-tags列出所有标签

```yaml
vim tags_httpd.yml

# tags example
---
- hosts: web
  remote_user: root
  gather_facts: no
  
  tasks:
  - name: install httpd
    yum: name=httpd state=present
  - name: install configure file
    copy: src=files/httpd.conf dest=/etc/httpd/conf/
    tags: conf
  - name: start httpd service
    tags: service
    service: name=httpd state=started enabled=yes

# 执行选择tags  使用-t选项
ansible-playbook -t conf,service tags_httpd.yml
```

## 7、Playbook变量使用

> 变量名：仅能有字母、数字和下划线组成，且只能以字母开头

**变量定义：**

```bash
variable=value
```

**范例：**

```bash
http_port=80
```

**调用方式：**

通过{{ variable_name }}调用变量，且变量名前后建议加空格，有事用"{{ variable_name }}"才生效

**变量来源：**

1. ansible的setup facts远程主机的所有变量都可直接调用(在playbook中收集了才能调用)

2. 通过命令行指定变量，优先级最高

   ```bash
   ansible-playbook -e varname=value
   ```

### 使用setup模块中的变量

```bash
vim var.yml

---
- hosts: web
  remote_user: root
  
  tasks: 
    - name: create log file
      file: name=/data/{{ ansible_nodename }}.log state=touch owner=g mode=600
```

### 在playbook命令行中定义变量

```yaml
vim var2.yml

---
- hosts: web
  remote_user: root
  tasks:
    - name: install package
      yum: name={{ pkname }} state=present

# 在执行playbook时，在命令行中执行命令，使用-e对变量进行赋值
ansible-playbook -e pkname=httpd var2.yml
```

### 在playbook文件中定义变量

```yaml
vim var3.yml

---
- hosts: web
  remote_user: root
  vars:
    - username: user1
    - groupname: group1
    
  tasks:
  - name: create group
    group: name={{ groupname }} state=present
  - name: create user
    user: name={{ username }} group={{ groupname }} state=present
    
ansible-playbook -e "username=user2 groupname=group2” var3.ym1
```

### 使用变量文件

> 单独创建一个变量文件存放变量，在playbook中引用变量文件中的变量，比playbook中定义的变量优先级高

```yaml
vim vars.yml

# 变量文件
---
package_name: nginx
service_name: nginx


vim install_var_app.yml

--- 
# install app
- hosts: web
  remote_user: root
  gather_facts: yes
  vars_files:
    - vars.yml   # 调用变量文件
  
  tasks:
    - name: install {{ package_name }}
      yum: name={{ package_name }}
      tags: install
    - name: start {{ package_name }} service
      service: name={{ package_name }} state=started enabled=yes
```

### 主机清单中定义变量

1、主机变量：主机清单文件中为指定的主机定义变量以便于在playbook中使用

```bash
vim /etc/ansible/hosts

[web]
192.168.201.123 host=node1
192.168.201.124 host=node2
[web:vars]
www=gyq.com

# 执行命令修改主机名
ansible web -m hostname -a 'name={{ host }}.{{ www }}'
```

2、组变量

见上方`[web:vars]`，如果变量相同，优先主机变量，而后组变量

## 8、Playbook模板template

### jinja2语言

### template

> template文件必须放在templates目录下，且命名为`.j2`结尾
>
> `yaml/yml`文件需与templates目录平级，调用的时候就不用写路径

利用template同步nginx配置文件：

```yaml
mkdir templates
# 创建模板文件，将本机的nginx配置移过去当做模板文件
mv /etc/nginx/nginx.conf templates/nginx.conf.j2
# 修改如下，作用是参考cpu核数，服务启动后就生成几个进程
worker_processes {{ ansible_processor_vcpus }};

vim tempnginx.yml

---
- hosts: web
  remote_user: root
  
  tasks:
  - name: template config to remote hosts
    template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
  - name: restart nginx
    service: name=nginx state=restarted

ansible-playbook temnginx.yml  --limit 192.168.201.123 # 执行任务

# 模板中修改worker_processes 6;使用命令查看nginx的进程
[root@node1 g]# pstree | grep nginx
        |-nginx---6*[nginx]

# 也可进行加减乘除运算，修改如下内容即可：
worker_processes {{ ansible_processor_vcpus*2 }};
```

### 在template中使用for和if

> 实现动态生成文件功能

例1：

```yaml
# tempnginx2.yml
---
- hosts: web 
  remote_user: root
  vars:
    nginx_vhosts: 
      - 81
      - 82
      - 83
  tasks:
    - name: template config
      template: src=nginx.conf2.j2 dest=/data/nginx.conf  # 测试路径

# templates/nginx.conf2.j2
{% for vhost in nginx_vhosts %}
server {
    listen {{ vhost }}
}
{% endfor %}

ansible-playbook  tempnginx2.yml
# 执行命令生成文件如下

[root@ansible ansible]# ansible web -a 'cat /data/nginx2.conf'
192.168.201.123 | CHANGED | rc=0 >>
server {
    listen 81
}
server {
    listen 82
}
server {
    listen 83
}
```

例2：

```yaml
# tempnginx3.yml
---
- hosts: web
  remote_user: root
  vars:
    nginx_vhosts: 
      - listen: 8080
  tasks:
  - name: config file
    template: src=nginx.conf3.j2 dest=/data/nginx3.conf

# templates/nginx.conf3.j2
{% for vhost in nginx_vhosts %}
server {
    listen {{ vhost.listen }}  # 字典取值
}
{% endfor %}

ansible-playbook  tempnginx3.yml
```

例3：

```yaml
# tempnginx4.yml
---
- hosts: web
  remote_user: root
  vars:
    nginx_vhosts: 
      - listen: 8080
        server_name: "web1.com"
        root: "/var/www/ nginx/web1/"
      - listen: 8081
        server_name: "web2.com"
        root: "/var/www/ nginx/web2/"
      - {listen: 8082, server_name: "web3.com", root: "/var/www/nginx/web3/"
  tasks: 
    - name: template config
      template: src=nginx.conf4.j2 dest=/data/nginx4.conf


{% for vhost in nginx_vhosts %}
server {
    listen {{ vhost.listen }}  # 字典取值
    server_name {{ vhost.server_name }}
    root {{ vhost.root }}
}
{% endfor %}
```

### join

```bash
# eg:
{{ a | join(',')}}   # 将变量a中的元素使用<,>逗号连接
```



##  9、Roles角色

> 模块化实现角色调用

```bash
# 创建roles文件夹
# 在roles文件夹创建如下目录
[root@ansible roles]# mkdir httpd/{tasks,files,handlers} -pv
[root@ansible roles]# tree
.
└── httpd
    ├── files
    ├── handlers
    └── tasks

4 directories, 0 files
```

部署一个服务一般如下几步：创建用户/组，安装包，配置文件，服务启动

### 实现httpd角色

例，配置httpd角色：

```bash
vim roles/httpd/tasks/install.yml

- name: install httpd package
  yum: name=httpd
  
vim roles/httpd/tasks/config.yml
# src中的配置文件放在files文件夹下面
# notify引用的restart在handlers目录的main中
- name: config file
  copy: src=httpd.conf dest=/etc/httpd/conf/ backup=yes
  notify: restart

vim roles/httpd/tasks/index.yml
# src中的index文件放在files文件夹下面
- name: index.html
  copy: src=index.html dest=/var/www/html/

vim roles/httpd/tasks/service.yml

- name: start service
  service: name=httpd state=started enabled=yes

# 此时编写完毕之后的目录结构如下
[root@ansible ansible]# tree
.
└── roles
    └── httpd
        ├── files
        │   ├── httpd.conf
        │   └── index.html
        ├── handlers
        └── tasks
            ├── config.yml
            ├── index.yml
            ├── install.yml
            └── service.yml

5 directories, 6 files

# tasks中的yaml文件的执行先后需要main来控制，名称必须为main
vim roles/httpd/tasks/main.yml

- include: install.yml
- include: config.yml
- include: index.yml
- include: service.yml

# 触发动作
vim roles/httpd/handlers/main.yml

- name: restart
  service: name=httpd state=restarted


# 此时编写完毕之后的目录结构如下  
[root@ansible ansible]# tree 
.
└── roles
    └── httpd
        ├── files
        │   ├── httpd.conf
        │   └── index.html
        ├── handlers
        │   └── main.yml
        └── tasks
            ├── config.yml
            ├── index.yml
            ├── install.yml
            ├── main.yml
            └── service.yml

5 directories, 8 files
```

在playbook中调用角色，注意调用者必须和roles目录平级：

```yaml
# 创建role_httpd.yml
vim role_httpd.yml

- hosts: web
  remote_user: root

  roles:
    - httpd

# 还可以提前创建用户和组 安装的时候会自动创建
vim roles/httpd/tasks/group.yml 

- name: create apache group
  group: name=apache system=yes gid=80
  
vim roles/httpd/tasks/user.yml 

- name: create apache
  user: name=apache system=yes shell=/sbin/nologin home=/var/www/ uid=80 group=apache



# 查看目录结构
[root@ansible ansible]# tree
.
├── role_httpd.yml
└── roles
    └── httpd
        ├── files
        │   ├── httpd.conf
        │   └── index.html
        ├── handlers
        │   └── main.yml
        └── tasks
            ├── config.yml
            ├── group.yml
            ├── index.yml
            ├── install.yml
            ├── main.yml
            ├── service.yml
            └── user.yml
```

## shell忽略错误

> [参考](https://bingostack.com/2021/03/ansible-shell-command/)

```bash
# 设置下面属性
ignore_errors: yes
```

## 获取ip信息

```jinja2
# hostvars获取ip信息
{{ hostvars[groups['control'][0]]['ansible_' + hostvars[groups['control'][0]]['network_interface']]['ipv4']['address'] }}
# net_mask
{{ hostvars[groups['control'][0]]['ansible_' + hostvars[groups['control'][0]]['network_interface']]['ipv4']['network'] }}/{{ hostvars[groups['control'][0]]['ansible_' + hostvars[groups['control'][0]]['network_interface']]['ipv4']['netmask'] }}
```









