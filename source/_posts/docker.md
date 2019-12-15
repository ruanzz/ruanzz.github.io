---
title: Docker
tags: [Docker]
toc: true
date: 2019-11-13 21:22:13
category: 云计算
---

Docker是一种容器引擎，主要用于应用的打包和运行，保证环境的一致性，目前与微服务配合一起使用的比较多。Docker官方说法是与集装箱功能类似，操作对象是应用，对应用进行Build -> Share -> Run。目前Docker为Kubernetes的默认容器引擎，当然还可以选择rkt，后续会写Kubernetes系列文章，本文先介绍一下Docker，为后续文章做一下铺垫。

<!-- more -->

## 安装

这里用CentOS7.7来做宿主机做演示，官方文档写的很详细，可以参照官方文档来操作。

首先关闭防火墙和selinux

```shell
systemctl stop firewalld && systemctl disable firewalld
sed -i 's/enforcing/disabled/' /etc/selinux/config
setenforce 0
```
 
添加aliyun提供的Docker源，加速docker-ce的下载
```shell
wget -O /etc/yum.repos.d/docker-ce.repo  https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

直接通过yum安装docker-ce即可
```shell
yum install docker-ce-18.06.1.ce-3.el7 -y
```

启动docker，并设置开机启动
```shell
systemctl enable docker && systemctl start docker
```

查看docker版本`docker version`以及docker信息`docker info`。

先来回顾一下我们之前说的内容，Docker主要用于应用的打包和运行，打包其实就是构建镜像，运行就是根据打包出来的镜像运行一个容器。所以下面我们来看一下镜像和容器。

## 镜像

镜像，与OpenStack，VMware这些说的镜像是同一个东西吗？其实他们的作用是类似的，但是实现区别非常大，OpenStack和VMware的镜像是一个完整的操作系统，Docker的镜像是一个不包含Linux内核的精简系统，这个与他们各自的架构有关。

Docker官方默认提供了一个公共的镜像仓库Docker Hub，我们通过`docker pull`从Docker Hub上拉取镜像。但是这个是国外，虽然可以访问，但是比较慢，所以一般都会将镜像仓库设置为`https://registry.docker-cn.com`或者aliyun提供的镜像仓库`https://uxk0ognt.mirror.aliyuncs.com`。

修改Docker镜像仓库为aliyun,加速镜像拉取
```shell
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://uxk0ognt.mirror.aliyuncs.com"]
}
EOF
```

重新加载并重启Docker
```shell
systemctl daemon-reload && systemctl restart docker
```

拉取nginx的最新最新镜像到本地,不添加版本号默认拉取最新`latest`版本，需要特定版本指定版本即可。
```shell
[root@dev ~]# docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx                                                        8d691f585fa8: Pull complete                                                               5b07f4e08ad0: Pull complete                                                               abc291867bca: Pull complete 
Digest: sha256:922c815aa4df050d4df476e92daed4231f466acc8ee90e0e774951b0fd7195a4
Status: Downloaded newer image for nginx:latest
```

查看本地镜像列表
```shell
root@dev ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              540a289bab6c        3 weeks ago         126MB
```

镜像是分层的，可以查看镜像的分层信息
```shell
[root@dev ~]# docker history nginx
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
540a289bab6c        3 weeks ago         /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B                  
<missing>           3 weeks ago         /bin/sh -c #(nop)  STOPSIGNAL SIGTERM           0B                  
<missing>           3 weeks ago         /bin/sh -c #(nop)  EXPOSE 80                    0B                  
<missing>           3 weeks ago         /bin/sh -c ln -sf /dev/stdout /var/log/nginx…   22B                 
<missing>           3 weeks ago         /bin/sh -c set -x     && addgroup --system -…   57MB                
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV PKG_RELEASE=1~buster     0B                  
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV NJS_VERSION=0.3.6        0B                  
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV NGINX_VERSION=1.17.5     0B                  
<missing>           4 weeks ago         /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B                  
<missing>           4 weeks ago         /bin/sh -c #(nop)  CMD ["bash"]                 0B                  
<missing>           4 weeks ago         /bin/sh -c #(nop) ADD file:74b2987cacab5a6b0…   69.2MB              
```

