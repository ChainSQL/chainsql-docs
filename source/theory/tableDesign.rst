=================
数据库功能设计
=================


1. 引言
=================

------------------
1.1 编写目的
------------------

        该文档旨在设计ChainSQL的架构和技术需求，通过此设计，合并区块链技术和传统分布式数据库的特性，建立一个去中心化的分布式数据库系统。

------------
1.2 背景
------------

        基于以下技术实现

        - ChainSQL 区块链技术实现
        - RethinkDB 实时分布式数据库
        - MySQL 以及 sqlite


2. 总体说明
=================

---------------
2.1 运行环境
---------------
        区块链网络的搭建，底层数据库可以直接适配 ``SQLITE`` ，``MySQL`` ，通过 ``mycat`` 间接适配其它各种类型的数据库。上层提供 ``javascript`` 与 ``java`` 版本的 api 对数据表进行增删改查，用户也可以通过 ``kingshard`` 使用原生的数据库接口进行数据库表的各种操作。

----------------        
2.2 系统架构
----------------

        .. image::  ../../images/ChainSQL.png

-----------------        
2.3 元素说明
-----------------


============  =====================================================================
名称           定义
============  =====================================================================
用户            ChainSQL 中的帐号，数据库表的操作者     
表              对应ChainSQL中的特殊资产及新的ledge tree node		
读写授权        表的所有者赋予其它帐号一定的读写权限，其它帐号才可能操作对应的数据库表
TableList       自定义的LEDGER NODE TYPES，记录用户创建的表 
TableEntry      TableList中TableEntries数据中元素，记录数据库表中的基本信息
============  =====================================================================

3. 基本功能
================

----------------
3.1 数据同步
----------------

        各节点在接入网络后，能进行数据的同步，并最终和网络上其它节点的数据保存一致。数据同步后，支持从不同的节点读数据。

---------------------------------
3.2 当前数据和日志（交易）分离
---------------------------------

        在最新区块记录当前的所有用户的最新数据状态及日志（交易）id。各节点可以配置要同步及保留的区块数量，例如最近的1000个区块或全部（full_history）。

-------------------------
3.3 表的创建及权限授予
-------------------------

        用户要往链上写入数据记录之前，必须先创建表，定义表的结构（字段属性等）。默认只有创建该表的用户才有此表的读写权限, 其它用户得先让表的所有者赋予一定权限后才能对表进行相应的操作。


.. _表同步设置:

-------------------
3.4 选择要同步的表
-------------------

        - 可配置同步某账户下所有表
        - 节点默认不主动同步自定义表的数据，需要节点在配置文件中指定要同步的表。只要知道表的所有者地址及表名即可同步。
        - 如果创建表时选择了加密 ``Raw`` 字段，则此处在表名后需要添加拥有同步权限的账户的私钥。
        - 如有需要，可以配置同步条件，意为只符合同步条件之前的数据。目前同步条件支持两种，同步到指定条件之前或者同步过程中跳过指定条件。
        - 对于同步到指定条件之前，目前条件可以设置为时间及ledger索引，时间支持标准格式，如 ``2016-12-29 12:00:00`` ，ledger索引为整数，如2000。对于同步过程中跳过指定条件，目前条件可以设置为交易Hash及ledger索引，交易Hash唯一标识某一交易，ledger索引为整数，如2000。

        配置的参考格式如下：
        
