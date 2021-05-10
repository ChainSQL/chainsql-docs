分片使用说明
############################

1. 版本获取
****************************

ChainSQL分片\ `源码 <https://gitlab.peersafe.cn/chainsql/chainsqld/-/tree/feature/shard>`_\ 及\ `Release版本 <>`_\ 均可从github开源仓库获取。

.. _分片手册节点配置:

2. 节点配置
****************************

ChainSQL分片版本对节点配置文件进行一些调整。

1. 禁用了配置单元 ``[ips]`` ， ``[ips_fixed]`` ， ``[validators]`` 。
2. 新增了配置单元 ``[shard]`` ， ``[lookup_ips]`` ， ``[lookup_public_keys]`` ， ``[shard_ips]`` ， ``[shard_validators]`` ， ``[committee_ips]`` ， ``[committee_validators]`` ， ``[sync_ips]`` ， ``[lookup_relay_interval]`` 。
3. 修改了配置单元 ``[node_db]`` 。

下面一一对所有新增和修改的配置项进行说明。

[shard]
============================

.. code-block:: ini

    [shard]
    role = shard
    shard_count = 2
    shard_index = 1

* **role** ：配置节点在分片网络中的充当的角色，可取值：lookup，sync，shard，committee，其中lookup与sync可以同时设置，用’,’分隔。
* **shard_count** ：配置分片网络规划的总片区数，不包括委员会片区。
* **shard_index** ：可选，节点角色为shard时，应配置，表示当前节点所属的分片标号，从1开始。

[lookup_ips]
============================

.. code-block:: ini

    [lookup_ips]
    192.168.1.1:5016
    192.168.1.2:5016

* 参考\ :ref:`[ips] <ips>`\ 配置单元。
* 配置分片网络中所有或本节点要与其建立P2P关系的Lookup节点，每行配置一个。

[lookup_public_keys]
============================

.. code-block:: ini

    [lookup_public_keys]
    n94feqNjDCNBUtUWYhnEUhyAhKB6zDa6RDTSosAuaz2TcQbyQydW
    n9Mwy4qkUaYMKjAuwLxEjVHjfmSLRc4iNfRodxeohnM5BRAz7gjt

* 参考\ :ref:`[validators] <validators>`\ 配置单元。
* 配置所有受信任的Lookup节点的公钥。

.. _shard_ips:

[shard_ips]
============================

.. code-block:: ini

    [shard_ips]
    192.168.1.3:5016,192.168.1.4:5016
    192.168.1.5:5016,192.168.1.6:5016

* 参考\ :ref:`[ips] <ips>`\ 配置单元。
* 配置分片网络中所有角色为分片的节点。
* 每行可配置多个，用逗号分隔，每行列出一个片区的所有节点。
* 第一行配置片区1中的节点，第二行配置片区2中的节点，以此类推。

[shard_validators]
============================

.. code-block:: ini

    [shard_validators]
    n9Kgr6tqprYF9scLnFBitpDFN2ztMYsQCHbRtRB8RZ6VuRVFvdWA,n9J5TxDkECMCfP4LCAkzsbsUa51MX1vHXMfzW2AAe5jgrvv6s8rd
    n9Kk2bqFysmoV9jR37Mj6msk9eGEsWgcms7JEQEEVYM6QgduZFAS,n9JgeybK9GRBV4NJ46YS1y8MSEf49gbBaW6CFDxFHka24wVqvsGg

* 参考\ :ref:`[validators] <validators>`\ 配置单元和\ :ref:`[shard_ips] <shard_ips>`\ 配置单元。
* 配置所有受信任的分片节点的公钥。

[committee_ips]
============================

.. code-block:: ini

    [committee_ips]
    192.168.1.7:5016
    192.168.1.8:5016

* 参考\ :ref:`[ips] <ips>`\ 配置单元。
* 配置分片网络中所有或本节点要与其建立P2P关系的Committee节点，每行配置一个。

[committee_validators]
============================

.. code-block:: ini

    [committee_validators]
    n9LuxesD97vZ7shE7euRnQ54TfybQRCJ1kHrBE2LWwrVp28Dq5DL
    n94M2CfgMsqQ8yR7DJAG3cH6ycoFk3AkESu3ANQsyf7M4U6rjbPj

