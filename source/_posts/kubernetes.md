---
title: Kubernetes
tags: [K8s]
toc: true
date: 2019-04-07 13:12:47
category: 云计算
---

Kubernetes是Google开源的容器编排引擎，是Google内部Borg系统的开源版本，是现在十分火热的技术，堪比之前的Hadoop和OpenStack。源代码开源在Github，是CNCF云原生基金会的第一个毕业项目，开启了云原生时代。本文是Kubernetes的入门。
<!-- more -->

Kubernetes简称k8s，k和s中间有8个单词所以简称k8s，Kubernetes采用Go语言开发，一般Kubernetes使用的容器技术有Docker或者rkt，Docker也是Go语言开发的，而且更加有意思的是Go语言本身也是用Go来写的，Go目前在云计算、中间件、区块链等领域疯狂的攻城略地,可以预见在不久的将来，Go将会成为主流语言。

在正式进入Kubernetes之前，首先得来了解一下Docker，Docker目前是很多公司都拥抱的技术，可以在开发、测试、运维阶段都保证环境一致，并且很容易通过编排引擎来调度容器，这个是后话。

## Docker

Docker是一种轻量级的虚拟化技术，是在操作系统级别虚拟化，各个容器之间共享操作系统资源，每个容器都是操作系统的一个进程，通过CGroup和NameSpace技术来达到资源的限制以及隔离，和虚拟机相比，隔离的不是很彻底，所以安全性一直被诟病，但是这个不影响它的欢迎程度，基本的计算、存储、网络资源隔离都是不错的，并且启动一个容器是秒级的，虚拟机是分钟级，所以容器可以在秒级别进行动态扩容和释放，这个和云最初的设想完全吻合，也是容器大受欢迎的原因。

我的环境是OS X系统，可以直接通过`brew cask install docker`来安装Docker，其他的平台可以参考官方文档来安装，Docker是C/S架构，安装完成之后命令行客户端就完成了，启动服务端之后就可以通过客户端来与后台的dockerd进程来通信了。客户端的话是通过`docker`命令进行交互，与服务端通信也是通过的客户端发送命令，然后由客户端与服务端进行通信，通过命令`docker info`可以查看Docker服务端的信息。

Docker主要是三大块的内容:容器，镜像，仓库，这三个最主要的是镜像，其他两个跟镜像有关，像仓库就是存储镜像的地方，容器是从镜像来启动的，镜像也是我们开发人员来重点关注和维护的，之前我们的打包出来给运维人员是一个jar包或者war包，现在我们打包出来的是镜像，直接push到我们内部搭建的仓库中，测试和运维直接从仓库中拉取镜像来启动容器即可，这就保证了开发，测试，运维的环境一致。

镜像的构建由开发人员来维护，构建Docker镜像需要Dockerfile文件，这个就是构建Docker镜像的描述文件，这里边有一系列的语法，这里不做赘述，下面我们来看一下Dockerfile.
```Docker
FROM daocloud.io/nginx
COPY html/* /usr/share/nginx/html
```
第一行是`FROM`开头，表示从哪个镜像开始构建，这里是以`daocloud.io/nginx`为原始镜像，第二行是`COPY`拷贝`html`这个文件夹的文件到原始镜像的html文件夹中，这样子就会新构建了一层镜像，这就要求在Dockerfile同级目录下有一个html文件夹，html文件夹下有一个文件index.html。
```shell
➜  html more index.html 
<h1>Hello Docker</h1>
```

接下来我们使用`docker build -t nginx-demo .` 来构建镜像，镜像名为`nginx-demo` ，`.`代表的是构建的上下文环境，这样子就可以直接找到html文件夹，一般这个上下文环境只要包含我们必须的文件，不然构建镜像会相当的慢。
```shell
➜  demo docker build -t nginx-demo .
Sending build context to Docker daemon  3.584kB
Step 1/2 : FROM daocloud.io/nginx
latest: Pulling from nginx
Digest: sha256:dabecc7dece2fff98fb00add2f0b525b7cd4a2cacddcc27ea4a15a7922ea47ea
Status: Downloaded newer image for daocloud.io/nginx:latest
 ---> 2bcb04bdb83f
Step 2/2 : COPY html/* /usr/share/nginx/html
 ---> 2289cd357a0e
Successfully built 2289cd357a0e
Successfully tagged nginx-demo:latest
```
这是构建的过程，非常的清晰，每一步都会生成一层镜像，第一步就是拉镜像，生成第一次层，id为2bcb04bdb83f，第二步是拷贝，生成第二层，id为2289cd357a0e，最后生成了镜像nginx-demo:latest，如果没有指定镜像标签的，默认的是latest，实际开发中一般是要指定tag的，tag就是对应的版本。好了，我们通过命令`docker image ls`来查看一下镜像信息
```shell
➜  demo docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx-demo          latest              2289cd357a0e        5 minutes ago       109MB
daocloud.io/nginx   latest              2bcb04bdb83f        11 days ago         
```
接下来就是启动容器`docker run --name nginx-80 -d -p 8080:80 nginx-demo `,指定容器名称为nginx-80，`-d`是指后台运行，`-p`指的是端口映射，将Docker Host的8080端口映射到容器的80端口，`nginx-demo`是镜像名，即这个容器是通过`nginx-demo`这个镜像来启动的。启动成功之后命令返回的是容器的长id，然后我们通过访问nginx来看看效果
```shell
➜  demo curl http://127.0.0.1:8080
<h1>Hello Docker</h1>
```