查看镜像详情
```shell
[root@dev ~]# docker inspect nginx
[
    {
        "Id": "sha256:540a289bab6cb1bf880086a9b803cf0c4cefe38cbb5cdefa199b69614525199f",
        "RepoTags": [
            "nginx:latest"
        ],
        "RepoDigests": [
            "nginx@sha256:922c815aa4df050d4df476e92daed4231f466acc8ee90e0e774951b0fd7195a4"
        ],
        "Parent": "",
        "Comment": "",
        "Created": "2019-10-23T00:26:03.830480202Z",
        "Container": "77b8bfc5e16274066a5d4c14915ea5e7387c062f8540cd970c54e9b6e38b1011",
        "ContainerConfig": {
            "Hostname": "77b8bfc5e162",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.17.5",
                "NJS_VERSION=0.3.6",
                "PKG_RELEASE=1~buster"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"nginx\" \"-g\" \"daemon off;\"]"
            ],
            "ArgsEscaped": true,
            "Image": "sha256:2e2fa75c52fdfe182fb66455d6db04849c683ef01d14a526211ba37831c66791",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
            },
            "StopSignal": "SIGTERM"
        },
        "DockerVersion": "18.06.1-ce",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.17.5",
                "NJS_VERSION=0.3.6",
                "PKG_RELEASE=1~buster"
            ],
            "Cmd": [
                "nginx",
                "-g",
                "daemon off;"
            ],
            "ArgsEscaped": true,
            "Image": "sha256:2e2fa75c52fdfe182fb66455d6db04849c683ef01d14a526211ba37831c66791",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
            },
            "StopSignal": "SIGTERM"
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 126215561,
        "VirtualSize": 126215561,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/073913f9b86f825ee0431de0bfdd01703b4410a87470009ba9a2ac96353c790b/diff:/var/lib/docker/overlay2/cea7c862cabda73f6d9395eb93b4c48a4b3798629d6663bd00e2d805fc6b02a4/diff",
                "MergedDir": "/var/lib/docker/overlay2/9ff16a2be5a6c4c32e4c27a87520be3f117486be9d89718f68799a58c25333fc/merged",
                "UpperDir": "/var/lib/docker/overlay2/9ff16a2be5a6c4c32e4c27a87520be3f117486be9d89718f68799a58c25333fc/diff",
                "WorkDir": "/var/lib/docker/overlay2/9ff16a2be5a6c4c32e4c27a87520be3f117486be9d89718f68799a58c25333fc/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:b67d19e65ef653823ed62a5835399c610a40e8205c16f839c5cc567954fcf594",
                "sha256:6eaad811af0237b78ba8b44a282d1564259d90007d628a032c5df7e3e2bbb613",
                "sha256:a89b8f05da3a2cbe459ef3fecfec8076fd0a7568db81f9164147b6f642e2dadf"
            ]
        },
        "Metadata": {
            "LastTagTime": "0001-01-01T00:00:00Z"
        }
    }
]
```

删除镜像
```shell
[root@dev ~]# docker image rm nginx
Untagged: nginx:latest
Untagged: nginx@sha256:922c815aa4df050d4df476e92daed4231f466acc8ee90e0e774951b0fd7195a4
Deleted: sha256:540a289bab6cb1bf880086a9b803cf0c4cefe38cbb5cdefa199b69614525199f
Deleted: sha256:ab18af7cee69bfb22c1771e54d5e0e68b1a1bf57bb46516142da0380b1771f4a
Deleted: sha256:02f7daf1e14541cd61a3dda1a61cc0f78fee8de2984d488b8ba5bbd3cbad9b57
Deleted: sha256:b67d19e65ef653823ed62a5835399c610a40e8205c16f839c5cc567954fcf594
```

删除所有镜像
```shell
[root@dev ~]# docker image rm $(docker image ls  -aq)
Untagged: busybox:latest
Untagged: busybox@sha256:1303dbf110c57f3edf68d9f5a16c082ec06c4cf7604831669faf2c712260b5a0
Deleted: sha256:020584afccce44678ec82676db80f68d50ea5c766b6e9d9601f7b5fc86dfb96d
Deleted: sha256:1da8e4c8d30765bea127dc2f11a17bc723b59480f4ab5292edb00eb8eb1d96b1
```

