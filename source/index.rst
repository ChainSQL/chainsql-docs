
.. image:: ../images/logo.png
    :width: 301px
    :alt: ChainSQL logo
    :align: center

ChainSQL
===============
ChainSQL是全球首个基于区块链的数据库应用平台，由 `众享比特 <http://www.peersafe.cn>`_ 公司开发维护。

设计目的
===============
    在不改原有项目整体结构的前提下，在逻辑层与数据层之间加入区块链，使得对数据库的操作记录不可更改、可追溯，并且与传统数据库相关项目对接比较方便。

功能
===============
支持功能包括：

1. 基本功能
---------------------

- 网关配置数字资产
- 数字资产的托管
- 去中心化交易所

2. 数据库表操作
---------------------
- DDL：建表、表授权、删除表、表重命名
- DML：插入、更新、删除、查询（支持复杂查询）
- 数据库事务：可同时对多张表进行操作

3. 智能合约
------------------------------------------
- 无缝兼容EVM智能合约（基于solidity版本：``0.4.25-nightly.2018.8.1+commit.21888e24`` ）
- solidity语法中增加了对表操作的指令
- solidity语法中增加对数字资产的操作指令
- 支持wasm智能合约虚拟机

系统架构
===============
    .. image:: ../images/ChainSQL.png
        :width: 622px
        :alt: ChainSQL Framework
        :align: center

版本获取
===============
- `Chainsql节点 <https://github.com/ChainSQL/chainsqld/releases>`_
- :ref:`Java API <Java环境>`
- :ref:`Node.js API <Node环境>`

版本变化
===============
详细版本变化请参考 `github <https://github.com/ChainSQL/chainsqld/releases>`_ ，这里只列出自0.30.3版本开始的主要变化：

3.4.5
---------------------
- 实现以太坊布隆过滤器，及相关Web3接口
- 增加新特性： ``BloomFilter`` ，特性生效时会在区块头中增加Bloom字段，用于存储布隆值
- 增加预编译合约: ``ToolsPrecompiled``
- 新特性生效时间由2周改为12小时
- 共识优化：当出现2v2（最新共识过的区块号不同导致无法达成共识）的情况时，增加拉取最新区块投票的方式以达到共识
- Bug修复

3.3.1
---------------------
- 增加实名认证功能，实名认证账户才可以将系统币转出
- 增加 ``GasPriceCompress`` 特性，特性启用时，最终消耗的手续费为之前的千分之一
- 增加多个Web3接口实现，增加 :ref:`[eth] <eth>` 配置用于配置以太坊本地签名账户私钥
- fullbelow缓存优化：没有区块同步任务时，每一分钟请求一次最新区块，以保持fullbelow缓存不被全部释放
- 修复POP共识Bug：同步区块切换前要检查与前面共识过的区块是否是一条链的
- 修复以太坊交易哈希计算方式
- 修复区块同步Bug：会导致区块同步不完整，导致交易无法从交易池移除
- 已有 `chainID` 状态的链， ``eth_chainId`` 返回现有 `chainID` 的后2个字节转成的整数
- 修复Web3应用调用过程中的bug

3.2.0
---------------------
- 支持wasm合约虚拟机
- 支持 ``Prometheus`` 对节点的监控，参考配置 :ref:`[prometheus] <prometheus>` 
- 增加peer连接证书准入机制，参考配置 :ref:`[peer_x509_root_path] <peer_x509_root_path>` 及 :ref:`[peer_x509_cred_path] <peer_x509_cred_path>`
- 支持与sdk之间的ssl连接，支持国密与非国密证书，参考配置 :ref:`[server] <server>` 及 :ref:`[trusted_ca_list] <trusted_ca_list>`
- 支持用户证书吊销功能
- 超级管理员治理功能，参考配置 :ref:`[govenance] <govenance>`
- 增加删除子链功能
- 增加对子链单独执行stop与start操作的功能
- 增加新特性： ``ContractStorage,TableGrant,PromethSLEHideInMeta``

3.1.0
---------------------
- 增加tx_result接口，只获取交易在链上的共识结果
- 增加 :ref:`ledger_tx_tables<cfgledger_tx_tables>`\ 配置项，可配置节点是否写Transactions表
- 修改 :ref:`sync_tables<表同步设置>`\ 配置项，支持同步某账户下所有表
- 新增一张表一个SLE的特性 ``TableSLEChange`` ，新特性下一个账户下不再有只能建100张表的限制
- 新增预编译合约功能，以及表的预编译合约接口
- 增加对交易时间、交易所在区块号自填充字段的支持，详见 ``JavaSdk`` 或 ``NodeSDK`` 的 ``insert/update`` 接口
- 智能合约交易，合约执行过程中的异常可以在交易结果中返回
- 建表时对多个字段创建索引，之前是创建联合索引，现改为为每个加 `index` 关键字的字段单独创建一个索引
- 共识算法优化

