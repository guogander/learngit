## 目录结构

```bash
[root@ansible uos-build-jobs]# tree
.
├── build_container_image.yml
├── build_iso.yml
├── dib_baremetal.yml
├── fetch_registry.yml
├── fetch_upackages.yml
├── group_vars
│   └── all.yml
├── inventory
├── isotest.sh
├── roles
│   ├── common
│   │   ├── defaults
│   │   │   └── main.yml
│   │   └── tasks
│   │       ├── Kylin.yml
│   │       ├── main.yml
│   │       ├── RedHat.yml
│   │       └── Uniontech.yml
│   ├── isobuild
│   │   ├── defaults
│   │   │   └── main.yml
│   │   ├── files
│   │   │   ├── isolinux
│   │   │   │   └── isolinux.cfg
│   │   │   ├── isotest.sh
│   │   │   ├── ustack_ks.cfg
│   │   │   └── ustack.repo
│   │   ├── meta
│   │   │   └── main.yml
│   │   └── tasks
│   │       └── main.yml
│   ├── kolla
│   │   ├── defaults
│   │   │   └── main.yaml
│   │   ├── files
│   │   │   ├── copy_cacerts.sh
│   │   │   ├── healthcheck_curl
│   │   │   ├── healthcheck_filemod
│   │   │   ├── healthcheck_listen
│   │   │   ├── healthcheck_mariadb
│   │   │   ├── healthcheck_port
│   │   │   ├── healthcheck_socket
│   │   │   ├── kolla_bashrc
│   │   │   └── kolla_template_overrides.j2
│   │   ├── meta
│   │   │   └── main.yml
│   │   ├── tasks
│   │   │   ├── CentOS.yml -> RedHat.yml
│   │   │   ├── hacker.yml
│   │   │   ├── Kylin.yml
│   │   │   ├── main.yaml
│   │   │   ├── RedHat.yml
│   │   │   └── Uniontech.yml
│   │   └── templates
│   │       ├── deployment_dockerfile.j2
│   │       ├── kolla_build.conf.j2
│   │       └── ustack_base_image_dockerfile.j2
│   ├── registry
│   │   ├── defaults
│   │   │   └── main.yml
│   │   ├── meta
│   │   │   └── main.yml
│   │   └── tasks
│   │       └── main.yml
│   └── upackages
│       ├── meta
│       │   └── main.yml
│       └── tasks
│           └── main.yml
├── setup.cfg
├── setup.py
├── test-requirements.txt
├── tests
│   └── playbooks
│       └── run.yml
├── tox.ini
└── uosp_release.yml

31 directories, 57 files
```



## meta

**meta目录中的main.yml文件包含Role元数据，包含的依赖关系。如果这个角色依赖于另一个角色，我们可以在这里定义。例如，nginx角色取决于安装SSL证书的ssl角色。约定必须包含main.yml文件。**

```yaml
---
dependencies:
  - { role: ssl }
```

**如果我调用了“nginx”角色，它将尝试首先运行“ssl”角色。
否则我们可以省略此文件，或将角色定义为没有依赖关系：**

```yaml
---
dependencies: []
```

## defaults

**用于为当前角色设定默认变量，此目录应当包含一个main.yml文件，我们可以在这个角色的其他地方使用这三个变量。我们在上面的模板中看到它们的使用，但是我们也可以在我们定义的任务中看到它们。**

## files

**用来存放由copy模块或script模块等模块调用的文件，如配置文件**

## templates

**用来存放jinjia2模板，template模块会自动在此目录中寻找jinjia2模板文件**

```bash
# setup
ansible_distribution 获取的是CentOS
ansible_os_family  获取的是RedHad
```



```bash
执行入口文件：build_container_image.yml→执行kolla角色→meta文件夹依赖角色common→进入common角色tasks文件夹的main.yml文件→设置时间戳变量timestamp
```



```bash
# 程序跑完执行此命令查看镜像列表
kolla-build --config-file /tmp/kolla_build.conf --list-images
# 对比group_vars/all.yaml的rocky_registry_images:镜像列表

如果是train+source，就是source_full_registry_images里面后缀相同的镜像

# main.yaml  程序跑到这即可
name: prepare kolla conf
```

## rocky_registry_images

