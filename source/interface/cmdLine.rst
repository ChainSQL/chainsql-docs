命令行接口
############################

chainsqld可以通过指定一个API接口方法名来启动RPC客户端模式。
也可以查询服务节点状态和账本数据。
通过配置文件或者命令行参数指定服务节点的地址和端口。

基本命令
*****************************

下面展示几个常用的命令。

.. _validation_create:

validation_create
+++++++++++++++++++++++++++++++

validation_create属于管理命令，通过椭圆曲线加密算法，使用种子生成非对称加密秘钥对（可以做为验证节点的证书）。

语法：

::

    ./chainsqld --conf="./chainsqld-example.cfg"  validation_create [algType] [secret]

.. note::

    选项 --conf执行配置文件路径，如不指定，则使用当前路径下的\ ``chainsqld.cfg``\ 。

参数说明：

.. list-table::

    * - **参数**
      - **类型**
      - **可选值**
      - **描述**
    * - algType
      - 字符串
      - secp256k1/gmalg/ed25519
      - 算法类型，指定生成何种算法的非对称密钥对，不指定默认使用secp256k1，如果使用secret参数，则必须指定。
    * - secret
      - 字符串
      - 无
      - 秘钥种子，如果省略，则使用随机的种子，相同的种子生成相同的证书。

示例：

1. secp256k1：

::

    ./chainsqld validation_create

返回结果：

::

    {
        "validation_key" : "BELA STAG CAB KUDO NUT RENT NET KUDO SIGN MAO COME TUCK",
        "validation_private_key" : "pnZvrMV2LCNZH2GmrWCrE9zbDEgRFVNPKL4UHADcnK4okJgqbit",
        "validation_public_key" : "n9LDNrsWjwFPQu5JeHje2tpfKLdgGP3TwRExtXQNQae3zXtyZEuw",
        "validation_public_key_hex" : "02DBAB0AD21A7DC4AF9F0CC44A190FE49CA9E46BCF57C21CA3AC7FD4B678004E51",
        "validation_seed" : "xnqezWeMcYukumPrwYRA2cW6kmW53"
    }

结果说明：

.. list-table::

    * - **参数**
      - **类型**
      - **描述**
    * - validation_key
      - 字符串
      - 证书私钥 RFC-1751格式。
    * - validation_private_key
      - 字符串
      - 验证私钥 Base58编码，但是不使用，ChainSQL中使用validation_seed。
    * - validation_public_key
      - 字符串
      - 验证公钥 Base58编码。
    * - validation_seed
      - 字符串
      - 公私钥种子Base58编码，ChainSQL中作为验证私钥使用，填写到验证节点配置文件中 。

2. gmalg：

::

    ./chainsqld validation_create gmalg

返回结果：

::

    {
        "validation_private_key" : "pcViNj73TgAXDqfWKx2kh3dz92KEQhsbVceousZDHD8GvSHgy55",
        "validation_public_key" : "pEng3cMSNHzGUjXpDiHouC2GTmsDKVusZBdexVAdMXQWob8pph6f2KJm7S5h3M2mU1pJTL19rhwm9EVAZxrDwPjVSLV35gES"
    }

结果说明：

.. list-table::

    * - **参数**
      - **类型**
      - **描述**
    * - validation_private_key
      - 字符串
      - 验证私钥 Base58编码，填写到验证节点配置文件中validation_seed处。
    * - validation_public_key
      - 字符串
      - 验证公钥 Base58编码。

submit
+++++++++++++++++++++++++++++++

submit属于交易命令，用来提交一个交易。有两种模式：

仅提交模式
===============================

语法：

::

    ./chainsqld submit <tx_blob>

参数说明：

.. list-table::

    * - **参数**
      - **类型**
      - **描述**
    * - tx_blob
      - 字符串
      - 签名后的交易的16进制格式。

示例：

::

    submit 1200002280000000240000000361D4838D7EA4C6800000000000000000000000000055534400000000004B4E9C06F24296074F7BC48F92A97916C6DC5EA968400000000000000A732103AB40A0490F9B7ED8DF29D246BF2D6269820A0EE7742ACDD457BEA7C7D0931EDB74473045022100D184EB4AE5956FF600E7536EE459345C7BBCF097A84CC61A93B9AF7197EDB98702201CEA8009B7BEEBAA2AACC0359B41C427C1C5B550A4CA4B80CF2174AF2D6D5DCE81144B4E9C06F24296074F7BC48F92A97916C6DC5EA983143E9D4A2B8AA0780F682D136F7A56D6724EF53754