好了，Docker我们暂时就说到这了，Docker本身是一个相对底层的容器引擎，和KVM，Xen这种属于同一级别的东西，现实中我们不可能直接这样子操作，像KVM有一个OpenStack云平台来管理操作，那是正常人使用的东西，像这种用命令行一个一个启动容器的对用户不太友好，而且调度也不方便。所以我们来看看今天的主角Kubernetes。

## Kubernetes架构

先从大的方面来了解一下Kubernetes，Kubernetes集群包含一个master节点和多个node节点，master是控制集群的中心，运行有多个服务，包括面向用户的API Server,负责维护集群状态的Controller Manager，负责调度任务的Scheduler。node是提供CPU，内存和存储资源的节点，每个node节点运行这kubelet和kube-proxy，kubelet是一个agent客户端，负责维护node节点的运行状态以及和master进行通信，kube-proxy主要是实现集群网络服务。

这个可以从Kubernetes的官方网站去了解，而且官方文档的还有一个特别棒的就是可以实时交互一个k8s集群，这样子就可以一边看Tutorial文档，一边实际操作，跟着文档做一遍下来就会对Kubernetes比较清晰了，这里要赞一下官方文档，真的写的很清晰，我的渣渣英语水平都可以很顺畅看完整个Tutorial。

## 搭建Kubernetes集群

我们这里不是真实的生产环境，真实的生产环境搭建还是根据官方文档来参考搭建，我们是试验一个东西，官方文档推荐使用Minikub来搭建一个单节点的集群来做开发测试，当然，如果资源足够的话还是搭建一个完整的真实集群。

先来安装客户端kubectl，这个可以跟Kubernetes集群通信,还有Minikube也一起安装
```shell
brew install kubectl
brew cask install minikube
```
安装完成之后我们来安装一个单节点的k8s集群，使用VirtualBox来做虚拟层。
```shell
minikube start --vm-driver virtualbox
```
运行命令之后我们打开VirtualBox看到已经自动创建了一台虚拟机，并且开始安装相关的服务，因为这个是单节点，master和node都是它，所以服务还是有点多的，等待的时间会比较长，如果安装失败了，删掉再重来即可，运行命令`minikube delete`会把虚拟机删掉重新，然后再重新安装，生成新的虚拟机来安装即可。

安装完成之后minikube会自动配置kubectl，把它指向k8s的API服务，运行命令`kubectl config current-context`查看
```shell
➜  html kubectl config current-context
minikube
```

这样子我们就可以使用kubectl和k8s集群通信了，安装好了集群之后我们可以通过命令`minikube start`和`minikube stop`来启动和停止集群。还要k8s提供了UI界面，通过命令`minikube dashboard`来打开，这样子就可以通过页面来操作编排了，但是作为一个开发人员，还是通过命令来和k8s集群通信比较舒服，下面我们就通过命令行来操作编排。

## 部署服务

在部署之前，我们先将本地Docker客户端和k8s主机的Docker Host的服务端建立联系，运行命令
`eval $(minikube docker-env)`,这样子就可以将二者关联起来，关联起来主要是是想构建镜像，模拟开发中的滚动的发布，这个实际开发中一般是通过的Jenkins来触发，我们为了简单就直接手动触发了。

之前我们已经构建过镜像，不过是在本地，现在本地的docker客户端已经连到了k8s的Docker Host，这个时候是没有刚才我们构建的镜像，通过命令`docker build -t k8s-demo:0.1 .`构建，构建完成了再通过命令`docker image ls`可以看到`k8s-demo`这个镜像，tag是0.1。


