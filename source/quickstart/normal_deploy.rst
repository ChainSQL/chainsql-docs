===========================
通过二进制程序建链
===========================

在这节课中，你可以学习到如何获取chainsql节点版本，如何在一台机器上启动单个节点，搭建4节点区块链，以及简单的查看节点状态命令

1. 拉取二进制程序
============================

chainsql的节点版本都发布在 `ChainSQL 节点github地址 <https://github.com/ChainSQL/chainsqld/releases>`_

以 ``v3.0.0`` 为例，打开 ``v3.0.0`` 的tag，页面下方有 chainsqld linux环境的安装包： ``chainsqld-linux-x64-3.0.0.tar.gz``

通过下面的命令可将版本拉取到本地

::

    wget https://github.com/ChainSQL/chainsqld/releases/download/v3.0.0/chainsqld-linux-x64-3.0.0.tar.gz

2. 单节点启动
============================
在将chainsql节点程序下载到本地后，执行如下操作：

1. 创建chainsql文件夹

::

    mkdir chainsql

2. 执行命令将节点程序解压到 chainsql目录

::

    tar zxvf chainsqld-linux-x64-3.0.0.tar.gz -C chainsql

3. 解压后执行 ``cd chainsql`` 进入到chainsql目录，然后执行 ``ls`` 命令可以看到目录下有 ``chainsqld`` 与 ``chainsqld.cfg`` 两个文件，其中 ``chainsqld.cfg`` 是可单点执行的配置文件
4. 执行如下命令后台启动chainsql节点

::

    nohup ./chainsqld &

5. 执行下面的命令可以查看节点状态，注意其中的 ``server_status`` 字段，大约等待10秒后，可以看到 ``server_status`` 的值由 ``abnormal`` 变成了 ``normal`` ，表示节点可用，可正常接收交易。

::

    watch ./chainsqld server_info

6. 执行 ``Ctrl+C`` 退出 ``watch``
7. 执行下面的可以看到有一个chainsql进程正在运行

::

    ps -ef|grep chainsqld

8. 执行如下命令停止运行chainsql进程，然后通过执行 ``ps -ef|grep chainsqld`` 发现列表为空，可确认节点已经退出

::

    ./chainsqld stop

3. 启动4节点网络
============================

配置说明
---------------------
``ChainSQL`` 的配置文件一般放在与可执行文件 ``chainsqld`` 同级的目录中，名称为 ``chainsqld.cfg``

具体的配置信息可参考 `ChainSQL 配置文件详解 <http://docs.chainsql.net/theory/cfg.html>`_

配置多个节点需要注意其中的 ``[ips]`` 与 ``[validators]`` 需要节点间两两配置

众享实验室提供了一个 `配置文件生成工具 <https://github.com/ChainSQL/chainsql-tools>`_ ，可以将根据配置模板将 ``[ips]`` 与 ``[validators]`` 都配置好

下载打包好的4个节点配置文件
--------------------------------------

1. 执行以下命令下载 chainsql-cfgs.tar.gz 

::

    cd ~ && wget http://chainsql.net/chainsql-cfgs.tar.gz

2. 执行以下命令创建新的文件夹

::

    mkdir chainsql-4nodes


3. 执行以下命令 进行解压

::

    tar zxvf chainsql-cfgs.tar.gz -C ~/chainsql-4nodes

4. 执行以下命令拷贝可执行文件

::

    cp ~/chainsql/chainsqld ~/chainsql-4nodes

启动节点，并查看状态
----------------------------------------
1. 执行以下命令启动4个节点

::

    cd ~/chainsql-4nodes && ./startAll.sh

2. 执行以下命令可以看到4个chainsqld的进程

::

    ps -ef|grep chainsqld

3. 执行以下命令可以看到当前1节点有3个邻节点

::

    cd ~/chainsql-4nodes/1 && ./chainsqld peers

4. 执行以下命令等待10秒（ ``uptime`` 字段的值>=10）左右， ``server_status`` 值会变成 ``normal``

::

    ./chainsqld server_info
