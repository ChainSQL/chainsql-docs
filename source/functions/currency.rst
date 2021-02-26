代币发行
###########################

代币发行的功能涉及到 ChainSQL 网络中的网关的概念以及网关发行代币，具体介绍如下：

------------------------------------

网关
*************************

概念
===============

在ChainSQL中，网关指的是代币在ChainSQL网络中流通经过的关口。网关的主要作用是发行代币，它允许使用者把法定货币注入、抽离ChainSQL网络，并可充当借、贷双方的桥梁；任意一个ZXC账户，都可以成为一个网关。

用途
===============

一般来说，ChainSQL网关主要有以下几种用途：

- 金融机构通过在ChainSQL网络中发行代币，可以加速资金流转，降低金融成本，最典型的例子是降低通过SWIFT跨国转账收取的高昂手续费；
- 金融机构可以利用区块链可靠，可追溯的特性，实现资金的自动清算，资金通过ChainSQL网络进行流通，不会出现对账不成功的问题，减少审计成本；
- 商家发行代币，可以作为一种积分、优惠券，可以很方便的实现积分的互转，并且所有记录自动上链，不可篡改；
-  ``ICO`` ，一个创业型公司可以通过代币发行，来进行融资；

术语
===============

- ``Trustline`` ，也就是信任通道，在ChainSQL中，一个账户只是把自己设置为网关是没有意义的，必须有人来信任它发行的代币才行，这个信任关系，就叫trustline。
- ``Rippling`` ，一个网关必须开启Rippling功能，信任它的账户之间才可以互转这一网关发行的代币，不然这一网关就是关闭的，发行的货币无法通过它进行流通。
- 费率( ``TransferRate`` )，代币的流通（转账或者挂单）会经过网关，网关可以选择设置费率对这些交易收取手续费，在ChainSQL中仅支持设置百分比作为费率。
- 发行账户( ``Issuing Address`` )，网关账户，也就是代币发行账户。
- 持币账户( ``Operatinal Address`` )，网关发行代币的过程，一般会涉及到另一个账户，也就是持币账户，网关发行的代币量会在最开始打到这个账户，后面由这个账户向其它账户进行转账。

------------------------------------

网关发行代币
*************************

下面举一个例子来说明，在ChainSQL网络中是如何通过网关来发行代币的，代币名称为AAA，要发行总量是10000。

准备工作
==============

在一个新建的ChainSQL网络上发币需要准备和涉及到的账户如下：

1. 根账户A：ChainSQL节点部署完成之后，对应会生成一个根账户，根账户会初始一定数量的系统币。

2. 网关账户B：发行代币的账户，后续所有普通账户需要做信任操作，信任该账户发行的代币。

3. 普通账户C：普通账户,由发行代币的账户转账一定数量的代币给该账户，该账户用于代币的接收以及转账。

4. 普通账户D：普通账户，作用同账户C。

------------------------------------

发币流程
==============

- 1 激活账户
- 2 配置网关属性
- 3 信任网关代币
- 4 转账发行代币
- 5 普通账户相互转移发行代币
- 6 冻结发行代币
- 7 解冻发行代币

激活账户
+++++++++++++++

通过根账户A给网关账户B，普通账户C以及普通账户D转账，激活账户B,C,D。

- RPC代码

激活账户B

.. code-block:: json

        {
            "method": "submit",
            "params": [
                {
                    "offline": false,
                    "secret": "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb",
                    "tx_json": {
                        "Account": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
                        "Amount": 10000000000,
                        "Destination": "zKQwdkkzpUQC9haHFEe2EwUsKHvvwwPHsv",
                        "TransactionType": "Payment"
                    }
                }
            ]
        }

激活账户C

.. code-block:: json

        {
            "method": "submit",
            "params": [
                {
                    "offline": false,
                    "secret": "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb",
                    "tx_json": {
                        "Account": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
                        "Amount": 10000000000,
                        "Destination": "zPcimjPjkhQk7a7uFHLKEv6fyGHwFGQjHa",
                        "TransactionType": "Payment"
                    }
                }
            ]
        }

激活账户D

.. code-block:: json

    {
        "method": "submit",
        "params": [
            {
                "offline": false,
                "secret": "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb",
                "tx_json": {
                    "Account": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
                    "Amount": "1000000000000",
                    "Destination": "zKQwdkkzpUQC9haHFEe2EwUsKHvvwwPHsv",
                    "TransactionType": "Payment"
                }
            }
        ]
    }

- java代码

