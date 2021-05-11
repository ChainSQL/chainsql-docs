======================
ChainSQL网络搭建
======================


版本获取
==============
ChainSQL 的节点程序可在 `Github开源仓库 <https://github.com/ChainSQL/chainsqld/releases>`_ 获取到，有 windows,linux 两个版本的可执行程序。


软硬件环境要求
==============

    .. image:: ../../images/environment.png
        :width: 600px
        :alt: ChainSQL Framework
        :align: center



一、区块链网络搭建
===============================

需要至少 4 个验证节点，下图是最简单的ChainSQL网络拓扑图：

    .. image:: ../../images/Simple-Network.png
        :width: 600px
        :alt: ChainSQL Network
        :align: center

下面以一个验证节点为例进行说明，想要得到更多节点，重复以下步骤即可。

.. IMPORTANT::

    但是节点使用的密码算法类型一经确定，所有参与区块链网络的节点必须使用同一种密码算法，
    即验证节点公私钥使用同一种密码算法生成，配置文件的[crypto_alg]部分保持一致。

1.	验证节点公私钥的生成
----------------------------
.. WARNING::
    在0.30.3版本之前，执行验证节点公私钥的生成这一命令要提前启动chainsqld进程，是因为下面的validation_create命令要向进程发送rpc请求，如果进程启动不成功，命令会返回错误。0.30.3及之后的版本可以不启动chainsqld程序直接返回结果。

- 0.30.3版本以前需要先按照以下操作启动chainsql: 将可执行程序与配置文件放在用户目录，先执行 

.. code-block:: bash

    nohup ./chainsqld &

.. IMPORTANT::

    如果配置文件在当前目录，且名称为 ``chainsqld.cfg``  ，可直接运行 ``nohup ./chainsqld &`` 命令即可启动节点，否则需要用 ``--conf`` 指定配置文件路径: ``./chainsqld --conf="./ chainsqld-example.cfg" &``

确认chainsqld程序已经启动，输入 ``ps -ef|grep chainsqld`` ，看是否列出chainsqld进程。

- 使用0.30.3之前版本已成功启动chainsql或者使用0.30.3及更高版本执行下面的命令：

1-1. secp256k1算法生成 ``validation_public_key`` 及 ``validation_seed`` , 输入:

.. code-block:: bash

    ./chainsqld validation_create secp256k1
    
返回结果如下：

.. code-block:: json

    {
        "status" : "success",
        "validation_key" : "TUCK NUDE CORD BERN LARD COCK ENDS ETC GLUM GALE CASK KEG",
        "validation_public_key" : "n9L9BaBQr3KwGuMoRWisBbqXfVoKfdJg3Nb3H1gjRSiM1arQ4vNg",
        "validation_seed" : "xxjX5VuTjQKvkTSw6EUyZnahbpgS1"
    }

1-2. 国密sm2算法生成 ``validation_public_key`` 及 ``validation_private_key(等同validation_seed)`` , 输入:

.. code-block:: bash

    ./chainsqld validation_create gmalg
    
返回结果如下：

.. code-block:: json

    {
        "validation_private_key" : "pcGRX6z6fdGzA58j1uh2xH196JvCMyau9QCZmcLGXGSiBrjT4d9",
        "validation_public_key" : "pEn2MTzZQc3kCfu19FJoNFExSpKf5U77cMzrh561roCJmQnmGA3XRzhXDuTqkyUugiBpCnLhUc67hooWATktuUN3vQui3ZX3"
    }



2.	配置文件的修改
---------------------------
以下仅针对部分字段进行说明，针对配置文件的详细说明参考 :ref:`配置文件详解 <配置文件>` 。

``[sync_db]``

  配置ip，port，db，mysql安装时设置的(user,pass)等。

  Chainsql中的事务与行级控制要求每个节点必须配置数据库，如果用不到这两个特性，也可以选择只在需要查看数据的节点配置数据库。

  例如

::

	[sync_db]
	type=mysql
	host=localhost
	port=3306
	user=root
	pass=root
	db=chainsql
	first_storage=0
	unix_socket=/var/lib/mysql/mysql.sock

.. note::

	使用localhost连接时，会默认使用 ``sock`` 方式连接，默认sock路径是 ``/var/run/mysqld/mysqld.sock`` 在非ubuntu系统中，这个路径是不对的，会导致连接数据库失败，需要用 ``unix_socket`` 选项来指定 ``sock`` 路径，如果用ip去连接，会使用 ``tcp`` 方式连接，就不会有这个问题

``[node_db]``

- windows平台: type=NuDB
- Ubuntu平台: type=RocksDB 或 type=NuDB

``[ips_fixed]``

  chainsql尝试进行对等连接的IP地址或主机名及端口号

例如：

::

	[ips_fixed]
	192.168.0.80 5123
	192.168.0.81 5123
	192.168.0.82 5123

``[validators]`` 或 ``[validators_file]``

  添加其他(三个)节点的 ``validation_public_key`` ；

例如：