签名和提交模式
==================================

签名和提交模式，需要提供账户的私钥。

语法：

::

    ./chainsqld submit <secret> <json>

参数说明：

.. list-table::

    * - **参数**
      - **类型**
      - **描述**
    * - secret
      - 字符串
      - 提交此交易的账户的私钥，用来签名交易。
    * - json
      - 字符串
      - 具体交易的json格式。

示例：

根账户\ ``zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh``\ 向账户\ ``zHYfrrZyyfAMrNgm3akQot6CuSmMM6MLda``\ 转账10000个ZXC。
``xnoPBzXtMeMyMHUVTgbuqAfg1SUTb``\ 是根账户的私钥。

::

    ./chainsqld submit xnoPBzXtMeMyMHUVTgbuqAfg1SUTb '{"Account":"zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh","Amount":"10000000000","Destination":"zHYfrrZyyfAMrNgm3akQot6CuSmMM6MLda","TransactionType":"Payment"}' 

返回结果：

::

    {
        "result": {
            "engine_result": "tesSUCCESS",
            "engine_result_code": 0,
            "engine_result_message": "The transaction was applied. Only final in a validated ledger.",
            "status": "success",
            "tx_blob": "12000022800000002400000002201B0002FA0E614000000005F5E10068400000000000000A73210330E7FC9D56BB25D6893BA3F317AE5BCF33B3291BD63DB32654A313222F7FD0207446304402207E88AA09F5C23A8E7AB29EC9BE5258B0C0A3F751AD8A8C26096FD6F022EC26FF0220112A2140F206679085B0015A2273BB4F802E23BFE64EF58F851F606BF6861ED68114B5F762798A53D543A014CAF8B297CFF8F2F937E88314934CD4FACC490E3DC5152F7C1BAD57EEEE3F9C77",
            "tx_json": {
                "Account": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
                "Amount": "10000000000",
                "Destination": "zHYfrrZyyfAMrNgm3akQot6CuSmMM6MLda",
                "Fee": "10",
                "Flags": 2147483648,
                "LastLedgerSequence": 195086,
                "Sequence": 2,
                "SigningPubKey": "0330E7FC9D56BB25D6893BA3F317AE5BCF33B3291BD63DB32654A313222F7FD020",
                "TransactionType": "Payment",
                "TxnSignature": "304402207E88AA09F5C23A8E7AB29EC9BE5258B0C0A3F751AD8A8C26096FD6F022EC26FF0220112A2140F206679085B0015A2273BB4F802E23BFE64EF58F851F606BF6861ED6",
                "hash": "1A4CA19291EED3A1F7D3FD8218B5FE1FF82D0A93368746A0188285E4CF60F6C1"
            }
        }
    }

server_info
+++++++++++++++++++++++++++++++

server_info属于公共命令，用来查看节点的运行状态。

语法：

::

    ./chainsqld server_info

返回结果示例：

.. code-block:: json

    {
        "id" : 1,
        "result" : {
            "info" : {
                "build_version" : "0.30.3+DEBUG",
                "complete_ledgers" : "1-555",
                "hostid" : "a-virtual-machine",
                "io_latency_ms" : 1,
                "last_close" : {
                    "converge_time_s" : 2,
                    "proposers" : 0
                },
                "load" : {
                    "job_types" : [
                        {
                            "in_progress" : 1,
                            "job_type" : "clientCommand"
                        },
                        {
                            "avg_time" : 1,
                            "job_type" : "acceptLedger",
                            "peak_time" : 3
                        },
                        {
                            "job_type" : "peerCommand",
                            "per_second" : 1
                        }
                    ],
                    "threads" : 6
                },
                "load_factor" : 1,
                "peers" : 1,
                "pubkey_node" : "n9M6KKeKxpP61t63EW6cKKACyhGJyQSokDbA8ipHsZJWCv1dJ3Cq",
                "pubkey_validator" : "n9M15Yj6Jdao2Tnpn8pQe8CeDkFYXid1jJLV9cmHMZngpVCdcPkk",
                "server_state" : "proposing",
                "state_accounting" : {
                    "connected" : {
                        "duration_us" : "72050340",
                        "transitions" : 1
                    },
                    "disconnected" : {
                        "duration_us" : "1191980",
                        "transitions" : 1
                    },
                    "full" : {
                        "duration_us" : "2442353290",
                        "transitions" : 1
                    },
                    "syncing" : {
                        "duration_us" : "0",
                        "transitions" : 0
                    },
                    "tracking" : {
                        "duration_us" : "3",
                        "transitions" : 1
                    }
                },
                "validated_ledger" : {
                    "base_fee_zxc" : 1e-05,
                    "close_time_offset" : 18753,
                    "hash" : "2D1E46FAD9EC8AAD34E8B472F1556A56407528A8F8218081B1F7BB2E0CC4CC5C",
                    "reserve_base_zxc" : 5,
                    "reserve_inc_zxc" : 1,
                    "seq" : 555
                },
                "uptime" : 2428,
                "validation_quorum" : 2,
                "validator_list_expires" : "never"
            },
            "status" : "success"
        }
    }


