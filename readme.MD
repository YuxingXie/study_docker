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

学习过程建议使用windows、macOS等我们日常使用的系统下安装docker，不建议在虚拟机下的centos等terminal终端操作的系统安装，因为：
1. 不方便拷贝文本；
2. 可能被GFW干扰访问dockerHub。

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

windows启动docker:启动Desktop Docker，小鲸鱼图标。

#### macOS安装docker

未实战，先略过。

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

## 三、 Docker常用命令

要记的东西较多，按照我的思维，理解大于记忆，所以准备一份参考字典： https://www.runoob.com/docker/docker-command-manual.html

### 3.1 启动与帮助类
* 启动：systemctl start docker
* 停止：systemctl stop docker
* 重启：systemctl restart docker
* 查看状态：systemctl status docker
* 开机启动：systemctl enable docker
* 查看概要信息：docker info
* 查看命令帮助：docker [command] --help
### 3.2 镜像命令
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

### 3.3 容器命令

####  3.3.1.创建容器

以Docker使用Ubuntu镜像为例。

docker pull ubuntu拉取Ubuntu镜像；

docker使用的linux内核与docker安装所在的操作系统内核不是一回事哦。

centos内核过大，使用Ubuntu可减小占用空间。


#### 3.3.2 启动容器

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

#### 3.3.3 退出容器
在伪终端中用exit退出，容器停止，使用ctrl+p+q退出，容器不停止。

#### 3.3.4 重启容器

镜像一旦run就创建了一个容器，并分配了一个id,可以用如下命令查看:
```text
docker ps -a
```
可以看到CONTAINER ID一列。

重启容器可以用restart容器id或名称来启动：
```text
# 运行已停止的
docker start <id或name>
# 重启正在运行的
docker restart <id或name>
``` 
#### 3.3.5 停止容器
```text
docker kill  <id或name>
# 据说stop比kill优雅
docker stop  <id或name>

```
#### 3.3.6 删除容器
```text
docker rm  <id或name>
```
-f强制删除

理解这个命令: docker rm -f $(docker ps -a),危险，生产环境勿用。

#### 3.3.7 重新进入交互式容器

守护式容器：run -d在后台运行即为守护式容器。而交互式容器当ctrl+p+q退出后，容器在后台运行也称为守护式容器。 

` docker后台运行的守护式容器容器，必须有一个前台进程。如果不是运行一个一直挂起的命令，容器启动后会立即自杀，因为docker觉得自己无事可干。` 
解决方法是带上-it或-d参数让它有个前台守护。

重新进入转入守护式的容器，使用exec或attach命令：
```text
docker exec -it 容器id
docker attach -it 容器id
```
区别：exec在使用exit退出后，容器不会停止，而attach会。建议使用exec。

#### 3.3.8 拷贝

概念不要模糊，容器运行在docker，容器的镜像也在docker，docker运行在主机。

##### 3.3.8.1 查看容器内文件

这是一个命令组合技巧：
```text
docker exec 容器ID ls -al /
```
可以查看容器内的根目录结构.

我用这个命令查看了redis和ubuntu的根目录，发现没什么不同。

##### 3.3.8.2 拷贝容器文件到主机
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

##### 3.3.8.3 docker export

