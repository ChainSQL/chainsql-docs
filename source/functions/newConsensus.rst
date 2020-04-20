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
* 测试环境：``ucloud/aliyun + ubuntu16.04/centos7.1 + 8核16G``
* 测试工具：``jmeter`` + 自定义脚本
* 节点数量：4台共识节点

3.2 测试步骤：
===============
1. 生成一批账户（同一账户无法实现并行发送交易）
2. 根据账户文件生成测试数据（payment或insert交易的签名数据）
3. 配置jmeter执行脚本（1000线程，每个线程循环100次，总共10万笔交易）
4. 启动链（配置数据库）
5. 在链上建表、授权所有人可插入
6. 开启区块及交易数量监控工具
7. 执行jmeter脚本向链上发送交易（4台中的一台接收交易）

3.3 测试结果
====================
* 发送：

::

    Created the tree successfully using GenerateSignTx.jmx
    Starting the test @ Thu Aug 29 15:49:40 CST 2019 (1567064980563)
    Waiting for possible Shutdown/StopTestNow/HeapDump/ThreadDump message on port 4445
    summary +  87871 in 00:00:19 = 4609.3/s Avg:   166 Min:     1 Max:  3311 Err:     2 (0.00%) Active: 887 Started: 1000 Finished: 113
    summary +  12129 in 00:00:05 = 2658.1/s Avg:   129 Min:     1 Max:  1214 Err:     0 (0.00%) Active: 0 Started: 1000 Finished: 1000
    summary = 100000 in 00:00:24 = 4232.3/s Avg:   162 Min:     1 Max:  3311 Err:     2 (0.00%)
    Tidying up ...    @ Thu Aug 29 15:50:04 CST 2019 (1567065004565)
    ... end of run

* 共识(ucloud)：

::

  ledger	time	txn_count	txn_success	txn_failure
  133		1		3562			3562				0
  134		1		4902			4902				0
  135		1		4817			4817				0
  136		1		7670			7670				0
  137		3		10000			10000				0
  138		2		10000			10000				0
  139		2		10000			10000				0
  140		3		10000			10000				0
  141		4		10000			10000				0
  142		2		10000			10000				0
  143		2		10000			10000				0
  144		2		9049			9049				0

* 共识(aliyun 3.1GHz cpu)：

::

  ledger	time	txn_count	txn_success	txn_failure
  187		0		1525			1525				0
  188		1		1866			1866				0
  189		1		2930			2930				0
  190		1		3369			3369				0
  191		1		2714			2714				0
  192		1		4873			4873				0
  193		1		3955			3955				0
  194		1		2843			2843				0
  195		1		3949			3949				0
  196		1		5250			5250				0
  197		1		5530			5530				0
  198		1		4401			4401				0
  199		1		3035			3035				0
  200		1		4404			4404				0
  201		1		4787			4787				0
  202		1		4535			4535				0
  203		1		4476			4476				0
  204		1		6058			6058				0
  205		1		5895			5895				0
  206		1		2748			2748				0
  207		1		1067			1067				0
  208		1		1914			1914				0
  209		1		4377			4377				0
  210		1		5949			5949				0
  211		1		5467			5467				0
  212		1		2073			2073				0