.. code-block:: bash

            [sync_tables]
            #表的发行帐户地址(只同步账户下所有非加密表)
            z9VF7yQPLcKgUoHwMbzmQBjvPsyMy19ubs 
            #表的发行帐户地址 解密的私钥（账户下加密表与非加密表都同步）
            z9VF7yQPLcKgUoHwMbzmQBjvPsyMy19ubs xxWFBu6veVgMnAqNf6YFRV2UENRd3
            #表的拥有者帐户地址 表名
            z9VF7yQPLcKgUoHwMbzmQBjvPsyMy19ubs table1
            #表的拥有者帐户地址 表名 同步到ledgerSeq2000
            z9VF7yQPLcKgUoHwMbzmQBjvPsyMy19ubs table1 2000
            #表的拥有者帐户地址 表名 同步到2016-12-29 12:00:00
            z9VF7yQPLcKgUoHwMbzmQBjvPsyMy19ubs table2 2016-12-29_12:00:00
            #表的拥有者帐户地址 表名 解密的私钥
            z9VF7yQPLcKgUoHwMbzmQBjvPsyMy19ubs table3 xxWFBu6veVgMnAqNf6YFRV2UENRd3
            #表的拥有者帐户地址 表名 跳过ledgerSeq2000
            z9VF7yQPLcKgUoHwMbzmQBjvPsyMy19ubs table3 ~2000
            #表的拥有者帐户地址 表名 跳过指定的交易hash
            z9VF7yQPLcKgUoHwMbzmQBjvPsyMy19ubs talbe4 ~860689E0F4A20F4CC0B35804B66486D455DEEFA940666054F780A69F770135C0
            #表的拥有者帐户地址 表名 解密的私钥 同步到2016-12-29 12:00:00
            z9VF7yQPLcKgUoHwMbzmQBjvPsyMy19ubs talbe4 xxWFBu6veVgMnAqNf6YFRV2UENRd3 2016-12-29_12:00:00
            #表的拥有者帐户地址 表名 跳过ledgerSeq2000 解密的私钥
            z9VF7yQPLcKgUoHwMbzmQBjvPsyMy19ubs talbe4 ~2000 xxWFBu6veVgMnAqNf6YFRV2UENRd3

---------------------
3.5 配置数据库
---------------------

        Chainsql支持多种数据库，``mysql`` 、 ``sqlite`` 、 ``mycat`` 转换后支持 ``mongodb`` 、 ``db2`` 、 ``sqlserver`` 、 ``PostgreSQL`` 等，本地通过配置文件插件式管理，配置的参考格式如下：

::

        [sync_db]
        type=mysql //数据库类型
        host=localhost //数据库的IP地址
        port=3306 //数据库端口
        user=root //数据库用户名
        pass=root //数据库密码
        db=chainsql //数据库中使用的Scheme名称
        charset=utf8 //是否使用UTF-8编码，以支持中文

-----------------------
3.6 先入库后共识
-----------------------
        

        在配置好 ``[sync_db]`` 的情况下，默认情况下先入库后共识处于关闭状态，如开启，先将数据写入数据库事务中，事务并不提交。如果共识成功，那么事务提交，否则，事务回滚。在特定情况下，可以将其关闭。配置的参考格式如下：

::

        [sync_db]
        first_storage=0

|       使用本功能的前提是按照3.5节配置好数据库。


--------------------
3.7 自动同步开关
--------------------
        

        此开关为关闭状态时，只同步 ``[sync_tables]`` 中填写的表。对于某些情况，需要实时同步区块链中新创建的表，那么可以将此开关打开。
        
        配置的参考格式如下：

.. code-block:: bash

        [auto_sync]
        1　　　　#1为打开此开关，0为关闭此开关


----------------------------------
3.8 通过ChainSQL API对表的写操作
----------------------------------

        | 表的写操作（添加记录、修改记录、删除记录），需要发到全网经过共识验证后才能存档。
        | 用户应该在每次写操作后，对操作的结果进行确认后，再进行其它读写操作。

---------------------------------
3.9 通过ChainSQL API对表的读操作
---------------------------------

        表的读操作，直接传入底层读本地的数据库

-------------------
3.10 数据的回滚
-------------------

        可以根据日志表进行数据的回滚，或整个表的重建

-------------------
3.11 事务的支持
-------------------

        上层API提供事务操作的接口，使用本功能的前提是按照3.5配置好本地数据库，然后按照3.7打开自动同步开关。


.. _Raw_Confidential:

---------------------
3.12 Raw字段加密
---------------------

- 如果出于保密性考虑，对于某张表的操作不想让其它用户看到，可以选择在操作表时对Raw字段加密，密码在创建表时随机生成，用生成的密码对Raw字段进行对称加密，密码使用公钥加密存放，只有表的创建者与被授权的用户可以用自己的私钥去解密，拿到解密后的密码之后再对Raw字段进行对称解密，才能看到Raw字段的明文。
- 如果需要同步某张使用Raw字段加密的表，需要在节点的配置文件中配置拥有读权限的用户私钥，配置格式参考3.4。
- 需要注意的是，对于先入库功能，需要在配置先入库的节点提前配置用户私钥，对于事务类型的交易，因为交易中会出现查询类型的交易，其中包含加密的raw字段，所以需要在共识节点配置用户私钥才能共识通过。


--------------------
3.13 Strict模式
--------------------

