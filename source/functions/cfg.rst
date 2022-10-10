配置文件详解
===============

.. _配置文件:

简介
---------
- ``.cfg`` 是chainsqld节点的配置文件，文件名默认为 chainsqld.cfg
- 文档中以 ``#`` 开头的内容是注释
- 文档中以 ``[]`` 修饰一个配置项单元的名称
- 文档中需要空格的地方只允许一个空格，多于一个空格会识别不了

配置文件示例
----------------

.. code-block:: bash

    #端口配置列表
    [server]
    port_rpc_admin_local
    port_peer
    port_ws_admin_local

    #http端口配置
    [port_rpc_admin_local]
    port = 5005
    ip = 0.0.0.0
    admin = 127.0.0.1
    protocol = http

    #peer端口配置，用于p2p节点发现
    [port_peer]
    port = 5126
    ip = 0.0.0.0
    protocol = peer

    #websocket端口配置
    [port_ws_admin_local]
    port = 6006
    ip = 0.0.0.0
    admin = 127.0.0.1
    protocol = ws


    #-------------------------------------------------------------------------------
    # 缓存级别
    [node_size]
    medium

    # 区块数据存储配置，windows下用 ``NuDB`` ，Linux/Mac下用 ``RocksDB`` 或 ``NuDB`` 
    [node_db]
    type=RocksDB
    path=./rocksdb
    open_files=2000
    filter_bits=12
    cache_mb=256
    file_size_mb=8
    file_size_mult=2

    #是否全节点
    [ledger_history]
    full

    #sqlite数据库（存储区块头数据，交易概要数据）
    [database_path]
    ./db

    # This needs to be an absolute directory reference, not a relative one.
    # Modify this value as required.
    [debug_logfile]
    ./debug.log

    #时间服务器，用于不同节点单时间同步
    [sntp_servers]
    time.windows.com
    time.apple.com
    time.nist.gov
    pool.ntp.org

    # 要连接的其它节点的Ip及端口
    [ips]
    127.0.0.1 5127
    127.0.0.1 5128

    # 信任节点列表（信任节点的公钥列表）
    [validators]
    n94ngNasveyfF2KttLuNni6nHPcUtw1Se3969nUginy8cf2Kzb4Z
    n9LacsFGc9VrpAXZidEeAipLchuC2r7wPV243ugR2KDA4En818sM

    # 本节点私钥（如不配置，不参与共识）
    [validation_seed]
    xpvEo9rKgjc6uabELVRymX9scieGC

    # 本节点公钥
    [validation_public_key]
    n9Ko6z3Ua9ShbTaTdqr1x457vaHzAnsrQFLH5uo1Mtu6pgE6

    #日志级别，一般设置为warning级别
    [rpc_startup]
    { "command": "log_level", "severity": "warning" }


    [features]
    DeletableAccounts
    MultiSign
    Flow
    OwnerPaysFee
    fix1781
    DepositAuth
    DepositPreauth
    TableSLEChange          #3.1.0
    ContractStorage         #3.1.2
    TableGrant              #3.1.4
    PromethSLEHideInMeta    #3.1.4

    #禁用某些支持但不需要启用的特性
    [veto_amendments]
    B4E4F5D2D6FB84DF7399960A732309C9FD530EAE5941838160042833625A6076 NegativeUNL


    #chainsql数据库配置，根据自己的机子
    [sync_db]
    type=mysql
    host=localhost
    port=3306
    user=root
    pass=123456
    db=chainsql
    first_storage=0
    charset=utf8

    # 开户自动同步后，节点运行情况下会去自动同步新建的表，开启这个开关，或者使用sync_tables标签的配置，否则无法同步表
    [auto_sync]
    1
    # 开启prometheus监控
    [prometheus]
    port = 7007

