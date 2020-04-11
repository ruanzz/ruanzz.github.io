---
title: 高性能Nginx(一)
date: 2018-12-22 09:56:20
tags: [Nginx]
toc: true
category: Web
---
<!-- ![Nginx服务网关](/asset/img/nginx/nginx.png) -->

Nginx是后端开发人员应该都会接触的一款基础软件，以支持高并发访问面世，在开发环境中经常会配置一些Nginx参数来支持我们开发，那么问题来了，你有系统的了解过吗？

<!-- more -->
## Nginx是什么

Nginx是一个高性能的Web和反向代理服务器。

## Nginx应用场景
### Web服务器
Web服务器一般用来处理JavaScript，HTML，CSS等静态资源。同类产品有Apache，IIS,但是Nginx是C语言写的，并且使用的epoll and kqueue开发模型，占用更少资源，却拥有更高的性能。一般谈到Web服务器的时候往往会谈到Web应用服务器，比如Tomcat、Jetty等，Web应用服务器是处理Servlet程序的载体，Web应用服务器的并发能力远低于Web服务器。

### 反向代理服务器
首先理解一下什么是代理，默认是正向代理，代理的是客户端，通过客户端的设置，实现让一台服务器(代理服务器)来代理客户端，客户端所有的请求都交由代理服务器来处理，比如翻墙用的梯子就是正向代理最好案例。认识了代理，反向代理就和正向代理正好相反，用一台服务器代理真实的服务器，当有用户进行访问请求时，不再访问真实的服务器，而是访问代理服务器，有代理服务器将请求转发交给真实的服务器处理。
Nginx用来当反向代理服务器时，需要在Nginx的配置文件中配置好反向代理的规则，不同的请求交给不同的真实服务器处理，当请求到达Nginx时，Nginx会根据已经定义好的规则进行请求的转发，从而实现路由的功能，并且可以解决应用中通过域名访问的时候的端口问题。

在开发中，我们更多是把他当做网关，因为它具备网关必备的功能：
- 反向代理
- 负载均衡
- 动态路由
- 请求过滤

和SpringCloud中的Zuul网关一样，但是Zuul网关我们是用来做服务内部调用的网关，带有鉴权作用，Nginx作为外部网关，因为并发能力太强了，单机可以支持50000个并发，直接碾压Tomcat的200个并发。

## Nginx实践
安装和启动这里就直接略过了，看到Nginx的欢迎页面就说明Nginx启动好了。

![访问Nginx](/asset/img/nginx/visit-nginx.png)

### 配置文件
nginx.conf是Nginx的配置文件，它是由特定的标识符(指令符)分为多个不同的模块，这个文件就是前文讲过的配置反向代理规则的地方，接下来了解下Nginx的配置文件。
指令符分为简单指令和块指令:
- 简单指令格式： name value;
- 块指令格式：  {name value;} 

块指令也可以称为上下文(e.g. events,http,server,location)，值得注意的是，所有不属于块指令的简单指令都属于main上下文，http块指令也属于main上下文，server块指令属于http上下文

### 配置静态访问
Nginx默认在http上下文中配置了一个server的块指令，监听80端口，访问`/`的时候就进入这个server中
，并且配置了server的根目录和首页。
```shell
server {
    listen       80;
    server_name  localhost;
    location / {
        root   html
        index  index.html index.htm;
    }
}
```
### 配置反向代理
server可以配置多个，相当于配置多台server提供服务，但是都是通过80端口访问Nginx之后，通过Nginx的proxy_pass进行转发。
```shell
server {
    listen       80;
    server_name  tomcat111;
    location / {
        proxy_pass  http://10.10.111.111:8090
        index  index.html index.htm;
    }
}
server {
    listen       80;
    server_name  tomcat112;
    location / {
        proxy_pass  http://10.10.111.112:8090
        index  index.html index.htm;
    }
}
```

### 配置负载均衡

1.轮询(默认)
每个请求按照时间顺序逐一分配到不同的后端服务器，按照服务器列表顺序来提供服务，如果down掉了，可以自动跳过。
```shell
upstream backserver{
    server  10.10.111.111;
    server  10.10.111.112;
}
```

2.指定权重

指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况
```shell
upstream backserver{
    server 10.10.111.111 weight=10;
    server 10.10.111.112 weight=5;
}
```