3.0.0
---------------------
- 支持 `主子链架构 <theory/schema.html>`_
- 支持 `共识可插拔 <theory/consensus.html>`_
- 节点启动自动加载最新区块，不需要再用 ``--load`` 启动


.. _版本2.0.0-shard:

2.0.0-shard
-------------------
- 支持\ :ref:`分片共识 <分片设计>`
- :ref:`Java SDK <分片手册JavaSDK>`\ 版本更新，支持分片，但与2.0.0-shard之前的节点不兼容

.. _版本1.1.4-pop:

1.1.4-pop
-------------------
- :ref:`国密算法支持 <密码算法支持>`
- 预编译合约
- 单条交易最大500KB
- 表交易中新增 LONGTEXT 类型字段
- 新添配置选项 :ref:`[crypto_alg] <crypto_alg>` 
- 配置选项修改 :ref:`validation_create <validation_create>`  :ref:`wallet_propose <wallet_propose>`
- Node.js SDK  更新到版本 0.70.1
- JAVA  SDK 更新到版本 1.5.7

1.0.2-pop
-------------------
- 替换智能合约虚拟机执行器，由原先的evmjit替换为Interpreter，兼容新智能合约字节码
- 防止SQL注入
- raw字段查询条件支持null
- 增加字段sfTxsHashFillField，实现表交易的历史哈希信息记录
- 提高内存的释放速度
- Bug修复

1.0.1-pop
-------------------
- 使用新共识算法，将tps提高到4000-6000，并且默认不生成空区块，减小空区块对存储的浪费
- 新共识原理参考：`POP共识算法原理 <theory/newConsensus.html>`_
- 新共识使用参考：`POP共识版本使用 <functions/newConsensus.html>`_
- 兼容0.30.6版本上的所有功能

------------

0.30.6
--------------------
- 调整区块缓存时间以及数量的默认值 
- 新增加命令行接口:  :ref:`ledger_objects <LedgerObjects>`   , :ref:`node_size <NodeSize>` 
- 新添配置选项  :ref:`ledger_acquire <LedgerAcquire>`   ,   :ref:`missing_hashes <MissingHashes>`
- 其它


0.30.5
--------------------
- 新增 :ref:`CA功能  <CAFeatures>` 
- 新增API接口 useCert。 :ref:`java  <UseCertJava>` :ref:`nodejs <UseCertNodeJS>`
- 新添配置选项 :ref:`x509_crt_path <X509CrtPath>`   , :ref:`ca_certs_keys <CACertsKeys>`  , :ref:`ca_certs_sites <CACertsSites>`
- 新增1分钟空区块特性 :ref:`DecreaseStorage  <DecreaseStorage>` 
- 其它

0.30.4
--------------------
- 智能合约添加数字资产接口,支持通过智能合约发数字资产
- 表相关交易费用通过配置项可进行修改 :ref:`drops_per_byte <DropsPerByte>`
- 一次查询条数上限可配置 :ref:`select_limit <SelectLimit>`
- 其它


0.30.3
---------------------
- 增加智能合约功能，并添加表操作指令
- 增加接口：``table_auth``，``r_get_sql_user``，``r_get_sql_admin`` 
- ``r_get`` 查询接口请求中需要附加账户私钥的签名，以更准确地验证用户查询权限
- 查询接口默认一次最多只能查询200条记录
- 其它修改

.. toctree::
   :maxdepth: 2
   :caption: 入门教程

   tutorial/commonSense
   tutorial/deploy

.. toctree::
   :maxdepth: 2
   :caption: 快速开始

   quickstart/normal_deploy
   quickstart/docker_deploy
   quickstart/common_use

.. toctree::
    :maxdepth: 2
    :caption: 设计文档

    theory/tableDesign
    theory/smartContractDesign
    theory/amendments
    theory/popConsensus
    theory/cryptoAlgorithm
    theory/shard
    theory/consensus
    theory/schema

.. toctree::
    :maxdepth: 2
    :caption: 使用说明

    functions/cfg
    functions/currency
    functions/tableOperation
    functions/smartContract
    functions/recordLevel
    functions/raw
    functions/ca
    functions/popConsensus
    functions/shard
    functions/consensus
    functions/schema
    functions/web3

.. toctree::
   :maxdepth: 2
   :caption: 交互

   interface/cmdLine
   interface/rpc
   interface/websocket
   interface/javaAPI
   interface/nodeAPI

.. toctree::
   :maxdepth: 1
   :caption: 运维相关

   maintenance/errCode
   maintenance/FAQ