.. code-block:: java
  
  public  void testActive(){
    
    String sUserB = "zKQwdkkzpUQC9haHFEe2EwUsKHvvwwPHsv";
    String sUserBSec = "xnJn5J5uYz3qnYX72jXkAPVB3ZsER";
    String sUserC = "zPcimjPjkhQk7a7uFHLKEv6fyGHwFGQjHa";
    String sUserCSec = "xxCosoAJMADiy6kQFVgq1Nz8QewkU";
    String sUserD= "z4ypskpHPpMDtHsZvFHg8eDEdTjQrYYYV6";
    String sUserDSec = "xxXvas5HTwVwjpmGNLQDdRyYe2H6t" ;


    System.out.print("activate >>>>>>>>>>>>>>>\n");
    JSONObject jsonObj = c.pay(sUserB, "1000").submit(SyncCond.validate_success);
    System.out.print("     sUserB:" + jsonObj + "\n");
    jsonObj = c.pay(sUserC, "1000").submit(SyncCond.validate_success);
    System.out.print("     sUserC:" + jsonObj + "\n");
    jsonObj = c.pay(sUserD, "1000").submit(SyncCond.validate_success);
    System.out.print("     sUserD:" + jsonObj + "\n");
    System.out.print("activate <<<<<<<<<<<<<<<\n");
  }

- Node.js代码

.. code-block:: javascript

  var testActive = async function () {

      var userB = {
          address: "zKQwdkkzpUQC9haHFEe2EwUsKHvvwwPHsv",
          secret: "xnJn5J5uYz3qnYX72jXkAPVB3ZsER"
      }
      var userC = {
          address: "zPcimjPjkhQk7a7uFHLKEv6fyGHwFGQjHa",
          secret: "xxCosoAJMADiy6kQFVgq1Nz8QewkU"
      }
      var userD = {
          address: "z4ypskpHPpMDtHsZvFHg8eDEdTjQrYYYV6",
          secret: "xxXvas5HTwVwjpmGNLQDdRyYe2H6t"
      }

      var amount = 1000;
      console.log("----------- active >>>>>>>>>>>>>");
      let res = await c.pay(userB.address, amount).submit({ expect: 'validate_success' })
      console.log("   active issuer", issuer.address, ":", res)
      res = await c.pay(userC.address, amount).submit({ expect: 'validate_success' })
      console.log("\n   active user", user.address, ":", res)
      res = await c.pay(userD.address, amount).submit({ expect: 'validate_success' })
  }

---------------

配置网关属性
++++++++++++++++++

配置网关账户B，设置发行账户B的 ``DefaultRipple`` 标志为true,并设置网关费率等信息，这个过程用到 ``AccountSet Flags``  交易：

- RPC代码 

.. code-block:: json

    {
        "method": "submit",
        "params": [
            {
                "offline": false,
                "secret": "xnJn5J5uYz3qnYX72jXkAPVB3ZsER",
                "tx_json": {

                    "TransactionType": "AccountSet",
                    "Account" : "zKQwdkkzpUQC9haHFEe2EwUsKHvvwwPHsv",
                    "SetFlag": 8,
                    "TransferRate":1002000000,
                    "TransferFeeMin":10,
                    "TransferFeeMax":15

                }
            }
        ]
    }


- java代码

.. code-block:: java
  
  c.as(sUserB, sUserBSec);
  JSONObject jsonObj = c.accountSet(8, true).submit(SyncCond.validate_success);
  System.out.print("set gateWay:" + jsonObj + "\ntrust gateWay ...\n");
  jsonObj = c.accountSet("1.002", "10", "15").submit(SyncCond.validate_success);

- node.js代码

.. code-block:: javascript

  let res;
  console.log("----------- GateWay >>>>>>>>>>>>>");
  var opt = {
      enableRippling: true,
      rate: 1.002,
      min: 10,
      max: 15
  }
  c.as(userB);
  res = await c.accountSet(opt).submit({ expect: 'validate_success' });
  console.log("\n   accountSet issuer", issuer.address, ":", res)


--------------------------------

信任网关代币
++++++++++++++++++++++++

账户C和账户D信任网关账户B的代币AAA，信任的代币限额即代币发行数量10000，这个过程用到  ``TrustSet`` 交易

- RPC代码 

账户C 信任网关账户B的代币 ``AAA`` ，信任代币的额度为10000

