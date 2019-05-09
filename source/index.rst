
.. image:: ../images/logo.png
    :width: 301px
    :alt: ChainSQL logo
    :align: center

ChainSQL
===============
ChainSQL是全球首个基于区块链的数据库应用平台，基于开源项目Ripple建立，由 `众享比特 <http://www.peersafe.cn>`_ 公司开发维护。

版本获取
===============
- `Chainsql节点 <https://github.com/ChainSQL/chainsqld/releases>`_
- :ref:`Java API <Java环境>`
- :ref:`Node.js API <Node环境>`

设计目的
===============
    在不改原有项目整体结构的前提下，在逻辑层与数据层之间加入区块链，使得对数据库的操作记录不可更改、可追溯，并且与传统数据库相关项目对接比较方便。

功能
===============
2016年初开始开发， 目前最新版本是0.30.3，支持功能包括：

1. Ripple原有功能
---------------------

- 网关发行代币
- 代币的托管
- 去中心化交易所

2. 数据库表操作
---------------------
- DDL：建表、表授权、删除表、表重命名
- DML：插入、更新、删除、查询（支持复杂查询）
- 数据库事务：可同时对多张表进行操作

3. 智能合约
------------------------------------------
- 无缝兼容以太坊智能合约（基于solidity版本：``0.4.25-nightly.2018.8.1+commit.21888e24`` ）
- solidity语法中增加了对表操作的指令
- solidity语法中增加对Ripple代币的操作指令（进行中...）

系统架构
===============
    .. image:: ../images/ChainSQL.png
        :width: 622px
        :alt: ChainSQL Framework
        :align: center

.. toctree::
   :maxdepth: 2
   :caption: 入门教程

   tutorial/commonSense
   tutorial/deploy

.. toctree::
    :maxdepth: 2
    :caption: 原理

    theory/tableDesign
    theory/smartContractDesign
    theory/cfg
    theory/amendments

.. toctree::
    :maxdepth: 2
    :caption: 功能介绍

    functions/currency
    functions/tableOperation
    functions/smartContract
    functions/recordLevel
    functions/raw

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
   maintenance/QA