::

	[validators]
	n9MRden4YqNe1oM9CTtpjtYdLHamKZwb1GmmnRgmSmu3JLghBGGJ
	n9Ko97E3xBCrgTy4SR7bRMomytxgkXePRoQUBAsdz1KU1C7qC4xq
	n9Km65gnE4uzT1V9L7yAY9TpjWK1orVPthCkSNX8nRhpRaeCN6ga

``[validation_public_key]``

  添加本节点的validation_public_key。此字段可不配置，但方便后续查阅，建议配置。

例如：

::

	[validation_public_key]
	n9Jq6dyM2jbxspDu92qbiz4pq7zg8umnVCmNmEDGGyyJv9XchvVn

``[validation_seed]``

  添加本节点的 ``validation_seed`` 。只有验证节点需要配 ``validation_seed`` ，普通节点不需要这一配置。

例如：

::

	[validation_seed]
	xnvq8z6C1hpcYPP94dbBib1VyoEQ1

``[auto_sync]``

::

	[auto_sync]
	1

auto_sync配置为1表示开启表自动同步，开启后，在节点正常运行的情况下，新建表会自动入同步到数据库。

如果不想自动同步，只想同步需要同步的表，使用 ``sync_tables`` 配置项。

``[sync_tables]``

::

	[sync_tables]
	zBUunFenERVydrqTD3J3U1FFqtmtYJGjNP tablename
	zxryEYgWvpjh6UGguKmS6vqgCwRyV16zuy tablename2

配置格式：

- 非加密表格式：	建表账户 表名
- 加密表格式：		建表账户 表名 可解密账户私钥

``[crypto_alg]``

::

	[crypto_alg]
	node_alg_type=secp256k1
    hash_type=sha

配置格式：

- node_alg_type：	支持值：gmalg/secp256k1
- hash_type：		支持值：sm3/sha

此配置项可不填，默认使用secp256k1和sha，不填时validation_seed和validation_public_key均需为secp256k1算法生成。
即node_alg_type的类型必须同validation_seed和validation_public_key生成算法一致。

3.	架设网络
---------------------------
启动chainsqld程序
进入chainsqld应用程序目录，执行下面的命令

::

	nohup ./chainsqld &

每个网络节点均要执行上述命令，使chainsql服务在后台运行。

检查是否成功
进入chainsql应用程序目录，执行命令::

	watch ./chainsqld server_info

**等待2分钟左右**，当输出结果中，字段 ``complete_ledgers``  :有值，类似 "1-10"，则chainsqld服务启动成功
每个网络节点的chainsql服务都要求成功运行

查看其它节点的运行情况：::

	watch ./chainsqld peers

链重启/节点重启
节点全部挂掉的情况：

- 如果想要清空链，将 ``db,rocksdb/NuDb`` 文件夹清空，然后重新执行节链启动过程；
- 如果想要加载之前的区块链数据启动，在某一全节点下执行下面的命令::

	nohup ./chainsqld --load &

其它节点执行：

::

	nohup ./chainsqld &

这样即可加载原来的数据启动链

还有节点在运行的情况

只要网络中还有节点还在跑，就不需要用 ``load`` 方式重启链，只需要启动挂掉的节点即可：::

		nohup ./chainsqld &

4.退出终端
---------------------------
在终端输入 ``exit`` 退出，不然之前在终端上启动的chainsqld进程会退出

二、数据库安装配置（可选）
===============================

.. IMPORTANT::
    用户可在配置文件中配置本地数据库，也可以配置远程数据库。需要注意的是 ``mysql`` 数据库安装完后，需要将默认编码改为 ``utf8`` 编码，否则表中的中文会显示为乱码。

1. 安装mysql
-------------------------

在需要安装mysql数据库的节点上按照提示安装mysql 以ubuntu 16.04为例，安装配置步骤如下：

.. code-block:: bash

	sudo apt-get install mysql-server

如果apt-get install不成功，可以选择 安装过程中会提示设置密码，要记下密码，在后面的配置文件中会用到。

2.检查是否安装成功
-------------------------
检查是否安装成功::

	mysql --version

能查询到mysql版本号则表示安装成功。 

检查是否能正常登录:

.. code-block:: bash

	mysql -uroot –p

上面命令输入之后会提示输入密码，此时正确输入密码就可以登录到mysql。

3.	创建数据库并支持utf8编码
------------------------------------------
登入mysql 后，创建名字为chainsql的database：

.. code-block:: sql

	CREATE DATABASE IF NOT EXISTS chainsql DEFAULT CHARSET utf8 

也可以将mysql的默认编码设置为utf8，然后直接创建数据库

.. code-block:: sql

	create database chainsql;

设置mysql 默认UTF8编码:
修改/etc/mysql/mysql.conf.d/mysqld.cnf文件

``[mysqld]`` 下添加：

::

	character_set_server = utf8

然后在配置文件最后添加如下配置：

::

	[mysql.server]
	default-character-set = utf8
	[client]
	default-character-set = utf8

