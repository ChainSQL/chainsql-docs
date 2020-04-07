PoP共识算法
#################

配置文件配置项
*****************

1. 修改的配置项

* ``[validators]``

  要求验证节点间两两互配。

2. 新增的配置项

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

弱化交易提交验证
*****************

验证内容只包括：

* 签名
* 账户与签名公钥的一致性
* 账户Sequence
* LastLedgerSequence
* 账户余额是否足够（当交易出现排队时会不准确）

区块中包含失败交易
************************

1. 错误交易的行为与之前\ ``tec``\ 错误码一样。

  * 会扣掉交易费用。
  * 会改变账户sequence。

2. 增加 ``ledger_txs`` 接口。

可查询区块中的成功、失败交易数，以及错误交易的hash及错误码，可配合 ``ledger_data`` 接口使用。

接口详情可查看\ :ref:`命令行接口 <cmdledger_txs>`\ 、\ :ref:`RPC接口 <rpcledger_txs>`\ 、\ :ref:`JAVA接口 <javaledger_txs>`\ 、Node.js接口。


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

修改节点组网方式
*****************

1. 与RPCA共识版本一样，支持start模式和load(\ ``--load``\ 或\ ``--ledger``\ )模式。

  * start模式下，等90秒后网络可用。
  * load模式下跟原来一样。

2. server_info中增加字段\ ``server_status``\ ，与\ `0.30.6版本 <https://github.com/ChainSQL/chainsqld/releases/tag/v0.30.6>`_\ 一样都是用\ ``server_status``\ 判断节点当前状态是否可用。

  * ``normal`` 表示当前共识网络状态正常。
  * ``abnormal`` 表示当前共识网络状态异常。