给镜像打标签，镜像列表多了一条记录，镜像ID还是一样的。
```shell
[root@dev ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              020584afccce        2 weeks ago         1.22MB
nginx               latest              540a289bab6c        3 weeks ago         126MB
[root@dev ~]# docker tag busybox:latest busybox:v1.0
[root@dev ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              020584afccce        2 weeks ago         1.22MB
busybox             v1.0                020584afccce        2 weeks ago         1.22MB
nginx               latest              540a289bab6c        3 weeks ago         126MB
```

导出镜像
```shell
[root@dev ~]# docker save busybox:v1.0 >  busybox-1.0.tar
[root@dev ~]# ls
anaconda-ks.cfg  busybox-1.0.tar
```

载入镜像
```shell
[root@dev ~]# docker rmi busybox:v1.0
Untagged: busybox:v1.0
Deleted: sha256:020584afccce44678ec82676db80f68d50ea5c766b6e9d9601f7b5fc86dfb96d
Deleted: sha256:1da8e4c8d30765bea127dc2f11a17bc723b59480f4ab5292edb00eb8eb1d96b1
[root@dev ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              540a289bab6c        3 weeks ago         126MB
[root@dev ~]# docker load < busybox-1.0.tar 
1da8e4c8d307: Loading layer  1.437MB/1.437MB
Loaded image: busybox:v1.0
[root@dev ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             v1.0                020584afccce        2 weeks ago         1.22MB
nginx               latest              540a289bab6c        3 weeks ago         126MB
```

构建镜像 `docker commit` 和 `docker build`，后边在Dockerfile详细讲解。

## 容器

启动一个容器，`-i`交互式启动，`-t`分配一个伪终端，`-d`后台运行，`--name`给容器起一个名称，启动之后返回容器长ID值。
```shell
[root@dev ~]# docker run  -itd --name busy-service busybox:v1.0
1479e27334802659474255201a6f125d38c3e568e57b74aa828a2c94b8602d4e
```

查看当前运行的容器,所有的容器加上`-a`
```shell
[root@dev ~]# docker ps 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
1479e2733480        busybox:v1.0        "sh"                12 hours ago        Up 12 hours                             busy-service
```

进入容器
```shell
[root@dev ~]# docker attach busy-service
/ # ls
bin   dev   etc   home  proc  root  sys   tmp   usr   var
/ # 
```

退出容器可以输入`exit`或者使用快捷键`Ctrl+p+q`。这样子退出终端之后容器也就退出了，所以铜通常不会这样子进入容器，一般是使用`docke exec`进入和退出。

```shell
[root@dev ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@dev ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
1479e2733480        busybox:v1.0        "sh"                12 hours ago        Exited (0) 5 minutes ago                       busy-service
[root@dev ~]# docker start busy-service
busy-service
[root@dev ~]# docker exec -it busy-service sh
/ # ls
bin   dev   etc   home  proc  root  sys   tmp   usr   var
/ # exit
[root@dev ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
1479e2733480        busybox:v1.0        "sh"                12 hours ago        Up 22 seconds                           busy-service
```