然后重启mysql：

::

	/etc/init.d/mysql restart

确认是否为utf8编码：

.. code-block:: sql

	show variables like 'character%';

显示如下图则认为database是utf8编码

::

	+-------------------------------+----------------------------+
	| Variable_name                 | Value                      |
	+-------------------------------+----------------------------+
	| character_set_client  	| utf8                       |
	| character_set_connection	| utf8                       |
	| character_set_database   	| utf8                       |
	| character_set_filesystem 	| binary                     |
	| character_set_results    	| utf8                       |
	| character_set_server     	| utf8                       |
	| character_set_system     	| utf8                       |
	| character_sets_dir       	| /usr/share/mysql/charsets/ |



4.	最大连接数设置（可选）
---------------------------------------
.. code-block:: sql

	show variables like '%max_connections%';

| 默认是151， 最大可以达到16384。修改方法有两种。
| 第一种，命令行修改：

.. code-block:: sql
	
	set GLOBAL max_connections = 10000;

| 这种方式有个问题，就是设置的最大连接数只在mysql当前服务进程有效，一旦mysql重启，又会恢复到初始状态。

| 第二种，修改配置文件：

| 这种方式也很简单，只要修改MySQL配置文件my.cnf的参数 ``max_connections`` ，
| 将其改为 ``max_connections=10000`` ，然后重启MySQL即可。



++++++++++++++++

三、Docker 搭建ChainSQL网络
==============================================================

ChainSQL 节点的 Docker 镜像地址 为  ``docker pull peersafes/chainsql:v0.30.6`` 


下面以4个验证节点组建网络为例，介绍Docker搭建ChainSQL网络的过程。

1.	生成4个验证节点的配置文件
--------------------------------------------------------
通过 docker镜像  ``peersafes/chainsql-tools`` 完成节点配置文件的生成。下面的命令生成了4个节点的配置文件，其中节点的IP分别为
``192.168.0.1`` ``192.168.0.2`` ``192.168.0.3`` ``192.168.0.4`` 。
 
.. code-block:: bash

	# 启动镜像
	docker run -itd --name chainsql-tools  -v ~/docker/cfg:/opt/chainsql-tools/cfg  peersafes/chainsql-tools:v0.1.0 /bin/sh

	# 生成节点配置文件
	docker exec -it  chainsql-tools  /bin/sh  /opt/chainsql-tools/genCfg.sh 4 "192.168.0.1;192.168.0.2;192.168.0.3;192.168.0.4"


生成配置文件后，目录的结构如下，其中目录 1 , 2 , 3 , 4 下的配置文件分别表示节点1，2，3，4的配置文件 。

.. code-block:: bash

	# 目录结构为
	├── 1
	│   └── chainsqld.cfg
	├── 2
	│   └── chainsqld.cfg
	├── 3
	│   └── chainsqld.cfg
	└── 4
	│   └── chainsqld.cfg

 
如果需要节点需要配置数据库，需修改对应节点的配置文件  ``chainsqld.cfg`` , 具体配置参考 :ref:`配置数据库 <SyncDB>`

++++++++

2.	启动ChainSQL的Docker镜像
--------------------------------------------------------

拷贝上一步生成的配置文件到4个节点

.. code-block:: bash

	scp ./1/chainsqld.cfg root@192.168.0.1:/opt/chainsql/
	scp ./2/chainsqld.cfg root@192.168.0.2:/opt/chainsql/
	scp ./2/chainsqld.cfg root@192.168.0.3:/opt/chainsql/
	scp ./3/chainsqld.cfg root@192.168.0.4:/opt/chainsql/


依次启动节点1,2,3,4

.. code-block:: bash

	# 登录节点1 后 , 启动节点1
	docker run -d --name node1 -p 5125:5125 -v /opt/chainsql/chainsqld.cfg:/opt/chainsql/chainsqld.cfg peersafes/chainsql:v0.30.6

	# 登录节点2 后 , 启动节点2
	docker run -d --name node2 -p 5125:5125 -v /opt/chainsql/chainsqld.cfg:/opt/chainsql/chainsqld.cfg peersafes/chainsql:v0.30.6

	# 登录节点3 后 , 启动节点3
	docker run -d --name node3 -p 5125:5125 -v /opt/chainsql/chainsqld.cfg:/opt/chainsql/chainsqld.cfg peersafes/chainsql:v0.30.6

	# 登录节点4 后 , 启动节点4
	docker run -d --name node4 -p 5125:5125 -v /opt/chainsql/chainsqld.cfg:/opt/chainsql/chainsqld.cfg peersafes/chainsql:v0.30.6

++++++++


3. 查看网络的状态
--------------------------------------------------------

通过 节点的 ``peers`` , ``server_info``  等命令查看网络的状态

.. code-block:: bash

	# 通过 server_info 查看网络状态 , 返回字段server_status为normal时表示ChainSQL网络正常运行
	docker exec -it node1 /opt/chainsql/chainsqld server_info