# Docker
---
## 目录：
 * [基本概念](#基本概念)
 * [Docker在Linux上的安装](#Docker在Linux上的安装)
 * [在虚拟机中获取镜像](#在虚拟机中获取镜像)
 * [列出本地已有镜像](#列出本地已有镜像)
 * [创建镜像](#创建镜像)
 * [使用Dockerfile来创建镜像](#使用Dockerfile来创建镜像)
 * [镜像的存出和载入](#镜像的存出和载入)

---

### <span id="基本概念">基本概念</span>
Docker包括三个基本概念
* 镜像(Image)
    >Docker镜像就是一个只读的模板。  
    例如:一个镜像可以包含一个完整的Ubuntu操作系统环境,里面仅安装了Apache或用户需要的其他的应用程序。
* 容器(Container)
    >Docker利用容器来运行应用。  
    容器是从镜像创建的运行实例。它可以被启动、开始、停止、删除。每个容器都是相互隔离的，保证安全的平台。  
    可以把容器看作是一个简易版的Linux环境和运行其中的应用程序。
* 仓库(Repository)
    >仓库是集中存放镜像文件的场所。有时候会把仓库和仓库注册服务器（Registy）混为一谈，并不严格区分。实际上，仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签（Tag）。

理解了这三个概念,就理解了Docker的整个生命周期  

### <span id="Docker在Linux上的安装">Docker在Linux上的安装</span>
* #### 虚拟机上的安装
    1. #### CentOS6上的安装
        对于CentOS6，可以使用EPEL库安装Docker，命令如下：
        >  
        ```bash
        $ sudo yum install http://mirros.yun-idc.com/epel/6/i386/epel-release-6-8.noarch.rpm  
        $ sudo yum install docker-io
        ```
    2. #### CentOS7上的安装
        对于CentOS7系统CentOS-Extras库中已带Docker，可以直接安装：
        >  
        ```bash
        $ sudo yum install docker
        ```
* ### 启动Docker服务，并设置开机自动启动。
    ```bash
    $ sudo service docker start  
    $ sudo chkconfig docker on
    ```

### <span id="在虚拟机中获取镜像">在虚拟机中获取镜像</span>
可以使用``` docker pull```命令来从仓库中获取所需要的镜像，例如获取一个ubuntu12.04的操作系统镜像。下载过程中，会输出镜像的每一层信息。
```bash
$ sudo docker pull ubuntu:12.04
Pulling repository ubuntu
ab8e2728644c: Pulling dependent layers
511136ea3c5a: Download complete
5f0ffaa9455e: Download complete
a300658979be: Download complete
904483ae0c30: Download complete
ffdaafd1ca50: Download complete
d047ae21eeaf: Download complete
```

完成后,即可随时使用该镜像了，例如创建一个容器,让其中运行bash应用。  
```bash
$ sudo docker run -t -i ubuntu:12.04 /bin/bash
root@fe7fc4bd8fc9:/#
```

### <span id="列出本地已有的镜像">列出本地已有的镜像</span>
使用``` docker images```显示本地已有的镜像。  
```bash
[root@Sucre ~] docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
centos              7.2.1511            1a8a3dacb0e6        5 months ago        194.6 MB
ubuntu              12.04               62b726df5062        12 months ago       103.6 MB
```
在列出的信息中，可以看到几个字段信息
* 来自于哪个仓库，比如centos
* 镜像的标记，比如12.04
* 它的ID号
* 创建时间
* 镜像大小

### <span id="创建镜像">创建镜像</span>
创建镜像的方法有很多，可以用docker hub上获取已有的镜像，也可以利用本地文件系统创建一个。  
1. 先使用下载的镜像启动容器。
    ```bash
    $ sudo docker run -t -i centos:7.2.1511 /bin/bash
    [root@35c0f26924bb /]# 
    ```
2. 改变镜像里的内容。
    ```bash
    $ touch test.txt
    ```
3. 保存改变后的镜像。
    ```bash 
    #另开终端窗口进行提交操作,先退出再保存可能导致刚才的改变无法保存.
    $ sudo docker commit -m 'create a test file' -a 'Docker Newbee'  35c0f26924bb centos:v2
    ```
4. 查看镜像。
    ```bash
    [root@Sucre ~] docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED                 VIRTUAL SIZE
    centos              v2                  e65d405d40cc        9 minutes ago           194.6 MB
    centos              7.2.1511            1a8a3dacb0e6        5 months ago            194.6 MB
    ubuntu              12.04               62b726df5062        12 months ago           103.6 MB
    ```
5. 启动新创建的镜像。
    ```bash
    [root@Sucre ~] docker run -t -i centos:v2 /bin/bash
    [root@6dc2a5d0d6d8 /]#
    ```

### <span id="使用Dockerfile来创建镜像">使用Dockfile来创建镜像</span>
使用``` docker commit```扩展一个镜像比较简单，但是不利于在团队中分享。所以使用``` docker build```来创建一个新的镜像。  
>首先启动docker容器的时候使用host模式解决不能联网的问题  
>```bash
>docker run -t -i --net=host centos:v2 /bin/bash
>```
1. 首先需要创建一个Dockerfile，包含一些如何创建镜像的指令。
    ```bash
    $ mkdir centos
    $ cd centos
    $ touch Dockerfile
    ```
2. 然后往Dockerfile中添加指令信息.
    >Dockerfile的基本语法是:
    >* 使用#来表注释
    >* FROM是告诉Docker使用哪个镜像来作为基础
    >* MAINTAINER是告诉维护者的信息
    >* RUN 开头的指令是会在创建中运行，比如安装一个安装包，这里使用yum。
    >* ADD是添加本地文件到镜像里面去
    >* EXPOSE命令来向外部开放端口
    >* CMD命令用来描述容器启动之后运行的程序。
    ```bash
    #This is a comment
    FROM centos:latests
    MAINTAINER Sucre <Sucreyy@163.com>
    RUN yum -y install lrzsz
    RUN yum -y install initscripts
    ADD myApp /var/www
    EXPOSE 80
    ```
3. 然后使用``` docker build -t='centos:v3' .```来执行Dockerfile。
    ```bash
    $ [root@Sucre run] docker build -t='centos:v3' .
    ```
    其中-t是用来标记该镜像的tag信息。  
    ==.== 是用来表明Dockerfile所在位置。  
4. 上传镜像到远程仓库
    ```bash
    $ [root@Sucre run] docker push sucreyy/centos
    ```
    
### <span id="镜像的存出和载入">镜像的存出和载入</span>
1. 存出镜像
    >如果要到处镜像到本地文件,可以使用``` docker save```命令。
    
    ```bash
    $ [root@Sucre run] docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    sucreyy/centos      latest              a59a881cb552        5 days ago          315.7 MB
    centos              latests             a59a881cb552        5 days ago          315.7 MB
    $ [root@Sucre run] docker save -o centos:latest.tar centos:latests
    ```
2. 载入镜像
    >可以使用``` docker load```从导出的文件中再导入本地镜像库，例如：

    ```bash
    docker load --input centos:latests.tar
    ```
    或者
    ```bash
    docker load < centos:latests.tar 
    ```
