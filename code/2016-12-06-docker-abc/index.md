# Docker速查


docker是一款很牛逼的虚拟化工具

<!--more-->

#### 1. 安装

1. 要求
 - 64位系统
 - kernel版本为3.10或以上

2. 以下使用的是Ubuntu 14.04 LTS 64位

参考[官方文档](https://docs.docker.com/engine/installation/linux/ubuntulinux/)

#### 2. 使用(需添加用户组避免每次都sudo)

1. 查看当前镜像列表

 ```
 docker images
 ```

2. 启动镜像: 创建容器并进入容器内的shell

 ```
 docker run -i -t <dockername> /bin/bash
 ```
 - docker run - 运行一个容器
 - -t - 分配一个（伪）tty (link is external)
 - -i - 交互模式 (so we can interact with it)
 - <dockername> - 使用 dockername 镜像
 - /bin/bash - 运行命令 bash shell
  
<!--more-->

3. 离开容器
  - 关闭容器: `Ctrl+C`
  - 用户通过 `exit` 命令或 `Ctrl+d` 来退出终端时，所创建的容器立刻终止。
  - 离开容器, 让容器继续运行: ` Ctrl+P, Ctrl+Q `

4. 查看当前运行的容器
 - ` sudo docker ps -a` -a为查看所有的容器，包括已经停止的。

5. 重新连接容器
 - `docker attach <containerID>`, 其中containerID是容器名或ID, 可以使用缩写

6. 从停止的容器恢复
 - `docker start <containerID>`

7. 从容器创建Docker镜像
 - `docker commit -m "commit" -a "author" 0b2616b0e5a8 ouruser/sinatra:v2`
 - 其中`0b2616b0e5a8`为容器ID, `ouruser/sinatra`为用户/镜像名, `:v2`为版本号

#### 3. 其他
1. 容器-宿主互传文件

 ```
 docker cp foo.txt mycontainer:/foo.txt
 docker cp mycontainer:/foo.txt foo.txt
 ```

2. 启动镜像时挂载host的文件夹

 ```
 docker run -v /host/directory:/container/directory <镜像名>
 ```
 
3. 通过导出导入的原理缩减镜像大小[Reference](http://tuhrig.de/flatten-a-docker-container-or-image/)

 ```
 # export the container to a tarball
 docker export <CONTAINER ID> > /home/export.tar
 
 # import it back
 cat /home/export.tar | docker import - some-name:latest
 ```
 
> to be continued...