.. code-block:: json

    {
        "method": "submit",
        "params": [
            {
                "offline": false,
                "secret": "xxCosoAJMADiy6kQFVgq1Nz8QewkU",
                "tx_json": {
                    "Account": "zPcimjPjkhQk7a7uFHLKEv6fyGHwFGQjHa",
                    "LimitAmount": {
                        "currency": "AAA",
                        "issuer": "zKQwdkkzpUQC9haHFEe2EwUsKHvvwwPHsv",
                        "value": "10000"
                    },
                    "TransactionType": "TrustSet"
                }
            }
        ]
    }

---------------

账户D 信任网关账户B的代币 ``AAA```，信任代币的额度为10000

  .. code-block:: json

    {
        "method": "submit",
        "params": [
            {
                "offline": false,
                "secret": "xxXvas5HTwVwjpmGNLQDdRyYe2H6t",
                "tx_json": {
                    "Account": "z4ypskpHPpMDtHsZvFHg8eDEdTjQrYYYV6",
                    "LimitAmount": {
                        "currency": "AAA",
                        "issuer": "zKQwdkkzpUQC9haHFEe2EwUsKHvvwwPHsv",
                        "value": "10000"
                    },
                    "TransactionType": "TrustSet"
                }
            }
        ]
    }

------------------

- java代码

.. code-block:: java
  
    c.as(sUserC, sUserCSec);
    jsonObj = c.trustSet("10000", "AAA", sUserB).submit(SyncCond.validate_success);
    System.out.print("     user: " + jsonObj + "\n");
    c.as(sUserD, sUserDSec);
    jsonObj = c.trustSet("10000", "AAA", sUserB).submit(SyncCond.validate_success);

    System.out.print("acountLines ...\n");
    jsonObj = c.connection.client.GetAccountLines(sUserC);
    System.out.print("     sUserC: " + jsonObj + "\n");
    jsonObj = c.connection.client.GetAccountLines(sUserD);
    System.out.print("     sUserD " + jsonObj + "\n");
    System.out.print("trust <<<<<<<<<<<<<<<\n");

- node.js代码

.. code-block:: javascript

    var amount = {
        value: 10000,
        currency: "AAA",
        issuer: sUserB.address
    }
    //
    c.as(sUserC);
    res = await c.trustSet(amount).submit({ expect: 'validate_success' });
    console.log("\n   trustSet sUserC", sUserC.address, ":", res)
    c.as(sUserD);
    res = await c.trustSet(amount).submit({ expect: 'validate_success' });
    console.log("\n   trustSet sUserD", sUserD.address, ":", res)
    //

--------------------------

转账发行代币
++++++++++++++++++++++++

发行账户B向账户C转账10000个AAA,并给账户D转账10000个AAA，这个过程用到 ``Payment`` 交易：

- RPC代码 

网关账户B向账户C转账5000个AAA

.. code-block:: json

    {
        "method": "submit",
        "params": [
            {
                "offline": false,
                "secret": "xnJn5J5uYz3qnYX72jXkAPVB3ZsER",
                "tx_json": {
                    "Account": "zKQwdkkzpUQC9haHFEe2EwUsKHvvwwPHsv",
                    "Amount" : {
                        "currency" : "AAA",
                        "value" : "5000",
                        "issuer" : "zKQwdkkzpUQC9haHFEe2EwUsKHvvwwPHsv"
                    },
                    "Destination": "zPcimjPjkhQk7a7uFHLKEv6fyGHwFGQjHa",
                    "TransactionType": "Payment"
                }
            }
        ]
    }

网关账户B向账户D转账5000个AAA

.. code-block:: json

    {
        "method": "submit",
        "params": [
            {
                "offline": false,
                "secret": "xnJn5J5uYz3qnYX72jXkAPVB3ZsER",
                "tx_json": {
                        "Account": "zKQwdkkzpUQC9haHFEe2EwUsKHvvwwPHsv",
                        "Amount" : {
                            "currency" : "AAA",
                            "value" : "5000",
                            "issuer" : "zKQwdkkzpUQC9haHFEe2EwUsKHvvwwPHsv"
                        },
                        "Destination": "z4ypskpHPpMDtHsZvFHg8eDEdTjQrYYYV6",
                        "TransactionType": "Payment"
                }
            }
        ]
    }

-----------------

- java代码

.. code-block:: java
  
      CString sCurrency = "AAA";
      System.out.print("pay >>>>>>>>>>>>>>>\n");

      c.as(sUserB, sUserBSec);
      jsonObj = c.pay(sUserC, "5000", sCurrency, sUserB).submit(SyncCond.validate_success);
      System.out.print("    sUserC:\n     " + jsonObj + "\n");
      jsonObj = c.connection.client.GetAccountLines(sUserC);
      System.out.print("  sUserC  lines: " + jsonObj + "\n");
      c.as(sUser, sUserSec);
      jsonObj  = c.pay(sUserD, "5000", sCurrency, sUserB).submit(SyncCond.validate_success);
      System.out.print("    sUserD:\n     " + jsonObj + "\n");
      jsonObj = c.connection.client.GetAccountLines(sUserD);
      System.out.print("  sUserD  lines: " + jsonObj + "\n");
      System.out.print("pay <<<<<<<<<<<<<<<\n");

- node.js代码

.. code-block:: javascript

    var amount = {
        value: 5000,
        currency: "AAA",
        issuer: sUserB.address
    }

    //
    c.as(sUserB);
    res = await c.pay(sUserC.address, amount).submit({ expect: 'validate_success' })
    console.log("\n   transfer currency(sUserB 2 sUserC)", issuer.address, user.address, ":", res)

    res = await c.pay(sUserD.address, amount).submit({ expect: 'validate_success' })
    console.log("\n   transfer currency(sUserB 2 sUserD)", user.address, user1.address, ":", res)
    console.log("\n----------- GateWay <<<<<<<<<<<<<");

---------------------------------------------------------


普通账户相互转移发行代币
++++++++++++++++++++++++++++++++++++++++++++++++

- RPC代码

账户C向账户D转账1000个AAA

.. code-block:: json

    {
        "method": "submit",
        "params": [
            {
                "offline": false,
                "secret": "xxCosoAJMADiy6kQFVgq1Nz8QewkU",
                "tx_json": {
                    "Account": "zPcimjPjkhQk7a7uFHLKEv6fyGHwFGQjHa",
                    "Amount" : {
                        "currency" : "AAA",
                        "value" : "1000",
                        "issuer" : "zKQwdkkzpUQC9haHFEe2EwUsKHvvwwPHsv"
                    },
                    "SendMax":{
                        "currency" : "AAA",
                        "value" : "1015",
                        "issuer" : "zKQwdkkzpUQC9haHFEe2EwUsKHvvwwPHsv"
                    },                    
                    "Destination": "z4ypskpHPpMDtHsZvFHg8eDEdTjQrYYYV6",
                    "TransactionType": "Payment"
                }
            }
        ]
    }

账户D向账户C转账1000个AAA

.. code-block:: json

    {
        "method": "submit",
        "params": [
            {
                "offline": false,
                "secret": "xxXvas5HTwVwjpmGNLQDdRyYe2H6t",
                "tx_json": {
                    "Account": "z4ypskpHPpMDtHsZvFHg8eDEdTjQrYYYV6",
                    "Amount" : {
                        "currency" : "AAA",
                        "value" : "1000",
                        "issuer" : "zKQwdkkzpUQC9haHFEe2EwUsKHvvwwPHsv"
                    },
                    "SendMax":{
                        "currency" : "AAA",
                        "value" : "1015",
                        "issuer" : "zKQwdkkzpUQC9haHFEe2EwUsKHvvwwPHsv"
                    },                    
                    "Destination": "zPcimjPjkhQk7a7uFHLKEv6fyGHwFGQjHa",
                    "TransactionType": "Payment"
                }
            }
        ]
    }

----------

- java代码

.. code-block:: java
  
      CString sCurrency = "AAA";
      System.out.print("pay >>>>>>>>>>>>>>>\n");

      c.as(sUserC, sUserCSec);
      jsonObj  = c.pay(sUserD, "1000", sCurrency, sUserB).submit(SyncCond.validate_success);
      System.out.print("   sUserC to sUserD:\n     " + jsonObj + "\n");

      c.as(sUserD, sUserDSec);
      jsonObj  = c.pay(sUserC, "1000", sCurrency, sUserB).submit(SyncCond.validate_success);
      System.out.print("   sUserD to sUserC :\n     " + jsonObj + "\n");

      jsonObj = c.connection.client.GetAccountLines(sUserC);
      System.out.print("  sUserC  lines: " + jsonObj + "\n");

      jsonObj = c.connection.client.GetAccountLines(sUserD);
      System.out.print("  sUserD  lines: " + jsonObj + "\n");
      System.out.print("pay <<<<<<<<<<<<<<<\n");


- Node.js 代码

.. code-block:: javascript
  
    var amount = {
        value: 1000,
        currency: "AAA",
        issuer: sUserB.address
    }

    //
    c.as(sUserC);
    res = await c.pay(sUserD.address, amount).submit({ expect: 'validate_success' })
    console.log("\n   transfer currency(sUserC 2 sUserD)", issuer.address, user.address, ":", res)

    c.as(sUserD);
    res = await c.pay(sUserC.address, amount).submit({ expect: 'validate_success' })
    console.log("\n   transfer currency(sUserD 2 sUserC)", user.address, user1.address, ":", res)
    console.log("\n----------- GateWay <<<<<<<<<<<<<");

--------------------

冻结发行代币
++++++++++++++

冻结发行代币主要是由网关发起，目的在于冻结已发行的代币。`详细信息 <https://xrpl.org/freezes.html#individual-freeze>`_


