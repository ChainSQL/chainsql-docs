===========================
通过docker建链
===========================
在这节课程中，你将学到如何使用 ``docker`` 对 ``chainsql`` 进行部署，我们将部署4个 ``docker`` 容器，每个容器中一个 ``chainsql`` 进程

1. 拉取镜像
=============================

ChainSQL 节点的 Docker 镜像地址为：
`ChainSQL docker地址 <https://hub.docker.com/r/peersafes/chainsql/tags?page=1&ordering=last_updated>`_

下面以最新版本 ``latest`` 为例进行操作

拉取镜像，执行命令

::
    
    docker pull peersafes/chainsql:latest

2. 生成配置文件
=======================

1. 执行命令下载 ``chainsql-tools.tar.gz`` 

::

    wget https://github.com/ChainSQL/chainsql-tools/releases/latest/download/chainsql-tools.tar.gz

2. 执行命令对工具解压
   
::
    
    tar zxvf chainsql-tools.tar.gz

3. 执行如下命令在生成4个配置文件，用 ``tree`` 命令可以看到有 ``1``, ``2``, ``3``, ``4`` 4个文件夹,每个文件夹下有一个 ``chainsqld.cfg`` 文件

::

    ./genCfg.sh
    tree

3. Docker启动
================================
1. 拉取docker部署相关的配置文件：

::
    
    cd ~ && wget http://chainsql.net/chainsql-deploy.tar.gz && tar zxvf chainsql-deploy.tar.gz && rm chainsql-deploy.tar.gz

2. 执行以下命令，用docker启动4个节点

::
    
    docker-compose -f chainsql-deployment.yaml up -d

3. 用如下命令查看容器列表，可以看到我们启动了4个容器，名称分别为 ``node1``, ``node2``, ``node3``, ``node4``

::

    docker ps

4. 进入节点1的容器
   
::
    
    docker exec -it node1 sh

5. 查看节点状态
   
::
    
    ./chainsqld server_info
 
6. 查看邻接节点信息

::
    
    ./chainsqld peers

7. 退出docker终端 
   
::
    
    exit

4. 关闭容器
===========================

1. 执行下面的命令关闭容器

::
    
    docker-compose -f chainsql-deployment.yaml down

2. 用如下命令去查看当前运行的docker容器，会发现列表是空的

::

    docker ps -a