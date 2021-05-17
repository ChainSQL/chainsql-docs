.. _POP共识算法版本:

PoP共识算法版本
#################

1. 与旧共识版本的不同
**************************

1.1 配置文件配置项
======================

1. 修改的配置项

* ``[validators]``

  要求验证节点间两两互配（在旧共识版本中，验证节点之间不强制要求两两互配）

2. 新增的配置项（**可不配置，有默认值**）

.. _pconsensus配置:

* ``[pconsensus]``

  .. code-block:: cfg

    [pconsensus]
    omit_empty_block=true
    min_block_time=1000
    max_block_time=1000
    max_txs_per_ledger=10000
    max_txs_in_pool=100000
    time_out=3000

  .. list-table::

    * - **配置名**
      - **默认值**
      - **可选值**
      - **释义**
    * - omit_empty_block
      - true
      - true/false
      - 是否忽略空区块。
    * - min_block_time
      - 1000
      - >=1000
      - 区块最小生成时间（单位毫秒，有交易的情况下，最小出块时间）。
    * - max_block_time
      - 1000
      - >=1000 && >= min_block_time
      - 最大区块生成时间（单位毫秒，无交易且允许空区块情况下的出块时间）。
    * - max_txs_per_ledger
      - 10000
      - >0
      - 区块中最大交易数。
    * - max_txs_in_pool
      - 100000
      - >0
      - 交易池中最大交易数。
    * - time_out
      - 3000
      - >=3000
      - 交易集共识超时时间(单位毫秒)。

1.2 弱化交易提交验证
=======================

在新共识中，交易只在交易集共识完成，记入区块时执行一次，在节点收到提交的交易时，不执行交易，只验证下面的内容，验证通过放到交易池：

* 签名
* 账户与签名公钥的一致性
* 账户Sequence
* LastLedgerSequence
* 账户余额是否足够（当交易出现排队时会不准确）

1.3 区块中包含失败交易
==========================

1. 错误交易的行为与之前\ ``tec``\ 错误码一样。

  * 会扣掉交易费用。
  * 会改变账户sequence。

2. 增加 ``ledger_txs`` 接口。

可查询区块中的成功、失败交易数，以及错误交易的hash及错误码，可配合\ `ledger_data <https://xrpl.org/ledger_data.html>`_\ 接口使用。

接口详情可查看\ :ref:`命令行接口 <cmdledger_txs>`\ 、\ :ref:`RPC接口 <rpcledger_txs>`\ 、\ :ref:`JAVA接口 <javaledger_txs>`\ 、\ :ref:`Node.js <jsledger_txs>`\ 接口。

3. 修改交易订阅，交易状态增加\ ``validate_error``\ 。

应答格式：

::

    {
        "error_message": "Table exist and not deleted.",
        "tx_hash": "F4A88D1AD3F2FCA729A73FB874CA0BD1F3EF6CEBDEBAC976A5FE938606870AA2",
        "error": "tefTABLE_EXISTANDNOTDEL",
        "status": "validate_error"
    }

关于订阅交易，可以参考各个模块相关接口：\ :ref:`Websocket接口 <websocket订阅交易>`\ 、\ :ref:`Java接口 <javaAPI订阅交易>`\ 、\ :ref:`Node.js接口 <nodeAPI订阅交易>`\ 。

2. 组网
*****************

1. 与RPCA共识版本一样，支持start模式和load(\ ``--load``\ 或\ ``--ledger``\ )模式。

  * start模式下，等90秒后网络可用。
  * load模式下跟原来一样。

2. server_info中增加字段\ ``server_status``\ ，与\ `0.30.6版本 <https://github.com/ChainSQL/chainsqld/releases/tag/v0.30.6>`_\ 一样都是用\ ``server_status``\ 判断节点当前状态是否可用。

  * ``normal`` 表示当前共识网络状态正常。
  * ``abnormal`` 表示当前共识网络状态异常。


3. 性能测试
********************
3.1 环境
===========
* 测试环境：阿里云高主频计算型实例规格族hfc7ECS

  * CPU: ``Intel(R) Xeon(R) Platinum 8369HC CPU @ 3.3GHz`` ,4核8线程
  * 内存： ``16G``

