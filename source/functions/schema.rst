=================
主子链使用说明
=================

主子链是 ``ChainSQL 3.0`` 的基础架构:

- 一般来说，主链用来初始化数据，管理子链节点，子链用来做各种业务，
- 子链是依赖于主链的，可以只有主链没有子链，但是有子链就一定会有主链

需要注意的有如下几点：

- ``ChainSQL 3.0`` 中不存在之前版本的普通节点（不配置 ``validation_seed`` ），所有网络中的节点都要有身份（ ``validation_publickey`` ）。
- 子链都存储在主链配置文件中 ``schema_path`` 配置的路径下面，用子链ID进行区分

1. 配置项说明
=================

--------------------------------
1.1 主链中新增与子链相关配置
--------------------------------

配置示例：

.. code-block:: bash

    [schema]
    schema_path=/mnt/schema
    auto_accept_new_schema = 1
    only_validate_for_schema = 1

配置项的说明参考 :ref:`Schema配置 <Schema>`

.. warning::

    | ``ChainSQL 3.0`` 中所有节点必须配置 ``validation_seed`` ，不然无法识别节点身份。
    | 目前主链中可通过配置 ``only_validate_for_schema=1`` 表示节点为主链的非共识节点。
    | 目前子链不支持非共识节点
    
.. warning::
    
    当一个节点配置为主链非共识节点时( ``only_validate_for_schema=1`` )，需要配置主链所有共识节点公钥，不然会出现当前节点的 ``server_status`` 偶尔变成 ``abnormal`` 的情况

------------------------------
1.2 子链配置文件
------------------------------

子链配置文件是节点创建子链时自动生成的，配置文件中移除了主链的配置文件的如下配置：

======================== =======================      ================================
配置项                      作用                        移除原因
======================== =======================      ================================
server                    端口配置                      共享主链配置
rpc_startup               设置日志级别                  共享主链配置
sntp_servers              时间服务器                    共享主链配置
validation_seed           节点共识私钥                  共享主链配置
debug_logfile             日志文件路径                  共享主链配置
schema                    子链相关配置                  子链节点不需要
ips_fixed                 固定连接的ip端口               子链节点不需要
sync_db                   数据库连接                    无法共享，需单独配置
auto_sync                 自动同步                      无法共享，需单独配置
sync_tables               手动配置同步表                 无法共享，需单独配置
validator_list_keys       信任的unl-server公钥           子链节点不需要
validator_list_sites      unl-server地址                子链节点不需要
======================== =======================      ================================

而有些配置与主链不同，在生成时进行了修改：

======================== ===========================      ===================================
配置项                      作用                                修改原因
======================== ===========================      ===================================
node_db的子项path           链数据的存储路径                 子链的数据都在子链ID标识的文件夹下
database_path               索引数据库路径                  子链的数据都在子链ID标识的文件夹下
ips                         子链要连接的节点ip:端口          子链要连接的节点受主链控制
validators                  子链共识节点公钥列表             子链要连接的节点受主链控制
======================== ===========================      ===================================

2. 子链相关交易 
===================

--------------------------------
2.1 SchemaCreate
--------------------------------
- :ref:`Rpc接口 <rpcSchemaCreate>`
- :ref:`Java接口 <javaSchemaCreate>`
- :ref:`Node.js接口 <nodecreateSchema>`
  
--------------------------------
2.2 SchemaModify
--------------------------------
- :ref:`Rpc接口 <rpcSchemaModify>`
- :ref:`Java接口 <javaSchemaModify>`
- :ref:`Node.js接口 <nodemodifySchema>`

3. 子链相关其它API接口
===========================

--------------------------------
3.1 schema_list
--------------------------------
查询子链列表，具体使用请参考以下各接口：

- :ref:`命令行接口 <cmdSchemaList>`
- :ref:`Rpc接口 <rpc查询子链列表>`
- :ref:`Java接口 <javagetSchemaList>`
- :ref:`Node.js接口 <nodegetSchemaList>`
  
--------------------------------
3.2 schema_info
--------------------------------
查询子链信息，具体使用请参考以下各接口：

- :ref:`命令行接口 <cmdSchemaInfo>`
- :ref:`Rpc接口 <rpc查询子链信息>`
- :ref:`Java接口 <javagetSchemaInfo>`
- :ref:`Node.js接口 <nodegetSchemaInfo>`
  
--------------------------------
3.3 schema_accept
--------------------------------
子链参与节点接受加入子链，是一个admin权限的接口，一般使用命令行调用：

- :ref:`命令行接口 <cmdSchemaAccept>`

--------------------------------
3.4 sign_for
--------------------------------
| 对于节点加入子链的确认方式中，通过多方签名交易可以让节点在建链交易共识通过后直接创建子链。
| 如果是节点签名，增加可选字段 ``for_node`` ，设置为true，示例如下：

.. code-block:: json 

    {
        "method":"sign_for",
        "params":[
            {
                "secret":"x████████████████████████████",
                "for_node":true,
                "tx_json":{
                    "Account":"z9dCd5pdwWJTGUaJC1gVp5Rf6wAGEGP6L8",
                    "Amount":{
                        "currency":"USD",
                        "issuer":"z6aCd5pdwWJTGUaJC1gVp5Rf6wAGEGP6L5",
                        "value":"1"
                    },
                    "Destination":"zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
                    "TransactionType":"Payment"
                },
                "fee_mult_max":1000
            }
        ]
    }



4. 接口访问主链/子链
=======================

使用各种接口访问子链，都需要指定子链ID（其实是一个hash值，长度为64的字符串），主链的ID为全部为0

--------------------------------
4.1 命令行接口
--------------------------------

通过 ``--schemaid`` 参数指定要操作哪个子链，示例如下查询子链ID为 ``55AC1593D5F9859092EC4763B3EC9A52B6AFDB4B562ABB5C61390A70C4A2B0AB`` 的子链上区块100的信息:

.. code-block:: bash

    ./chainsqld ledger 100 --schemaid=55AC1593D5F9859092EC4763B3EC9A52B6AFDB4B562ABB5C61390A70C4A2B0AB

返回值格式与主链一样

--------------------------------
4.2 RPC接口
--------------------------------
通过参数 ``schema_id`` 指定子链ID，如:发送交易：

.. code-block:: json

    {
        "method": "submit",
        "params": [{
            "secret": "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb",
            "schema_id":"55AC1593D5F9859092EC4763B3EC9A52B6AFDB4B562ABB5C61390A70C4A2B0AB",
            "tx_json": {
                "TransactionType": "Payment",
                "Account": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
                "Destination": "zwNSN5J1b67bKqzddvJ9G7HdB87DeML2ak",
                "Amount": "5000000000"
            }
        }]
    }

在子链上查询区块数据

.. code-block:: json

    {
        "method": "ledger_data",
        "params": [
            {
                "schema_id":"AC69A827983A1DDAC7BD408EB8DC93DCB3F4ABF22C3989BAA5781B1C6154E9A2",
                "ledger_index": 100,
                "full": false,
                "accounts": false,
                "transactions":true,
            }
        ]
    }

--------------------------------
4.3 Java接口
--------------------------------
通过 ``setSchema`` 接口来设置子链ID，参考 :ref:`JavaAPI setSchema 接口说明 <javasetSchema>` 

--------------------------------
4.4 Node.js接口
--------------------------------

通过 ``setSchema`` 接口来设置子链ID，参考 :ref:`Node.js API setSchema 接口说明  <javasetSchema>` 