- 在限制模式下，语句共识通过的条件是期望的表的快照HASH与预期一致。
- 第一次建表时，快照HASH=HASH(建表的Raw)。
- 增删改操作时，快照HASH=HASH(上次的快照HASH+操作的Raw)。
- 授权、改表名、删除表时不修改快照HASH。

----------------------
3.14 表的行级控制(P2)
----------------------

- 表的增删改查支持行级控制
- 插入表可设置默认填写字段（账户字段、交易哈希字段）
- 插入表可限制单个账户的插入条数
- 更新表可限制允许更新的字段
- 更新、删除、查询表可限制条件，规则参见8.Raw字段详解

-------------------------
3.15 表、交易的订阅(P2)
-------------------------

- 通过提供表的创建者账户地址与表名订阅一张表
- 订阅表成功后，与表相关的交易结果（共识或入库）都会通过回调返回
- 通过提供交易哈希订阅单个交易
- 交易订阅成功后交易的共识结果与入库结果（Chainsql）会通过回调返回
- 取消订阅必需与要取消的订阅在同一个websocket连接中执行

---------------------
3.16 表的重建(P2)
---------------------

- 通过表的重建功能可对区块链进行“瘦身”
- 可通过表的重建功能将表的创建点移到新建区块
- 重建表之前通过dump导出表相关交易
- 表重建后可通过重新发送交易重建表的数据

------------
3.17 dump
------------

- 将数据库表的操作以文档的形式进行记录，可以分多次对同一张表进行dump。
- 实现方式：通过Commandline方式进行操作。
- 命令形式： ``chainsqld "para1" "para2"``
- ``Para1`` : 参考数3.4节中的设置，与“数据库表的同步设置”保持一致。
- ``Para2`` : 数据库表操作保存的目标路径。
- 例::

        chainsqld t_dump "z9VF7yQPLcKgUoHwMbzmQBjvPsyMy19ubs table1 2000" "/chainsql/table1.dmp"

------------
3.18 审计
------------

- 对数据库表的指定条目特定字段进行追根溯源，将所有影响了指定条目特定字段的数据库表操作都记录下来。
- 实现方式：通过 ``Commandline`` 方式进行操作。
- 命令形式： ``chainsqld "para1" "para2" "para3"``
- ``Para1`` : 参考数3.4节中的设置，与“数据库表的同步设置“保持一致。
- ``Para2`` : sql查询语句，表明指定条目特定字段，如“select name, salary from - table1 where id=1”，代表审计数据库表table1中id=1的条目的name与salary字段，所有与对数据库表table1的操作中影响到id=1的条目中的name与salary字段的操作将被记录。
- ``Para3`` : 数据库表操作保存的目标路径。
- 例::

        chainsqld t_audit "z9VF7yQPLcKgUoHwMbzmQBjvPsyMy19ubs table1 2000" "select name, salary from table1 where id=1” “/chainsql/table1.dmp"

----------------
3.19 数据同步
----------------
- 前提：节点保存链上的所有表且所有表已经是最新的，并且在清理数据期间没有别的对表的操作。
- 操作步骤：

    1. 用dump命令将表导出至文件存档，以备以后检索
    2. 调用API接口发送recreate类型的交易
    3. 依次处理网络中的所有节点，每个节点均按照下列流程处理

            - 停止节点程序
            - 将节点的配置文件中 ``[ledger_history]`` 改为512.
            - 定位程序生成的区块数据文件存放路径（ ``[node_db]`` 和 ``[database_path]`` ），将文件目录删除。默认为当前程序目录下 ``db`` 和 ``rocksdb`` 目录
            - 启动节点程序，用server_info（使用说明见附录）查看，确定节点正确同步上
    4. 清理实际存储数据库：将每个节点连接的实际数据库清空，对于没有连接数据库的节点则无需此操作
    5. 修改网络中所有节点配置，依次重启

            - 将网络中的所有节点的配置文件中 ``[ledger_history] `` 均改回 ``full`` ，然后节点依次重启，每一个节点重启后用 ``server_info`` 查看，确定节点正确同步上区块，直至全部处理完毕
    6. 在本地配置好同步的表，然后启动本地节点，开始同步，待确认表建立完成后，进行下一步
    7. 客户准备好需要恢复的数据交易，发往网上参与共识，待共识通过后，至此对区块链的数据清理和恢复完成

4. 性能指标
============

-------------------
4.1 数据的一致性
-------------------
        各节点在完成同步后，数据的一致性要达到100%，多节点同时读写的情况下，数据的一致性不小于99%

