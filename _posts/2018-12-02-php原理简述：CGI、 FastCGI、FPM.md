---
layout: post
title: 【笔记】php原理简述：CGI、 FastCGI、FPM
categories: 编程
tags: 后端 建站
author: adai
---

* content
{:toc}


在配置Nginx服务器的时候，发现php需要配置fastcgi才能执行，简单了解了一下

## 基础

### 静态网页请求处理

![静态网页请求处理]({{site.url}}/assets/2018-12-2/webserver.png)

### 动态网页请求处理
![动态网页请求处理]({{site.url}}/assets/2018-12-2/cgi.png)

### 概念

- CGI: WebServer 和 Web Application 之间的**数据交换协议**
- FastCGI: 对CGI协议的性能优化
- PHP-CGI: PHP提供的CGI协议接口程序
- PHP-FPM: PHP提供的CGI协议接口程序，提供智能任务管理


##FastCGI

### 架构：C/S

![fastcgi工作原理]({{site.url}}/assets/2018-12-2/fastcgi.png)

### 工作原理概述
1. web server启动时启动FastCG进程管理程序
2. FastCG进程管理程序初始化，启动多个CGI解释器进程,等待web server 连接
3. FastCGI管理器分配请求处理给子进程
4. 子进程处理完后响应数据给webserver，等待下个请求

### fastcgi vs cgi

**cgi:**
web server请求 ==》启动cgi程序进程，处理请求==》响应处理

**fastcgi：**
web server启动==》初始化fastcgi管理器
web server请求==》管理器分配子进程处理===》子进程响应处理、结束===》等待下个请求

## PHP-FPM
php-cgi 是php提供的cgi处理程序，仅处理请求cgi解析，返回响应结果
php-fpm 是一个fastcgi管理器，用于管理进程池，处理web server请求


![思维导图]({{site.url}}/assets/2018-12-2/php_cgi_mindmap.svg)


# 参考资料
https://www.awaimai.com/371.html#iHPf62z9AfU8yy2YG
