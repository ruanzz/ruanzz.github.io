---
title: 单点登录
tags: [sso]
toc: true
date: 2019-04-13 21:34:10
category: Web
---
单点登录应该是大部分公司都有的一个系统，这个系统的作用就是用户只需登录一次就可以在多个系统之间切换而不用重新登录。刚毕业的时候接触过点单点登录，不过久了有点忘了，这篇文章来讲述单点登录相关知识点，并且根据两种主流实现方式CAS和JWT来讲解。
<!-- more -->

## 单点登录背景

我们先从最开始的方案来说，以前的单点登录是基于Cookie来实现的，这种方式是将用户名和密码加密之后存放在Cookie中，之后访问系统时通过过滤器来校验用户权限，如果有权限就从Cookie中取出用户名和密码来进行登录，让用户感觉上只用登录一次。

基于Cookie的这种方式的缺点很明显，多个系统都有登录逻辑的代码，如果有修改那就多个系统都得修改，还有的就是多次传输用户名字和密码，即使是加密的也还是存在增加被盗风险，还有一个非常不方便的就是跨域，这个对现在的系统来说是不合适的，如果是二级域的还可以通过设置cookie范围为顶域，这个时候cookie还能共享，但是现代化的系统非常多，可能顶级域名都不是一样，这个时候cookie就没办法共享了，这种方案就不合适了。

接下来我们看一下统一认证中心方案，基于Cookie方案的登录认证过于分散，而且代码逻辑重复，现在我们将认证统一化，形成一个独立的服务，当用户需要进行登录操作的时候，统一重定向到认证服务，流程如下：
1. 用户访问A系统，A系统的过滤器判断用户是否登录，如果没有登录，则重定向到认证服务。
2. 重定向到认证服务，输入用户名和密码登录，认证服务将用户登录的信息记录到服务器的session中
3. 认证服务发给浏览器一个特殊的凭证，浏览器将凭证交给A系统，A系统拿着凭证去认证服务验证凭证的有效性，从而判断用户是否登录成功。

## CAS 

CAS是Yale大学开源的一个统一认证服务，CAS包括两个部分：CAS Server 和 CAS Client，CAS Server负责对用户的认证工作，需要独立部署，CAS Client与受保护的客户端一起部署，通过Filter形式保护资源，负责处理对客户端受保护资源的访问请求，需要对请求方重定向到CAS Server进行认证。

工作流程如下：
1、用户访问A系统，A系统是需要登录，但是用户现在没有登录
2、重定向到CAS Server，即UAP(统一登录平台)，UAP也没有登录，跳转到登录页面
3、用户填写用户名、密码，UAP进行认证后，将登录状态写入UAP的session中，浏览器写入UAP域下的cookie
4、UAP登录完成后会生成一个ST(Service Ticket),然后跳转到A系统，同时将ST作为参数传递给A系统
5、A系统拿到ST后，后台过滤器拦截到，CAS Client向UAP发送请求，验证ST是否有效
6、验证通过后，A系统将登录状态写入session并设置A系统域下的cookie

这样子用户就完成了A系统的登录，接下来我们看一下用户继续访问B系统

1、用户访问B系统，B系统没有登录，重定向到UAP
2、用户已登录过UAP，不需要重新登录认证
3、UAP生成ST，跳转到B系统，将ST作为参数传给B系统
4、B系统拿到ST之后，CAS Client向UAP验证ST是否有效
5、验证通过后，B系统将登录状态写入session并设置B系统域下cookie

通过对这个流程分析我们发现，用户登录了A系统之后再访问B系统就不会再出现重定向到登录页面，这样就完成了单点登录，即使A系统和B系统时不同的域，session不同享也都没关系。

接下来做个CAS的DEMO

先来完成cas服务端的部署,从Github上将cas的代码检出来，然后build war包，放到tomcat里边执行
```
git clone https://github.com/apereo/cas-overlay-template.git
git checkout remotes/origin/5.3  # master分支是gradle构建，5.x是maven构建
./mvnw clean package

```
等待打包执行完毕，在target目录下有cas.war,然后把这个war包放到tomcat8上就可以了，因为cas已经采用springboot改造过了，打出来的war包最好放在tomcat8或者更高的版本之上，这样子才不会有问题的，启动完之后访问 http://localhost:8080/cas/login 就可以看到登录页面了，默认的用户名和密码为casuser/Mellon,这个写在springboot的配置文件application.properties文件中。

