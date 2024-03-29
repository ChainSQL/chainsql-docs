===================
基础知识
===================

1. 账户
==================

1.1 根账户
------------------

- 根账户是在创世区块中就存在的，区块链中所有的众享数字资产(1000亿 ``ZXC`` )默认都在根账户中。
- 根账户地址和公私钥对根据节点使用的非对称密码算法不同而不同。
- 一般在链初始成功后，会进行根账户转移操作，将所有 ``ZXC`` 转移到一个隐私账户中，以实现对根账户的保护。

生成/查看根账户的命令如下：

1.1.4-pop之前版本

.. code-block:: bash

    ./chainsqld wallet_propose masterpassphrase

1.1.4-pop之后版本,需要指定算法名称，具体用法参见 :ref:`wallet_propose <wallet_propose>`

.. code-block:: bash

    ./chainsqld wallet_propose secp256k1 masterpassphrase

- ``secp256k1`` 算法根账户的地址及种子（种子可生成私钥及公钥，在Chainsql中一般不直接使用私钥，而是使用种子）

.. code-block:: json

    {
        "account_id" : "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
        "public_key" : "cBQG8RQArjx1eTKFEAQXz2gS4utaDiEC9wmi7pfUPTi27VCchwgw",
        "master_seed" : "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb"
    }

- 国密算法根账户的地址及私钥（国密账户中私钥的作用等同于其他算法账户中的种子 ）

.. code-block:: json

    {
        "account_id" : "zN7TwUjJ899xcvNXZkNJ8eFFv2VLKdESsj",
        "public_key" : "pYvWhW4azFwanovo5MhL71j5PyTWSJi2NVurPYUrE9UYaSVLp29RhtxxQB7xeGvFmdjbtKRzBQ4g9bCW5hjBQSeb7LePMwFM",
        "master_seed" : "p97evg5Rht7ZB7DbEpVqmV3yiSBMxR3pRBKJyLcRWt7SL5gEeBb"
    }

1.2 账户的生成与激活
---------------------------

在chainsql中，除了根账户外，其它账户都是需要激活才能生效的，下面以节点默认使用 ``secp256k1`` 算法为例进行说明。
其他算法使用流程一样，但是具体命令或者函数需要参考其在不同算法中的使用方式进行调用。
我们可以在命令行中使用  ``wallet_propose`` 命令生成一个新的账户，也可以调用  ``generateAddress`` 接口来生成一个新的账户：

命令行：

.. code-block:: bash

    ./chainsqld wallet_propose

Node.js:

.. code-block:: js

    let account = c.generateAddress();
    console.log(account);

Java:

.. code-block:: java

    Chainsql c = new Chainsql();
    JSONObject account = c.generateAddress();
    System.out.println(account);

新生成的账户在链中是无效的，在node.js中查询账户信息：

.. code-block:: js

    let info = await c.api.getAccountInfo(account.address);
    console.log(info);

会输出 ``actNotFound`` 的错误信息，想要使用一个账户，需要使用 ``pay`` 接口给账户打钱：

Node.js:

.. code-block:: js

    var account = {
        secret:"xnnUqirFepEKzVdsoBKkMf577upwT",
        address:"zpMZ2H58HFPB5QTycMGWSXUeF47eA8jyd4"
    }
    var owner = {
        secret: "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb",
        address: "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh"	
    }

    await c.connect('ws://106.75.99.244:6006');
    console.log('连接成功')
    c.as(owner);    //这里owner指一个有足够zxc的账户，第一个转账操作肯定要用根账户
            
    let rs = await c.pay(account.address,200).submit({expect:'validate_success'});
    console.log(rs);

Java:

.. code-block:: java

    String rootAddress = "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh";
    String rootSecret = "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb";
    String newAddress = "zpMZ2H58HFPB5QTycMGWSXUeF47eA8jyd4";

    c.connect("ws://106.75.99.244:6006");
    c.connection.client.logger.setLevel(Level.SEVERE);
    c.as(rootAddress,rootSecret);

    JSONObject obj = c.pay(newAddress,"200").submit(SyncCond.validate_success);
    if(obj.has("error_message")){
        System.out.println("激活或转账失败。 失败原因: " + obj.getString("error_message"));
    }else {
        System.out.println("激活或转账成功");
    }