版本变化
----------------
    - 3.2.0 版本增加配置项 ``[prometheus]``, ``[govenance]`` 等
    - 3.1.0 版本增加配置项 :ref:`cfgledger_tx_tables`
    - 3.0.0 版本增加配置项 :ref:`schema <Schema>`, :ref:`consensus <Consensus>`
    - 1.0.1-pop版本之后，新添配置选项 :ref:`crypto_alg <crypto_alg>`
    - 0.30.6版本以后, 新添配置选项   :ref:`ledger_acquire <LedgerAcquire>`   , :ref:`ca_certs_keys <CACertsKeys>`  , :ref:`missing_hashes <MissingHashes>`
    - 0.30.5版本以后，新添加配置选项 :ref:`x509_crt_path <X509CrtPath>`   , :ref:`ca_certs_keys <CACertsKeys>`  , :ref:`ca_certs_sites <CACertsSites>`
    - 0.30.4版本以后，新添加配置选项 :ref:`drops_per_byte <DropsPerByte>`   , :ref:`select_limit <SelectLimit>`

配置项说明
----------------

[auto_sync]
********************
是否开启表自动同步，可设置为0或1，默认为0

.. important:: 

    auto_sync只影响表的创建，如auto_sync为0，在创建新表的交易共识过后，不会在数据库中建表。但是如果表已经存在，这时向表中插入数据，是不受auto_sync值影响的


[ca_certs_keys]
******************************
    配置信任X509证书服务器公钥列表

.. code-block:: bash

    [ca_certs_keys]
    029d1f40fc569fff2a76417008d98936a04417db0758c8ab123dee6dbd08d79398


.. _CACertsSites:

[ca_certs_sites]
******************************
    配置信任X509证书服务器列表

.. code-block:: bash

    [ca_certs_sites]
    http://192.168.29.112:8081/


.. _Consensus:

[consensus]
***************************
    配置共识参数

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

.. _crypto_alg:

[crypto_alg]
******************************
    配置节点组网及共识使用的非对称密码算法和哈希算法
    
    - node_alg_type配置项可选值：gmalg/secp256k1/ed25519;

    没有此配置项时，默认使用secp256k1算法

.. code-block:: bash

    [crypto_alg]
    node_alg_type=gmalg


[database_path]
******************
    sqlite 数据库的存储路径，可以是全路径 ，也可以是相对路径（相对于当前配置文件路径）

[debug_logfile]
******************
    日志文件路径

.. _DropsPerByte:

[drops_per_byte]
******************************
    该选项表示表交易中每字节数据消耗的drops，默认为 976(10^6 /1024)，表示1KB数据消耗1 ZXC，范围为[1,10^6]

.. code-block:: bash

    [voting]
    drops_per_byte = 976

.. important:: 

    为了保证与低版本的API的兼容性，drops_per_byte的默认值为976(10^6 /1024)，表示1KB数据消耗1 ZXC。drops_per_byte不配置或者配置为默认值，保持与老版本API的向下兼容性；如果该值配置为非默认值，那么与老版本的API都不兼容，无法发起表交易。

[features]
**************
    要在节点启动时就在本节点启用的特性，特性的具体介绍参考 :ref:`features <amendments>` ，这里不再赘述

[govenance]
**************
    治理相关配置，目前支持配置超级管理员地址、是否默认开启账户授权

.. code-block:: bash

    [govenance]
    admin=xxx
    default_authority_enabled=0

.. _ips:

[ips]
******************
    | 要连接的其它节点的Ip及端口，一行只允许出现一个ipv4地址及端口
    | 配置的端口为要连接节点的peer协议端口

[ledger_history]
*****************
- 节点要维护的最小历史区块数量
- 若不想维护历史区块，则设置为 ``none``，若想维护全部历史区块（全节点），则设置为 ``full`` 还可以设置为一个数字
- 默认值为256，如果 [node_db]中有 ``online_delete`` 配置项，[ledger_history] 的值必须 <= ``online_delete`` 的值

**配置非全节点的示例** :

.. code-block:: bash

    [node_db]
    type=RocksDB
    path=/data/chainsql/db1/rocksdb
    open_files=2000
    filter_bits=12
    cache_mb=256
    file_size_mb=8
    file_size_mult=2
    online_delete=2000
    advisory_delete=0

    #[ledger_history]
    #full

.. _LedgerAcquire:

[ledger_acquire]
******************************
    同步区块相关的配置

    - skip_blocks 表示同步区块时要跳过的区块，也就是缺失的区块，用逗号分隔多个缺失的域。示例：skip_blocks=5000-6000,8000,9000-10000

