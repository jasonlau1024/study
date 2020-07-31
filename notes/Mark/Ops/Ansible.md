

Ansible 是款使用基于Python开发，轻量级的自动化运维工具，可用来批量系统配置、批量程序部署、批量运行命令等。

相关链接：

Github地址：<https://github.com/ansible/ansible/>

官网地址：[https://docs.ansible.com](https://docs.ansible.com/)

playbook分享平台：[https://galaxy.ansible.com](https://galaxy.ansible.com/)

# 概述

## 基本架构

![“ansible infrastructure”的图片搜索结果"](http://leopard.in.ua/presentations/rubyc_2015/pictures/ansible_fig.png)

- 核心：Ansible
- 核心模块(Core Modules)：ansible自带模块
- 自定义模块(Custom Modules)：添加自定义模块，扩展功能。
- 插件(Plugins)：作为模块功能的补充，如：借助插件完成日志记录、邮件等功能。
- 剧本(Playbooks)：作为ansible实现配置和部署的编排，描述远程执行策略或过程。
- 连接插件(Connection Plugins)：ansible通过连接插件连接到各主机上，负责和被管理节点实现通信。
- 主机清单(Host Inventory)：定义远程管理的主机。

## 工作机制

Ansible执行命令时，通过其底层传输连接模块，将一个或数个文件，或者定义一个Play或Command命令传输到远程服务器/tmp目录的临时文件，并在远程执行这些Play/Command命令，然后删除这些临时文件，同时回传整体命令执行结果

![Ansible架构](https://cdn.educba.com/academy/wp-content/uploads/2019/10/ansible-architecture.png)

#### Puppet vs Saltstack vs Ansible

| 项目                         | Puppet                         | Saltstack       | Ansible            |
| ---------------------------- | ------------------------------ | --------------- | ------------------ |
| 开发语言                     | Ruby                           | Python          | Python             |
| 是否有客户端                 | 是                             | 是              | 否                 |
| 是否支持二次开发             | 不支持                         | 支持            | 支持               |
| 服务器与远程机器是否相互验证 | 是                             | 是              | 是                 |
| 服务器与远程机器通信是否加密 | 是，标准SSL协议                | 是，使用AES加密 | 是，使用OpenSSH    |
| 是否提供WEB UI               | 提供                           | 提供            | 提供，但是商业版本 |
| 配置文件格式                 | Ruby语法                       | YAML            | YAML               |
| 命令行执行                   | 不支持，但可以通过配置模块实现 | 支持            | 支持               |

# 常用模块介绍

## ping 模块

判断远程主机的网络是否畅通

`ansible cluster_hosts -m ping`

## copy模块

`copy`模块在ansible里的角色就是把ansible执行机器上的文件拷贝到远程节点上。(与`fetch`模块相反的操作)

- `src=/src_path` 指定本机文件的绝对路径。

  如果指定的是目录，将拷贝整个目录；如果以`/`作为结尾，值拷贝目录里的所有东西，类似rsync。

- `dest=/dest_path` 指定远程主机文件的绝对路径。

- `backup=yes|no` 指定在拷贝前是否备份远程主机源文件。

实例①.拷贝前备份

`ansible node01 -m copy -a 'src=/etc/nginx/nginx.conf dest=/etc/nginx/nginx.conf backup=yes'`

**其它常用模块参数：**

| 参数名         | 是否必须 | 默认值 | 选项   | 说明                                                         |
| -------------- | -------- | ------ | ------ | ------------------------------------------------------------ |
| content        | no       |        |        | 用来替代src，用于将指定文件的内容，拷贝到远程文件内          |
| directory_mode | no       |        |        | 这个参数只能用于拷贝文件夹时候，这个设定后，文件夹内新建的文件会被拷贝。而老旧的不会被拷贝 |
| follow         | no       | no     | yes/no | 当拷贝的文件夹内有link存在的时候，那么拷贝过去的也会有link   |
| force          | no       | yes    | yes/no | 默认为yes,会覆盖远程的内容不一样的文件（可能文件名一样）。如果是no，就不会拷贝文件，如果远程有这个文件 |
| group          | no       |        |        | 设定一个群组拥有拷贝到远程节点的文件权限                     |
| mode           | no       |        |        | 等同于chmod，参数可以为“u+rwx or u=rw,g=r,o=r”               |
| owner          | no       |        |        | 设定一个用户拥有拷贝到远程节点的文件权限                     |

## shell 模块

向远程主机发送执行命令，shell 模块是通过/bin/sh进行执行，所以shell 模块可以执行任何命令。

- `chdir=/path` 指定远程主机的工作目录，类似`cd`到目录再执行命令。

实例①.让所有节点进入scripts目录运行somescript.sh并把log输出到somelog.txt

`ansible -i hosts all -m shell -a "sh somescript.sh >> somelog.txt" chdir=scripts/`

实例②.cd到编译目录，编译安装

`ansible -i hosts all -m shell -a "./configure && make && make insatll" chdir=/opt/nginx`

**其它常用模块参数：**

| 参数    | 是否必须 | 默认值 | 选项 | 说明                                             |
| ------- | -------- | ------ | ---- | ------------------------------------------------ |
| creates | no       |        |      | 跟command一样的，如果某个文件存在则不运行shell   |
| removes | no       |        |      | 跟command一样的，如果某个文件不存在则不运行shell |

## command 模块

ansible默认模块，用于运行系统命令，不支持管道符和变量等（"<", ">", "|", and "&"等）。用法类似shell

实例①.关闭远程主机

`nsible -i hosts all -m command -a "/sbin/shutdown -t now"`

**常用模块参数：**

| 参数       | 是否必须 | 默认值 | 选项 | 说明                                                     |
| ---------- | -------- | ------ | ---- | -------------------------------------------------------- |
| chdir      | no       |        |      | 运行command命令前先cd到这个目录                          |
| creates    | no       |        |      | 如果这个参数对应的文件存在，就不运行command              |
| executable | no       |        |      | 将shell切换为command执行，这里的所有命令需要使用绝对路径 |
| removes    | no       |        |      | 如果这个参数对应的文件不存在，就不运行command            |

## fetch 模块

拉取模块，将远程主机中的文件拷贝到本机中。

- `dest=/path` 指定本地存放目录，在该目录下回自动添加远程主机的主机名的目录，拉取文件保存在对应主机的目录下。
- `src=/path/file` 指定拉取远程主机的文件，这里不能是目录

实例①.`ansible node01 -m fetch -a "src=/etc/nginx/nginx.conf dest=/opt/backup"`

**其它常用模块参数：**

| 参数              | 必填 | 默认值 | 选项   | 说明                                                         |
| ----------------- | ---- | ------ | ------ | ------------------------------------------------------------ |
| Fail_on_missing   | No   | No     | Yes/no | 当源文件不存在的时候，标识为失败                             |
| Flat              | No   |        |        | 允许覆盖默认行为从hostname/path到/file的，如果dest以/结尾，它将使用源文件的基础名称 |
| Validate_checksum | No   | Yes    | Yes/no | 当文件fetch之后进行md5检查                                   |

## file 模块

主要用来设置文件、链接或目录的属性，也可用于移除。

- `path=/path` 指定要控制的文件路径
- `owner=user` 指定文件属主
- `group=group` 指定文件属组
- `mode` 指定文件权限
- `recurse=yes|no` 当指定的是目录是，是否递归设置。

实例①.`ansible node01 -m file -a "path=/test/file owner=centos group=centos mode=0644"`

实例②.`ansible node01 -m file -a "path=/tmp/dir/ owner=centos group=centos mode=0644 recurse=yes"`

**其它常用模块参数：**

| 参数   | 必填 | 默认 | 选项   | 说明                                                         |
| ------ | ---- | ---- | ------ | ------------------------------------------------------------ |
| Follow | No   | No   | Yes/no | 这个标识说明这是系统链接文件，如果存在，应该遵循             |
| Force  | No   | No   | Yes/no | 强制创建链接在两种情况下：源文件不存在（过会会存在）；目标存在但是是文件（创建链接文件替代） |
| Src    | No   |      |        | 文件链接路径，只有状态为link的时候，才会设置，可以是绝对相对不存在的路径 |

## yum 模块

安装包管理模块。

- `conf_file` 指定远程执行yum时所依赖的yum配置文件。
- `list` 相当于`yum list xx`
- `name=pakname` 指定安装的包名。
- `state=present|latest|absent `指定包的动作，present/latest表示安装，absent表示remove
- `update_cache` 安装包前执行更新list

实例①.`ansible node01 -m yum -a “name=httpd`

## service 模块

用于服务管理，类似centos6的`service`命令，centos7的`systemctl`命令

- `name=service` 指定要管理的服务名。
- `enabled=yes|no` 设置是否开机启动。
- `state=stared|stoped|restarted|reloaded` 指定管理服务的动作。

实例①.`ansible node01 -m service -a "name=httpd state=started"`

实例②.`ansible node01 -m service -a "name=httpd enabled=yes"`

## cron 模块

用于管理计划任务

实例：

`ansible test -m cron -a 'name="a job for reboot" special_time=reboot job="/some/job.sh"'`

`ansible test -m cron -a 'name="yum autoupdate" weekday="2" minute=0 hour=12 user="root`

`ansible test -m cron  -a 'backup="True" name="test" minute="0" hour="5,2" job="ls -alh > /dev/null"'`

## user 模块

该模块调用的是useradd,userdel,usermod三个指令。

- `name=username` 指定用户名
- `groups=groupname `指定用户组
- `password=passwd` 指定用户密码，这里的`passwd`不能是明文，因为`passwd`会直接写入`/etc/shadow`文件中，所以需要预先加密，centos6和centos7使用SHA512加密，

## script 模块

在远程主机执行本地脚本

示例：`ansible node01 -m script -a /tmp/hello.sh`

# playbook

ansible的playbook的文件格式为YAML格式。

核心元素

- `tasks` 任务，由模板定义的操作列表。
- `variables` 变量。
- `template` 模板，即使用模板语法的文件。
- `handlers` 处理器，当某条件满足时，触发执行得操作。
- `roles` 角色。



ansible-playbook 命令：`ansible-playbook playbook.yml [options]`





常用模块配置



hosts 和 users 介绍

```shell
---
- hosts: abc               #指定主机组，可以是一个或多个组。
  remote_user: root                #指定远程主机执行的用户名
```

tasks 是剧本的主体，tasks列表中定义了在远程hosts依次执行的任务。

注①.如果一个host执行task失败，整个tasks都会回滚，需修正错误，重新执行。

注②.play中只要执行得命令返回值不为0，就会抛出错误，停止tasks。

添加`ignore_errors: True` 忽略错误。

```shell
# cat start_httpd.yml
---
- hosts: 192.168.3.173    //指定主机
  remote_user: centos     //指定在被管理的主机上执行任务的用户
  become: yes             //使用sudo
  become_method: sudo
  tasks:                  //任务列表↓
  - name: disable seliunx    //任务名
    command: '/sbin/setenforce 0'  //任务动作
    ignore_errors: True      //忽略错误
  - name: start httpd
    service: name=httpd state=started    //调用service模块 开启httpd 服务
# ansible-playbook start_httpd.yml --syntax-check    #检查yaml文件的语法是否正确
# ansible-playbook start_httpd.yml
```



notify 和 handlers 介绍

notify 和 handlers 组合使用，handlers定义task列表，notify 通过定义的name值调用handlers定义的task，减少重复指令的编写，一般只用于触发系统重启操作。



引用变量{{vars}}

`vars:`  自定义变量

```shell
---
- hosts: etcd-cluster
  remote_user: centos
  become: yes
  become_method: sudo
  vars:
  - myip: "{{ansible_all_ipv4_addresses}}"  //定义变量myip，ansible_all_ipv4_addresses是ansible内置变量。
  tasks:
  - name: copy file
    copy: content={{myip}} dest=/opt/iplist_txt
```



条件判断

when，指定一个条件表达式，条件成立，task才执行

组合条件判断：

1）when指定多条表达式，需同时满足多个条件，task才执行

2）使用 or 和 and 组合

```shell
## when指定多条表达式
tasks:
  - name: "shut down CentOS7 systems"
  command: /sbin/shutdown -r now
  when:
  - ansible_distribution == "CentOS"
  - ansible_distribution_major_version == "7"
## 使用 or 和 and 组合
tasks:
  - name: "shut down CentOS6 and Debian7 systems"
  command: /sbin/shutdown -r now
  when: (ansible_distribution == "CentOS" and ansible_distribution_major_version == "6") or (ansible_distribution == "Debian" and ansible_distribution_major_version == "7")
```




示例①.指定一个主机名，对这个主机进行配置操作。

```shell
## 目录结构
config-ansible
    |___config_hosts.yml
    |___roles
             |___config_hosts
                        |___tasks
                                |___main.yml
                                |___config.yml
## 说明
config_hosts.ym 作为总入口，调用roles/config_hosts/tasks目录下的脚本
执行命令ansible-playbook config_hosts.yml 运行剧本
# cat config_hosts.yml
---  ## 说明该文件是YAML文件，非必须
- hosts: node01  ## 定义该playbook针对的目标主机
roles:  ## 定义角色目录
- config_hosts  ## 定义角色
## main.yml的内容为
---
- include: config.yml  ## 指定此角色要导入的task文件
## config.yml文件，该文件才是真正执行得任务代码
---
- name: copy test.file  ## 使用copy模块，拷贝文件
copy:
src: /home/test.file
dest: /home/test.file
owner: root
group: root
mode: 0777
force: yes

- name: exec hello world script  ## 使用script模块，在目标主机执行/home目录下的helloworld.sh
script: /home/helloworld.sh

- name: rm test.file
file: path=/home/test.file state=absent  ## 使用file模块把目标主机的/home/test.file文件删除
```