```
# rocky_registry_images
1 : barbican-api
3 : barbican-keystone-listener
4 : barbican-worker
6 : chrony
11 : cinder-volume
10 : cinder-scheduler
7 : cinder-api
8 : cinder-backup
12 : cron
14 : etcd
74 : garen
15 : haproxy
75 : hawkeye-alertmanager
76 : hawkeye-blackbox-exporter
77 : hawkeye-cadvisor
78 : hawkeye-consul
80 : hawkeye-grafana
81 : hawkeye-haproxy-exporter
82 : hawkeye-memcached-exporter
83 : hawkeye-mysqld-exporter
84 : hawkeye-node-exporter
85 : hawkeye-nuntius
86 : hawkeye-nvidia-gpu-prometheus-exporter
87 : hawkeye-openstack-exporter
88 : hawkeye-prometheus
89 : hawkeye-rabbitmq-exporter
90 : hawkeye-redis-exporter
16 : ironic-api
18 : ironic-conductor
20 : ironic-inspector
30 : ironic-neutron-agent
21 : iscsid
22 : keepalived
23 : kolla-toolbox
91 : lulu
24 : memcached
25 : mistral-api
27 : mistral-engine
28 : mistral-event-engine
29 : mistral-executor
38 : neutron-metadata-agent
35 : neutron-l3-agent
36 : neutron-lbaas-agent
33 : neutron-dhcp-agent
41 : neutron-openvswitch-agent
42 : neutron-server
47 : nova-api
54 : nova-novncproxy
52 : nova-consoleauth
56 : nova-scheduler
51 : nova-conductor
55 : nova-placement-api
49 : nova-compute
53 : nova-libvirt
59 : nova-ssh
66 : openvswitch-vswitchd
65 : openvswitch-db-server
68 : prometheus-server
69 : rabbitmq
70 : redis
72 : redis-sentinel
73 : tgtd
92 : unitedstack-ceilometer-exporter
93 : unitedstack-neutron-plugin
94 : ustack-blazar-api
96 : ustack-blazar-manager



2 : barbican-base
5 : base
9 : cinder-base
13 : dnsmasq
17 : ironic-base
19 : ironic-pxe
26 : mistral-base
31 : neutron-base
32 : neutron-bgp-dragent
34 : neutron-infoblox-ipam-agent
37 : neutron-linuxbridge-agent
39 : neutron-metadata-agent-ovn
40 : neutron-metering-agent
43 : neutron-server-opendaylight
44 : neutron-server-ovn
45 : neutron-sfc-agent
46 : neutron-sriov-agent
48 : nova-base
50 : nova-compute-ironic
57 : nova-serialproxy
58 : nova-spicehtml5proxy
60 : novajoin-base
61 : novajoin-notifier
62 : novajoin-server
63 : openstack-base
64 : openvswitch-base
67 : prometheus-base
71 : redis-base
79 : hawkeye-docker-state-exporter
95 : ustack-blazar-base
```

## 角色

### setup

```bash
# 查看系统收集信息
ansible localhost -m setup -a 'filter=item'
# setup
ansible_distribution 获取的是CentOS
ansible_os_family  获取的是RedHad
rocky_container_images : x86_64
```

### common

```bash
[root@ansible uos-build-jobs]# tree roles/common/
roles/common/
├── defaults
│   └── main.yml
└── tasks
    ├── Kylin.yml
    ├── main.yml
    ├── RedHat.yml
    └── Uniontech.yml
    
# tasks/main.yml
---
- name: Gather facts
  setup:

- name: generate timestamp
  set_fact:
    timestamp: "{{ lookup('pipe', 'date +%Y%m%d-%H%M') }}"  # 定义变量生成时间戳

- include_tasks: "{{ ansible_os_family.split(' ')[0] }}.yml"  # 引入RedHat.yml剧本


# RedHat.yml
---
- name: list repo files  # 获取上面的find命令所获取的信息赋给repo变量
  command: "find /etc/yum.repos.d/ -name *.repo"
  register: repo   

- name: absent all repo files  # 将上面repo变量中的repo.stdout_lines对应的信息全部删除
  file:
    path: "{{ item }}"
    state: absent
  loop: "{{ repo.stdout_lines }}"  # loop循环读取的内容赋给item

- name: Add devel repofiles  # 添加开发文件
  get_url:
    url: "{{ item }}"
    dest: "/etc/yum.repos.d/"
    force: yes
  loop: "{{ devel_repofiles_url }}"
  when: repo_type == 'devel'

- name: Add production repofiles  # 添加生产文件 when条件判断（默认）
  get_url:
    url: "{{ item }}"
    dest: "/etc/yum.repos.d/"
    force: yes
  loop: "{{ production_repofiles_url }}"
  when: repo_type == 'production'

- name: Ensure yum-plugin-priorities is present  # 保证yum优先级插件存在
  package:
    name: "yum-plugin-priorities"
    state: "present"
```