.. code-block:: bash

    [ledger_acquire]
    skip_blocks=5000-6000,8000,9000-10000

.. _cfgledger_tx_tables:

[ledger_tx_tables] 
******************************
    ``transaction.db`` 存储交易相关配置，``1.1.5-pop`` 及以上版本支持

    - ``use_tx_tables`` 是否向 ``transaction.db`` 中存储交易信息，配置为0表示不存储，节点不能对外提供查询交易的服务，默认值为1
    - ``use_trace_table`` 是否使用 ``TraceTransactions`` 表(提供查询上一个，下一个交易功能，以及记录与表/合约相关的交易有哪些)，默认值为1

.. code-block:: bash

    [ledger_tx_tables] 
    use_tx_tables = 1  
    use_trace_table = 1 

.. _MissingHashes:

[missing_hashes]
******************************
    手动配置节点获取不到区块哈希的区块，每一行配置一个对应的区块号和区块哈希，用冒号分隔区块号和区块哈希

.. code-block:: bash

    [missing_hashes]
    8999:<hash of ledger 8999>
    7999:<hash of ledger 7999>


[node_size]
**************
    | 缓存大小，可设置的值有 "tiny", "small", "medium", "large", "huge"，我们建议一开始设置一个默认值，如果运行一段时间发现还有内存空余，则将缓存增大
    | 默认值为"tiny"

.. _node_db:

[node_db]
**************
    Chainsql中创建了4个sqlite数据库来存储交易、区块等信息的索引，完整的区块内容存储到node_db配置项中

示例：

.. code-block:: bash

    [node_db]
    type=RocksDB
    path=./rocksdb
    online_delete=2000
    advisory_delete=0

配置项的具体内容：

- ``type`` node_db可选类型有两种，一个是 NuDB（平台兼容，可运行在linux/windows)，一个是 RocksDB（只支持linux）
- ``path`` node_db 可配置绝对路径也可以配置相对路径（相对于当前配置文件）

可选配置项（用于开启非全节点）：

- ``online_delete`` 最小值为256，节点最小维持的区块数量，这个值不能小于 ``ledger_history`` 配置项的值
- ``advisory_delete`` 0为禁用，1为启用。如果启用了，需要调用admin权限接口 ``can_delete`` 来开启区块的在线删除功能

[peer_x509_root_path]
**************************
    配置信任的根证书，用于在建立p2p连接的过程中验证对等节点的子证书

.. code-block:: bash

    [peer_x509_root_path]
    ./ca.cert  

[peer_x509_cred_path]
**************************
    配置节点自身的子证书文件
    
.. code-block:: bash

    [peer_x509_cred_path]
    ./peer1.cert  

[prometheus]
*****************
    Prometheus监控对外提供的http访问端口

.. code-block:: bash

    [prometheus]
    7007

[remote_sync]
******************
表同步是否支持远程同步，默认支持，配置为0表示只走本地同步

.. code-block:: bash

    [remote_sync]
    0

[rpc_startup]
*****************
    节点启动时要执行的JSON-RPC命令，一般用于设置日志级别：

.. code-block:: json

    {
        "command":"log_level",
        "severity":"warning"
    }

日志级别包括：trace, debug, info, warning, error, fatal

.. _Schema:

[schema]
*********************
    子链相关配置，包括子链路径，是否自动同意加入子链，是否只参与子链共识

    示例：

.. code-block:: bash

    [schema]
    schema_path=/mnt/schema
    auto_accept_new_schema = 1
    only_validate_for_schema = 1

配置项说明：

- ``schema_path`` 子链数据存储路径，节点作为子链节点的配置文件，schema_path文件夹示例如下：

.. code-block:: bash

    schema_path/
    ├── 4F26F023FBA6E23CD5FAFBF0751E72EEB306FE944F1BAF9AEB7D8753D5719B15
    └── C6970A8B603D8778783B61C0D445C23D1633CCFAEF0D43E7DBCD1521D34BD7C3
        ├── chainsqld.cfg
    ├── schema_info
        ├── db
        └── rocksdb