.. _serverInfo-return:

结果说明：

.. list-table::

    * - **参数**
      - **类型**
      - **描述**
    * - build_version
      - 字符串
      - 节点运行的chainsqld版本。
    * - complete_ledgers
      - 字符串
      - 本节点上完整的区块序列，如果本节点上没有任何完整的区块
        （可能刚接入网络，正在于网络同步），则值为empty。
    * - load
      - 对象
      - 节点当前的负载详情。
    * - peers
      - 整形
      - 与本节点直接连接的其他chainsqld节点的数量。
    * - pubkey_node
      - 字符串
      - 节点与节点通信时，用来验证这个节点的公钥。节点在启动时自动生成的。
    * - pubkey_validator
      - 字符串
      - 该验证节点的公钥，由上面的validation_create命令生成。
    * - server_state
      - 字符串
      - 节点当前状态。
    * - state_accounting
      - 对象
      - 节点在每个状态下的运行时长。
    * - validated_ledger
      - 对象
      - 最近完成共识的区块的信息。
        如果不存在，则会替换为closed_ledger域，表示最近关闭但还没有完成共识的区块信息。
    * - validated_ledger.base_fee_zxc
      - 整形
      - 账本的基本费用，交易、记账以这个数额为基础，单位：zxc。
    * - validated_ledger.close_time_offset
      - 整形
      - 表示账本关闭多长时间了。
    * - validated_ledger.hash
      - 字符串
      - 区块的哈希。  
    * - validated_ledger.reserve_base_zxc
      - 整形
      - 账户必须预留的费用。
    * - validated_ledger.reserve_inc_zxc
      - 整形
      - 账户每增加一个对象（比如一个表）需要额外预留的费用增加这个数值。
    * - validated_ledger.seq
      - 整形
      - 区块的序号。
    * - uptime
      - 整形
      - 节点已运行时长。
    * - validation_quorum
      - 整形
      - 账本达成共识需要的验证数。
    * - validator_list_expires
      - 字符串
      - 新特性，验证节点列表相关的。

.. note::

    若返回结果中，字段\ ``complete_ledgers``\ 类似 "1-10"，则表示chainsqld服务启动成功。

peers
+++++++++++++++++++++++++++++++

peers属于管理命令，查看已连接的其他节点的连接状态和同步状态。

语法：

::

    ./chainsqld peers

返回结果示例：

.. code-block:: json

    {
        "id" : 1,
        "result" : {
            "cluster" : {},
            "peers" : [
                {
                    "address" : "127.0.0.1:5115",
                    "complete_ledgers" : "18850253 - 18851277",
                    "latency" : 0,
                    "ledger" : "5724E7C9B0E7B9E6D7F359A15B260216D896968C0BD782B94F423B10AE0B59FB",
                    "load" : 152,
                    "public_key" : "n9M6KKeKxpP61t63EW6cKKACyhGJyQSokDbA8ipHsZJWCv1dJ3Cq",
                    "uptime" : 4195,
                    "version" : "chainsqld-0.30.3+DEBUG"
                }
            ],
            "status" : "success"
        }
    }

结果说明：