common角色执行完毕开始kolla角色

### kolla

```yaml
[root@ansible uos-build-jobs]# tree roles/kolla/
roles/kolla/
├── defaults
│   └── main.yaml
├── files
│   ├── copy_cacerts.sh
│   ├── healthcheck_curl
│   ├── healthcheck_filemod
│   ├── healthcheck_listen
│   ├── healthcheck_mariadb
│   ├── healthcheck_port
│   ├── healthcheck_socket
│   ├── kolla_bashrc
│   └── kolla_template_overrides.j2
├── meta
│   └── main.yml
├── tasks
│   ├── CentOS.yml -> RedHat.yml
│   ├── hacker.yml
│   ├── Kylin.yml
│   ├── main.yaml
│   ├── RedHat.yml
│   └── Uniontech.yml
└── templates
    ├── deployment_dockerfile.j2
    ├── kolla_build.conf.j2
    └── ustack_base_image_dockerfile.j2

5 directories, 20 files


# tasks/main.yaml
---
- name: Environment cleaning  # 环境清理，卸载kolla 删除以往程序运行遗留文件
  shell: |
    while [ `docker ps -q | wc -l` -gt 0 ]; do for i in `docker ps -q`; do docker rm -f $i; done; done
    while [ `docker images -q | wc -l` -gt 0 ]; do for i in `docker images -q`; do docker rmi -f $i; done; done
    pip uninstall -y kolla
    rm -rf {{ usr_prefix }}/share/ustack-container-image/
    rm -rf {{ usr_prefix }}/share/kolla/
    rm -rf /tmp/ustack-kolla/*
    rm -rf /tmp/openstack-kolla/

- name: Install packages  # 安装包，引入RedHat.yml剧本
  include: "{{ ansible_distribution.split(' ')[0] }}.yml"
```
```yaml
# 此处开始执行RedHat.yml
# tasks/RedHat.yml
---
- name: Update yum cache  # 更新yum缓存
  command: "{{ item }}"
  loop:
    - "yum clean all"
    - "yum makecache"  # 把服务器的包信息下载到本地电脑缓存起来

- name: Install packages for redhat  # 安装docker
  yum:
    name: "{{ item }}"
    state: present
  loop:
    - "docker"

- name: Update packages for redhat  # 安装python依赖
  yum:
    name: "{{ item }}"
    state: latest  # noqa 403
  loop:
    - "python-pbr"
    - "python-setuptools"
```