* 测试工具：``jmeter`` + 自定义脚本
* 节点数量：4台共识节点，一台Jmeter交易发送节点

3.2 测试步骤（模拟实际生产）：
=====================================
1. 生成一批账户（同一账户无法实现并行发送交易）
2. 根据账户文件生成测试数据（payment或insert交易的签名数据）
3. 配置jmeter执行脚本（100线程，每个线程循环10000次，总共100万笔交易）
4. 启动链（配置数据库）
5. 在链上建表、授权所有人可插入
6. 开启区块及交易数量监控工具（订阅发布的区块）
7. 执行jmeter脚本向链上发送交易（4台中的一台接收交易）

3.3 测试结果
====================
* 发送

.. code-block:: console

    Creating summariser <summary>
    Created the tree successfully using testChainsql.jmx
    Starting standalone test @ Tue Apr 27 14:09:06 CST 2021 (1619503746683)
    Waiting for possible Shutdown/StopTestNow/HeapDump/ThreadDump message on port 4445
    summary + 186116 in 00:00:23 = 8106.8/s Avg:    10 Min:     0 Max:   262 Err:     0 (0.00%) Active: 100 Started: 100 Finished: 0
    summary + 219767 in 00:00:30 = 7324.6/s Avg:    13 Min:     1 Max:   330 Err:     0 (0.00%) Active: 100 Started: 100 Finished: 0
    summary = 405883 in 00:00:53 = 7663.7/s Avg:    12 Min:     0 Max:   330 Err:     0 (0.00%)
    summary + 206505 in 00:00:30 = 6884.4/s Avg:    14 Min:     1 Max:   574 Err:     0 (0.00%) Active: 100 Started: 100 Finished: 0
    summary = 612388 in 00:01:23 = 7381.9/s Avg:    12 Min:     0 Max:   574 Err:     0 (0.00%)
    summary + 201322 in 00:00:30 = 6663.4/s Avg:    14 Min:     0 Max:   393 Err:     0 (0.00%) Active: 100 Started: 100 Finished: 0
    summary = 813710 in 00:01:53 = 7190.1/s Avg:    13 Min:     0 Max:   574 Err:     0 (0.00%)
    summary + 186290 in 00:00:29 = 6331.2/s Avg:    14 Min:     0 Max:   518 Err:     0 (0.00%) Active: 0 Started: 100 Finished: 100
    summary = 1000000 in 00:02:23 = 7012.9/s Avg:    13 Min:     0 Max:   574 Err:     0 (0.00%)
    Tidying up ...    @ Tue Apr 27 14:11:29 CST 2021 (1619503889637)
    ... end of run


* 出块及包含的交易量

.. code-block:: console

    root@iZ0jl0b8uobbm4a6tpq55aZ:~/testChainsql# sh testSubLedger.sh 
    Apr 27, 2021 2:08:48 PM com.peersafe.base.client.Client log
    INFO: Connecting to ws://172.16.144.180:6006
    connect success
    ledger index:140,ledger time:2,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:141,ledger time:2,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:142,ledger time:4,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:143,ledger time:2,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:144,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:145,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:146,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:147,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:148,ledger time:2,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:149,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:150,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:151,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:152,ledger time:2,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:153,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:154,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:155,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:156,ledger time:2,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:157,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:158,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:159,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:160,ledger time:2,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:161,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:162,ledger time:4,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:163,ledger time:2,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:164,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:165,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:166,ledger time:2,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:167,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:168,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:169,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:170,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:171,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:172,ledger time:2,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:173,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:174,ledger time:4,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:175,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:176,ledger time:2,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:177,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:178,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:179,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:180,ledger time:4,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:181,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:182,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:183,ledger time:2,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:184,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:185,ledger time:2,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:186,ledger time:5,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:187,ledger time:2,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:188,ledger time:3,txn_count:20000,txn_success:20000,txn_failure:0
    ledger index:189,ledger time:2,txn_count:20000,txn_success:20000,txn_failure:0

*  Tps统计

.. code-block:: console

    root@iZ0jl0b8uobbm4a6tpq55aZ:~/testChainsql# sh calc_tps.sh 
    ledger time total = 141
    txn_count = 1000000
    txn_success = 1000000
    txn_failure = 0
    每秒落账交易的TPS = 7092.19/s