.. list-table::

    * - **参数**
      - **类型**
      - **描述**
    * - cluster
      - 对象
      - 如果配置了集群，则返回集群中其他节点的信息。
    * - peers
      - 数组
      - 已连接的其他节点的连接状态和同步状态。
    * - address
      - 字符串
      - 对端节点与本节点连接使用的IP地址和端口号。
    * - complete_ledgers
      - 字符串
      - 对端节点中有哪些完整的账本。
    * - latency
      - 整数
      - 与对端节点的网络延迟。单位：毫秒。
    * - ledger
      - 字符串
      - 对端节点最后一个关闭的账本的哈希。
    * - load
      - 整数
      - 衡量对等服务器在本地服务器上加载的负载量。数字越大表示负载越大。（测量负载的单位未正式定义。）
    * - public_key
      - 字符串
      - 用来验真对端节点消息完整性的公钥。
    * - uptime
      - 整数
      - 对端节点自启动以来，连续运行的时长。单位：秒。
    * - version
      - 字符串
      - 对端节点运行的chainsqld版本。

.. _wallet_propose:

wallet_propose
+++++++++++++++++++++++++++++++

生成一个账户地址和秘钥对，之后必须通过转账交易，发送足够的ZXC给该账户，才能使账户真正进入账本。

语法：

::

    ./chainsqld wallet_propose [algType] [passphrase]

参数说明：

.. list-table::

    * - **参数**
      - **类型**
      - **可选值**
      - **描述**
    * - algType
      - 字符串
      - secp256k1/gmalg/ed25519
      - 算法类型，指定生成何种算法的非对称密钥对，不指定默认为节点使用的非对称密码算法，如果使用passphrase参数，则必须指定。
    * - passphrase
      - 字符串
      - 无
      - 秘钥种子，如果省略，则使用随机的种子，相同的种子生成相同的账户地址和证书。

返回结果示例：

1. secp256k1：

.. code-block:: json

    {
        "result" : {
            "account_id" : "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
            "account_id_hex" : "B5F762798A53D543A014CAF8B297CFF8F2F937E8",
            "key_type" : "secp256k1",
            "master_key" : "I IRE BOND BOW TRIO LAID SEAT GOAL HEN IBIS IBIS DARE",
            "master_seed" : "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb",
            "master_seed_hex" : "DEDCE9CE67B451D852FD4E846FCDE31C",
            "public_key" : "cBQG8RQArjx1eTKFEAQXz2gS4utaDiEC9wmi7pfUPTi27VCchwgw",
            "public_key_hex" : "0330E7FC9D56BB25D6893BA3F317AE5BCF33B3291BD63DB32654A313222F7FD020",
            "status" : "success"
        }
    }

2. gmalg：

.. code-block:: json

    {
        "result" : {
            "account_id" : "zLzooEnenjmeVaPZYykdx8jGJBV5j7uMN9",
            "account_id_hex" : "D091744D1737B0D574A9C908B3B97E646A7E87F4",
            "key_type" : "gmalg",
            "private_key" : "p92iRuvDiFnmRBfSGXGA5QNLuFx1rFucvkQpaSMgoVpYg5g7U8B",
            "public_key" : "pYvfKPYdmfkdTpQg8NFpxxzpGsr77WT4fDA93sd3mdBhnG66UCapMF296eCFZ7boLEWpeUNQvSRAVeuXXEnxpDmqhyfF7Eb7",
            "public_key_hex" : "4746CE7928E8D4464F3CA3E35EAC75BEEA210A9A3DAE606F75D4658A133E15BF3B44581F42A208DC06053BFE600166E8FE6E435BE84D8980689889C3CA2EA3E126",
            "status" : "success"
        }
    }

结果说明：

.. list-table::

    * - **参数**
      - **类型**
      - **描述**
    * - status
      - 字符串
      - 标识命令是否执行成功。
    * - account_id
      - 字符串
      - 生成的账户地址。
    * - account_id_hex
      - 字符串
      - 生成的账户地址原始十六进制格式内容。
    * - master_seed
      - 字符串
      - 账户的种子（私钥），国密算法没有此项。
    * - private_key
      - 字符串
      - 账户的私钥，国密算法使用此项。
    * - public_key
      - 字符串
      - 账户的公钥。
    * - public_key_hex
      - 字符串
      - 账户的公钥原始十六进制格式内容。


.. _cmdledger_txs:

ledger_txs
+++++++++++++++++++++++++++++++

查询区块中的成功、失败交易数，以及错误交易的hash及错误码。

语法：

::

    ./chainsqld ledger_txs <ledger_seq> [include_success] [include_failure]

参数说明：

.. list-table::

    * - **参数**
      - **类型**
      - **描述**
    * - ledger_seq
      - 整形
      - 要查询的区块号。
    * - include_success
      - 字符串
      - 若省略，则返回结果中，不包括成功的交易的hash。
    * - include_failure
      - 字符串
      - 若省略，则返回结果中，不包括错误交易的hash及错误码。

