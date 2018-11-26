---
layout: post
title: 【笔记】阿里云 centos ftp服务搭建
categories: 编程
tags: 后端 建站
author: adai
---

* content
{:toc}

## ftp 安装
```shell
yum install vsftpd
```

## 修改配置文件

1. **修改/etc/vsftpd/vsftpd.conf**

```
# 禁止匿名用户登录
anonymous_enable=NO
# 设置用户不允许离开主目录
chroot_local_user=YES
# 设置限定不允许离开主目录用户列表
chroot_list_file=/etc/vsftpd/chroot_list
# 设置只允许/etc/vsftpd/user_list中用户登录
userlist_deny=NO
```

2. **创建用户**

```shell
# 创建用户，限制用户登录
useradd $用户名 -s /sbin/nologin -d /home/ftp
#设置用户名
passwd $用户名

```

3. **修改配置文件**

```
修改 /etc/vsftpd/user_list 添加允许访问用户
修改 /etc/vsftpd/chroot_list 添加限制离开用户
```

## 常用命令
```shell
# 配置开机启动
chkconfig vsftpd on
# 启动ftp服务
service vsftpd start
# 停止ftp服务
service vsftpd stop
# 查看ftp服务状态
service vsftpd status
# 查看ftp服务进程
ps -ef | grep vsftpd
ps aux | grep ftp
```