映射容器端口到使用主机端口` -p hostPort:containerPort`
```shell
[root@dev ~]# docker run -itd -p 8080:80  --name nginx nginx:latest        
00e1743963f38a0eae3c05fe94d8ae9974ea30cf300aae86b257951ab4ff4336
[root@dev ~]# curl localhost:8080
<!DOCTYPE html>                                                                           <html>
<head>                                                                                    <title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and               working. Further configuration is required.</p>
                                                                                          <p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

查看容器日志,`-f`动态刷新
```shell
[root@dev ~]# docker logs -f nginx
172.17.0.1 - - [15/Nov/2019:00:28:21 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"
```

多请求几次nginx,发现日志不断刷新
```shell
[root@dev ~]# docker logs -f nginx
172.17.0.1 - - [15/Nov/2019:00:28:21 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"
172.17.0.1 - - [15/Nov/2019:00:32:49 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"
172.17.0.1 - - [15/Nov/2019:00:32:54 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"
172.17.0.1 - - [15/Nov/2019:00:32:54 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"
172.17.0.1 - - [15/Nov/2019:00:32:55 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"
172.17.0.1 - - [15/Nov/2019:00:32:55 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"
```

在使用容器跑业务的时候，运行了一段时间就会发现容器宿主机存储不够了，这种时候有可能就是日志文件太大了，docker容器运行的日志文件在宿主机的目录`/var/lib/docker/containers`,以JSON格式来存储。
```shell
[root@dev 00e1743963f38a0eae3c05fe94d8ae9974ea30cf300aae86b257951ab4ff4336]# pwd
/var/lib/docker/containers/00e1743963f38a0eae3c05fe94d8ae9974ea30cf300aae86b257951ab4ff4336
[root@dev 00e1743963f38a0eae3c05fe94d8ae9974ea30cf300aae86b257951ab4ff4336]# ll
total 28
-rw-r-----. 1 root root 1026 Nov 15 08:32 00e1743963f38a0eae3c05fe94d8ae9974ea30cf300aae86b257951ab4ff4336-json.log
drwx------. 2 root root    6 Nov 15 08:28 checkpoints
-rw-------. 1 root root 2902 Nov 15 08:28 config.v2.json
-rw-r--r--. 1 root root 1463 Nov 15 08:28 hostconfig.json                                 -rw-r--r--. 1 root root   13 Nov 15 08:28 hostname
-rw-r--r--. 1 root root  174 Nov 15 08:28 hosts
drwx------. 3 root root   17 Nov 15 08:28 mounts
-rw-r--r--. 1 root root   81 Nov 15 08:28 resolv.conf
-rw-r--r--. 1 root root   71 Nov 15 08:28 resolv.conf.hash
[root@dev 00e1743963f38a0eae3c05fe94d8ae9974ea30cf300aae86b257951ab4ff4336]# more 00e1743963f38a0eae3c05fe94d8ae9974ea30cf300aae86b257951ab4ff4336-json.log 
{"log":"172.17.0.1 - - [15/Nov/2019:00:28:21 +0000] \"GET / HTTP/1.1\" 200 612 \"-\" \"cur
l/7.29.0\" \"-\"\r\n","stream":"stdout","time":"2019-11-15T00:28:21.759024147Z"}
{"log":"172.17.0.1 - - [15/Nov/2019:00:32:49 +0000] \"GET / HTTP/1.1\" 200 612 \"-\" \"cur
l/7.29.0\" \"-\"\r\n","stream":"stdout","time":"2019-11-15T00:32:49.231094319Z"}
{"log":"172.17.0.1 - - [15/Nov/2019:00:32:54 +0000] \"GET / HTTP/1.1\" 200 612 \"-\" \"cur
l/7.29.0\" \"-\"\r\n","stream":"stdout","time":"2019-11-15T00:32:54.030918643Z"}
{"log":"172.17.0.1 - - [15/Nov/2019:00:32:54 +0000] \"GET / HTTP/1.1\" 200 612 \"-\" \"cur
l/7.29.0\" \"-\"\r\n","stream":"stdout","time":"2019-11-15T00:32:54.677989285Z"}          {"log":"172.17.0.1 - - [15/Nov/2019:00:32:55 +0000] \"GET / HTTP/1.1\" 200 612 \"-\" \"cur
l/7.29.0\" \"-\"\r\n","stream":"stdout","time":"2019-11-15T00:32:55.263751426Z"}
{"log":"172.17.0.1 - - [15/Nov/2019:00:32:55 +0000] \"GET / HTTP/1.1\" 200 612 \"-\" \"cur
l/7.29.0\" \"-\"\r\n","stream":"stdout","time":"2019-11-15T00:32:55.758060729Z"}
```

## 存储卷

Docker常用的存储方式有两种`volumes`和`bind mount`,`volumes`存储在`/var/lib/docker/volumes`，`bind mount`可以是宿主机的任意位置。

可以通过`docker volume ls`查看volume列表
```shell
[root@dev volumes]# docker volume ls
DRIVER              VOLUME NAME
local               09e847e4927f1c7d71748aaa34b4d018680ad2446c1e2b6c81f88a9cd79d6d66
local               jenkins
```

创建一个卷
```shell
[root@dev volumes]# docker volume create nginx-volume
nginx-volume
```

查看卷详情
```shell
[root@dev volumes]# docker volume inspect nginx-volume
[
    {
        "CreatedAt": "2019-11-15T16:21:40+08:00",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/nginx-volume/_data",
        "Name": "nginx-volume",
        "Options": {},
        "Scope": "local"
    }
]
```

创建容器的时候使用这个卷，如果没有指定卷的话会默认创建一个卷。
```shell
[root@dev _data]# docker run -itd -p 8080:80  --name nginx --mount src=nginx-volume,dst=/usr/share/nginx/html nginx
7499b405bca77aea5723a194f7956455cce8bb3aedea1b4158e233ad0db3bfa9
```

容器起来之后发现文件已经写出来到宿主机本地了
```shell
[root@dev _data]# pwd
/var/lib/docker/volumes/nginx-volume/_data
[root@dev _data]# ls
50x.html  index.html
[root@dev _data]# 
```

修改宿主机文件，然后通过访问容器内的nginx来证明文件是同步的
```shell
[root@dev _data]# echo 111 > test.html
[root@dev _data]# ls
50x.html  index.html  test.html
[root@dev _data]# curl localhost:8080/test.html
111
[root@dev _data]# 
```

进入容器确认文件是否存在
```shell
[root@dev _data]# docker exec nginx more  /usr/share/nginx/html/test.html
::::::::::::::
/usr/share/nginx/html/test.html
::::::::::::::
111
```

删除一个卷使用`docker volume rm `即可

`bind mount` 需要指定类型,并且绑定宿主机的目录需要提前创建好
```shell
[root@dev opt]# mkdir /opt/html
[root@dev opt]# docker run -itd -p 8081:80  --name nginx-8081 --mount type=bind,src=/opt/html,dst=/usr/share/nginx/html nginx 
a2709612f0adee5a48501e12f1e1d65ffd62024223026b19dde4f2b560c2ace7
```

目前宿主机的目录是空的，容器内的目录也是空的，在宿主机内写文件，容器内目标目录也会有同样的文件,反之亦然。
```shell
[root@dev opt]# docker exec  nginx-8081 ls /usr/share/nginx/html 
[root@dev opt]# ls
html
[root@dev opt]# cd html/
[root@dev html]# ls
[root@dev html]# echo 111 > index.html
[root@dev html]# docker exec  nginx-8081 ls /usr/share/nginx/html 
index.html
[root@dev html]# curl localhost:8081
111
[root@dev html]# docker exec nginx-8081 touch /usr/share/nginx/html/test.html
[root@dev html]# docker exec  nginx-8081 ls /usr/share/nginx/html
index.html
test.html
[root@dev html]# ls
index.html  test.html
```

这个`bind mount`还可以简写,比如可以写为`docker run -itd -p 8081:80  --name nginx-8081 -v /opt/html:/usr/share/nginx/html nginx`

## 网络

Docker支持5种网络模式

1. `bridge`  默认网络，Docker安装的时候默认创建一个docker0网桥，默认创建的容器也是添加到这个网桥中。

2. `host`   容器不会获得一个独立的network namespace，而是与宿主机共用一个。

3. `none`  获取独立的network namespace，但不为容器进行任何网络配置。

4. `container`  与指定的容器使用同一个network namespace，网卡配置也都是相同的。

5. `custom`     自定义网桥，默认与bridge网络一样。

看一下docker的网络，并查看一下宿主机的docker0网桥
```shell
[root@dev ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
1974df6aac06        bridge              bridge              local
8de4b3cf8127        host                host                local
5473f4de6898        none                null                local
[root@dev ~]# ip addr show docker0
4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:52:e3:d1:61 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:52ff:fee3:d161/64 scope link 
       valid_lft forever preferred_lft forever
[root@dev ~]# 
```

docker0上有一对veth pair进行连接

```shell
[root@dev ~]# docker exec -it busybox sh                                                  / # ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever                          
17: eth0@if18: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue 
    link/ether 02:42:ac:11:00:04 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.4/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
/ # exit
[root@dev ~]# ip addr show if18
18: veth882df3c@if17: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether d2:fb:14:6b:26:60 brd ff:ff:ff:ff:ff:ff link-netnsid 2
    inet6 fe80::d0fb:14ff:fe6b:2660/64 scope link 
       valid_lft forever preferred_lft forever
[root@dev ~]# brctl show                                                                  bridge name     bridge id               STP enabled     interfaces
docker0         8000.024252e3d161       no              veth882df3c
                                                        vethc3a6616
```
可以看到容器内的eth0和宿主机docker0网桥上的veth882df3c是一对veth pair。

## Dockerfile

Docker镜像通常是使用Dockerfile来构建，可以清晰控制每一层镜像。下面是常用的指令

1. `FROM` 基于哪个基础镜像去构建，例如`FROM centos:7`。
2. `MAINTAINER` 镜像维护者名字或者邮箱，例如`MAINTAINER rzz1995@live.com`
3. `RUN` __构建镜像时运行__的shell命令，例如`RUN yum install vim`或者`RUN ["yum","install","vim"]`。
4. `CMD` __运行容器时执行__的shell命令，例如`CMD /usr/sbin/sshd -D`或者`CMD ["/usr/sbin/sshd","-D"]`。
5. `EXPOSE` 暴露容器内部服务端口，例如`EXPOSE 443`。
6. `ENV` 设置容器内环境变量，例如`ENV MYSQL_ROOT_PASSWORD root`
7. `ADD` 拷贝文件或者目录到镜像，如果是URL或者压缩包会自动下载或者解压，例如`ADD https://xxx.com/html.tar.gz /var/www/html`或者`ADD html.tar.gz /var/www/html`。
8. `COPY` 拷贝文件或者目录到镜像，例如`COPY ./start.sh /start.sh`。
9. `ENTRYPOINT` __运行容器时执行__执行的shell命令，例如`ENTRYPOINT ["/bin/bash","-c","/start.sh"]`或者`ENTRYPOINT /bin/bash -c './start.sh'`。
10. `VOLUME` 指定容器挂载点到宿主机自动生成的目录或其他容器。例如`VOLUME ["/var/lib/mysql"]`。
11. `USER` 为`RUN`、`CMD`、`ENTRYPOINT`执行命令指定运行用户，例如`USER root`。
12. `WORKDIR` 为`RUN`,`CMD`,`ENTRYPOINT`,`COPY`,`ADD`设置工作目录，例如`WORKDIR /data`。
13. `HEALTHCHECK` 健康检查，例如`HEALTHCHECK --interval=5m --timeout=3s --retries=3 CMD curl -f http:localhost/ || exit 1`。
14. `ARG` 在构建镜像时指定一些参数，例如Dockerfile中有一行`ARG uer`，那build镜像的时候可以这样子传参`docker build --build-arg user=ruanzz Dockerfile .`。

```shell
[root@dev demo]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              020584afccce        6 weeks ago         1.22MB
busybox             v1.0                020584afccce        6 weeks ago         1.22MB
nginx               latest              540a289bab6c        7 weeks ago         126MB
[root@dev demo]# ll
total 4
-rw-r--r--. 1 root root 46 Dec 15 10:52 Dockerfile
drwxr-xr-x. 2 root root 24 Dec 15 10:54 html
[root@dev demo]# more html/index.html 
Hello Nginx
[root@dev demo]# more Dockerfile 
FROM nginx
COPY html/* /usr/share/nginx/html

[root@dev demo]# docker build -t test-nginx:1.0 .
Sending build context to Docker daemon  3.584kB
Step 1/2 : FROM nginx
 ---> 540a289bab6c
Step 2/2 : COPY html/* /usr/share/nginx/html
 ---> 7debe9edccb6
Successfully built 7debe9edccb6
Successfully tagged test-nginx:1.0
[root@dev demo]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
test-nginx          1.0                 7debe9edccb6        6 seconds ago       126MB
busybox             latest              020584afccce        6 weeks ago         1.22MB
busybox             v1.0                020584afccce        6 weeks ago         1.22MB
nginx               latest              540a289bab6c        7 weeks ago         126MB
[root@dev demo]# docker run -it -d --name test-nginx -p 80:80  test-nginx:1.0
05d5cbe48f90983b8470982ee0e81a8125d52129363779ad927ea46fc94afc60
[root@dev demo]# curl localhost:80
Hello Nginx
```