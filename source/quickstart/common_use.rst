===========================
基础操作
===========================

在这节课中，我们将学习chainsql的基本使用

1. Mysql数据库启动及配置
=================================

给节点配置mysql数据库，这里使用mysql的docker镜像，实际项目中一般直接安装mysql-server

docker启动mysql实例
-------------------------------
1. 拉取mysql的docker镜像:

::
    
    docker pull mysql:5.7

2. 启动mysql实例：
   
::

    docker run --name docker-mysql -e MYSQL_ROOT_PASSWORD=1234 -d -p 3306:3306 mysql:5.7

在mysql中创建chainsql的数据库
--------------------------------------
1. 执行如下命令进入mysql的docker
   
::

    docker exec -it docker-mysql sh

2. 执行如下命令，然后输入密码 ``1234`` 登录mysql

::

    mysql -uroot -p

3. 执行命令在mysql中创建chainsql数据库

::

    CREATE DATABASE IF NOT EXISTS chainsql DEFAULT CHARSET utf8;

1. 执行命令`exit`退出mysql登录，然后执行`exit`退出mysql的docker

::

    exit
    exit

2. 在节点中配置mysql数据库
==========================================
1. 执行如下命令对节点1的配置文件进行编辑
::

    cd ~/chainsql/1 && vim chainsqld.cfg

2. 在文件结尾插入以下内容，然后按 ``:wq`` 保存并退出文件：

::

    [sync_db]
    type=mysql
    host=127.0.0.1
    port=3306
    user=root
    pass=1234
    db=chainsql
    first_storage=0
    charset=utf8

    [auto_sync]
    1


3. 可以用如下命令查看配置文件，确认已经正确添加相关配置

::

    cat chainsqld.cfg

4. 执行命令启动4个节点
   
::

    cd ~/chainsql && ./startAll.sh
   
3. 激活账户及查询账户信息
==================================

ChainSQL在创世块中有一个根账户（ ``zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh`` ），所有的系统币( ``ZXC`` )都在这个根账户中，可以执行下面的命令
查看根账户信息，可以看到它的余额信息 ``Balance`` 字段

::

    ./chainsqld account_info zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh

1. 用如下命令生成一个新的账户，并复制结果中的 ``account_id`` 字段的值，如 ``z9t6fxjiRMx7P5wTJEX65p6ys9JFxWSN93``
   
::

    ./chainsqld wallet_propose

2. 执行以下命令查看账户信息，会报一个 ``actNotFound`` 的错误，说明账户在链上不存在
   
::

    ./chainsqld account_info z9t6fxjiRMx7P5wTJEX65p6ys9JFxWSN93

3. 执行下面的命令激活新的账户，注意将 ``Destinaion`` 字段值成之前复制的 ``account_id`` 值

::

    curl -H "Content-Type: application/json" -X POST -d '
    {
        "method": "submit",
        "params": [{
            "offline": false,
            "secret": "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb",
            "tx_json": {
                "Account": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
                "Amount": "100000000",
                "Destination": "z9t6fxjiRMx7P5wTJEX65p6ys9JFxWSN93",
                "TransactionType": "Payment"
            }
        }]
    }
    ' http://127.0.0.1:5005

返回结果中可以看到 ``"engine_result":"tesSUCCESS"`` 表示交易发送成功，返回结果最后一个字段是 ``hash`` ，它的值是当前发送交易的hash值

4. 执行命令查看账户信息，可以看到余额正是我们激活交易中的 ``Amount`` 值

::

    ./chainsqld account_info z9t6fxjiRMx7P5wTJEX65p6ys9JFxWSN93

4. 建表，插入，查询操作
================================
这里都用根账户进行操作

1. 建表交易，执行下面的命令发送建表交易

::

    curl -H "Content-Type: application/json" -X POST -d '
    {
        "method": "t_create",
        "params": [{
            "secret": "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb",
            "tx_json": {
                "TransactionType": "TableListSet",
                "Account": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
                "Tables": [
                    {
                        "Table": { "TableName": "test_chainsql" }
                    }
                ],
                "OpType": 1,
                "Raw": [
                    {
                        "field": "id",
                        "type": "int",
                        "length": 11,
                        "PK": 1,
                        "NN": 1,
                        "UQ": 1
                    },
                    {
                        "field": "age",
                        "type": "int"
                    },
                    {
                        "field": "name",
                        "type": "varchar",
                        "length": 16
                    }
                ],
                "Confidential": false
            }
        }]
    }
    ' http://127.0.0.1:5005


通过上面的请求可以知道，我们建了一张名为 ``test_chainsql`` 的表，表中有 ``id,name,age`` 3个字段，并且是一张非加密表

2. 表插入交易，执行下面的命令向表中插入两条数据：

::

    curl -H "Content-Type: application/json" -X POST -d '
    {
        "method": "r_insert",
        "params": [{
            "offline": false,
            "secret": "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb",
            "tx_json": {
                "TransactionType": "SQLStatement",
                "Account": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
                "Owner": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
                "Tables":[
                    {
                        "Table": { "TableName": "test_chainsql" }
                    }
                ],
                "Raw": [
                    {
                        "id": 1,
                        "name": "张三",
                        "age": 11
                    },
                    {
                        "id": 2,
                        "name": "李四",
                        "age": 12
                    }
                ],
                "OpType": 6
            }
        }]
    }' http://127.0.0.1:5005


3. 在数据库中查看表

- 执行命令进入到mysql容器

::

    docker exec -it docker-mysql sh

- 执行以下命令，然后输入密码 `1234` 登录mysql

::

    mysql -uroot -p

- 执行命令切换到chainsql数据库

::

    use chainsql

- 执行以下命令 可以看到数据库中有两张表， ``SyncTableState``为管理表， ``t_`` 开头的是我们刚刚在链上建的表

::

    show tables;

- 可以执行命令查看管理表的内容

::

    select * from SyncTableState;

- 执行命令 `exit` 退出mysql登录，然后执行 `exit` 退出mysql的docker

::

    exit
    exit

4. 查询表中的数据，执行下面的命令查询表中所有内容，注意将其中的 ``t_37F7B9AE97A20933D90D41AA86F76226EF467C5D`` 换成数据库中真正的表名

::

    curl -H "Content-Type: application/json" -X POST -d '
        {
            "method": "r_get_sql_admin",
            "params": [
                {
                    "sql": "select * from t_37F7B9AE97A20933D90D41AA86F76226EF467C5D"
                }
            ]
        }' http://127.0.0.1:5005


返回结果中，可以看到我们刚刚插入的两条数据

5. 区块查询，交易查询
=================================
先执行命令切换到chainsql节点的目录

::

    cd ~/chainsql/1

2. 查询区块信息

- 可以执行命令查看当前最新区块信息

::

    ./chainsqld ledger

- 可执行命令指定区块号来查询区块信息
  
::

    ./chainsqld ledger 2

3. 查询交易信息

- 按账户查询，执行以下命令查询根账户在1-100区块之间的所有交易

::

    ./chainsqld account_tx zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh 1 100

- 按区块查询， 执行以下第一条命令可以查询区块2上的交易数量，执行第二条命令可以查询到区块2上成功或失败的交易hash

::

    ./chainsqld ledger_txs 2
    ./chainsqld ledger_txs 2 include_success include_failure

- 按交易hash查询，以下命令查询交易信息，注意将其中的tx_hash换成真正的交易hash

::

    ./chainsqld tx tx_hash