返回结果示例：

.. code-block:: json

    {
        "id" : 1,
        "result" : {
            "ledger_index" : 2,
            "status" : "success",
            "txn_failure" : 0,
            "txn_failure_detail" : [],
            "txn_success" : 1,
            "txn_success_detail" : [
              {
                "hash" : "41521F8535F1A6A581528BFB56F3085F9D4B09EBE913A6C854B1C9453BD0C46D",
                "transaction_result" : "tesSUCCESS"
              }
            ]
        }
    }

结果说明：

.. list-table::

    * - **参数**
      - **类型**
      - **描述**
    * - status
      - 字符串
      - 标识命令是否执行成功。
    * - txn_failure
      - 整形
      - 区块包含的错误交易个数。
    * - txn_success
      - 整形
      - 区块包含的成功交易个数。
    * - txn_failure_detail
      - 对象数组
      - 包含每个错误交易的哈希和错误码。
    * - txn_success_detail
      - 对象数组
      - 包含每个成功交易的哈希。

.. warning::

  此命令为\ :ref:`PoP共识版本 <PoP共识版本>`\ 新增命令，只适用于PoP共识版本。

chainsqld命令
*****************************

t_dump
+++++++++++++++++++++++++++++++

将数据库表的操作以文档的形式进行记录，可以分多次对同一张表进行dump。

语法：

::

    chainsqld t_dump <param> <out_file_path>

示例：

::

    ./chainsqld t_dump "zNRi42SAPegzJYzXYZfRFqPqUfGqKCaSbx Table1 262754" ./Table1.dump

参数说明：

.. list-table::

    * - **参数**
      - **类型**
      - **描述**
    * - param
      - 字符串
      - 与数据库表的同步设置保持一致。详情参见数据库表同步设置。
    * - out_file_path
      - 字符串
      - 输出文件路径。

返回结果：

.. code-block:: json

    {
        "id" : 1,
        "result" : {
            "command" : "t_dump",
            "status" : "success",
            "tx_json" : [
                "zNRi42SAPegzJYzXYZfRFqPqUfGqKCaSbx Table1 262754",
                "./table1.dmp"
            ]
        }
    }

t_dumpstop
+++++++++++++++++++++++++++++++

停止dump一张表。

语法：

::

    chainsqld t_dump <owner_address> <table_name>

参数说明：

.. list-table::

    * - **参数**
      - **类型**
      - **描述**
    * - owner_address
      - 字符串
      - 表的创建者账户地址。
    * - table_name
      - 字符串
      - 表名。

返回结果示例：

.. code-block:: json

    {
        "id" : 1,
        "result" : {
            "command" : "t_dumpstop",
            "status" : "success",
            "tx_json" : [ 
                "zNRi42SAPegzJYzXYZfRFqPqUfGqKCaSbx", 
                "Table1" 
            ]
        }
    }

t_audit
+++++++++++++++++++++++++++++++

对数据库表的指定记录（由SQL查询条件指定）的一列或多列进行追根溯源，将所有影响了指定记录的列的操作都记录下来。

语法：

::

    chainsqld t_audit <param> <sql_query_statement> <out_file_path>

示例：

::

    ./chainsqld t_audit "zNRi42SAPegzJYzXYZfRFqPqUfGqKCaSbx Table1 262754" "select * from Table1 where id=1" ./Table1.audit

参数说明：

.. list-table::

    * - **参数**
      - **类型**
      - **描述**
    * - param
      - 字符串
      - 与数据库表的同步设置保持一致。详情参见数据库表同步设置。
    * - sql_query_statement
      - 字符串
      - 由SQL语句指定审计的记录和列。
    * - out_file_path
      - 字符串
      - 输出文件路径。

返回结果：

.. code-block:: json

    {
        "id" : 1,
        "result" : {
            "command" : "t_audit",
            "nickName" : "5C9DD983025F6F654EA23FAFC0ADFC1BD0CAF58E",
            "status" : "success",
            "tx_json" : [
                "zNRi42SAPegzJYzXYZfRFqPqUfGqKCaSbx Table1 263498",
                "select * from Table1 where id=1",
                "./Table1.audit"
            ]
        }
    }

结果说明：