* 参考\ :ref:`[validators] <validators>`\ 配置单元。
* 配置所有受信任的Committee节点的公钥。

[sync_ips]
============================

.. code-block:: ini

    192.168.1.9:5016
    192.168.1.10:5016

* 可选配置，如果分片网络中存在独立的sync角色节点，则需进行配置。
* 参考\ :ref:`[ips] <ips>`\ 配置单元。
* 配置分片网络中所有或本节点要与其建立P2P关系的Sync节点，每行配置一个。

[lookup_relay_interval]
============================

.. code-block:: ini

    [lookup_relay_interval]
    500

* 可选配置，对Lookup节点生效，配置Lookup节点对交易进行分片、打包、并转发到分片节点的频率。
* 单位：毫秒，默认值：500。

[node_db]
============================

.. code-block:: ini

    [node_db]
    sync_wait = 1

* 可选配置，在原\ :ref:`[node_db] <node_db>`\ 配置单元里增加了一个配置项 **sync_wait** 。
* **sync_wait** 配置为1时，确保区块在KV数据库持久化完成后再发布，只对RocksDB有效（NuDB后端只支持同步持久化）。
* 可选值：0或1，默认值：0。

.. _分片手册UNLServer:

3. UNL Server
****************************

利用UNL Server可对分片网络中的验证节点公钥列表进行初始配置，也可在区块链网络运行过程中对其进行热修改。

UNL Server每项信任列表项中增加分片相关的角色信息以及分片节点所属片区号。示例如下：

.. code-block:: json

    {
        "sequence": 2,
        "expiration": "2020-02-07 16:50:00",
        "validators": [
            {
                "validation_public_key": "0306891C7BA2C276B4180AA4E8E5DECF88F06686B3A9AFB6709064393D51654325",
                "shard_role": "lookup,sync"
            },
            {
                "validation_public_key": "02028663F808393964CD4CE236056ECEF4CED217F9CE28F43537B0FDE14F62619E",
                "shard_role": "shard ",
                "shard_index": "1 "
            },
            {
                "validation_public_key": "03A7E886143819F704238FBE0CE2431BC36D392E81036D39BBD21063FDCD558D3E",
                "shard_role": "committee"
            }
        ],
    }

.. _分片手册客户端:

4. 客户端
****************************

ChainSQL分片版本对客户端提交交易做了以下修改：

1. 客户端（SDK或RPC客户端）必须自己维护链上账户的Sequence。
2. 对于分片中\ :ref:`调用智能合约的交易 <分片设计智能合约>`\ ，交易体中可增加 ``Priority`` 字段。当交易应用失败并返回 ``tefCONTRACT_DIFF_SHARD`` 错误码时，使用此字段用来指定交易所属片区为委员会片区，避免交易应用失败。

目前支持分片的SDK有Java SDK。

下面分别对客户端在使用Sign-and-submit模式以及Java SDK的使用针对分片的修改做说明。

4.1 Sign-and-submit模式
============================

* 提交交易时，交易的tx_json对象中必须包含发起交易账户的Sequence字段。否则交易提交不成功，返回 ``rpcINVALID_PARAMS`` 错误码。

下面是一个正确的tx_json示例：

.. code-block:: json

    {
        "method": "submit",
        "params": [
            {
                "offline": false,
                "secret": "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb",
                "tx_json": {
                    "Account": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
                    "Sequence": 1,
                    "Amount": "1000000000000",
                    "Destination": "z9PyRF5WDzEpiBmFr9G4W5DiDcBXMV7kKq",
                    "TransactionType": "Payment"
                },
                "fee_mult_max": 1000
            }
        ]
    }

* 交易体中可设置 ``Priority`` ，指定交易所属片区为委员会片区。只对调用智能合约的交易类型生效。

当设置 ``Priority`` 值为1时，显示设置交易被划分到委员会分片。