```yaml
# 上面执行完毕回到main.yaml继续执行
- name: Install ustack-container-image   # 克隆仓库镜像到本地，默认为stable/rocky分支
  git:
    repo: "https://git.ustack.com/devops/ustack-container-image"
    dest: "{{ usr_prefix }}/share/ustack-container-image"  # usr_prefix=/usr(默认变量判断不为Uniontech)
    version: "stable/{{ branch }}"

- name: Use specify version for openstack-kolla
  block:
    - name: get specify version code  # 获取指定版本的kolla
      git:
        repo: "{{ kolla_git_repo }}"  # 默认地址
        dest: /tmp/openstack-kolla
        version: "{{ kolla_refs }}"   # kolla_refs: "stable/{{ branch }}"  默认rocky

    - name: install kolla
      command: "pip install file:///tmp/openstack-kolla -i https://devpi.apps.ustack.com/root/pypi/+simple -c https://git-mirror.ustack.com/cgit/openstack/requirements/plain/upper-constraints.txt?h=stable/{{ branch }} --ignore-installed"  # 忽略已经安装的包
  become: true

- name: Hack for kolla  # 开始hacker.yml剧本
  include: hacker.yml
```
```yaml
# hacker.yml
# lineinfile :将<path>路径的文件的<regexp>匹配的一行替换为<line>里面的内容;
---
- name: Fix BUG 1865119 (Review 710498) for kolla_toolbox_pip_virtualenv_package
  lineinfile:
    path: "{{ usr_prefix }}/share/kolla/docker/kolla-toolbox/Dockerfile.j2"
    regexp: 'kolla_toolbox_pip_virtualenv_packages|customizable("pip_packages")'
    line: '{% raw %}RUN {{ macros.install_pip(kolla_toolbox_pip_virtualenv_packages|customizable("pip_virtualenv_packages"), constraints=false) }} \{% endraw %}'

- name: Change barbican-api to use mod_wsgi with apache
  lineinfile:
    path: "{{ usr_prefix }}/share/kolla/docker/barbican/barbican-api/Dockerfile.j2"
    regex: 'USER barbican'
    line: 'USER root'

- name: Change octavia-api to use mod_wsgi with apache
  lineinfile:
    path: "{{ usr_prefix }}/share/kolla/docker/octavia/octavia-api/Dockerfile.j2"
    regex: 'USER octavia'
    line: 'USER root'

- name: Enhancement for kolla log
  lineinfile:
    path: "{{ python_path }}/kolla/common/utils.py"
    regexp: 'logging.BASIC_FORMAT'
    line: '        handler.setFormatter(logging.Formatter("%(asctime)s.%(msecs)03d %(process)d %(thread)d %(threadName)s %(levelname)s %(name)s [-] %(message)s"))'
```
```yaml
# 继续执行main.yaml
- name: ensure docker service started # 启动docker服务
  systemd:
     name: docker
     state: started
     enabled: yes

- name: generate production kolla rpm_setup_config  # 设置rpm_setup_config变量(默认)
  set_fact:
     rpm_setup_config: "ustack-{{ branch }}-production.repo,ustack-ceph-{{ ceph_branch }}.repo,elasticsearch.repo"
  when: repo_type == 'production'

- name: generate devel kolla rpm_setup_config  # 开发版的rpm_setup_config变量
  set_fact:
     rpm_setup_config: "ustack-{{ branch }}-production.repo,ustack-{{ branch }}-devel.repo,ustack-ceph-{{ ceph_branch }}.repo,elasticsearch.repo"
  when: repo_type == 'devel'

- name: create dir  # 创建目录
  file:
    path: "{{ item }}"
    state: directory
    mode: 0755
  loop:
    - "{{ work_dir }}"  # work_dir: /tmp/ustack-kolla/
    - "{{ logs_dir }}"  # logs_dir: /tmp/ustack-kolla/
    - "{{ ustack_base_image_dir }}"  # ustack_base_image_dir: /tmp/ustack_base_image/
    - "{{ deployment_image_dir }}"   # deployment_image_dir: "/tmp/deployment_image/"

- name: prepare kolla override files   # 准备文件
  copy:
    src: "{{ item.src }}"
    dest: "{{ item.dest }}"
  loop: "{{ kolla_override_files }}"

- name: Set container_images for rocky on x86_64  # 设置rocky版本 容器镜像  x86_64平台
  set_fact:
    container_images: "{{ rocky_container_images }}"   # all.yml   rocky容器镜像列表
  when:
    - ansible_architecture == 'x86_64'  # "ansible_architecture": "x86_64"
    - branch == 'rocky'

- name: Set container_images for rocky on aarch64  # 设置rocky版本 容器镜像  aarch64平台
  set_fact:
    container_images: "{{ rocky_container_images | difference(rocky_container_images_aarch64_exclude) }}"
  when:
    - ansible_architecture != 'x86_64'
    - branch == 'rocky'

- name: Set container_images for train on x86_64  # 设置train版本容器镜像 x86_64平台
  set_fact:
    container_images: "{{ train_container_images }}"
  when:
    - ansible_architecture == 'x86_64'
    - branch == 'train'
    - install_type == 'binary'   # 安装类型二进制

- name: Set container_images for train on aarch64  # 设置train版本容器镜像  aarch64平台
  set_fact:
    container_images: "{{ train_container_images | difference(train_container_images_aarch64_exclude) }}"
  when:
    - ansible_architecture != 'x86_64'
    - branch == 'train'
    - install_type == 'binary'

- name: Set source_container_images for train on x86_64
  set_fact:
    container_images: "{{ source_container_images }}"
  when:
    - ansible_architecture == 'x86_64'
    - branch == 'train'
    - install_type == 'source'

- name: Set source_container_images for rocky on aarch64
  set_fact:
    container_images: "{{ source_container_images | difference(train_container_images_aarch64_exclude) }}"
  when:
    - ansible_architecture != 'x86_64'
    - branch == 'train'
    - install_type == 'source'

- name: prepare kolla conf
  template:
    src: kolla_build.conf.j2
    dest: "{{ kolla_build_conf }}"

- name: set all of docker dir needs to copy repo
  set_fact:
    docker_dirs:
      - "{{ kolla_dir }}/base/"

- name: Copy repo file to docker_dirs
  get_url:
    url: "http://repo.ustack.com/repofiles/{{ item.0 }}"
    dest: "{{ item.1 }}"
    force: yes
  loop: "{{ rpm_setup_config.split(',')|product(docker_dirs)|list }}"

- name: Login to "{{ registry }}"
  docker_login:
    registry: "{{ registry }}"
    username: "{{ docker_user }}"
    password: "{{ docker_pass }}"
  tags: docker_login

- name: prepare ustack base image dockerfile
  template:
    src: ustack_base_image_dockerfile.j2
    dest: "{{ ustack_base_image_dir }}/Dockerfile"

- name: Build ustack base image
  docker_image:
    name: "{{ base_image }}"
    path: "{{ ustack_base_image_dir }}"
    tag: "{{ base_tag }}"
    push: true
  tags:
    - build_base_image

# skip this for train daily release now, fix it later when train has maine and kolla-ansible rpm package
- block:
    - name: prepare deployment image dockerfile
      template:
        src: deployment_dockerfile.j2
        dest: "{{ deployment_image_dir }}/Dockerfile"

    - name: Build deployment image
      docker_image:
        name: "{{ deployment_image }}"
        path: "{{ deployment_image_dir }}"
        tag: "{{ base_tag }}"
        push: true

    - name: synchronize the deployment image to the repository
      shell:
        executable: /bin/bash
        cmd: |
          docker pull registry.ustack.com/centos/deployment:{{ base_tag }}
          docker save registry.ustack.com/centos/deployment:{{ base_tag }} -o /opt/deployment.tar

          pushd /opt
          tar -zcvf deployment-{{ release }}-{{ ansible_architecture }}-{{ timestamp }}.tar.gz deployment.tar
          /usr/bin/md5sum deployment-{{ release }}-{{ ansible_architecture }}-{{ timestamp }}.tar.gz > /opt/deployment-{{ release }}-{{ ansible_architecture }}-{{ timestamp }}.tar.gz.md5sum
          popd

          # Copy image to repo
          export RSYNC_DEST="root@repo.ustack.com:/opt/repos/deploy-tools/{{ ansible_architecture }}/{{ branch }}/deployment"
          /usr/bin/rsync -aP /opt/deployment-{{ release }}-{{ ansible_architecture }}-{{ timestamp }}.tar.gz ${RSYNC_DEST}/deployment-{{ release }}-{{ ansible_architecture }}-{{ timestamp }}.tar.gz

          /usr/bin/rsync -aP /opt/deployment-{{ release }}-{{ ansible_architecture }}-{{ timestamp }}.tar.gz.md5sum ${RSYNC_DEST}/deployment-{{ release }}-{{ ansible_architecture }}-{{ timestamp }}.tar.gz.md5sum

          /usr/bin/rsync -aP /opt/deployment-{{ release }}-{{ ansible_architecture }}-{{ timestamp }}.tar.gz ${RSYNC_DEST}/deployment-{{ release }}-{{ ansible_architecture }}.tar.gz

          /usr/bin/rsync -aP /opt/deployment-{{ release }}-{{ ansible_architecture }}-{{ timestamp }}.tar.gz.md5sum ${RSYNC_DEST}/deployment-{{ release }}-{{ ansible_architecture }}.tar.gz.md5sum

          # Clean
          docker rmi registry.ustack.com/centos/deployment:{{ base_tag }}
          rm -f /opt/deployment*
  when:
    - branch != 'train'

# run kolla_build
- name: Build kolla image
  shell:
    executable: /bin/bash
    cmd: |
      set -o pipefail
      kolla-build --config-file {{ kolla_build_conf }} 2>&1 | tee {{ logs_dir }}kolla_build.log
  register: kolla_build_logs

# Only build in one main release
- block:
  - name: build docs images
    docker_image:
      name: "{{ registry ~ '/' ~ doc_namespace ~ '/' ~ item }}"
      tag: "{{ kolla_tag }}"
      build:
        path: "{{ work_dir  ~ 'docker/' ~ item }}"
      push: yes
    loop: "{{ doc_images }}"
  when:
    - branch != 'train'

- name: Get kolla build logs
  debug:
    msg: "{{ kolla_build_logs.stdout_lines }}"
```

```bash
# 遗留问题
1、定位到创建容器镜像的地方，仔细查看
2、试试看能不能使用命令行过滤即将输出的容器镜像列表
```