.. list-table::

    * - **参数**
      - **类型**
      - **描述**
    * - nickName
      - 字符串
      - 审计任务名称，用来停止审计任务。

t_auditstop
+++++++++++++++++++++++++++++++

停止审计。

语法：

::

    chainsqld t_auditstop <nickname>

参数说明：

.. list-table::

    * - **参数**
      - **类型**
      - **描述**
    * - nickname
      - 字符串
      - 启动审计任务时，返回的审计任务名。

返回结果：

.. code-block:: json

    {
        "id" : 1,
        "result" : {
            "command" : "t_auditstop",
            "status" : "success",
            "tx_json" : [ 
                "5C9DD983025F6F654EA23FAFC0ADFC1BD0CAF58E"
            ]
        }
    }



.. _LedgerObjects:

ledger_objects
+++++++++++++++++++++++++++++++

统计账本中各类别状态的个数。

语法：

::

    chainsqld ledger_objects <ledger_hash>|<ledger_index>

参数说明：

.. list-table::

    * - **参数**
      - **类型**
      - **描述**
    * - <ledger_hash>|<ledger_index>
      - 字符串
      - 账本哈希值 或者 账本号


返回结果：

.. code-block:: json

    {
      "id" : 1,
      "result" : {
          "ledger_hash" : "2C7279A24D8ED1A3B10F1C0497D245FEE87496FEDBB661176ABC3D1188F7CAE8",
          "ledger_index" : 1000,
          "state" : {
            "account" : 1,
            "amendments" : 1,
            "directory" : 0,
            "escrow" : 0,
            "fee" : 0,
            "hashes" : 2,
            "offer" : 0,
            "payment_channel" : 0,
            "signer_list" : 0,
            "state" : 0,
            "table" : 0,
            "ticket" : 0
          },
          "status" : "success",
          "tx" : 0,
          "validated" : true
      }
    }

------------

.. _NodeSize:

node_size
+++++++++++++++++++++++++++++++

查询和设置节点的缓存级别。不指定 ``type`` 表示查询节点的缓存级别，指定 ``type`` 表示设置节点的缓存级别。

语法：

::

    chainsqld node_size [<type>]

参数说明：

.. list-table::

    * - **参数**
      - **类型**
      - **描述**
    * - type
      - 字符串
      - 缓存级别,包括 tiny small medium large huge。


返回结果：

- 查询结果

.. code-block:: json

    {
      "id" : 1,
      "result" : {
          "node_size" : "medium",
          "status" : "success"
      }
    }


- 设置结果

.. code-block:: json

    {
      "id" : 1,
      "result" : {
          "status" : "success"
      }
    }


.. _MallocTrim:

malloc_trim
+++++++++++++++++++++++++++++++

释放由glibc维护的，未还给系统的内存。

语法：

::

    chainsqld malloc_trim

参数说明：


返回结果：

.. code-block:: json

    {
      "id" : 1,
      "result" : {
          "status" : "success",
          "value" : 1
      }
    }

.. _cmdSchemaList:

schema_list
+++++++++++++++++++++++++++++++

查询子链列表。

语法：

::

    chainsqld schema_list <running>

参数说明：

.. list-table::

    * - **参数**
      - **类型**
      - **描述**
    * - running
      - 字符串 running
      - 加此参数查询当前节点正运行的子链，如果不加，则查询所有子链

返回结果：

.. code-block:: json

    {
      "id" : 1,
      "result" : {
          "status" : "success",
          "value" : 1
      }
    }

.. _cmdSchemaInfo:

schema_info
+++++++++++++++++++++++++++++++

查询子链信息。

语法：

::

    chainsqld schema_info <schema_id>

参数说明：

.. list-table::

    * - **参数**
      - **类型**
      - **描述**
    * - schema_id
      - 字符串 子链ID
      - 查询指定子链信息

返回结果：

.. code-block:: json

    {
      "id" : 1,
      "result" : {
          "status" : "success",
          "value" : 1
      }
    }


.. _cmdSchemaAccept:

schema_accept
+++++++++++++++++++++++++++++++

接受加入子链。

语法：

::

    chainsqld schema_accept <schema_id>

参数说明：

.. list-table::

    * - **参数**
      - **类型**
      - **描述**
    * - schema_id
      - 字符串 子链ID
      - 查询指定子链信息

返回结果：

.. code-block:: json

    {
      "id" : 1,
      "result" : {
          "status" : "success",
          "value" : 1
      }
    }