- RPC代码

账户 B 冻结发行给账户 D 的代币 ``AAA``, 额度为 10000

.. code-block:: json

    {
        "method": "submit",
        "params": [
            {
                "offline": false,
                "secret": "xnJn5J5uYz3qnYX72jXkAPVB3ZsER",
                "tx_json": {
                    "Account": "zKQwdkkzpUQC9haHFEe2EwUsKHvvwwPHsv",   
                    "Flags": 1048576,
                    "LimitAmount": {
                        "currency": "AAA",
                        "issuer": "z4ypskpHPpMDtHsZvFHg8eDEdTjQrYYYV6",
                        "value": "10000"
                    },
                    "TransactionType": "TrustSet"
                }    
            
            }
        ]
    }

----------------

- java代码

.. code-block:: java
  
    CString sCurrency = "AAA";
    System.out.print("freeze currency >>>>>>>>>>>>>>>\n");

    c.as(sUserB, sUserBSec);
    jsonObj = c.freezeCurrency("10000", sCurrency,sUserD,true).submit(SyncCond.validate_success);
    System.out.print("     freeze " + jsonObj + "\n");

------------

- Node.js 代码

.. code-block:: javascript
  
    var amount = {
        value: 10000,
        currency: "AAA",
        issuer: sUserB.address
    }

    //
    c.as(sUserB);
    let ret = await  c.freezeCurrency(userD.address,sCurrency);

    console.log('deFreezeCurrency ret',ret);