1. [RUNOOB.COM的命令解释](https://www.runoob.com/docker/docker-export-command.html)



##### 3.3.8.4 docker export :
将文件系统作为一个tar归档文件导出到STDOUT。

###### 语法
```text
docker export [OPTIONS] CONTAINER

OPTIONS说明：
-o :将输入内容写到文件。
```


###### 实例
将id为a404c6c174a2的容器按日期保存为tar文件。
```text
runoob@runoob:~$ docker export -o mysql-`date +%Y%m%d`.tar a404c6c174a2
runoob@runoob:~$ ls mysql-`date +%Y%m%d`.tar
mysql-20160711.tar
```

2. 视频作者的解释

导出容器的内容流作为一个tar归档文件

###### 实例
```text
docker export 61279fc3cc6a > abcd.tar.gz
```
这里使用了一个“>”操作符。

##### 3.3.8.5 docker import

import是export的逆操作吗？不是，export是从容器实例导出为输出流，
而import不是把输入流写入容器实例，而是写入到一个镜像中。

1. [runoob的命令详解](https://www.runoob.com/docker/docker-import-command.html)

###### docker import : 
从归档文件中创建镜像。

###### 语法
docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
OPTIONS说明：

-c :应用docker 指令创建镜像；

-m :提交时的说明文字；

###### 实例
从镜像归档文件my_ubuntu_v3.tar创建镜像，命名为runoob/ubuntu:v4
```text
runoob@runoob:~$ docker import  my_ubuntu_v3.tar runoob/ubuntu:v4  
sha256:63ce4a6d6bc3fabb95dbd6c561404a309b7bdfc4e21c1d59fe9fe4299cbfea39
runoob@runoob:~$ docker images runoob/ubuntu:v4
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
runoob/ubuntu       v4                  63ce4a6d6bc3        20 seconds ago      142.1 MB
```
runoob/ubuntu:v4中的runoob表示镜像的组。

2. 视频作者解释
###### 实例

```text
cat abcd.tar | docker import - atguigu/ubuntu:3.7.0
```
我认为视频作者的命令是非标准命令，还是先使用标准命令格式吧。

现在run这个镜像，就创建了一个容器实例。

## 四、 Docker镜像

* 如何自己制作一个镜像供他人使用？

* docker commit/push

### 4.1 关于镜像

前面解除过镜像，只知道它可以从docker repository search一个镜像，pull它，并run生成容器实例。

这里需要更进一步了解它。

#### 4.1.1 UnionFS（联合文件系统）

我们前面看到的一些镜像和实例，比如hello-world,ubuntu,redis,tomcat等，它们是一个个在linux中可运行的程序，
有些需要java环境，有些需要别的环境，有些甚至是另一个linux版本的操作系统。我们可以想一想，这些程序是如何在docker中运行起来的。
可以肯定，它们都有一个linux底层操作系统核心，有各自的运行环境，以及各自的可执行程序。linux核心我们可以猜测它们都共用docker所提供的linux核心，
而其它的依赖环境呢？

docker pull的时候，我们查看控制台，发现它是一层层pull的。

```text
docker pull tomcat
...
Pulling fs layer
...
Pull complete
...
Downloading [=======================>       ] 72.44 MB/122.1 MB
...
## a、b、c、d、e、f是我胡乱写的，实际过程中是类似UUID的不同字符串
aaaaaaaaaaa:Waiting
bbbbbbbbbbb:Waiting
ccccccccccc:Waiting
ddddddddddd:Waiting
eeeeeeeeeee:Waiting
fffffffffff:Waiting
```

#### 4.1.2 bootFS

镜像分层的最内层，包含linux最基础的kernel，bootloader，用于引导启动最基础版的linux。

#### 4.1.3 rootFS

包含典型的linux系统中的/dev，/proc，/bin，/etc 等标准目录和文件。
我想暂时不用理解太多，也理解不了。

#### 4.1.4 镜像层

在有了bootfs和rootfs之后，程序可以运行在它之上。我们启动的docker容器可看做是一个安装了应用程序的linux了。
这些合起来我们称之为镜像层。

#### 4.1.4 容器层

在镜像层之上，我们继续操作，比如再安装一个另外的软件，此时我们相对于镜像层进行了增量操作，类似于git中所做的修改操作。
我们增加的这一层被称作容器层。

    如果我们不停的修改容器层，并不断导出为镜像，在新的镜像运行的容器实例再进行修改，又导出为镜像，这个操作可无限下去。
    就像git提交都基于一个父提交一样无限提交下去。在docker中每一次生成镜像，则镜像便多了一层(layer)。
    
#### 4.1.4 docker commit

现在这个命令登场是顺理成章的了，它会把我们对容器层的修改提交为新的镜像。

https://www.runoob.com/docker/docker-commit-command.html

##### docker commit :
从容器创建一个新的镜像。

##### 语法
```text
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```
##### OPTIONS说明：

* -a :提交的镜像作者；

* -c :使用Dockerfile指令来创建镜像；

* -m :提交时的说明文字；

* -p :在commit时，将容器暂停。

和git commit 是不是超级像？

##### 实例
将容器a404c6c174a2 保存为新的镜像,并添加提交人信息和说明信息。
```text
runoob@runoob:~$ docker commit -a "runoob.com" -m "my apache" a404c6c174a2  mymysql:v1 
sha256:37af1236adef1544e8886be23010b66577647a40bc02c0885a6600b33ee28057
runoob@runoob:~$ docker images mymysql:v1
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mymysql             v1                  37af1236adef        15 seconds ago      329 MB
```
提交后生成了新的镜像。提交到了哪里呢?提交到了本地。就像git commit一样，保存了修改。

#### 4.1.4 docker push

跟git一样，docker在commit以后接着可以push。git push需要一个git remote repository，那么docker push又会push到哪里呢？

我制作了一个镜像myubuntu:1.0.0，使用如下命令测试了一下：
```text
docker push myubuntu:1.0.0
```
请求被拒绝。猜测是提交到dockerHub时未登录造成的，https://hub.docker.com/signup 去注册一个账号然后返回登录
```text
docker login
#登录到阿里云镜像服务器，可能不一定是qingdao
docker login --username=xxx registry.cn-qingdao.aliyuncs.com
```
继续push仍然拒绝,在dockerHub上建一个仓库名为myubuntu仍然失败，看到页面上提示的docker push command:
```text
docker push myname/myubuntu:tagname
```
仍然不对，找不到本地的镜像的myname组。

把仓库改为共有，仍然拒绝。后来想了想是不是GFW造成的，因为虚拟机上的centos无法科学上网，便用windows下的docker进行演示，
中途仍出现request denied，后来我pull了hello-world镜像，run后将实例生成镜像,命名为myubuntu:v1,添加了一个组，即myname/myubuntu:v1,
和dockerHub保持一致，
最后用docker push myname/myubuntu:v1成功push。所以我最终还是没有排除是不是GFW造成的。断开我的vpn，发现dockerHub并没有被墙。

视频作者使用的阿里云镜像服务，不太想用。不过如何push到阿里云镜像的命令还是要知道，即在myname/myubuntu:v1加上阿里云镜像仓库地址：
变成registry.cn-qingdao.aliyuncs.com/myname/myubuntu:v1即可。push之前还要自建命名空间“myname”和仓库myubuntu。

#### 4.1.5 docker私服

先pull docker registry，然后run，便创建了一个本地版本的dockerHub。

然后参考下阿里云如何push一个镜像到仓库，在push的目标仓库前加上阿里云仓库地址即可。私服也一样，用ip:port表示。

另外要做的事情挺多的，run registry带的参数也暂时理解不了，还要配支持http，端口映射什么的，不如要用到的时候再搞吧。

## 五、 Docker容器数据卷

卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能够绕过unionFS提供一些用于持久存储或共享数据的特性。

卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此docker不会在容器删除时删除其挂载的数据卷。

### 与数据卷相关的docker run options:

* --volume , -v: 绑定一个卷。
* --volumes-from=容器id或name: 数据卷继承自。

容器绑定卷时，主机目录或容器目录不存在的话，会自动帮助创建。

### 实例：

使用镜像 nginx:latest，以后台模式启动一个容器,将容器的 80 端口映射到主机的 80 端口,主机的目录 /data 映射到容器的 /data。

```text
docker run -p 80:80 -v /data:/data -d nginx:latest
```
另一个重要option: --privilege=true，开启特权，即让容器获得容器卷的root权限。这个建议加上，免得出现权限问题。

### 检查数据卷是否加载成功：
```text
docker inspect 容器ID
```
会输出一个json，其中Mounts节点可看到加载的数据卷映射情况。

## 五、 Docker常用软件安装

无新知识点，略。

## 六、 Dockerfile

Dockerfile用于构建一个镜像，我理解类似于gradle构建java工程吧。

既然gradle有函数、变量、闭包等用于构建java程序，那么Dockerfile也有自己的编程方式生成镜像。Dockerfile是通过一些大写的保留字指令生成镜像。

建议参照下面的链接学习，直观不啰嗦。

https://www.runoob.com/docker/docker-dockerfile.html 

下面的内容我是直接拷贝排版上面资料的内容而来。

### 6.1 使用 Dockerfile 定制镜像
这里仅讲解如何运行 Dockerfile 文件来定制一个镜像，具体 Dockerfile 文件内指令详解，将在下一节中介绍，这里你只要知道构建的流程即可。

1. 下面以定制一个 nginx 镜像（构建好的镜像内会有一个 /usr/share/nginx/html/index.html 文件）

在一个空目录下，新建一个名为 Dockerfile 文件，并在文件内添加以下内容：
```text
FROM nginx
RUN echo '这是一个本地构建的nginx镜像' > /usr/share/nginx/html/index.html

```
![](https://github.com/YuxingXie/study_docker/blob/master/assets/img/001.png)

2. FROM 和 RUN 指令的作用


* FROM：定制的镜像都是基于 FROM 的镜像，这里的 nginx 就是定制需要的基础镜像。后续的操作都是基于 nginx。

* RUN：用于执行后面跟着的命令行命令。有以下俩种格式：

shell 格式：
```text
RUN <命令行命令>
# <命令行命令> 等同于，在终端操作的 shell 命令。
```
exec 格式：
```text
RUN ["可执行文件", "参数1", "参数2"]
# 例如：
# RUN ["./test.php", "dev", "offline"] 等价于 RUN ./test.php dev offline
```

注意：Dockerfile 的指令每执行一次都会在 docker 上新建一层。所以过多无意义的层，会造成镜像膨胀过大。例如：
```text
FROM centos
RUN yum -y install wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN tar -xvf redis.tar.gz
```

以上执行会创建 3 层镜像。可简化为以下格式：
```text
FROM centos
RUN yum -y install wget \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && tar -xvf redis.tar.gz
```

如上，以 && 符号连接命令，这样执行后，只会创建 1 层镜像。

### 6.2 开始构建镜像
在 Dockerfile 文件的存放目录下，执行构建动作。

以下示例，通过目录下的 Dockerfile 构建一个 nginx:v3（镜像名称:镜像标签）。

注：最后的 . 代表本次执行的上下文路径，下一节会介绍。
```text
$ docker build -t nginx:v3 .
```
![](https://github.com/YuxingXie/study_docker/blob/master/assets/img/001.png)
以上显示，说明已经构建成功。

### 6.3 上下文路径

上一节中，有提到指令最后一个 . 是上下文路径，那么什么是上下文路径呢？
```text
$ docker build -t nginx:v3 .
```
上下文路径，是指 docker 在构建镜像，有时候想要使用到本机的文件（比如复制），docker build 命令得知这个路径后，会将路径下的所有内容打包。

解析：由于 docker 的运行模式是 C/S。我们本机是 C，docker 引擎是 S。实际的构建过程是在 docker 引擎下完成的，所以这个时候无法用到我们本机的文件。这就需要把我们本机的指定目录下的文件一起打包提供给 docker 引擎使用。

如果未说明最后一个参数，那么默认上下文路径就是 Dockerfile 所在的位置。

注意：上下文路径下不要放无用的文件，因为会一起打包发送给 docker 引擎，如果文件过多会造成过程缓慢。

### 6.4 指令详解

#### 6.4.1.COPY
复制指令，从上下文目录中复制文件或者目录到容器里指定路径。

格式：
```text
COPY [--chown=<user>:<group>] <源路径1>...  <目标路径>
COPY [--chown=<user>:<group>] ["<源路径1>",...  "<目标路径>"]
[--chown=<user>:<group>]：可选参数，用户改变复制到容器内文件的拥有者和属组。
<源路径>：源文件或者源目录，这里可以是通配符表达式，其通配符规则要满足 Go 的 filepath.Match 规则。例如：
COPY hom* /mydir/
COPY hom?.txt /mydir/
<目标路径>：容器内的指定路径，该路径不用事先建好，路径不存在的话，会自动创建。
```



#### 6.4.2.ADD
ADD 指令和 COPY 的使用格类似（同样需求下，官方推荐使用 COPY）。功能也类似，不同之处如下：

* ADD 的优点：在执行 <源文件> 为 tar 压缩文件的话，压缩格式为 gzip, bzip2 以及 xz 的情况下，会自动复制并解压到 <目标路径>。
* ADD 的缺点：在不解压的前提下，无法复制 tar 压缩文件。会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。具体是否使用，可以根据是否需要自动解压来决定。
#### 6.4.3.CMD
类似于 RUN 指令，用于运行程序，但二者运行的时间点不同:

* CMD 在docker run 时运行。
* RUN 是在 docker build。
作用：为启动的容器指定默认要运行的程序，程序运行结束，容器也就结束。CMD 指令指定的程序可被 docker run 命令行参数中指定要运行的程序所覆盖。

注意：如果 Dockerfile 中如果存在多个 CMD 指令，仅最后一个生效。

格式：
```text
CMD <shell 命令> 
CMD ["<可执行文件或命令>","<param1>","<param2>",...] 
CMD ["<param1>","<param2>",...]  # 该写法是为 ENTRYPOINT 指令指定的程序提供默认参数
```

推荐使用第二种格式，执行过程比较明确。第一种格式实际上在运行的过程中也会自动转换成第二种格式运行，并且默认可执行文件是 sh。

#### 6.4.4.ENTRYPOINT
类似于 CMD 指令，但其不会被 docker run 的命令行参数指定的指令所覆盖，而且这些命令行参数会被当作参数送给 ENTRYPOINT 指令指定的程序。

但是, 如果运行 docker run 时使用了 --entrypoint 选项，将覆盖 ENTRYPOINT 指令指定的程序。

优点：在执行 docker run 的时候可以指定 ENTRYPOINT 运行所需的参数。

注意：如果 Dockerfile 中如果存在多个 ENTRYPOINT 指令，仅最后一个生效。

格式：
```text
ENTRYPOINT ["<executeable>","<param1>","<param2>",...]
```
可以搭配 CMD 命令使用：一般是变参才会使用 CMD ，这里的 CMD 等于是在给 ENTRYPOINT 传参，以下示例会提到。

示例：

假设已通过 Dockerfile 构建了 nginx:test 镜像：
```text
FROM nginx

ENTRYPOINT ["nginx", "-c"] # 定参
CMD ["/etc/nginx/nginx.conf"] # 变参 
```

1. 不传参运行
```text
$ docker run  nginx:test

```
容器内会默认运行以下命令，启动主进程。
```text
nginx -c /etc/nginx/nginx.conf

```
2. 传参运行
```text
$ docker run  nginx:test -c /etc/nginx/new.conf
```
容器内会默认运行以下命令，启动主进程(/etc/nginx/new.conf:假设容器内已有此文件)
```text
nginx -c /etc/nginx/new.conf
```
#### 6.4.5.ENV
设置环境变量，定义了环境变量，那么在后续的指令中，就可以使用这个环境变量。

格式：
```text
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...
```

以下示例设置 NODE_VERSION = 7.2.0 ， 在后续的指令中可以通过 $NODE_VERSION 引用：
```text
ENV NODE_VERSION 7.2.0

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc"
```

#### 6.4.6.ARG
构建参数，与 ENV 作用一致。不过作用域不一样。ARG 设置的环境变量仅对 Dockerfile 内有效，也就是说只有 docker build 的过程中有效，构建好的镜像内不存在此环境变量。

构建命令 docker build 中可以用 --build-arg <参数名>=<值> 来覆盖。

格式：
```text
ARG <参数名>[=<默认值>]
```
#### 6.4.7.VOLUME
定义匿名数据卷。在启动容器时忘记挂载数据卷，会自动挂载到匿名卷。

作用：

* 避免重要的数据，因容器重启而丢失，这是非常致命的。
* 避免容器不断变大。
格式：
```text
VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>
```

在启动容器 docker run 的时候，我们可以通过 -v 参数修改挂载点。

#### 6.4.8.EXPOSE
仅仅只是声明端口。

作用：

* 帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射。

* 在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口。

格式：
```text
EXPOSE <端口1> [<端口2>...]
```

#### 6.4.9.WORKDIR
指定工作目录。用 WORKDIR 指定的工作目录，会在构建镜像的每一层中都存在。以后各层的当前目录就被改为指定的目录，如该目录不存在，WORKDIR 会帮你建立目录。

docker build 构建镜像过程中的，每一个 RUN 命令都是新建的一层。只有通过 WORKDIR 创建的目录才会一直存在。

格式：
```text
WORKDIR <工作目录路径>
```
#### 6.4.10.USER
用于指定执行后续命令的用户和用户组，这边只是切换后续命令执行的用户（用户和用户组必须提前已经存在）。

格式：
```text
USER <用户名>[:<用户组>]
```
#### 6.4.11.HEALTHCHECK
用于指定某个程序或者指令来监控 docker 容器服务的运行状态。

格式：
```text
HEALTHCHECK [选项] CMD <命令>：设置检查容器健康状况的命令
HEALTHCHECK NONE：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

HEALTHCHECK [选项] CMD <命令> : 这边 CMD 后面跟随的命令使用，可以参考 CMD 的用法。
```

#### 6.4.12.ONBUILD
用于延迟构建命令的执行。简单的说，就是 Dockerfile 里用 ONBUILD 指定的命令，在本次构建镜像的过程中不会执行（假设镜像为 test-build）。当有新的 Dockerfile 使用了之前构建的镜像 FROM test-build ，这时执行新镜像的 Dockerfile 构建时候，会执行 test-build 的 Dockerfile 里的 ONBUILD 指定的命令。

格式：
```text
ONBUILD <其它指令>
```
#### 6.4.13.LABEL
LABEL 指令用来给镜像添加一些元数据（metadata），以键值对的形式，语法格式如下：
```text
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```
比如我们可以添加镜像的作者：
```text
LABEL org.opencontainers.image.authors="runoob"
```


## 七、 docker network

docker run时可加上如下option使用network模式
```text
--network=[NETWORK_MODE]

# NETWORK_MODE可以是 bridge|host|none|container,以及自定义模式。
```

### 7.1 bridge模式

默认情况下，每一个docker容器有自己独立的网卡ip，所有容器共用一个网桥，就像所有容器都处于同一个局域网网段内一样。

### 7.2 host模式

在host模式下，每个docker容器不再单独获得独立的Network Namespace。容器不再虚拟出自己的网卡而是使用宿主机的网卡和ip。

使用host模式时，主机和容器之间的端口映射关系没有意义了，设置了也不起作用，还会得到一个警告。

此时程序就像运行在宿主机上一样,程序端口号不变，占用的话端口号递增(+1)。

### 7.3 none模式

docker容器不使用任何网络配置。这个很少用，了解一下即可。

### 7.4 container模式

一个容器与另一个容器共享网络配置：
```text
--network=container:容器id或name。
```
注意两个容器之间不要存在端口冲突。

当关闭有网络配置的那个容器时，另一个容器网络模式会变成host模式。

### 7.5 自定义模式

容器的ip有可能是动态分配的，有可能随时变动。为了让程序访问固定，比如一个容器直接访问另一个容器时，需要固定ip，我们需要自定义模式。

这里需要用到一个新命令：
```text
docker network create CUSTOM_NETWORK_NAME
```
docker会创建一个新的网络模式名称。
其实之前的bridge,none,host等都是一个网络模式名称，这里我们只是新增一个模式而已。

可以使用`docker network ps`命令查看docker上存在的网络模式。

所以可以使用--network=CUSTOM_NETWORK_NAME指定容器的网络模式。

说到这，怎么固定呢？其实这个时候ip仍然可能是变化的（我猜测的），但是我们可以用另一个策略进行容器互访：

假设在一个容器用ping命令查询另一个容器的网络是否通畅,以前：
```text
# 在192.168.3.1的容器内ping 192.168.3.2
ping 192.168.3.2
```

现在让192.168.3.2 容器使用自定义模式:
```text
# 在192.168.3.1的容器内ping ip为192.168.3.2，name为exampleName的容器
ping exampleName
```
当然容器名也可以是容器id。现在可以无视ip变化ping通了。

## 第八章、 docker compose

这是最后一个重要知识点了。

docker compose是一个管理docker容器集群的管理工具。它有点类似于spring的bean管理，我们也完全可以这样去类比，而且都能用.yml文件进行配置。

docker compose能管理容器实例的加载顺序和依赖关系，和spring bean管理类似。

虽然理解起来很容易，但却是需要掌握语法、关键字等内容的一章，这种内容还是在实战中锻炼吧。

### 8.1 下载安装

本章实战为主不搞理论研究了。

可参考： 
* https://docs.docker.com/compose/compose-file/
* https://www.runoob.com/docker/docker-compose.html

Portainer和CIG也不研究了。