.. code-block:: json

    {
        "method": "submit",
        "params": [
            {
                "offline": false,
                "secret": "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb",
                "tx_json": {
                    "Account": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
                    "Sequence": 15,
                    "ContractAddress": "zx1eNuwaP8YGygi1VPNaZP6ZAJhq3m31Kf",
                    "ContractData": "2DEF54E00000000000000000000000000000000000000000000000000000000000003039",
                    "Gas": 30000000,
                    "Priority": 1,
                    "ContractOpType": 2,
                    "TransactionType": "Contract"
                },
                "fee_mult_max": 1000
            }
        ]
    }

.. _分片手册JavaSDK:

4.2 Java SDK
============================

支持分片的\ `Java SDK源码 <https://gitlab.peersafe.cn/chainsql/java-chainsql-api/-/tree/feature/trackSequence>`_\ 和\ `Realease版本 <>`_\ 均可从github开源仓库获取。

* Java SDK已添加对链上账户Sequence的自动维护功能，不需要开发者自行维护与关心。
* Java SDK增加了对合约交易设置 ``Priority`` 的接口。

setPriority
----------------------------

.. code-block:: java

    public void setPriority(boolean flag);

**参数**

1. ``flag`` - ``boolean``：``true`` 表示设置交易体中 ``Priority`` 字段为1， ``false`` 表示表示清除(不设置)交易体中的 ``Priority`` 字段。

**示例**

.. code-block:: java

    // 加载合约
    Greeter contract = Greeter.load(c, "zKotgrRHyoc7dywd7vf6LgFBXnv3K66rEg", Contract.GAS_LIMIT);

    // 设置合约交易Priority
    contract.setPriority(true);

    // 调用合约
    JSONObject ret = contract.newGreeting("Well hello again3").submit(SyncCond.validate_success);
    System.out.println(ret);

5. Load启动
****************************

ChainSQL分片中只有Lookup角色节点为对接外部客户端的节点。Lookup接收客户端交易，处理订阅，并发布区块和交易。同时，在账本\ :ref:`持久化 <分片设计持久化>`\ 方面，只有Lookup节点和Sync节点对链上的原始交易详情和Meta Data进行了持久化。

因此，当运行中的分片网络由于运维或故障缘故需要重启整个链时，重启过程需遵循以下步骤，来保证分片网络的可用性和正确性。

1. 先 ``--load`` 启动分片网络中已完成持久化，并且区块高度最高的Lookup节点，需事先在索引数据库SQLite中查明。
2. 再 ``--ledger <number>`` 启动一个包含最高区块的委员会节点。其中 *number* 为步骤1中Lookup节点的区块高度。
3. 最后，正常启动分片中其它所有节点，等待整个网络完成初始化，并达成共识。

6. 常见问题
****************************

6.1 订阅db_success
============================

在ChainSQL分片中，数据库同步功能以插件式功能的设计思想置于独立的节点角色\ :ref:`Sync节点 <分片设计节点角色>`\ 中。而与客户端对接（接收交易、发布订阅）的节点角色又只有\ :ref:`Lookup节点 <分片设计节点角色>`\ 。这会导致客户端无法从独立的Lookup节点订阅到交易的 ``db_success`` 状态。

.. note::

    在ChainSQL分片网络中使用ChainSQL数据库表功能，在分片的网络规划部署阶段应将Lookup节点同时\ :ref:`配置 <分片手册节点配置>`\ 为Sync节点。

7. 目前缺陷
****************************

目前ChainSQL分片设计及实现已基本能满足高并发、大规模节点部署的应用的场景，但仍存在一些细节功能上的缺陷。在ChainSQL后续的开发计划中，将继续对分片方案及实现进行完善、优化。欢迎大家体验并反馈意见。

7.1 不支持国密算法
============================

ChainSQL从\ :ref:`1.1.4-pop <版本1.1.4-pop>`\ 版本开始支持国密算法，国密算法的代码暂时还没有合入分片版本中。未来分片必然将国密算法代码合入，支持国密算法。

7.2 数据库功能缺陷
============================

================= ===============================================================
功能               缺陷
================= ===============================================================
行级控制           不支持
严格模式           不支持
数据库事务交易      必须\ :ref:`设置NeedVerify <Java-setNeedVerify>`\ 为 ``false``
================= ===============================================================