2.Chainsql中的预留费用
================================

2.1 账户基础预留费
-------------------------------

账户预留费用为一个账户激活需要的最少费用，chainsql网络中默认为 ``5zxc``

2.2 对象增加预留费用
--------------------------------
为了防止每个账户恶意创建对象（如建表操作），导致整个区块链网络占用内存过大，每增加一个对象，chainsql会冻结1个 ``ZXC`` 作为对象增加预留费用，相对的，每减少一个对象，也会解除一个ZXC的保留费用冻结。
Chainsql中的对象包括：

 | 基础对象 ``Escrow,PayChannel,Offer,TrustLine``
 | 表对象 ``Table,Contract``

Chainsql预留费用 = 账户基础预留费用 + 对象增加预留费用

预留费用是被冻结的，不能用于转账操作

比如我新生成一个账户A，并且用 ``10ZXC`` 把它激活，那这时A账户中只有5个 ``ZXC`` 是能用的。
A账户要建一张表，建表交易费用为 ``0.5ZXC`` ，对象增加费用为 ``1ZXC`` ，那这时，A账户余额为 ``9.5ZXC``，总预留费用为 ``6ZXC`` ，可用余额为 ``3.5ZXC`` 。

3.交易费用
===================
Chainsql中自带系统数字资产 ``ZXC`` ，最小单位为 ``drop`` ， ``1ZXC = 1000000(1e+6) drop`` 

在Chainsql中交易费用将会被销毁，不会给任何人，也就是说，Chainsql网络中总的 ``ZXC`` 数量是随着交易不断减少的。

Chainsql中的基础交易费用 ``10drop`` ，一笔普通转账交易，正常情况下只需要10drop就可以

3.1 Chainsql交易费用计算规则
------------------------------------------------
Chainsql类型的交易 ``（TableListSet,SQLStatement,SQLTransaction）`` 基础费用为 ``1010drop`` ，也就是 ``0.00101zxc`` 

Chainsql类型交易费用 = ``0.00101(ZXC)`` + 交易中 ``Raw`` 字段字节数/ ``1024(ZXC)``

即交易费用分为基础交易费用0.00101ZXC，以及处理数据的费用，每有1kb数据就需要支付1ZXC。

比如我要建一张表，建表的rpc命令如下：

.. code-block:: json

    {
        "method": "t_create",
        "params": [
            {
                "offline": false,
                "secret": "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb",
                "tx_json": {
                    "TransactionType": "TableListSet",
                    "Account": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
                    "Tables":[
                        {
                            "Table":{
                                    "TableName":"aaa"
                            }
                        }
                    ],
                "OpType": 1,
                "Raw": [
                        {"field":"id","type":"int","length":11,"PK":1,"NN":1,"UQ":1},
                        {"field":"age","type":"int"},
                        {"field":"name","type":"varchar","length":64}
                ]
            
            }
            }
        ]
    }

这个建表操作中 ``Raw`` 字段较小，假设只有 ``0.1K`` ，那这个交易的交易费用为
``0.00101 + 0.1 = 0.10101(zxc)``

版本变化说明
+++++++++++++++++++++++++++++++++++++

0.30.4版本以后，新添加配置选项    :ref:`drops_per_byte <DropsPerByte>`  

Chainsql类型交易费用 = ``0.00101(ZXC)`` + 交易中 ``Raw`` 字段字节数 *  drops_per_byte / ``10^6(ZXC)``

例如： 上述建表操作中 ``Raw`` 字段较小，假设只有 ``100`` bytes，drops_per_byte = 2000 上述建表交易的交易费用为

``0.00101 + 100 * 2000 /10^6 = 0.20101(zxc)``

3.2 智能合约交易费用
-------------------------------
ChainSQL 中的智能合约是用的 ``EVM`` 技术实现的，智能合约中除了交易费用，合约在执行过程中还需要消耗 ``Gas`` ，在做智能合约交易时，需要指定 ``Gas`` 上限。

Gas只是一个数量，没有单位，真正的消耗的费用= ``Gas * GasPrice`` .

在ChainSQL中， ``GasPrice`` 是由ChainSQL网络决定的，正常情况下， ``GasPrice = 10drop`` ，网络状况拥堵的情况下，GasPrice会在 ``10drop`` 到 ``20drop`` 之间浮动