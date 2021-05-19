.. _共识可插拔使用:

共识可插拔
############################

.. _共识可插拔配置:

1. 节点配置
****************************

ChainSQL3.0对共识模块进行了\ :ref:`抽象设计 <共识可插拔设计>`\ ，实现接入了多种共识算法。目前已经支持的共识算法有\ `RPCA <https://xrpl.org/consensus.html>`_\ 、\ :ref:`POP <PoP共识版本>`\ 和\ `HotStuff <https://arxiv.org/pdf/1803.05069.pdf>`_\ 。可通过节点配置文件对链使用的共识算法类型和共识参数进行配置。

与原\ :ref:`1.x.x-pop <版本1.1.4-pop>`\ 和\ :ref:`2.x.x-shard <版本2.0.0-shard>`\ 版本进行对比，在共识配置方面的差异如下：

1. 禁用了单独对POP共识算法进行配置的配置单元 ``[pconsensus]``\ 。
2. 新增了可选的配置单元 ``[consensus]`` 可对节点使用的共识类型和共识参数进行配置。

.. note::

    如节点不对共识算法类型和共识参数进行配置，节点默认使用POP共识算法及其默认参数。

下面对 ``[consensus]`` 配置单元中支持的配置项进行说明。

.. code:: ini

    [consensus]
    type = pop
    min_block_time = 1000
    max_block_time =1000
    max_txs_per_ledger =10000
    omit_empty_block = true
    time_out = 3000
    init_time = 10
    max_txs_in_pool = 1000000

===================  ========  ======  =========  ===========================================================================================================
配置项                 类型      单位    默认值       说明
===================  ========  ======  =========  ===========================================================================================================
type                  字符串     N/A     POP       共识类型，可选值：RPCA/POP/HOTSTUFF，大小写不敏感
min_block_time        正整数     毫秒    1000      最小区块间隔时间，最小值1000
max_block_time        正整数     毫秒    1000      最大区块间隔时间，最小值1000，且大于或等于min_block_time
max_txs_per_ledger    正整数     N/A     10000     每个区块包含的最大交易数，小于或等于max_txs_in_pool
omit_empty_block      布尔       N/A     true      是否忽略空区块
time_out              正整数     毫秒    3000      对于POP共识，默认值为3000，并且如果配置的值小于或等于max_block_time，则值调整为max_block_time的2倍
time_out              正整数     毫秒    5000      对于HotStuff共识，默认值为5000，并且如果配置的值小于2倍max_block_time加1000，则值调整为2倍max_block_time加1000
init_time             正整数     秒      90        初始化时间，大于或等于2倍time_out
max_txs_in_pool       正整数     N/A     100000    交易池的容量大小
===================  ========  ======  =========  ===========================================================================================================

.. note::

    除了type配置项外，其它配置项均不影响RPCA共识算法

2. 查看共识信息
****************************

节点运行过程中，可通过命令行或者RPC接口查询节点的共识类型、共识参数和共识状态信息。

.. code:: console

    ./chainsqld consensus_info

返回信息的部分字段与配置项一一对应，如下：

.. code:: Json

    {
        "result" : {
            "info" : {
                "initialized" : false,
                "ledger_seq" : 2,
                "new_round" : 1,
                "parms" : {
                    "init_time" : 10,
                    "max_block_time" : 1000,
                    "max_txs_per_ledger" : 10000,
                    "min_block_time" : 1000,
                    "omit_empty_block" : true,
                    "time_out" : 5000
                },
                "proposing" : true,
                "synched" : true,
                "tx_count_in_pool" : 0,
                "tx_pool_capacity" : 100000,
                "type" : "hotstuff",
                "validating" : true
            },
            "status" : "success"
        }
    }