---------------------
4.2 数据的可靠性
---------------------
        单节点写入时，全网数据的可靠性达到100%，单点非法篡改数据不会同步到其它节点

----------------
4.3 写入速度
----------------
        支持每秒万级的写操作

5. 结构定义
============

---------------------------------
5.1 自定义的表: SyncTableState
---------------------------------
- 该表用于记录用户需要同步的表的状态


=================  ==================  ==============================================================
Field    	      Internal Type   	   Description
=================  ==================  ==============================================================
Owner               	String       	 所有者AccountID
TableName           	String 	         要同步的表名
TableNameInDB 	        String 	         要同步的实际表名
TxnLedgerHash        	String 	         同步到的影响该表的交易所在ledger hash
TxnLedgerSeq         	String 	         同步到的影响该表的交易ledger index
LedgerHash 	        String 	         同步到的ledger hash
LedgerSeq 	        String 	         同步到的ledger index
TxnUpdateHash        	String 	         同步到的交易Hash
deleted              	String 	         标识此表是否已经删除（0-未删除 1-已经删除）
AutoSync             	String 	         标识此表是否是自动同步（0-不是 1-是）
TxnLedgerTime 	        String 	         标识交易发生时的时间
PreviousCommit 	        String       	 该表的上次快照点
=================  ==================  ==============================================================

-----------------------------------------------
5.2 自定义的LEDGER NODE TYPES：TableList
-----------------------------------------------
- 新加一种LEDGER NODE TYPES（TableList），用于存储用户的表数据。TableList node定义如下：

=====================  ==================  ======================  ============================================================================================================================================
Field    	              JSON Type             Internal Type   	    Description
=====================  ==================  ======================  ============================================================================================================================================
OwnerNode               	String       	    UInt64                  A hint indicating which page of the owner directory links to this node, in case the directory consists of multiple nodes
PreviousTxnID           	String 	            Hash256                 The identifying hash of the transaction that most recently modified this node
PreviousTxnLgrSeq 	        Number 	            UInt32                  The index of the ledger that contains the transaction that most recently modified this node
TableEntries        	    Array 	            Array                   An array of TableEntry objects
=====================  ==================  ======================  ============================================================================================================================================

- TableEntry Object定义如下:

=====================  ==================  ======================  ============================================================================================================================================
Field    	           JSON Type            Internal Type   	   Description
=====================  ==================  ======================  ============================================================================================================================================
TableName                String              	Blob 	                数据库表名
NameInDB 	         String              	Hash128 	        对应底层数据库中实际的表名,（LedgerSequence+OwnerAccountID+表名）
CreateLgrSeq 	         Number              	UInt32 	                表创建交易所在ledger的前一个ledger 序列号
CreatedLedgerHash 	 Number              	Hash256              	表创建交易所在ledger的前一个ledger HASH
CreatedTxnHash 	         Number              	Hash256 	        表创建交易HASH
TxnLgrSeq 	         Number              	UInt32 	                本交易的ledger序列号
TxnLgrHash 	         String              	Hash256 	        本次交易的ledger HASH
PreviousTxnLgrSeq 	 Number              	UInt32 	                上次交易的ledger序列号
PreviousTxnLgrHash     	 String              	Hash256 	        上次交易的ledger HASH
TxCheckHash 	         String              	Hash256 	        上次TxCheckHash+本次交易raw字段，再进行哈希
Users 	                 Array               	Array                	授权用户列表
Users[] 	         Object              	Object 	                An association of an address and roles
Users[].Account 	 String              	AccountID            	被授予对应权限的ChainSQL账户地址
Users[].Flags 	         Number              	UInt32 	                A bit-map of boolean flags enabled for this account. 用户拥有的权限flags
Users[].Token          	 String              	Blob 	                Cipher encrypted by this user's publickey. 对Raw字段加解密密码使用用户公钥加密后的密文
=====================  ==================  ======================  ============================================================================================================================================

- Table Role Flags定义如下:

=================  ==================  ==============================================================
Flag Name    	         Hex Value   	 Decimal Value
=================  ==================  ==============================================================
lsfSelect           0x00010000       	    65536
lsfInsert           0x00020000 	            131072
lsfUpdate           0x00040000 	            262144
lsfDelete 	    0x00080000 	            524288
lsfExecute 	    0x00100000 	            1048576
=================  ==================  ==============================================================