一般我们开发人员会写一个deployment.yml,描述如何进行部署
```shell
apiVersion: extensions/v1beta1
kind: Deployment
metadata: 
 name: k8s-demo-deployment
spec:
 replicas: 3
 template:
  metadata:
   labels:
    app: k8s-demo
  spec:
   containers:
    - name: k8s-demo-pod
      image: k8s-demo:0.1
      ports:
       - containerPort: 80
```
这里边定义了部署的副本数是3个，每个部署都是一个pod，什么是pod？pod是k8s最小的资源调度单位，一般由多个container组成，这些container共享资源，还定义了container的相关信息，包括名称，使用的镜像以及容器端口。这样子我们通过命令来应用部署
```shell
➜  k8s kubectl create -f deployment.yml 
deployment.extensions/k8s-demo-deployment created
```
通过命令`kubectl get rs`来查看
```shell
➜  k8s kubectl get rs                  
NAME                             DESIRED   CURRENT   READY   AGE
k8s-demo-deployment-774878f86f   3         3         3       5m
```
然后查看pod相关信息，发现确实是3份
```shell
➜  k8s kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
k8s-demo-deployment-774878f86f-ct8hj   1/1     Running   0          6m
k8s-demo-deployment-774878f86f-gxrp7   1/1     Running   0          6m
k8s-demo-deployment-774878f86f-v77hl   1/1     Running   0          6m
```
Kubernetes会一直帮我们维护pod的数量为3，我们假设其中的一个pod挂了，看看会不会k8s会不会立即启动一个pod
```shell
➜  k8s kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
k8s-demo-deployment-774878f86f-ct8hj   1/1     Running   0          6m
k8s-demo-deployment-774878f86f-gxrp7   1/1     Running   0          6m
k8s-demo-deployment-774878f86f-v77hl   1/1     Running   0          6m
➜  k8s kubectl delete pod k8s-demo-deployment-774878f86f-ct8hj
pod "k8s-demo-deployment-774878f86f-ct8hj" deleted
➜  k8s kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
k8s-demo-deployment-774878f86f-gxrp7   1/1     Running   0          8m
k8s-demo-deployment-774878f86f-kfhm7   1/1     Running   0          16s
k8s-demo-deployment-774878f86f-v77hl   1/1     Running   0          8m
```
我们手动删了`k8s-demo-deployment-774878f86f-ct8hj`这个pod，然后k8s很快就起了一个新的pod`k8s-demo-deployment-774878f86f-kfhm7`来保证数量为3，这就非常完美了，这样子线上环境中某个节点挂了，那么k8s就可以帮我们快速的新建一个pod来维持数量，这样子就不会如果支撑不住，导致整个集群挂掉，那这可就惨了。

既然容器起来了，那我们访问一下nginx，现在容器是在k8s内部起来了，并没有做端口映射，所以目前我们还访问不到，k8s是通过service来做映射访问的，我们来看一下service定义文件serivce.yml
```shell
➜  k8s more service.yml 
apiVersion: v1
kind: Service
metadata:
 name: k8s-demo-service
 labels:
  app: k8s-demo
spec:
 type: NodePort
 ports:
  - port: 80
    nodePort: 30050
 selector:
  app: k8s-demo
```
重点关注selector，这里将会把label为`app: k8s-demo`的容器的80端口映射到node节点的30050端口，我们刚才的deployment.yml文件中定义的label就是`app: k8s-demo`,也可以看一下现在的运行的pod的描述
```shell
➜  k8s kubectl describe pod k8s-demo-deployment-774878f86f-gxrp7
Name:           k8s-demo-deployment-774878f86f-gxrp7
Namespace:      default
Node:           minikube/10.0.2.15
Start Time:     Sun, 07 Apr 2019 18:05:29 +0800
Labels:         app=k8s-demo
                pod-template-hash=3304349429
Annotations:    <none>
Status:         Running
IP:             172.17.0.6

.....

```
通过命令`kubectl create -f service.yml`来创建service
```shell
➜  k8s kubectl create -f service.yml
service/k8s-demo-service created
```
然后通过这个service把endpoint暴露出来
```shell
➜  k8s minikube service k8s-demo-service --url
http://192.168.99.100:30050
```
访问这个url
```shell
➜  k8s curl http://192.168.99.100:30050
<h1>Hello Docker</h1>
```
跟我们本地构建镜像启动容器访问的时候看到的是一样，只是这个端口映射我们是写成了yml配置文件，而本地镜像是启动容器的时候指定的。

