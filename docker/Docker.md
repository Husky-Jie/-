# 1.Docker

1. Docker是一个开源的应用容器引擎
2. 诞生于2013年初，基于Go 语言实现，dotCloud公司出品(后改名为Dockerlnc)Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移的容器中，然后发布到任何流行的Linux机器上。
3. 容器是完全使用沙箱机制，相互隔离
4. 容器性能开销极低。
5. Docker从17.03版本之后分为CE (CommunityEdition:社区版)和EE(EnterpriseEdition:企业版
6. **docker是一种容器技术，解决软件跨环境迁移的问题**

# 2.安装docker

```markdown
1、yum 包更新到最新
yum update

2、安装需要的软件包，yum-utils 提供yum-config-manager功能，另外两个是devicemapper驱动依赖
yum install -y yum-utils device-mapper-persistent-data lvm2

3、设置yum源
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

4、安装docker，出现输入的界面都按y
yum install -y docker-ce

5、查看docker版本，验证是否安装成功
docker -v
```

# 3.docker命令

## 3.1服务相关命令

```markdown
1、启动
systemctl start docker

2、停止
systemctl stop docker

3、重启
systemctl restart docker

4、查看状态
systemctl status docker

5、开机启动
systemctl enable docker
```



## 3.2镜像相关命令

```mark
1、查看镜像：查看本地所有镜像
docker images
docker images -q  查看所有镜像id

2、搜索镜像：从网络中查找需要的镜像
docker search 镜像名称

3、拉取镜像：从docker仓库下载镜像到本地，镜像名称格式为 名称:版本号 ，如果版本号不指定则是最新版本。
	如果不知道镜像版本，可以去docker hub（hub.docker.com）搜索对应镜像查看
docker pull 镜像名称:版本号

4、删除镜像：删除本地镜像
docker rmi 镜像id    删除指定镜像
docker rmi `docker images -q`  删除所有本地镜像
```



## 3.3容器相关命令

```markd
1、查看容器
docker ps 	查看正在运行的容器
docker ps -a	查看所有的容器

2、创建并启动容器
docker run 参数 镜像名称:版本号 /bin/bash
参数说明：
-i:保持容器运行。通常与-t同时使用。加入it这两个参数后，容器创建后自动进入容器中，退出容器后，容器自动关闭。
-t:为容器重新分配一个伪输入终端，通常与-i同时使用。
-d:以守护(后台)模式运行容器。创建一个容器在后台运行，需要使用docker exec 进入容器。退出后，容器不会关闭.
-it创建的容器一般称为交互式容器，-id创建的容器一般称为守护式容器
--name:为创建的容器命名

3、退出容器
exit

4、进入容器（退出容器，容器不会关闭）
docker exec 参数 容器名称 /bin/bash

5、停止容器
docker stop 容器名称或id

6、启动容器
docker start 容器名称或id

7、删除容器
docker rm 容器名称或id

8、查看容器信息
docker inspect 容器名称或id


```



# 3.数据卷

**数据卷（volume）**是一个虚拟目录，指向宿主机文件系统中的某个目录。

- 数据卷是宿主机中的一个目录或文件

- 当容器目录和数据卷目录绑定后，对方的修改会立即同步

- 一个数据卷可以被多个容器同时挂载

- 一个容器也可以被挂载多个数据卷

  

  **数据卷作用：**

- 容器数据持久化
- 外部机器和容器间接通信
- 容器之间数据交换



## 3.1挂载数据卷

创建启动容器时，使用 -v 参数设置数据卷

```mark
docker run 参数 -v 宿主机目录1:容器内目录1 -v 宿主机目录2:容器内目录2 镜像名称:版本号 /bin/bash

注意事项
1.目录必须是绝对路径
2.如果目录不存在，会自动创建
3.可以挂载多个数据卷
```



<img src="imgs\image-20221118155448176.png" alt="image-20221118155448176" style="zoom:50%;" />



**多个容器也可以挂载一个数据卷，容器A与容器B可通过数据卷间接交换数据**



## 3.2配置数据卷容器

<img src="imgs\image-20221119104958163.png" alt="image-20221119104958163" style="zoom:50%;" />

```markdown
1.创建启动c3数据卷容器， 使用 -v参数 设置数据卷
docker run -it --name=c3 -v /volume 镜像名称:版本号 /bin/bash

注意：这里不指定宿主机目录，系统会自动分配
使用docker inspect 容器名称或id 命令可查看系统分配的宿主机目录
"Mounts": [
            {
                "Type": "volume",
                "Name": "6efb5d85a11df7b2d3550bde9e9c9326a25f05c3d8965e75415da0206fd1d25d",
                "Source": "/var/lib/docker/volumes/6efb5d85a11df7b2d3550bde9e9c9326a25f05c3d8965e75415da0206fd1d25d/_data",
                "Destination": "/volume",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
宿主机目录为/var/lib/docker/volumes/6efb5d85a11df7b2d3550bde9e9c9326a25f05c3d8965e75415da0206fd1d25d/_data/volume
```

```markdown
2.创建启动c1 c2容器，使用 --volumes-from 参数 设置数据卷
docker run -it --name=c1 --volumes-from 数据卷容器c3 /bin/bash
docker run -it --name=c2 --volumes-from 数据卷容器c3 /bin/bash

注意：这两个数据卷指向的宿主机目录与数据卷容器的一样，而这两个容器也与c3数据交互
```



# 4.docker应用部署

<img src="imgs\image-20221119110155387.png" alt="image-20221119110155387" style="zoom:50%;" />

- 容器内的网络服务和外部机器不能直接通信
- 外部机器和宿主机可以直接通信
- 宿主机和容器可以直接通信
- 当容器中的网络服务需要被外部机器访问时，可以将容器中提供服务的端口映射到宿主机的端口上。外部机器访问宿主机的该端口，从而间接访问容器的服务。
- 这种操作称为:端口映射

 [docker应用部署.md](docker应用部署.md) 



# 5.DockerFile

## 5.1Docker镜像原理

<img src="imgs\image-20221119170728656.png" alt="image-20221119170728656" style="zoom:50%;" />

- Docker镜像是由特殊的文件系统叠加而成
- 最底端是bootfs，并使用宿主机的bootfs
- 第二层是root文件系统rootfs称为baseimage
- 然后再往上可以叠加其他的镜像文件
- 统一文件系统(Union File System)技术能够将不同的层整合成一个文件系统，为这些层提供了一个统一的视角这样就隐藏了多层的存在，在用户的角度看来，只存在一个文件系统。
- 一个镜像可以放在另一个镜像的上面。位于下面的镜像称为父镜像，最底部的镜像成为基础镜像
- 当从一个镜像启动容器时，Docker会在最顶层加载一个读写文件系统作为容器



```markdown
思考:
1.Docker镜像本质是什么?
· 是一个分层文件系统

2. Docker 中一个centos镜像为什么只有200MB，而一个centos操作系统的iso文件要几个个G?。
Centos的iso镜像文件包含bootfs和rootfs，而docker的centos镜像复用操作系统的bootfs，只有rootfs和其他镜像层

3.Docker中一个tomcat镜像为什么有500MB，而一个tomcat安装包只有70多MB?
由于docker中镜像是分层的，tomcat虽然只有70多MB，但他需要依赖于父镜像和基础镜像，所有整个对外暴露的tomcat镜像大小500多MB
```



## 5.2镜像制作

1.容器转为镜像（挂载的目录并不会在自定义的镜像中）

```markdown
docker commit 镜像id 自定镜像名称:自定义版本号
```



2.镜像转为压缩文件（.tar）

```markdown
docker save -o 文件名.tar 自定镜像名称:自定义版本号
```



3.压缩文件转为镜像

```markdown
docker load -i 文件名.tar
```



## 5.3dockerFile概念

- Dockerfile是一个文本文件
- 包含了一条条的指令
- 每一条指令构建一层，基于基础镜像，最终构建出一个新的镜像
- 对于开发人员:可以为开发团队提供一个完全致的开发环境
- 对于测试人员:可以直接拿开发时所构建的镜像或者通过Dockerfile文件构建一个新的镜像开始工作了
- 对于运维人员:在部署时，可以实现应用的无缝移植



## 5.4dockerFile 案列

 [dockerfile.md](dockerfile.md) 

![image-20221120164407914](imgs\image-20221120164407914.png)



## 5.5服务编排

**前言：**

微服务架构的应用系统中一般包含若干个微服务，每个微服务一般都会部署多个实例，如果每个微服务都要手动启停维护的工作量会很大

- 要从Dockerfile buildimage或者去dockerhub拉取image
- 要创建多个container
- 要管理这些container (启动停止删除)

服务编排: 按照一定的业务规则批量管理容器



### 5.5.1服务编排工具Docker Compose

Docker Compose是一个编排多容器分布式部署的工具，提供命令集管理容器化应用的完整开发周期，包括服务构建启动和停止。使用步骤:
1.利用 Dockerfile定义运行环境镜像
2.使用docker-compose.yml定义组成应用的各服务
3.运行 docker-composeup 启动应用

<img src="imgs\image-20221120173058192.png" alt="image-20221120173058192" style="zoom:50%;" />

 [docker-compose.md](docker-compose.md) 



# 6.docker私有仓库

Docker官方的Dockerhub (https://hub.docker.com)是一个用于管理共镜像的库，我们可以从上面拉取镜像到本地，也可以把我们自己的镜像推送上去。但是，有时候我们的服务器无法访问互联网，或者你不希望将自己的镜像放到公网当中，那么我们就需要搭建自己的私有仓库来存储和管理自己的镜像。

 [docker 私有仓库.md](docker 私有仓库.md) 



# 7.docker容器虚拟化与传统虚拟机比较

容器就是将软件打包成标准化单元，以用于开发、交付和部署。

- 容器镜像是轻量的、可执行的独立软件包，包含软件运行所需的所有内容:代码、运行时环境、系统工具系统库和设置。
- 容器化软件在任何环境中都能够始终如一地运行。
- 容器赋予了软件独立性，使其免受外在环境差异的影响，从而有助于减少团队间在相同基础设施上运行不同软件时的冲突。

<img src="imgs\image-20221120182306264.png" alt="image-20221120182306264" style="zoom:50%;" />

相同:

- 容器和虚拟机具有相似的资源隔离和分配优势

不同:

- 容器虚拟化的是操作系统，虚拟机虚拟化的是硬件传统虚拟机可以运行不同的操作系统，容器只能运行同一类型操作系统
