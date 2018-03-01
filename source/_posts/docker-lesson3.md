---
title: docker学习-第三课：镜像
date: 2018-02-02 21:36:08
categories: docker
tags: docker镜像
---

Docker	运行容器前需要本地存在对应的镜像,如果镜像不存在本地,Docker	会从镜像仓库下载(默认是
Docker	Hub	公共注册服务器中的仓库)。

本章将介绍更多关于镜像的内容,包括:
- 从仓库获取镜像;
- 管理本地主机上的镜像;
- 介绍镜像实现的基本原理。

## 获取镜像

在官方[Docker Hub](https://hub.docker.com/explore/)有大量高质量可用镜像。下面我们来看怎样获取这些镜像。

从Docker	镜像仓库获取镜像的命令是docker	pull。其命令格式为: 

    docker pull	[选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]

具体的先选可以通过`docker pull	--help`命令查看。镜像名称的格式：

- Docker 镜像仓库地址:地址的格式一般是`<域名/IP>[:端口号]`。默认地址是Docker Hub。
- 仓库名:如之前所说,这里的仓库名是两段式名称,即`<用户名>/<软件名>`。对于Docker Hub,如果不给出用户名,则默认为library,也就是官方镜像。

比如:

    $	docker	pull	ubuntu:16.04
    16.04:	Pulling	from	library/ubuntu
    bf5d46315322:	Pull	complete
    9f13e0ac480c:	Pull	complete
    e8988b5b3097:	Pull	complete
    40af181810e7:	Pull	complete
    e6f7c7e5c03e:	Pull	complete
    Digest:	sha256:147913621d9cdea08853f6ba9116c2e27a3ceffecf3b492983ae97c3d643fbbe
    Status:	Downloaded	newer	image	for	ubuntu:16.04
    
上面的命令中没有给出Docker镜像仓库地址,因此将会从Docker Hub获取镜像。而镜像名称是ubuntu:16.04	,因此将会获取官方镜像library/ubuntu	仓库中标签为16.04的镜像。    

例子： 

    mutian@mutian-ThinkPad-T440p:~$ sudo docker pull centos
    [sudo] password for mutian: 
    Using default tag: latest
    latest: Pulling from library/centos
    af4b0a2388c6: Pull complete 
    Digest: sha256:6247c7082d4c86c61b00f7f2e3edbf7f072a24aa8edc28b5b68b3de3101bc1ce
    Status: Downloaded newer image for centos:latest
    mutian@mutian-ThinkPad-T440p:~$ 
    

## 运行

有了镜像后,我们就能够以这个镜像为基础启动并运行一个容器。以上面的为例,如果我们打算启动里面的bash并且进行交互式操作的话,可以执行下面的命令。

`sudo docker run -it --rm centos bash`

    [root@5f6c0a6b41c1 ~]# cat /etc/os-release 
    NAME="CentOS Linux"
    VERSION="7 (Core)"
    ID="centos"
    ID_LIKE="rhel fedora"
    VERSION_ID="7"
    PRETTY_NAME="CentOS Linux 7 (Core)"
    ANSI_COLOR="0;31"
    CPE_NAME="cpe:/o:centos:centos:7"
    HOME_URL="https://www.centos.org/"
    BUG_REPORT_URL="https://bugs.centos.org/"
    
    CENTOS_MANTISBT_PROJECT="CentOS-7"
    CENTOS_MANTISBT_PROJECT_VERSION="7"
    REDHAT_SUPPORT_PRODUCT="centos"
    REDHAT_SUPPORT_PRODUCT_VERSION="7"
    
    [root@5f6c0a6b41c1 ~]# 

通过上面信息我们可以看到容器内系统信息。

命令说明：

`docker	run`就是运行容器的命令,具体格式我们会在	容器	一节进行详细讲解,我们这里简要的说明一下上面用到的参数。

- `-it`:这是两个参数,一个是`-i`:交互式操作,一个是`-t`终端。我们这里打算进入`bash`执行一些命令并查看返回结果,因此我们需要交互式终端。
- `--rm`:这个参数是说容器退出后随之将其删除。默认情况下,为了排障需求,退出的容器并不会立即删除,除非手动`docker	rm`。我们这里只是随便执行个命令,看看结果,不需要排障和保留结果,因此使用`--rm`可以避免浪费空间。
- `centos`:这是指用`centos`镜像为基础来启动容器。
- `bash`:放在镜像名后的是命令,这里我们希望有个交互式`Shell`,因此用的是`bash`。

进入系统后我们可以执行任何linux下的命令。

最后我们通过`exit`退出了这个容器。
退出后，之前操作的所有内容将删除，重新进入系统，已经看不到。因为加了`--rm`命令。

## 列出所有的镜像

想列出所有已经下载的镜像，可以使用命令`docker	image ls`

    mutian@mutian-ThinkPad-T440p:~$ sudo docker image ls
    [sudo] password for mutian: 
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    centos              latest              ff426288ea90        7 weeks ago         207MB
    hello-world         latest              f2a91732366c        3 months ago        1.85kB


上面列表包含了`仓库名`、`标签`、`镜像ID`、`创建时间`以及所占用空间。
镜像	ID	则是镜像的唯一标识,一个镜像可以对应多个标签。

- 镜像体积SIZE：
上面个看到的镜像体积可能比Docker Hub上的大，因为Docker Hub上的是压缩的，本地的是解压后的。所有镜像的总体积会比
每个加起来的小，因为镜像是分层存储的，有些镜像共用相同部分。

你可以通过以下命令来便捷的查看镜像、容器、数据卷所占用的空间。

    mutian@mutian-ThinkPad-T440p:~$ sudo docker system df
    TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
    Images              2                   1                   207.2MB             207.2MB (99%)
    Containers          1                   0                   0B                  0B
    Local Volumes       0                   0                   0B                  0B
    Build Cache 


- 虚悬镜像

{%asset_img a.png %}

如上图，命令查出来的就是虚悬镜像。其产生的原因是：
由于新旧镜像同名,旧镜像名称被取消,从而出现仓库名、标签均为<none>的镜像。

这种镜像已经失去了存在的意义，是可以删除的，可以用下面命令删除：

    $ sudo docker image	prune

- 中间层镜像
为了加速镜像构建、重复利用资源,Docker会利用中间层镜像。所以在使用一段时间后,可
能会看到一些依赖的中间层镜像。默认的`docker image	ls`列表中只会显示顶层镜像,如果
希望显示包括中间层镜像在内的所有镜像的话,需要加	`-a`参数。


    $ sudo docker image ls -a

这样会看到很多无标签的镜像,与之前的虚悬镜像不同,这些无标签的镜像很多都是中间层镜像,是其它镜像所依赖的镜像。这些无标签镜像不应该删除,否则会导致上层镜像因为依赖丢失而出错。实际上,这些镜像也没必要删除,因为之前说过,相同的层只会存一遍,而这些镜像是别的镜像的依赖,因此并不会因为它们被列出来而多存了一份,无论如何你也会需要它们。只要删除那些依赖它们的镜像后,这些依赖的中间层镜像也会被连带删除。 

- 列出部分镜像
上面查看的是全部镜像，有的时候我们只是需要查看需要查看的镜像。

根据仓库名称列出镜像：
    
    sudo docker image ls centos
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    centos              latest              ff426288ea90        7 weeks ago         207MB

列出特定的某个镜像,也就是说指定仓库名和标签

{%asset_img b.png%}

过滤查找镜像，使用`--filter`。具体使用请百度。

- 以特定格式显示

只显示ID列：

    sudo docker image ls -q
    ff426288ea90
    f2a91732366c

用go模板语法定制格式显示
比如,下面的命令会直接列出镜像结果,并且只包含镜像ID和仓库名:

    mutian@mutian-ThinkPad-T440p:~$ sudo docker image ls --format "{{.ID}}:{{.Repository}}"
    ff426288ea90:centos
    f2a91732366c:hello-world

或者打算以表格等距显示,并且有标题行,和默认一样,不过自己定义列:

    mutian@mutian-ThinkPad-T440p:~$ sudo docker image ls --format "table{{.ID}}\t{{.Repository}}\t{{.Tag}}"
    IMAGE ID            REPOSITORY          TAG
    ff426288ea90        centos              latest
    f2a91732366c        hello-world         latest

## 删除本地镜像

可以使用`docker	image	rm`命令来删除本地镜像。格式如下：

    $	docker image rm [选项] <镜像1> [<镜像2> ...]

- 用	ID、镜像名、摘要删除镜像


    mutian@mutian-ThinkPad-T440p:~$ sudo docker image ls
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    nginx               latest              e548f1a579cf        7 days ago          109MB
    centos              latest              ff426288ea90        7 weeks ago         207MB
    hello-world         latest        

    #删除hello-world，段id，id一部分，能区分就行。
    mutian@mutian-ThinkPad-T440p:~$ sudo docker rm f2a91

用仓库名删除：
    
    $ sudo docker image	rm	hello-world    

当然,更精确的是使用镜像摘要删除镜像。

    mutian@mutian-ThinkPad-T440p:~$ sudo docker image ls --digests

- 用	`docker	image	ls`命令来配合

比如,我们需要删除所有仓库名为redis的镜像:

    $ docker image rm $(docker image ls -q redis)

## 利用commit理解镜像构成
镜像是一层一层构成的。

现在让我们以定制一个Web	服务器为例子,来讲解镜像是如何构建的。

    docker run --name webserver -d -p 80:80 nginx

在浏览器可查看：

{%asset_img c.png%}

现在,假设我们非常不喜欢这个欢迎页面,我们希望改成欢迎	Docker的文字,我们可以使用`docker	exec`命令进入容器,修改其内容 

    > sudo docker exec -it webserver bash
    root@3729b97e8226:/# echo '<h1>Hello,Docker!</h1>' >	/usr/share/nginx/html/index.html
    root@3729b97e8226:/#	exit

然后刷新浏览器，就可以看到更改了。

我们修改了容器的文件,也就是改动了容器的存储层。我们可以通过`docker	diff`命令看到具体的改动。

下面我们可以用下面的命令将容器保存为镜像：

    mutian@mutian-ThinkPad-T440p:~$ sudo docker commit \
    > --author "zmt" \
    > --message "修改了默认页面" \
    > webserver \
    > nginx:v2
    sha256:e8023c09eed50cf1dead0b2e9da1f8e324db7f1adf7fbb042371b0503ccd71c3
    
    #查看
    mutian@mutian-ThinkPad-T440p:~$ sudo docker image ls
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    nginx               v2                  e8023c09eed5        8 seconds ago       109MB
    nginx               latest              e548f1a579cf        7 days ago          109MB
    centos              latest              ff426288ea90        7 weeks ago         207MB
    hello-world         latest  

查看镜像内历史记录：

    $ docker history nginx:v2

新的镜像定制好后,我们可以来运行这个镜像。

    mutian@mutian-ThinkPad-T440p:~$ sudo docker run --name web2 -d -p 81:80 nginx:v2


在浏览器查看：http://localhost:81/
    
至此,我们第一次完成了定制镜像,使用的是`docker	commit`命令,手动操作给旧的镜像添加了新的一层,形成新的镜像,对镜像多层存储应该有了更直观的感觉。

> 注意：通常不会使用`docker commit`来创建镜像，这样容易导致镜像臃肿。另外也无法知道每次更改，变成黑箱操作。

## 使用Dockerfile定制镜像