通常在实际项目中，我们是用https来进行访问，下面来配置https访问
先用jdk工具生成秘钥库
```
keytool -genkey -alias cas-tomcat -keyalg RSA -keystore cas.keystore
```
```
➜  classes keytool -genkey -alias cas-tomcat -keyalg RSA -keystore cas.keystore
Enter keystore password:  
Re-enter new password: 
What is your first and last name?
  [Unknown]:  sso.demo.com
What is the name of your organizational unit?
  [Unknown]:  demo.com
What is the name of your organization?
  [Unknown]:  demo.com
What is the name of your City or Locality?
  [Unknown]:  gz
What is the name of your State or Province?
  [Unknown]:  gd
What is the two-letter country code for this unit?
  [Unknown]:  cn
Is CN=sso.demo.com, OU=demo.com, O=demo.com, L=gz, ST=gd, C=cn correct?
  [no]:  y

Enter key password for <cas-tomcat>
	(RETURN if same as keystore password):  
Re-enter new password:
```

导出证书
```
keytool -export -file cas.crt -alias cas-tomcat -keystore cas.keystore
```
```
➜  classes keytool -export -file cas.crt -alias cas-tomcat -keystore cas.keystore

Enter keystore password:  
Certificate stored in file <cas.crt>
```
将证书导入JDK中，不然认证了跳过来就会报错，ssl建立连接错误
```
keytool -import -keystore ${JAVA_HOME}/jre/lib/security/cacerts -file cas.crt -alias cas-tomcat
```
```
➜  classes sudo keytool -import -keystore /Library/Java/JavaVirtualMachines/jdk1.8.0_144.jdk/Contents/Home/jre/lib/security/cacerts -file cas.crt -alias cas-tomcat

Enter keystore password:  
Re-enter new password: 
Owner: CN=sso.demo.com, OU=demo.com, O=demo.com, L=gz, ST=gd, C=cn
Issuer: CN=sso.demo.com, OU=demo.com, O=demo.com, L=gz, ST=gd, C=cn
Serial number: 5890e418
Valid from: Sun May 26 11:41:41 CST 2019 until: Sat Aug 24 11:41:41 CST 2019
Certificate fingerprints:
	 MD5:  69:69:38:A9:52:03:D2:3D:DA:28:03:CD:53:59:48:23
	 SHA1: F2:9A:55:04:E8:A2:E6:4C:C2:85:8D:EA:9A:47:E6:27:C1:DC:41:B2
	 SHA256: 1B:25:A6:B9:F9:36:AD:E7:CD:28:2E:57:D8:30:8C:8A:9E:6C:07:CB:03:70:78:5E:AE:9C:AB:72:27:0A:13:B3
	 Signature algorithm name: SHA256withRSA
	 Version: 3

Extensions: 

#1: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: 93 1B 15 7E 7D 26 A3 51   35 ED 51 30 E4 EA 52 77  .....&.Q5.Q0..Rw
0010: 19 9C 93 C1                                        ....
]
]

Trust this certificate? [no]:  y
Certificate was added to keystore
```

接下来配置tomcat的server.xml
```
 <Connector port="443" protocol="org.apache.coyote.http11.Http11Protocol"
    maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
    clientAuth="false" sslProtocol="TLS" keystoreFile="/Users/***/apache-tomcat-8.5.41/webapps/cas/WEB-INF/classes/cas.keystore" keystorePass="123456"  />
```
配置域名映射`127.0.0.1  sso.demo.com`,这样子就可以通过https://sso.demo.com/cas 来进行访问了。

cas服务端就部署完成了，接下来我们将Cas Client和具体的应用系统结合起来。

具体代码见Github仓库中的cas模块，启动CasApplication类。
<div class="github-widget" data-repo="ruanzz/sso"></div>
cas模块是一个SpringBoot应用，跑起来之后访问https://localhost:8099/test 就会自动跳转到https://sso.demo.com/cas/login 认证过之后会重定向回来访问https://localhost:8099/test

到这里CAS基本认证流程就OK了。

## JWT
TODO