------------


解冻发行代币
++++++++++++++

- RPC代码

账户B 解冻发行给账户 D 的代币 ``AAA``, 额度为 10000

.. code-block:: json

        {
            "method": "submit",
            "params": [
                {
                    "offline": false,
                    "secret": "xnJn5J5uYz3qnYX72jXkAPVB3ZsER",
                    "tx_json": {
                        "Account": "zKQwdkkzpUQC9haHFEe2EwUsKHvvwwPHsv",   
                        "Flags": 2097152,
                        "LimitAmount": {
                            "currency": "AAA",
                            "issuer": "zKQwdkkzpUQC9haHFEe2EwUsKHvvwwPHsv",
                            "value": "10000"
                        },
                        "TransactionType": "TrustSet"
                    }    
                
                }
            ]
        }

----------------

- java代码

.. code-block:: java
  
    CString sCurrency = "AAA";
    System.out.print(" no freeze currency >>>>>>>>>>>>>>>\n");

    c.as(sUserB, sUserBSec);
    jsonObj = c.freezeCurrency("10000", sCurrency,sUserD,false).submit(SyncCond.validate_success);
    System.out.print("   no  freeze " + jsonObj + "\n");

------------

- Node.js 代码

.. code-block:: javascript
  
    var amount = {
        value: 10000,
        currency: "AAA",
        issuer: sUserB.address
    }

    //
    c.as(sUserB);
    let ret = await  c.deFreezeCurrency(userD.address,sCurrency);

    console.log('deFreezeCurrency ret',ret);

------------


完整的代码示例
++++++++++++++

- `node.js代币发行 <https://github.com/ChainSQL/node-chainsql-api/blob/master/test/testRipple.js/>`_

- `JAVA 代币发行 <https://github.com/ChainSQL/java-chainsql-api/blob/master/chainsql/src/test/java/com/peersafe/example/chainsql/TestRipple.java/>`_



智能合约对代币发行的支持
+++++++++++++++++++++++++++++

:ref:`智能合约代币支持 <SmartContract_Gateway_call>`

-----------------------------------------------

总结
*************************

上面是用网关来发行定量代币的过程，如果只是作为积分发行，机构只需要在想要分发代币时，用持币账户给它下面的用户转账代币就可以了，每个用户需要做的是在最开始信任下网关，然后就可以在网络中进行代币的交易。