其中，schema_info是一个文本文件，用来解决子链文件夹根据 schemaId不易读的问题，schema_info中的内容包括子链名称，建链策略等，示例如下::

    888D24BC38A7DA8CCEBB44DFD985EB3BDD72D110C70F04D031C4E4065D069950
    hello3
    zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh
    1
    0000000000000000000000000000000000000000000000000000000000000000
    n9MzFGhK9uGyxMX8t3i7Z2hqDKhrRcnmm3yQyGsA1DVs9UK423KE 0;n9Mi394bhLAXtqoX2jHBrPXYH2SVmemDSmnUyFNEaFqn9bhxZVfK 0;n9K7QUHEPN81aKpuujkJHsnRQLpHV8AhHxxtRq7qC51mYYyNXVx6 0
    127.0.0.1:5430;127.0.0.1:5431;127.0.0.1:5432


- ``auto_accept_new_schema`` 配置节点是否自动同意加入子链，默认为0。若新建子链交易不是多方签名交易，则需要节点在命令行执行一下 ``schema_accepet`` 命令手动执行接受操作，否则不执行子链的创建，也不参与后面子链的消息处理。
- ``only_validate_for_schema`` 是否节点只参与子链共识，默认为 0。 配置为1代表节点不参与主链共识，可以认为是主链上的非共识节点。只属于子链的节点需要配置 only_validate_for_schema = 1 ，不然节点无法同步主链区块，也就无法加入子链。

.. note::

    当节点配置 ``validation_seed`` 去同步主链区块时，因为POP/HotStuff共识的原因会导致同步区块失败，所以增加只参与子链而不参与主链的配置。

.. _SelectLimit:

[select_limit]
********************* 
    配置表查询相关接口返回的最大查询条数，默认值为200

.. code-block:: bash

    [select_limit]
    200

[server]
************

- 端口列表，chainsqld 会查找文件中具有与列表项相同名称的配置项，并用这些配置荐创建监听端口
- 列表中配置项的名称不会影响功配置功能

单个配置项示例如下：

.. code-block:: bash

    [port_rpc_admin_local]
    port = 5005
    ip = 0.0.0.0
    admin = 127.0.0.1
    protocol = http

    [port_wss_admin_local]
    port = 6006
    ip = 0.0.0.0
    admin = 127.0.0.1
    protocol = wss
    ssl_key = /Users/lascion/lcworkspace/cafile/server/server.key
    ssl_cert = /Users/lascion/lcworkspace/cafile/server/server.crt
    ssl_verify = 1


每个配置项包含如下内容：

    - ``port`` 配置端口
    - ``ip`` 哪些ip可以连接这一端口，如果有多个，以逗号（,）进行分隔， ``0.0.0.0`` 代表任意ip可以连接这一端口
    - ``admin`` chainsql中有一些命令（如peers,t_dump,t_audit）只有拥有admin权限的ip才能调用，配置方法与 ip 相同
    - ``protocol`` 协议名称，chainsql中支持协议有 http, https, ws, wss, peer
    - ``ssl_key`` 节点ssl服务证书密钥文件路径
    - ``ssl_cert`` 节点ssl服务证书文件路径
    - ``ssl_verify`` 是否开启双向认证，配置为1开启，0为不开启，默认为0，开启双向认证需配合配置项 ``[trusted_ca_list]`` 使用

[sntp_servers]
******************
    时间服务器，用于p2p节点间时间同步

.. important:: 

    在内网环境中，公网的时间服务器连不上，这时必须配置内网的时间服务器或手动将节点的时间调节一致，不然会出现节点发现不了，或者达不成共识等各种问题。

[sqlite]
******************
    sqlite存储相关配置

- synchronous 磁盘的同步模式，该模式控制积极的 SQLite 如何将数据写入物理存储，可配置值包括 ``off, normal, full, extra``，详见 https://www.runoob.com/sqlite/sqlite-pragma.html

.. code-block:: bash

    [sqlite]
    synchronous = off

.. _SyncDB:

[sync_db]
*****************
    配置同步表相关交易用的数据库，原生支持mysql, sqlite，可通过 mycat 支持其它数据库，示例如下:

.. code-block:: bash
    
    [sync_db]
    type=mysql
    host=localhost
    port=3306
    user=root
    pass=root
    db=chainsql
    first_storage=0 #可选
    charset=utf8 #可选
    unix_socket=/var/lib/mysql/mysql.sock #可选

    [sync_db]
    type=sqlite
    db=chainsql

配置项说明：
    - ``type`` 数据库类型，这里支持sqlite, mysql, mycat，配置mysql等同于mycat
    - ``host`` 连接数据库用的主机, localhost或127.0.0.1表示本机
    - ``port`` 数据库端口
    - ``user`` 登录数据库的用户名
    - ``pass`` 登录数据库的密码
    - ``db``   数据库名称
    - ``first_storage`` 是否开启先入库，0为不开启，1为开启，默认为0
    - ``charset`` 数据库编码
    - ``unix_socket`` 
        | 使用localhost连接时，会默认使用 sock 方式连接，默认sock路径是 /var/run/mysqld/mysqld.sock
        | 在非ubuntu系统中，这个路径是不对的，会导致连接数据库失败，需要用 unix_socket 选项来指定 sock 路径
        | 如果host写为ip（如127.0.0.1）去连接，会使用 tcp 方式连接，就不会有这个问题

[sync_tables]
*********************
    配置要同步的表，详细配置方法参考 :ref:`sync_tables <表同步设置>`，这里与auto_sync的不同在于：

    - auto_sync 配置为1，只能同步新建的表，而 sync_tables 还可以同步之前区块上建的表
    - sync_tables 可中配置同步加密表所用的解密私钥，加密表只有通过sync_tables的配置才可以同步下来
    - sync_tables 可配置各种同步条件，如同步到某个区块，同步到某个时间，跳过某个区块等

[trusted_ca_list]
**********************
    如果开启双向认证，配置用于验证客户端证书的根证书文件路径

.. code-block:: bash

    [trusted_ca_list]
    /Users/lascion/lcworkspace/cafile/root/root.crt

.. _validators:

[validators]
******************
    信任节点列表（信任节点的公钥列表）

[validation_seed]
********************
    本节点私钥（必填项）

[validation_public_key]
***************************
    本节点公钥（国密算法情况下为必填项，其他情况为可选项）


[veto_amendments]
********************
    要禁用的特性，在特性启用前禁用，会给特性的开启投反对票，赞成票低于80%，会导致特性无法开启，详情参考：:ref:`features <amendments>`


[voting]
***************
    配置账户预留费用、对象增加费用、GasPrice设置，示例如下:

.. code-block:: bash

    [voting]
    account_reserve = 10000000
    owner_reserve = 1000000
    gas_price = 10

配置项说明：

    - account_reserve 账户预留费用，指的是激活一个账户所需要的最小系统数字资产（ZXC）数量，也是一个账户的余额要保留的最小值，单位为drop，上面的配置表示账户预留费用为10ZXC
    - owner_reserve 增加一个对象，要增加的预留费用，这里的对象指的是要占用链上存储的对象，如账户与网关之间的trustline，账户新建的表等，上面的值表示每增加一个对象，账户的预留费用要增加1ZXC
    - gas_price 合约在执行过程中需要消耗，这里是 GasPrice的设置，默认值是10drops

.. _X509CrtPath:

[x509_crt_path]
******************************
    配置X509证书文件路径

.. code-block:: bash

    # X509证书文件路径
    [x509_crt_path]
    ./ca1.cert
    ./ca2.cert

.. _CACertsKeys:

[prometheus]
******************
    配置节点监控，可监控节点的区块高度、子链数量、交易数量、失败交易数量、合约数量、合约调用次数、账户数量、节点状态。示例如下:

.. code-block:: bash

    [prometheus]
    port = 7007

配置项说明：
    - port 节点开启的监控端口

注意事项：
    如果当前链没有配置prometheus监控，现在要配置prometheus监控，需注意
        1. 超过2/3共识节点sqlite数据库中存储了交易信息。
        2. 节点启动，查看监控数据是否有值，最好通过get请求http://ip:port/metrics查看每个节点的监控数据（其中ip是节点部署的服务器ip，port是节点开启的监控端口），然后再发送交易。
        3. 如果数据量比较大，可以设置节点启动时间长一点。