3.IP绑定
每个请求按照请求IP的HASH结果分配，这样子每个IP固定访问一个后端服务，这个可以解决集群的session共享问题。
```shell
upstream backserver{
    ip_hash;
    server  10.10.111.111;
    server  10.10.111.112;
}
```
### 配置Nginx解决跨域问题
跨域问题是浏览器为了安全性做的一个限制，请求的url和浏览器地址栏的url中端口|域名|协议不一样都属于跨域，我们可以借助Nginx的反向代理解决这个这个问题。
```shell
server {
    listen       80;
    server_name  www.xxx.com;
    location /order {
        proxy_pass  http://search.a.com:8090
        index  index.html index.htm;
    }
    location /manage {
        proxy_pass  http://manage.a.com:8090
        index  index.html index.htm;
    }
}
```

### 配置防盗链
利用referer来做防盗链检查，只允许某个域名来请求资源，如果不是从某个域名来的，返回403未授权

```shell
server {
    listen       80;
    server_name  www.xxx.com;
    location ~ .*\.(jpg|jpeg|JPG|png|gif|icon)$ {
        valid_referers blocked http://www.xxx.com www.xxx.com;
        if ($invalid_referer) {
            return 403;
        }
    }
}
```

## Nginx高可用
Nginx作为外部网关，如果只是一台的话存在单点故障，那么整个后端服务都不能提供服务，所以要搭建Nginx高可用，现在互联网应用基本都在关注这两点，高并发和高可用。我们借助Keepalived来实现高可用,那么接下来我们来了解一下Keepalived。
Keepalived从字面意思就可以知道它的作用--保持活着。官方的说辞是主要提供负载均衡(Load Balance)和高可用(high-availablity)功能。负载均衡实现需要依赖Linux的虚拟内核模块ipvs(IP虚拟服务器，用于实现网络服务的负载均衡)，高可用通过VRRP协议实现多台机器之间的故障转移业务。

### 配置Keepalived
Keepalived的配置文件为keepalived.conf，配置说明如下:
Master节点
```shell
global_defs {
   router_id nginx111 ##标识节点的字符串，通常为hostname
}

vrrp_script chk_nginx {
    script "/etc/keepalived/nginx_check.sh" ##执行脚本位置，检查节点状态，根据返回结果动态调整权重
    interval 2 ##检测时间间隔
    weight -20 ## 如果条件成立则权重减20（-20）
}

vrrp_instance VI_1 {
    state MASTER   ## 主节点为MASTER，备份节点为BACKUP
    interface en0  ## 绑定虚拟IP的网络接口（网卡）
    virtual_router_id 111  ## 虚拟路由ID号
    mcast_src_ip 10.10.111.111 ## 本机ip地址
    priority 100  ##优先级配置（0-254的值）
    Nopreempt  
    advert_int 1 ## 组播信息发送间隔，俩个节点必须配置一致，默认1s
    authentication {  
        auth_type PASS
        auth_pass 123456 ## 真实生产环境下对密码进行匹配
    }

    track_script {
        chk_nginx
    }

    virtual_ipaddress {
        10.10.111.110 ## 虚拟ip(vip)，可以指定多个
    }
}
```
Backup节点:

```shell
global_defs {
   router_id bhz006
}

vrrp_script chk_nginx {
    script "/etc/keepalived/nginx_check.sh"
    interval 2
    weight -20
}

vrrp_instance VI_1 {
    state BACKUP
    interface en0
    virtual_router_id 112
    mcast_src_ip 10.10.111.112
    priority 90 ##优先级配置,比master低
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 123456
    }

    track_script {
        chk_nginx
    }

    virtual_ipaddress {
        10.10.111.110
    }
}
```
nginx_check.sh 脚本:

```shell
#!/bin/bash
A=`ps -C nginx –no-header |wc -l`
if [ $A -eq 0 ];then
    /usr/local/nginx/sbin/nginx
    sleep 2
    if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
        killall keepalived
    fi
fi
```

### 测试
启动Nginx和Keepalived，检查vip是否配置在对应的网络接口，干掉Master，Nginx还是能够正常提供服务。一般当某些服务出现问题，特别是像Nginx这些重要组件出现问题的时候，一般会从脚本文件中加上发送邮件的代码，或者有监控平台的话会以各种各样的方式比如短信、微信、钉钉进行通知。