-----------------------------------------------
5.3 自定义的Transactions：TableListSet
-----------------------------------------------
- TableListSet Transactions 对应创建表、删除表、表改名、表授权、表重建等操作，只有表的创建者可以删除及授权等其它操作

=====================  ==================  ======================  ============================================================================================================================================
Field    	              JSON Type             Internal Type   	    Description
=====================  ==================  ======================  ============================================================================================================================================
Tables                    Array 	        Array                   必填，本次操作涉及到的表名
Table[] 	          Object   	        Object                  必填，表元素
Table[].TableName 	  String   	        Blob                    必填，上层表名
Table[].NameInDB 	  String   	        Hash160 	        选填，实际表名
Table[].TableNewName 	  String   	        Blob 	                选填，如果有则是表改名操作，如果是NULL则是删除表
User 	                  String   	        AccountID 	        选填，被授予对应权限的ChainSQL账户地址
Flags 	                  Number   	        UInt32 	                选填，公用字段，用来记录用户被授予的权限
OpType 	                  Number   	        UInt32           	必填，操作类型， 1：建表，2：删表，3：改表名，10：验证断言，11：授权，12：表重建，13：多链整合
Raw                       String   	        Blob 	                选填，建表/删表的sql或json
TxCheckHash 	          String   	        Hash256 	        选填，strict模式时设置的校验
Token 	                  String   	        Blob 	                选填，建表/授权表用户公钥加密的密文
OperationRule 	          Json Object 	        Blob             	行级控制规则
=====================  ==================  ======================  ============================================================================================================================================


- 建表要填写的字段:         Tables，Raw，Token
- 删表要填写的字段:         Tables，TableNewName=null
- 改名要填写的字段:         Table，TableNewName
- 授权要填写的字段:         Table，User，Flags，Token
- 取消授权要填写的字段:     Table，User，Flags=0


-----------------------------------------------
5.4 自定义的Transactions：SQLStatement
-----------------------------------------------
- SQLStatement Transactions 对应对表的select, insert, delete, update等操作

=====================  ==================  ======================  ============================================================================================================================================
Field    	              JSON Type             Internal Type   	    Description
=====================  ==================  ======================  ============================================================================================================================================
Owner 	                String           	AccountID 	        必填，表的创建者
Tables 	                Array            	Array            	必填，本次操作涉及到的表名
OpType 	                Number           	UInt32 	                必填，操作类型，6:插入记录, 8:更新记录,9:删除记录
AutoFillField 	        String           	Blob             	选填，指定自动填充的字段
Raw 	                String           	Blob 	                必填，select/insert/update/delete的sql或json
TxCheckHash      	String           	Hash256          	选填，strict模式时设置的校验
=====================  ==================  ======================  ============================================================================================================================================


-----------------------------------------------
5.5 自定义的Transactions：SQLTransaction
-----------------------------------------------
- SQLTransaction Transactions 对应对表的事务操作

====================================================  ==================  ======================  ============================================================================================================================================
Field    	                                        JSON Type             Internal Type   	    Description
====================================================  ==================  ======================  ============================================================================================================================================
Statements 	                                        Array            	Array 	            必填，事务操作json数组
Statements[] 	                                        Object           	Object              必填，事务操作json对象
Statements[].Tables 	                                Array            	Array 	            必填，本次事务操作涉及的表名等信息
Statements[].Tables.Table[] 	                        Object           	Object 	            必填，表元素
Statements[].Tables.Table[].TableName 	                String           	Blob          	    必填，上层表名
Statements[].Tables.Table[].NameInDB 	                String           	Hash160 	    选填，实际表名
Statements[].Tables.Table[].TableNewName 	        String           	Blob 	            选填，如果有则是表改名操作，如果是NULL则是删除表
Statements[].User 	                                String          	AccountID 	    选填，被授予对应权限的ChainSQL账户地址
Statements[].Flags 	                                Number          	UInt32              选填，公用字段，用来记录用户被授予的权限
Statements[].OpType                              	Number           	UInt32              必填，操作类型， 1：建表, 2：删表, 3：改表名, 6:插入记录, 8:更新记录, 9:删除记录, 10.验证断言
Statements[].AutoFillField 	                        String          	Blob                选填，指定自动填充的字段
Statements[].Raw 	                                String 	                Blob        	    选填，建表/删表的sql或json
Statements[].TxCheckHash 	                        String 	                Hash256  	    选填，strict模式时设置的校验
====================================================  ==================  ======================  ============================================================================================================================================