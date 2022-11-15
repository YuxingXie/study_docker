# docker学习

参考资料：

* https://www.youtube.com/playlist?list=PLmOn9nNkQxJFtOGw9fsoLHgtCxcki7TtK

* https://www.runoob.com/docker/docker-tutorial.html

* https://docs.docker.com/

* https://docs.docker.com/language/java/

以前看尚硅谷杨哥学的springCloud，感觉不太好，这次以官方文档和runoob为主要学习资料。

## 一、前言

### Docker的应用场景：

* Web 应用的自动化打包和发布。

* 自动化测试和持续集成、发布。

* 在服务型环境中部署和调整数据库或其他的后台应用。

* 从头编译或者扩展现有的 OpenShift 或 Cloud Foundry 平台来搭建自己的 PaaS 环境。

### Docker的安装

#### centOS安装docker

https://docs.docker.com/engine/install/centos/

```text
yum -y install gcc
yum -y install gcc-c++
#安装docker仓库
yum install -y yum-utils
#添加阿里云镜像
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum makecache fast
yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
#启动docker
systemctl start docker
ps -ef|grep docker
#启动hello-world
docker run hello-world
#curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

#### windows安装docker

https://docs.docker.com/desktop/install/windows-install/

达不到win10最低版本要求不要怕，docker运行时会提示修复。

#### 什么是容器？
简而言之，容器是您机器上的沙盒进程，与主机上的所有其他进程隔离。这种隔离利用了内核命名空间和 cgroups，这些特性在 Linux 中已经存在了很长时间。
Docker 致力于使这些功能变得平易近人且易于使用。总而言之，一个容器：

* 是镜像的可运行实例。您可以使用 DockerAPI 或 CLI 创建、启动、停止、移动或删除容器。
* 可以在本地机器、虚拟机上运行或部署到云端。
* 是可移植的（可以在任何操作系统上运行）。
* 与其他容器隔离并运行自己的软件、二进制文件和配置。

#### 什么是容器镜像？

运行容器时，它使用隔离的文件系统。此自定义文件系统由容器映像提供。由于镜像包含容器的文件系统，它必须包含运行应用程序所需的一切——所有依赖项、配置、脚本、二进制文件等。
镜像还包含容器的其他配置，例如环境变量、要运行的默认命令、和其他元数据。

后面部分深入研究图像，涵盖分层、最佳实践等主题。

## 二、Docker的使用

### 2.1 Docker Hello World

```text
docker run hello-world
```
第一次提示找不到这个镜像，docker会自动下载后再运行。

docker run时干了什么？
1. 寻找镜像:本地>docker Hub；
2. 本地无，找docker Hub，有则下载至本地，无则终止；
3. 以镜像为模板产生容器实例运行。

### 2.2 Docker常用命令

要记的东西较多，按照我的思维，理解大于记忆，所以准备一份参考字典： https://www.runoob.com/docker/docker-command-manual.html

#### 启动与帮助类
* 启动：systemctl start docker
* 停止：systemctl stop docker
* 重启：systemctl restart docker
* 查看状态：systemctl status docker
* 开机启动：systemctl enable docker
* 查看概要信息：docker info
* 查看命令帮助：docker [command] --help
#### 镜像命令
* images
了解一下镜像的表示法：repository:tag，仓库和版本号代表一个镜像

* search
* pull
* rmi
* system df
什么是数据卷

命令续传：
1. 使用docker images -qa查询到所有image id;
2. 将1中的结果传递给rmi（删除镜像）命令；
3. 合并后的命令将删除所有查询到的镜像。
```text
docker rmi -f $(docker images -qa)
```

虚悬镜像(dangling image)：无名称和版本号<none>的镜像，建议删除。

#### 容器命令

##### 1.创建容器

以Docker使用Ubuntu镜像为例。

docker pull ubuntu拉取Ubuntu镜像；

docker使用的linux内核与docker安装所在的操作系统内核不是一回事哦。

centos内核过大，使用Ubuntu可减小占用空间。


##### 2.启动容器

启动前先了解下run命令：
```text
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```
--name="nginx-lb": 为容器指定一个名称；
-d: 后台运行容器，并返回容器ID；

-i: 以交互模式运行容器，通常与 -t 同时使用；

-P: 随机端口映射，容器内部端口随机映射到主机的端口

-p: 指定端口映射，格式为：主机(宿主)端口:容器端口

-t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用

所以带上-it参数执行，可以有一个伪终端交互输入。-p进行端口映射。

```text
docker run -it ubuntu /bin/bash
```
这样就进入了docker所带的ubuntu伪终端。

##### 3.退出容器
在伪终端中用exit退出，容器停止，使用ctrl+p+q退出，容器不停止。

##### 4.重启容器

镜像一旦run就创建了一个容器，并分配了一个id,可以用如下命令查看:
```text
docker ps -a
```
可以看到CONTAINER ID一列。

重启容器可以用restart容器id或名称来启动：
```text
docker restart <id或name>
``` 
##### 5.停止容器
```text
docker kill  <id或name>
```
##### 6.删除容器
```text
docker rm  <id或name>
```
-f强制删除

理解这个命令: docker rm -f $(docker ps -a),危险，生产环境勿用。

##### 7.重新进入交互式容器

守护式容器：run -d在后台运行即为守护式容器。而交互式容器当ctrl+p+q退出后，容器在后台运行也称为守护式容器。 

` docker后台运行的守护式容器容器，必须有一个前台进程。如果不是运行一个一直挂起的命令，容器启动后会立即自杀，因为docker觉得自己无事可干。` 
解决方法是带上-it或-d参数让它有个前台守护。

重新进入转入守护式的容器，使用exec或attach命令：
```text
docker exec -it 容器id
docker attach -it 容器id
```
区别：exec在使用exit退出后，容器不会停止，而attach会。建议使用exec。

##### 8. 拷贝

概念不要模糊，容器运行在docker，容器的镜像也在docker，docker运行在主机。

###### 8.1 查看容器内文件

这是一个命令组合技巧：
```text
docker exec 容器ID ls -al
```
可以查看容器内的根目录结构

###### 8.2 拷贝容器文件到主机
```text
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH
```

OPTIONS说明：

-L :保持源目标中的链接

实例

将主机/www/runoob目录拷贝到容器96f7f14e99ab的/www目录下。
```text
docker cp /www/runoob 96f7f14e99ab:/www/

```

将主机/www/runoob目录拷贝到容器96f7f14e99ab中，目录重命名为www。
```text
docker cp /www/runoob 96f7f14e99ab:/www

```
将容器96f7f14e99ab的/www目录拷贝到主机的/tmp目录中。

docker cp  96f7f14e99ab:/www /tmp/