好了，到目前为止，我们已经部署了一个服务实例，回过头来看看，这个和我们本地启动容器有什么区别，或者优势在哪里？咋一看好像还变复杂了，看起来是复杂了，但其实对于运维人员来说变得相当简单了，而且这样子更加的规范，也更加的灵活了，还有很重要的一点就是比较符合现在大多数的部署场景。

梳理一下流程，开发人员写代码的时候一般源代码下边都会有一个Dockerfile文件，这样子每次提交代码的时候都会触发Jenkins来构建镜像并push到我们内部搭建私有仓库中，并且配套有deployment.yml和service.yml文件，做完了CI之后，接下来就是CD的过程，触发k8s应用这两个文件来完成滚动更新部署。这样子运维人员只需要点击一下发布按钮来触发，开发人员通过提交代码来触发，自动完成这个CI/CD过程，整个过程所需要关心的就是Dockerfile，deployment.yml,service.yml这三个文件，这么说来是不是觉得非常轻松了，而且运维还不用担心应用什么时候挂了，有了k8s，假期期间再也没有接到夺命连环call了。

Kubernetes确实非常好用，应用场景也会越来越多，目前我们内部的平台就只是用来做CI/CD，有些公司也用来搭建深度学习平台等等，以后肯定也会越来越多，但是k8s帮我们做了很多内部的事情，这个还是比较有分量，用好它也不是一件容易的事。我们已经部署出来了一个服务，下面我们来看看滚动更新，这样子就完整的介绍了CI/CD的这个场景下使用k8s。

##  滚动更新

假设我们修改了代码，我们这里就修改index.html的内容，方便做对比
```shell
➜  html more index.html 
<h1>Hello Kubernetes</h1>
```
接下来就是提交代码自动构建镜像，这里我们手动构建,tag修改为0.2
接下来我们需要修改一下deployment.yml这个文件来适配滚动跟新
```shell
apiVersion: extensions/v1beta1
kind: Deployment
metadata: 
 name: k8s-demo-deployment
spec:
 replicas: 3
 minReadySeconds: 10
 strategy:
  type: RollingUpdate
  rollingUpdate:
   maxUnavailable: 1
   maxSurge: 1
 template:
  metadata:
   labels:
    app: k8s-demo
  spec:
   containers:
    - name: k8s-demo-pod
      image: k8s-demo:0.2
      ports:
       - containerPort: 80
```
主要是增加滚动跟新的配置，如果一开始就有的话保持原样不用修改，`minReadySeconds: 10`指在更新了一个pod之后，需要在它进入正常状态10秒之后再更新下一个pod，`maxUnavailable: 1`指同时处于不可用状态pod不能超过1个，`maxSurge: 1`指多余的pod不能超过一个，这样子Kubernetes就会逐个替换所有的pod，运行命令`kubectl apply -f deployment.yml --record=true`来滚动更新，这里`--record=true`是让Kubernetes把这行命令记录到发布历史中备查
```shell
➜  k8s kubectl apply -f deployment.yml --record=true
deployment.extensions/k8s-demo-deployment configured
➜  k8s kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
k8s-demo-deployment-774878f86f-gxrp7   1/1     Running   0          1h
k8s-demo-deployment-774878f86f-v77hl   1/1     Running   0          1h
k8s-demo-deployment-86dbd79ff6-g45m8   1/1     Running   0          5s
k8s-demo-deployment-86dbd79ff6-mkxbx   1/1     Running   0          5s
```
可以看到当前正在pod正在替换，使用命令`kubectl rollout status deployment k8s-demo-deployment`可以实时查看更新状态，但是更新太快了，不太方便。但是我们可以看一下pod的age，
```shell
➜  k8s kubectl get pods                                     
NAME                                   READY   STATUS    RESTARTS   AGE
k8s-demo-deployment-86dbd79ff6-5l8gz   1/1     Running   0          22s
k8s-demo-deployment-86dbd79ff6-g45m8   1/1     Running   0          34s
k8s-demo-deployment-86dbd79ff6-mkxbx   1/1     Running   0          34s
```
之前都是1h左右，现在是几十秒，说明是新建的，我们来验证一下：
```shell
➜  demo curl http://192.168.99.100:30050
<h1>Hello Kubernetes</h1>
```
确实修改了，到这里我们就完整的讲述了CI/CD的整个流程。

到这里我们基本上把Kubernetes用起来，但是还有很多的东西值得我们去学习，官网的文档就是最好的学习资料，这里还要再次赞美Kubernetes的官方文档，真的真的写的非常通俗易懂。


