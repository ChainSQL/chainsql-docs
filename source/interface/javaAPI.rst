.. _JavaAPI_entry:

==========
Java接口
==========


ChainSQL提供JAVA-API与节点进行交互。实现ChainSQL区块链的基础交易发送、链数据查询、数据库操作、智能合约操作等交互操作。

.. _Java环境:

环境准备
===============

---------------
获取依赖的jar包
---------------


* 如果本地有maven环境，将以下代码加入本地开发环境pom.xml文件进行jar包下载。

.. code-block:: xml

  <dependency>
    <groupId>com.peersafe</groupId>
    <artifactId>chainsql</artifactId>
    <version>1.5.7</version>
  </dependency>


- 如果本地没有maven环境，可以直接下载项目依赖的jar包。项目依赖的jar包下载: `下载地址 <http://www.chainsql.net/libs.zip>`_

  *-* Eclipse。在"Project"-"Properyties"-"Java Build Path"-"Libraries"-"Add External JARs" 添加相应的jar包。

  *-* IDEA。在"Project Structure"-"Modules"-"Dependencies"中添加相应的jar包。

------

--------
引入
--------

使用下面的代码引入ChainSQL JAVA-API

.. code-block:: java

    import com.peersafe.chainsql.core.Chainsql;
    import com.peersafe.chainsql.core.Submit.SyncCond;
    import com.peersafe.chainsql.util.Util;

    // 引入之后使用new创建chainsql对象，之后使用chainsql对象进行接口操作
    Chainsql c = new Chainsql();

------

------------------------------------------
Java-SDK对于多密码算法的支持方式
------------------------------------------
Java-SDK的 :ref:`as <javaAS>` 确定Java-SDK以何种算法运行：

  - 当as了一个secp256k1算法账户时，会使用sha哈希算法，此时能跟同样使用secp256k1的节点进行交互；
  - 当as了一个国密算法账户时，会使用sm3哈希算法，此时能跟同样使用gmalg节点进行交互；
  - SDK使用的算法必须和节点一致

版本变化
===========

----------
1.5.7
----------
    - 插入和更新表交易中支持 ``txsHashFillField`` : :ref:`insert  <InsertJava>`  :ref:`update  <UpdateJava>` 
    - 支持国密算法。相关接口: :ref:`generateAddress  <GenerateAddress>`

------------------------

----------
1.5.6
----------
    - ``SQLStatement`` 类型的交易支持CA证书功能

------------------------

----------
1.5.5
----------
    - 插入表交易中支持 ``AutoFillField``

------------------------

----------
1.5.4
----------
    - 对应 ``Chainsqld`` 版本 ``0.30.5`` ，支持CA证书功能
    - 1.5.4版本增加新接口 :ref:`useCert  <UseCertJava>`  

------------------------

----------
1.5.3
----------
    - 修改maven库依赖 。依赖库 ``abi_chainsql`` 从版本 ``1.4.4`` 修改为 ``1.4.5``

------------------------

----------
1.5.2
----------
    - 对应 ``Chainsqld`` 版本 ``0.30.4`` ，支持智能合约对数字资产指令的扩展，支持费用调节功能
    - 1.5.2版本增加新添对多线程并发的支持类：   :ref:`ChainsqlPool <chainsql_pool>`  ，并提供了示例代码。

------------------------

----------
1.5.1
----------
    - 对应 ``Chainsqld`` 版本 ``0.30.3``
    - 1.5.1版本之前的版本对多线程的支持不好,新版本支持多线程中调用。
    - 1.5.1版本之前 ``pay`` 方法直接调会返回交易提交结果，而新版本需要在方法后接 ``.submit`` 指定是否共识成功返回。具体示例见 :ref:`示例 <my-reference-pay-sample>`.
    - 1.5.1版本之前对象可以使用Chainsql静态对象： ``Chainsql c = Chainsql.c`` ,现在删除了静态对象，需要用户自己调用 ``new`` ，例如
      ``Chainsql c = new Chainsql();``

------------------------

.. _Java返回值:

接口返回格式
================

--------------
交易类接口
--------------

交易类的接口包括网关交易以及表交易等相关的一系列接口。

.. _json-return:

交易类接口返回的JSON包含的各个域如下：

.. list-table::

    * - **域**
      - **类型**
      - **描述**
    * - tx_hash
      - 字符串
      - 交易的哈希值
    * - status
      - 字符串
      - 标识交易是否已被服务节点成功接收并且解析成功。
    * - tx_json
      - 对象
      - 签名后的交易的JSON格式。
    * - error
      - 字符串
      - 如果交易请求解析或者处理出错，返回错误类型码。
    * - error_code
      - 字符串
      - 与error关联的整形值。
    * - error_message
      - 字符串
      - 错误原因的描述。

成功示例

.. code-block:: json

    {
        "tx_hash":"CC552C401D7E0393A240D7441C4C4870DD5723F5D9306F89BA9848BDA6ED4816",
        "status":"db_success"
    }

失败示例

.. code-block:: json

      {
          "error_message":"Insufficient ZXC balance to send.",
          "tx_json":{
              "Account":"z3VGAJo59RWZ23CeM7zwMGVvsbWF8HabHf",
              "Destination":"zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
              "TransactionType":"Payment",
              "TxnSignature":"3045022100E3D6BE3070FBF7B7F891DD6A724F0EFA77A9BF0E440C739CF898F0E44FB25273022001D82BE41146FBA21F20145A14132943623632C33FB0844CD035FAE38086DB83",
              "SigningPubKey":"032B50FB18E894BB74B56A18A482C113219598650AE465F61161663D15C0EC4215",
              "Amount":"2000000000",
              "Fee":"11",
              "Flags":2147483648,
              "Sequence":2,
              "LastLedgerSequence":5116,
              "hash":"8441BD3D6777A647B0D63DEC5D37C8AF67DE38248D7045E3900E14B3AE56DC8B"
          },
          "error_code":104,
          "error":"tecUNFUNDED_PAYMENT",
          "status":"error"
      }

------

--------------
查询类接口
--------------

查询类的接口包括账户相关的信息查询以及表查询等相关的一系列接口。

查询类接口返回的JSON包含的各个域如下：

.. list-table::

    * - **域**
      - **类型**
      - **描述**
    * - status
      - 字符串
      - 标识交易是否已被服务节点成功接收并且解析成功。
    * - request
      - 对象
      - 查询接口的JSON格式。
    * - final_result
      - 布尔型
      - 调用是否成功的标志。
    * - error
      - 字符串
      - 如果交易请求解析或者处理出错，返回错误类型码。
    * - error_code
      - 字符串
      - 与error关联的整形值。
    * - type
      - 字符串
      - 返回类型      
    * - error_message
      - 字符串
      - 错误原因的描述。

成功示例

.. code-block:: json

    {
        "ledger_current_index":5397,
        "validated":false,
        "account_data":{
              "Account":"zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
              "OwnerCount":0,
              "PreviousTxnLgrSeq":4988,
              "LedgerEntryType":"AccountRoot",
              "index":"2B6AC232AA4C4BE41BF49D2459FA4A0347E1B543A4C92FCEE0821C0201E2E9A8",
              "PreviousTxnID":"E796A7B6A53900928F8016880A1593CD627B1FD4909532BBDDEBE274B815A4B2",
              "Flags":0,
              "Sequence":44,
              "Balance":"99999984779956509"
        }
    }

失败示例

.. code-block:: json

      {
            "error_message":"Table does not exist.",
            "request":{
            "signature":"3045022100A72D868B70FE66FDCD78DE2F411C751549B2A3F5D438AA14ABA251E8FFCE90B2022061C52B78311E6F2669E0293225602C3597942751160E82B4C852920AB7DDC76C",
            "tx_json":{
                  "LedgerIndex":7432,
                  "Account":"zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
                  "Owner":"zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
                  "Raw":"[[], {\"name\":\"hello\"}]",
                  "Tables":[
                    {
                      "Table":{
                      "TableName":"tTable2"
                      }
                    }
                  ]
                  },
                  "id":2,
                  "publicKey":"0330E7FC9D56BB25D6893BA3F317AE5BCF33B3291BD63DB32654A313222F7FD020",
                  "signingData":"{\"LedgerIndex\":7432,\"Account\":\"zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh\",\"Owner\":\"zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh\",\"Tables\":[{\"Table\":{\"TableName\":\"tTable2\"}}],\"Raw\":\"[[], {\\\"name\\\":\\\"hello\\\"}]\"}",
                  "command":"r_get"
            },
            "final_result":true,
            "error_code":76,
            "id":2,
            "error":"tabNotExist",
            "type":"response",
            "status":"error"
      }

------

基础接口
===========

.. _javaAS:

-------------------
as
-------------------

.. code-block:: java

  public void as(String address, String secret);

提供操作账户信息，指明一个全局的操作账户。

.. warning::
    部分接口与节点进行交互操作前，需要指明一个全局的操作账户，这样避免在每次接口的操作中频繁的提供账户。再次调用该接口即可修改全局操作账户。


参数
--------


1. ``address``  - ``String``: 账户地址.
2. ``secret``  - ``String``: 账户私钥


返回值
--------


示例


.. code-block:: java

    //提供操作账户信息
    c.as("zP8Mum8xaGSkypRgDHKRbN8otJSzwgiJ9M", "xcUd996waZzyaPEmeFVp4q5S3FZYB");

------

-------------------
use
-------------------

.. code-block:: java

  public void use(String address);

use接口主要使用场景是针对ChainSQL的表操作，为其提供表的 **拥有者账户地址** 。use接口和as接口的区别如下：

- as接口的目的是设置全局操作账户GU，这个GU在表操作接口中扮演表的操作者，并且默认情况下也是表的拥有者（即表的创建者）。
- 当as接口设置的账户不是即将操作的表的拥有者时，即可能通过表授权获得表的操作权限时，需要使用use接口设置表的拥有者地址。
- use接口只需要提供账户地址即可，设置完之后，表的拥有者地址就确定了，再次使用use接口可以修改表拥有者地址。


参数
----------


1. ``address`` - ``String``: 表的拥有者的账户地址


返回值
----------



示例


.. code-block:: java

    c.use("zP8Mum8xaGSkypRgDHKRbN8otJSzwgiJ9M");

------

.. note:: 进行表操作前必须调用use,来指定表的所有者。因为不同用户可能存在同名的表名

-------------------
connect
-------------------

.. code-block:: java

    public Connection connect(String url);
    public Connection connect(String url,String serverCertPath,String storePass);
    public Connection connect(String url,final Callback<Client> connectCb);
    public Connection connect(String url,final Callback<Client> connectCb,final Callback<Client> disconnectCb);
    public Connection connect(String url,String serverCertPath,String storePass,final Callback<Client> connectCb);
    public Connection connect(String url,String serverCertPath,String storePass,final Callback<Client> connectCb,
                       final Callback<Client> disconnectCb);

连接一个 ``websocket`` 地址.如果需要与节点进行交互，必须设置该节点的websocket地址。

参数
-------------------


1. ``url`` - ``String``: 节点的websocket访问地址,格式为:"ws://127.0.0.1:5006".
2. ``serverCertPath`` - ``String``: 认证路径.
3. ``storePass`` - ``String``: 认证密码
4. ``connectCb`` - ``Callback<Client>``:    已连接节点后的回调
5. ``disconnectCb`` - ``Callback<Client>``: 与节点断开连接后的回调

.. note:: 如果连接的websocket地址是域名地址，需要注意：按 ``rfc952`` 规范，主机名不可以包含下划线，否则会连接失败

返回值
--------------------


``Connection`` - 与节点连接后的对象


示例


.. code-block:: java

    // 同步连接
    // 如果无法建立连接,会抛出java.net.ConnectException;
    String url = "ws://192.168.0.162:6006";
    c.connect(url);

.. code-block:: java

  // 异步连接
  c.connect("ws://127.0.0.1:6006", new Callback<Client>() {
    @Override
    public void called(Client args) {

      System.out.println("Connected");
    }
  }, new Callback<Client>() {
    @Override
    public void called(Client args) {

      System.out.println("Disconnected  ");
    }
  });

------

-------------------
submit
-------------------

.. code-block:: java

  public JSONObject submit();
  public JSONObject submit(Callback cb)
  public JSONObject submit(SyncCond cond);

submit有3个重载函数，分为异步和同步，用户可以根据需求使用不同函数。

使用说明:

- 针对ChainSQL的交易类型的操作，需要使用submit接口执行提交上链操作，交易类型的操作是指需要进行区块链共识的操作。
- 还有一类ChainSQL的查询类操作，不需要使用submit接口，不需要进行区块链共识。 
- submit接口有使用前提，需要事先调用其他操作接口将交易主体构造，比如创建数据库表，需要调用createTable接口，然后调用submit接口，详细使用方法在具体接口处介绍。

参数
-------------------


1. ``cb``   - ``Callback``: 异步接口，参数为 回调函数
2. ``cond`` - ``SyncCond``: 同步接口，参数为 枚举类型;

.. code-block:: java

  enum SyncCond {
      send_success,    // 交易提交成功
      validate_success,// 交易共识通过
      db_success       // 交易成功同步到数据库
  };


返回值
-------------------


``JSONObject`` - JSON对象.可参考 :ref:`接口返回值 <json-return>`

1. 执行成功，则 ``JsonObject`` 中包含两个字段：

  * ``status`` - ``String`` : 为提交时的设定值，如果没有，则默认为 **send_success** 
  * ``tx_hash`` - ``String`` : 交易哈希值，通过该值可以在链上查询交易

2. 执行失败，有以下几种情况:

  * 第一种交易共识前的字段信息有效性检测出错，``JsonObject`` 中主要包含以下字段：

    - ``error`` - ``String`` : 错误类型；
    - ``error_message`` - ``String`` : 错误具体描述。

  * 第二种交易提交之后共识出错，``JsonObject`` 中包含以下字段：

    - ``error`` - ``String`` : 错误类型码，可参考 :ref:`交易类错误码 <tx-errcode>`；
    - ``error_message`` - ``String`` : 错误具体描述。

  * 第三种交易提交共识后出错，主要是数据库入库操作中的错误，``JsonObject`` 中包含以下字段：

    - ``status`` - ``String`` : 错误类型，有以下可能字段：

      - db_error
      - validate_timeout
      - db_noTableExistInDB
      - db_noDbConfig
      - db_noSyncConfig
      - db_noAutoSync

    - ``tx_hash`` - ``String`` : 交易哈希值。
    - ``error_message`` - ``String`` : [**可选**]在错误类型为 **db_error** 的时候，会额外附加错误信息。

  * 入库相关错误说明如下：

  ====================  ================================================================================
  字段    	              解释
  ====================  ================================================================================
  db_error               	入库语句执行失败
  validate_timeout        交易共识超时
  db_noTableExistInDB 	  要操作的表在数据库中不存在
  db_noDbConfig        	  未配置数据库
  db_noSyncConfig         加密表未配置解密私钥
  db_noAutoSync 	        配置文件中auto_sync为0，无法建表
  db_acctSecretError      加密表解密私钥错误
	db_notInSync			      表不在同步列表中
  ====================  ================================================================================

示例


.. code-block:: java


  // 1、
  JSONObject obj = c.pay(sNewAccountId,"10").submit(SyncCond.validate_success);

  if(obj.has("error_message")){
    System.out.println(obj);
  }else {
    JSONObject objRet = new JSONObject();
    objRet.put("status", obj.getString("status"));
    System.out.println( objRet);
  }

  // 2、
  obj = c.pay(sNewAccountId,"10").submit(new Callback<JSONObject>() {
          @Override
          public void called(JSONObject args) {
            System.out.println(args);
          }
        });



成功输出

.. code-block:: json

    {
      "status": "validate_success"
    }
    {
      "type": "singleTransaction",
      "transaction": {
        "Account": "zpMZ2H58HFPB5QTycMGWSXUeF47eA8jyd4",
        "Destination": "zcs4x6e64E3Jw59CPSPyZAtYmVuGxUM4gb",
        "TransactionType": "Payment",
        "TxnSignature": "3045022100815CA715BC88E199C36D9215F74E8AFBCD4F3863DF0521FCD45B64104D5F6BCA02206C9E6704B42941F52F02BE8E14EECF85CCC083E58E2B46A68BA99EC1C3F1E1DF",
        "SigningPubKey": "02080EF3E62711D21A502DC6B6290E2819474AA7701271D82CDFF73F65AFEE7276",
        "Amount": "10000000",
        "Fee": "11",
        "Flags": 2147483648,
        "Sequence": 20,
        "LastLedgerSequence": 23813,
        "hash": "C6D8EE4DC86105ECF07CD31AB96205A95B8F41972FD6C1D7E547D593B9B651AC"
      },
      "status": "validate_success"
    }


失败输出

.. code-block:: json

    {
      "error_message": "Insufficient ZXC balance to send.",
      "tx_json": {
        "Account": "zpMZ2H58HFPB5QTycMGWSXUeF47eA8jyd4",
        "Destination": "zcs4x6e64E3Jw59CPSPyZAtYmVuGxUM4gb",
        "TransactionType": "Payment",
        "TxnSignature": "30450221009BE2564CAEDF0819668A07F29F6CB30870ADBF95A0CA09DB9E31987CDFDE4F73022023141C6162142880EC2C110821409186C544D18659424CB8488A70BE0888C25D",
        "SigningPubKey": "02080EF3E62711D21A502DC6B6290E2819474AA7701271D82CDFF73F65AFEE7276",
        "Amount": "10000000000",
        "Fee": "11",
        "Flags": 2147483648,
        "Sequence": 6,
        "LastLedgerSequence": 23597,
        "hash": "5C6E284AAA281A7D9D5139B800E8C9CD19C27D5F3657F46C4BDE910345BE6634"
      },
      "error_code": 104,
      "error": "tecUNFUNDED_PAYMENT",
      "status": "error"
    }
    {
      "error_message": "Insufficient ZXC balance to send.",
      "tx_json": {
        "Account": "zpMZ2H58HFPB5QTycMGWSXUeF47eA8jyd4",
        "Destination": "zcs4x6e64E3Jw59CPSPyZAtYmVuGxUM4gb",
        "TransactionType": "Payment",
        "TxnSignature": "3044022005595A62EC4127C84E22782AA7AA287B4562A991E898A99155E194B991D208F502205969821251C596AF7353DA75F7105D0AC36741F54D5579BAC81591C3F49B554F",
        "SigningPubKey": "02080EF3E62711D21A502DC6B6290E2819474AA7701271D82CDFF73F65AFEE7276",
        "Amount": "10000000000",
        "Fee": "11",
        "Flags": 2147483648,
        "Sequence": 7,
        "LastLedgerSequence": 23597,
        "hash": "19184535CC6B7FF2DA7AF965071AB50EF15DDA3B60A7E1B28BA43004DA31B5D0"
      },
      "error_code": 104,
      "error": "tecUNFUNDED_PAYMENT",
      "status": "error"
    }


------

.. _pay-系统数字资产:

-----------------------
pay(转账系统数字资产)
-----------------------

.. code-block:: java

  public Ripple pay(String accountId, String value);

给用户转账,新创建的用户在转账成功之后才能正常使用(激活)。

.. note::
    这个函数有重载，除了可以转账系统数字资产外，还可以转账配置的数字资产：:ref:`pay(转账数字资产) <pay-数字资产>`

.. warning::
    1.5.1版本之前 pay 方法直接调用就行，现在需要.submit


参数
----------


1. ``accountId``   - ``String``: 接收转账方地址
2. ``value``       - ``String``: 转账金额（单位:ZXC）,默认情况下，最少需要5个ZXC才能激活一个账户


返回值
----------


``Ripple`` - Ripple对象,后面一般接submit进行连续操作,如示例。


.. _my-reference-pay-sample:


示例


.. code-block:: java

    JSONObject obj = c.pay("zcs4x6e64E3Jw59CPSPyZAtYmVuGxUM4gb","10000").submit(SyncCond.validate_success);
    if(obj.has("error_message")){
      System.out.println("激活或转账失败。 失败原因: " + obj.getString("error_message"));
    }else {
      System.out.println("激活或转账成功");
    }

------


.. _GenerateAddress:

-------------------
generateAddress
-------------------

.. code-block:: java

  public JSONObject generateAddress();
  public JSONObject generateAddress(String secret);
  public JSONObject generateAddress(JSONObject options);

生成一个新的ChainSQL账户。但是此时该账户未在链上有效，需要链上有效账户对新账户发起pay操作，新账户才有效。


参数
----------


1. ``secret``  - ``String``: 账户私钥
2. ``options`` - ``JSONObject``: : 指定算法类型或账户私钥,字段如下：

	* ``algorithm`` - ``String`` : 算法类型包括 ed25519 , secp256k1 , softGMAlg
	* ``secret`` - ``String`` : 账户私钥

返回值
----------


``JSONObject`` - JSON对象 

    * ``secret`` - ``String`` : 新账户私钥，是原始十六进制的base58编码。
    * ``address`` - ``String`` : 新账户地址，是原始十六进制的base58编码；
    * ``publicKey`` - ``String`` : 新账户公钥，是原始十六进制的base58编码；


示例


.. code-block:: java

    String rootSecret = "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb";
    System.out.println( c.generateAddress() );
    System.out.println( c.generateAddress(rootSecret) );
    
    // 指定生成国密算法的账户
    JSONObject options = new JSONObject();
    options.put("algorithm","softGMAlg");
    JSONObject ret = c.generateAddress(options);

输出:

.. code-block:: json    

  {
    "address": "zxSscHsDfb8XZj3tgCgFaTcZgDe2rveySE",
    "secret": "xpmWr564b9BLo8ZA2ysw7Vicrh76h",
    "publicKey": "cBQw6iDUjN2Z3Aca56TBRt5R9vhmsX5R7SSHnVo9vnE4pYUjAiV6"
  }
  {
    "address": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
    "secret": "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb",
    "publicKey": "cBQG8RQArjx1eTKFEAQXz2gS4utaDiEC9wmi7pfUPTi27VCchwgw"
  }
  {
    "address":"zMD3r5ZNUBbmLs68FBB7AK46fm7ScxDLQH",
    "secret":"p98kY6F2MA6H8z1nAXMjkAaFrTg4Pqc1b3dJ82Xe6gxn1kv5h2i",
    "publicKey": "pYvV28qfGdeymQ5f21qpQhbtmFQK36566gZnaJv8VuHoyiZActWerJAkK3MtZWdXV1DggJCdCzkhwmefK2YHebho9QhgBArX"
  }

------

-------------------
validationCreate
-------------------

.. code-block:: java

  public JSONObject validationCreate();
  public JSONArray  validationCreate(int count);
  public JSONObject validationCreate(JSONObject options);

生成验证key


参数
----------


1. ``count`` - ``int``: 生成的key的个数
2. ``options`` - ``JSONObject``: : 指定算法类型或验证key的私钥,字段如下：

	* ``algorithm`` - ``String`` : 算法类型包括 softGMAlg
	* ``secret`` - ``String`` : 验证key的私钥

返回值
----------


1. ``JSONObject`` -  一个有效的key,结构为{"seed":xxx,"publickey":xxx}
2. ``JSONArray``  -  一个或多个有效的key，每个key的结构同上


示例1

.. code-block:: java

    JSONObject json = c.validationCreate();
  
输出:

.. code-block:: json

    {
      "seed"     :"xnaKLBqkwZxCxCNk1LokjAekUQaWT",
      "publickey":"n9KrLAkaHZk3kns6TfZS9mRJmPrNJLjARxM8qUtM2CXpBpUcyTdD"
    }

------

示例2

.. code-block:: java

    JSONArray jsonArr = c.validationCreate(2);

输出:

.. code-block:: json

    [
     {"seed":"xxuvaugPX5ZTCcFvKdd9vzhAHFd27","publickey":"n94U13Uap8LQaDJQtbV9HGcgWH8qzWPscpZdqMv6SPz6U5Zazcdq"},
     {"seed":"xxqE8bBLKrKMMEpjqS4gwLwmRAGm6","publickey":"n9MdENDVAaQSDnmFdv3BzRbuuNH1AvUmpy8D7LMfN3evEx82us4Z"}
    ]

------

示例3

.. code-block:: java

    // 生成国密版本的验证key
    JSONObject gmOptions = new JSONObject();
    gmOptions.put("algorithm","softGMAlg");
    JSONObject validateCreate = c.validationCreate(gmOptions);

输出:

.. code-block:: json

    {
      "seed":"pf9D7hEKj2eDCCC8hkUuYshyncY37bVN4rnCMixE1Xyr3JAqjjc","publickey":"pEnYfGwmQ9XsPpXJ3Wj5efA8DnmNH8urEGZkAkW46qSBUbCYW4CS5TSFps6xBg3iCoQzFgQZwRiXAK8H7ZBQ4q7qUyyU5d9p"
    }   

------

-------------------
getServerInfo
-------------------

.. code-block:: java

  public JSONObject getServerInfo();

获取区块链信息.


参数
----------


返回值
----------


``JSONObject`` - 区块链信息.

1. ``JsonObject`` : 包含区块链基础信息，详细字段信息可在 **命令行接口 server_info** 中查看， 主要字段如下：

	* ``buildVersion`` - ``String`` : 节点程序版本
	* ``complete_ledgers`` - ``String`` : 当前区块范围
	* ``peers`` - ``Number`` : peer节点数量
	* ``validation_quorum`` - ``Number`` : 完成共识最少验证节点个数



示例


.. code-block:: java

    JSONObject json = c.getServerInfo();

输出:

.. code-block:: json

    {
        "info":{
          "build_version":"0.30.3+DEBUG",
          "peers":0,
          "hostid":"DESKTOP-M5MSU6I",
          "last_close":{
              "proposers":0,
              "converge_time_s":1.985
          },



          "validation_quorum":1,



          "complete_ledgers":"1-7888",
          "pubkey_validator":"n9KigtPo6tPTNSuyaz7AtHk7XijPZwEUuF8LfaQQhjmSwFBenk6Q",
          "server_state":"proposing",
          "validator_list_expires":"unknown"
        }
    }

------

-------------------
getChainInfo
-------------------

.. code-block:: java

    public JSONObject getChainInfo();

获取链信息


参数
----------



返回值
----------


1. ``JsonObject`` : 包含区块链基础信息，主要字段如下：

	* ``chain_time`` - ``int`` : 区块链运行时间
	* ``tx_count`` - ``JSONObject`` : 见  :ref:`tx_count <trans-count-return>`.

.. _get-chain-info-sample:


示例


.. code-block:: java

    System.out.println(c.getChainInfo());

输出:

.. code-block:: json

    {
        "chain_time":517500,
        "tx_count":{
              "all":562,
              "chainsql":502
        }
    }

------

-------------------
getUnlList
-------------------

.. code-block:: java

    public JSONObject getUnlList();

获取信任公钥列表


参数
----------



返回值
----------


``JSONObject`` - 信任公钥列表,详细字段见示例


示例


.. code-block:: java

    JSONObject json = c.getUnlList();

输出:

.. code-block:: json

    {
        "unl":[
          {
            "trusted":true,
            "pubkey_validator":"n9KigtPo6tPTNSuyaz7AtHk7XijPZwEUuF8LfaQQhjmSwFBenk6Q"
          }
        ]
    }

------

-------------------
getAccountInfo
-------------------

.. code-block:: java

    public JSONObject getAccountInfo(address);

从链上请求查询账户信息。


参数
----------


1. ``address`` - ``String``: 账户地址


返回值
----------


1. ``JsonObject`` : 包含账户基本信息。正常返回主要字段如下：

  * ``Sequence`` - ``Number`` : 该账户交易次数
  * ``Balance`` - ``String`` : 账户ZXC系统数字资产的余额


示例


.. code-block:: java

    String rootAddress = "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh";
    System.out.println(c.getAccountInfo(rootAddress));

输出:

.. code-block:: json    


      {
          "ledger_current_index":6363,
          "validated":false,
          "account_data":{
                "Account":"z3VGAJo59RWZ23CeM7zwMGVvsbWF8HabHf",
                "OwnerCount":1,
                "PreviousTxnLgrSeq":5113,
                "LedgerEntryType":"AccountRoot",
                "index":"6AFC4F3E9B190B8B6711BCC33EF03BF2D3FA1D374368BB3F8E80E5B744E8AACD",
                "PreviousTxnID":"8441BD3D6777A647B0D63DEC5D37C8AF67DE38248D7045E3900E14B3AE56DC8B",
                "Flags":0,
                "Sequence":3,
                "Balance":"399848588"
          }
      }

------

-------------------
getTransactionCount
-------------------

.. code-block:: java

    private JSONObject getTransactionCount();

获取交易数量， **getChainInfo** 中调用，存于返回的 ``tx_count`` 字段中


参数
----------


.. _trans-count-return:


返回值
----------

1. ``JsonObject`` : 获取交易数量

	* ``all``      - ``int`` : 所有交易数量
	* ``chainsql`` - ``int`` : chainsql交易数量


示例


:ref:`示例 <get-chain-info-sample>`

------

-------------------
getLedger
-------------------

.. code-block:: java

    // 同步接口
    public JSONObject getLedger();                    // 获取最新账本信息
    public JSONObject getLedger(Integer ledger_index);// 获取指定索引账本信息

    // 异步接口
    public void getLedger(Callback<JSONObject> cb);
    public void getLedger(Integer ledger_index,Callback<JSONObject> cb);


获取账本信息


参数
----------


1. ``ledger_index`` - ``Integer``: 账本索引

2. ``cb``           - ``Callback`` : 异步接口，参数为 回调函数


返回值
----------



1. ``JsonObject`` : 区块信息。正常返回主要字段如下：

    * ``ledger_index``     - ``int`` : 区块号 
    * ``ledger_hash``      - ``String`` : 区块哈希 
    * ``parent_hash``      - ``String`` : 父区块哈希 
    * ``close_time``       - ``String`` : 区块生成时间 
    * ``transactions``     - ``JSONArray`` : 区块包含的交易数组，数组元素为交易的哈希值 


示例


.. code-block:: java

    JSONObject json = c.getLedger(3833);

输出:

.. code-block:: json


  {
    "ledger": {
      "close_flags": 0,
      "ledger_index": "3833",
      "seqNum": "3833",
      "account_hash": "18D4485E6AC513C877487FF2F9EDC8084E9D7148556A6BA9E3CFA791C4F9E9BF",
      "close_time_resolution": 10,
      "accepted": true,
      "close_time": 643369840,
      "transactions": ["8AB4C5EBB3DCF0B828EAD31F273EDA6C24963F10F41E6925585EFBA2AEA9153F"],
      "close_time_human": "2020-May-21 09:50:40",
      "ledger_hash": "0C3F5E1D7532D8A13756FE721B00AE1877FE2A6AC800E34F11879E72D2F1B0EA",
      "total_coins": "99999999999491102",
      "closed": true,
      "totalCoins": "99999999999491102",
      "parent_close_time": 643369800,
      "hash": "0C3F5E1D7532D8A13756FE721B00AE1877FE2A6AC800E34F11879E72D2F1B0EA",
      "parent_hash": "7057B6F06A3D198F9CA52AA454346F8FBEBDA0680DF9496050F354AEE7BCF883",
      "transaction_hash": "BF019692876F5EA793BA66D8E871C781DA39C255C0672480B4F295842E4B8CBD"
    }, 
    "validated": true,
    "ledger_index": 3833,
    "ledger_hash": "0C3F5E1D7532D8A13756FE721B00AE1877FE2A6AC800E34F11879E72D2F1B0EA"
  } 


------

-------------------
getLedgerVersion
-------------------

.. code-block:: java

    public JSONObject getLedgerVersion();
    public void       getLedgerVersion(Callback<JSONObject> cb);

获取最新区块高度（区块号）


参数
----------


1. ``cb``      - ``Callback`` : 异步接口，参数为 回调函数


返回值
----------


``JSONObject`` 

  * ``ledger_current_index``  - ``int`` : 最新区块高度


示例


.. code-block:: java

  // 同步接口
  System.out.println(c.getLedgerVersion());

  // 异步接口
  c.getLedgerVersion( new Callback<JSONObject>() {
    @Override
    public void called(JSONObject args) {
      System.out.println(args);
    }
  });

输出:

.. code-block:: json    

  {
    "ledger_current_index":13796
  }

------

.. _javaledger_txs:

-------------------
getLedgerTxs
-------------------

.. code-block:: java

    // 同步接口
    public JSONObject getLedgerTxs(Integer ledgerSeq, boolean bIncludeSuccess, boolean bIncludefailure);

    // 异步接口
    public void getLedgerTxs(Integer ledgerSeq, boolean bIncludeSuccess, boolean bIncludefailure, Callback<JSONObject> cb);

获取区块中的成功、失败交易数，以及错误交易的hash及错误码。

参数
----------

1. ``ledgerSeq`` - ``Integer``: 账本索引

2. ``bIncludeSuccess`` - ``boolean``: 是否要获取所有成功交易的哈希

3. ``bIncludefailure`` - ``boolean``: 是否要获取所有失败交易的哈希和错误码

4. ``cb`` - ``Callback`` : 异步接口，参数为 回调函数

返回值
----------

1. 可参考RPC中的\ :ref:`ledger_txs <rpcledger_txs>`\ 接口。

示例


.. code-block:: java

    JSONObject json = c.getLedgerTxs(2,true,true);

输出:

.. code-block:: json

  {
    "txn_failure_detail":[],
    "txn_failure":0,
    "txn_success":1,
    "ledger_index":2,
    "txn_success_detail":[
      {
        "hash":"41521F8535F1A6A581528BFB56F3085F9D4B09EBE913A6C854B1C9453BD0C46D",
        "transaction_result":"tesSUCCESS"
      }
    ]
  }

.. warning::

  此API为\ :ref:`PoP共识版本 <PoP共识版本>`\ 新增API，只适用于PoP共识版本。

------

------------------------
getAccountTransactions
------------------------

.. code-block:: java

  public void       getAccountTransactions(String address,Callback cb);
  public void       getAccountTransactions(String address,int limit,Callback<JSONObject> cb);
  public JSONObject getAccountTransactions(String address,int limit);
  public JSONObject getAccountTransactions(String address);     

查询某账户的交易


参数
----------


1. ``address`` - ``String``: 查询交易的账户地址;
2. ``limit``   - ``int``: 获取的最大的交易数量;
3. ``cb``      - ``Callback`` : 异步接口，参数为 回调函数


返回值
----------


1. ``JsonObject`` : 包含账户基本信息。正常返回主要字段如下：

    * ``ledger_index_max`` - ``int`` : 最大区块号 
    * ``limit``            - ``int`` : 获取的最大的交易数量，默认为20
    * ``ledger_index_min`` - ``int`` : 最小区块号 
    * ``transactions``     - ``JSONArray`` : 交易数组，详见示例 
    * ``account``          - ``String`` : 查询交易的账户地址



示例


.. code-block:: java

  String rootAddress   = "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh";
  JSONObject  obj      =   c.getAccountTransactions(rootAddress,30);
  System.out.println(obj);

  // Example 2
  c.getAccountTransactions(rootAddress, 30, new Callback<JSONObject>() {

    public void called(JSONObject args) {
      System.out.println(args);
    }
  });


输出

.. code-block:: json

    {
        "ledger_index_max":911,
        "limit":30,
        "ledger_index_min":1,
        "transactions":[
          {
          "tx":{
              "date":609906292,
              "Account":"zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
              "TransactionType":"TableListSet",
              "ledger_index":431,
              "SigningPubKey":"0330E7FC9D56BB25D6893BA3F317AE5BCF33B3291BD63DB32654A313222F7FD020",
              "Fee":"151401",
              "Raw":"[{\"NN\":1,\"field\":\"id\",\"length\":11,\"PK\":1,\"type\":\"int\",\"UQ\":1}, {\"default\":null,\"field\":\"name\",\"length\":50,\"type\":\"varchar\"}, {\"field\":\"age\",\"type\":\"int\"}]",
              "Flags":2147483648,
              "Sequence":1,
              "LastLedgerSequence":434,
              "TxnSignature":"304402207E31292196C8004021A7A8D021E1EC39E2E997149DE886AF8AC3DFBFF17EAADA02200467DA6734FA000A3915806C4DC951F7307D3DFEEC0A6D75E715D1E5E51C54DC",
              "Tables":[
              {
              "Table":{
              "TableName":"c1235",
              "NameInDB":"79D9C64B0297611ED6A642B1B5980C9C05E8ECBD"
              }
              }
              ],
              "inLedger":431,
              "OpType":1,
              "hash":"7A836046F485A7F94A205476AFC4D4BB12EFE9E2C0EFA31402406F774DC86094"
              },
              "validated":true                      
          }                       
        ],
        "account":"zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh"
    }


------

------------------------
getTransaction
------------------------

.. code-block:: java

   public JSONObject getTransaction (String hash);     //同步
   public void getTransaction(String hash,Callback cb);//异步

查询某个hash下的交易信息


参数
----------


1. ``hash``    - ``String``:  交易哈希值;
2. ``cb``   - ``Callback`` : 异步接口，参数为一回调函数


返回值
----------


``JSONObject`` - 详细格式见示例


示例


成功

.. code-block:: json

  {
    "date": 610459191,
    "Account": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
    "TransactionType": "TableListSet",
    "ledger_index": 37885,
    "SigningPubKey": "0330E7FC9D56BB25D6893BA3F317AE5BCF33B3291BD63DB32654A313222F7FD020",
    "Fee": "151401",
    "Raw": "[{\"NN\":1,\"field\":\"id\",\"length\":11,\"PK\":1,\"type\":\"int\",\"UQ\":1}, {\"default\":null,\"field\":\"name\",\"length\":50,\"type\":\"varchar\"}, {\"field\":\"age\",\"type\":\"int\"}]",
    "Flags": 2147483648,
    "Sequence": 270,
    "LastLedgerSequence": 37888,
    "metaChain": {
      "TableChain": [
        {
          "NameInDB": "2A2494F5288EFF4A0C4A1E1388B3372CE130F35D",
          "PreviousHash": "",
          "NextHash": ""
        }
      ]
    },
    "TxnSignature": "304402205411DBA2A02A76867A637A8F8F02EFE411379D768F1D46928D425F274AE46055022002CE6971A749F6945201529E8530316A32DAED58BE2298FC280317E782873700",
    "validated": true,
    "meta": {},
    "Tables": [
      {}
    ],
    "inLedger": 37885,
    "OpType": 1,
    "hash": "A67DFF659F95A72EDE609FC128EEC0BABA0E7F36FCA770F281BDF7F6CA8A7BD6"
  }

失败

.. code-block:: json

  {
    "error_message": "Transaction not found.",
    "request": {
      "id": 1,
      "command": "tx",
      "transaction": "A67DFF659F95A72EDE609FC128EEC0BABA0E7F36FCA770F281BDF7F6CA8A7BD7"
    },
    "error_code": 29,
    "id": 1,
    "error": "txnNotFound",
    "type": "response",
    "status": "error"
  }



------

----------------
sign
----------------

.. code-block:: java

    public JSONObject sign(JSONObject tx,String secret);  //对交易签名
    public byte[]     sign(byte[] message,String secret); //对普通字符串进行签名

签名接口。


参数
----------


1. ``tx``      - ``JSONObject``:  交易对象，不同交易类型，结构不同
2. ``message`` - ``byte[]``    : 要签名的字符串
3. ``secret``  - ``String``    :  签名私钥


返回值
----------


1. ``JsonObject`` : 

	* ``tx_blob``  -  签名后的交易
	* ``hash``     -  哈希值

2.  ``byte[]`` - 签名后的字符串


示例


.. code-block:: java

  JSONObject obj = new JSONObject();
  JSONObject tx_json = new JSONObject();
  tx_json.put("Account", "rHb9CJAWyB4rj91VRWn96DkukG4bwdtyTh");
  tx_json.put("Amount", "10000000000");
  tx_json.put("Destination", "rBuLBiHmssAMHWQMnEN7nXQXaVj7vhAv6Q");
  tx_json.put("TransactionType", "Payment");
  tx_json.put("Sequence", 2);
  obj.put("tx_json", tx_json);

  JSONObject res = c.sign(obj, "snoPBrXtMeMyMHUVTgbuqAfg1SUTb");
  System.out.println(res);


输出

.. code-block:: json

  {
      "tx_blob":"120000228000000024000000026140000002540BE40068400000000000000B73210330E7FC9D56BB25D6893BA3F317AE5BCF33B3291BD63DB32654A313222F7FD0207446304402203C7734114C1F9EEB16C775D49B8392636361CC844839483979BCBF0EE39F4B390220233BE3C714E1374415E2B4F2EF155579DB2E97BA627A3B16A0342A78975E06BF8114B5F762798A53D543A014CAF8B297CFF8F2F937E883140EDBA2E944E1751BDA1FC4ACC7116C9B19394996",
      "hash":"706BF05DD43730C312A3B3F2F9BB7B16B04DEC827E29A38AD2C5C390ED0E75D2"
  }

.. code-block:: java

  String hello = "helloworld";
  byte[] signature = c.sign(hello.getBytes(), rootSecret);
  System.out.println( Util.bytesToHex(signature));

输出

.. code-block:: java

  "3044022002B4B80066E900E1EB4DB6DD843F8A31D5237E6A53536ED113694B2714EAF03902203BE9450E143CCAD62642ED7574377F2A580C3EA5ABC6A48E03A5126E2B3A45AA"


------

----------------
signFor
----------------

.. code-block:: java

    public JSONObject signFor(JSONObject tx,String secret);

用于交易多方签名


参数
----------


1. ``tx``     - ``JSONObject``: 交易JSON
2. ``secret`` - ``String``    : 签名私钥


返回值
----------


1. ``JSONObject`` :主要参数说明(格式见示例输出)

      * ``Account``        -  签名账户
      * ``TxnSignature``   -  签名后的交易
      * ``SigningPubKey``  -  签名公钥    
      * ``hash``           -  哈希值


示例
----------

.. code-block:: java

      JSONObject obj = new JSONObject();
      obj.put("secret", "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb");
      obj.put("account","zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh");
      JSONObject tx_json = new JSONObject();
      tx_json.put("Account", "zKvHeBUtEoNRW1wtvA42tfJx1bh7pqxZmT");
      tx_json.put("Amount", "1000000000");
      tx_json.put("Destination", "zMcXHEkD78T1pwAgG2pf6QWALyBKF1YvD1");
      tx_json.put("TransactionType", "Payment");
      tx_json.put("Sequence", 2);
      obj.put("tx_json", tx_json);

      JSONObject res = c.signFor(obj, "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb");
      System.out.println("sign_for payment signer:");
      System.out.println(res);

输出

.. code-block:: json

    {
      "Signer":{
        "Account":"zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
        "TxnSignature":"3045022100CF059AC2B07D3CAC1F3E877F0E81675731BDA21F6BF51000FC3DE3EE579A343002207320DB8614F45925D1AA392E0E61D468D1F3BA269574ED24F3789A887CC89F12",
        "SigningPubKey":"0330E7FC9D56BB25D6893BA3F317AE5BCF33B3291BD63DB32654A313222F7FD020"
      },
      "hash":"EB11BE3944E183EBD89A5CDB1EE55664DD2289D05E84BF99F47147108A5A728B"
    }


------

-------------------
getTableNameInDB
-------------------

.. code-block:: java

    public JSONObject getTableNameInDB(String owner,String tableName);

查询表在数据库中的记录的名字，在外定义的表名，经过ChainSQL会进行格式转换，此接口查询转换之后的名字。


参数
----------


1. ``owner`` - ``String``: 账户地址,表的拥有者；
2. ``tableName`` - ``String``:  原始表名


返回值
----------


 ``JSONObject`` : 主要参数如下

      * ``nameInDB``       -  数据库中的表名。该字段在正确时返回。 
      * ``error_message``  -  显示错误信息。该字段在错误时返回。
      

示例
----------

.. code-block:: java

  String rootAddress = "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh";
  System.out.println(c.getTableNameInDB(rootAddress,"test1"));

成功

.. code-block:: json

  {
    "nameInDB":"2A2494F5288EFF4A0C4A1E1388B3372CE130F35D"
  }

失败

.. code-block:: json

  {
    "error_message": "Table does not exist.",
    "request": {
      "id": 1,
      "tablename": "test1",
      "account": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
      "command": "g_dbname"
    },
    "error_code": 76,
    "id": 1,
    "error": "tabNotExist",
    "type": "response",
    "status": "error"
  }  

------

-------------------
getTableAuth
-------------------

.. code-block:: java

    public JSONObject getTableAuth(String owner,String tableName);
    public JSONObject getTableAuth(String owner,String tableName,List<String> accounts);
    public void getTableAuth(String owner,String tableName,Callback<JSONObject> cb);
    public  void getTableAuth(String owner,String tableName,List<String> accounts,Callback<JSONObject> cb)


获取某张表的授权列表


参数
----------


1. ``owner`` - ``String``: 表的拥有者地址
2. ``tableName`` - ``String``: 表名
3. ``accounts`` - ``List<String>``: 指定账户地址列表，只查这个列表中相关地址的授权
4. ``cb`` - ``Callback<JSONObject>``:回调函数


返回值
----------


``JSONObject`` - 授权列表


示例
----------

.. code-block:: java

  String rootAddress = "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh";
  System.out.println(c.getTableAuth(rootAddress,"tableName"));

.. code-block:: Json

    {
          "owner":"z3VGAJo59RWZ23CeM7zwMGVvsbWF8HabHf",
          "tablename":"tTable2",
          "users":[
            {
              "authority":{
              "select":true,
              "insert":true,
              "update":true,
              "delete":true
              },
              "account":"z3VGAJo59RWZ23CeM7zwMGVvsbWF8HabHf"
            }
          ]
    }

------

-------------------
getAccountTables
-------------------

.. code-block:: java

    public JSONObject getAccountTables(String address,boolean bGetDetail);
    public void getAccountTables(String address,boolean bGetDetail,Callback<JSONObject> cb);

获取某个账户建的表


参数
----------


1. ``address`` - ``String``: 账户地址
2. ``bGetDetail`` - ``boolean``: 是否获取详细信息（建表的raw字段）
3. ``cb`` - ``Callback<JSONObject>``:回调函数


返回值
----------


``JSONObject`` - 用户建的表（数组），详细信息见示例


示例
----------

.. code-block:: java

  String rootAddress = "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh";
  System.out.println(c.getAccountTables(rootAddress,true));

输出:

.. code-block:: json

    {
      "tables":[
        {
        "create_time":608208080,
        "ledger_index":6222,
        "ledger_hash":"8443D58F41A6B391B2833E7A58F8E4587A361441EC1B66E182F5CAD0F02EFE7F",
        "raw":"AB8B11AE95B9DD145AF82052FE87FADBF23DA6336095848D1C2918699AEC772E853E4CD9107E2AAB0C304DB62C03A85373249680DC6D56B55852C572A279EFBE307417EA81868DCAA422A5E6C22404803CE37FF48EC1FC95F40522C7763E1DC8EDA9D9BA10E4D4E51910FBF7852ACF7455F5FFFC07AEBFB3F7BBBFEF8096B559872BB64D3E0E91B6332018088CC76D83A5E85A189717B9262067F66F2A89FDE6",
        "tx_hash":"EFE119F4EA1481400D050C54981D20E5AE40C293A1C9FD0D80C3F73AF3FF1673",
        "tablename":"b1",
        "nameInDB":"D06DC2D2A7A8CB40B637B95BF2A9BF8EF62C89CA",
        "table_exist_inDB":false,
        "confidential":true
        }
      ]
    }

------


.. _UseCertJava:

-------------------
useCert
-------------------

.. code-block:: java

  public void useCert(String userCert)

用于设置账户的CA证书信息。


参数
----------


1. ``userCert``  - ``String``: 用户证书的文件内容.


返回值
----------


示例

.. code-block:: java

    c.userCert("-----BEGIN CERTIFICATE-----\n
  MIIC2TCCAcGgAwIBAgICEEkwDQYJKoZIhvcNAQELBQAwczELMAkGA1UEBhMCQ04x\n
  EDAOBgNVBAgMB1RpYW5qaW4xEDAOBgNVBAcMB1RpYW5qaW4xFTATBgNVBAoMDENI\n
  SU5BU1NMIEluYzEpMCcGA1UEAwwgQ0hJTkFTU0wgQ2VydGlmaWNhdGlvbiBBdXRo\n
  b3JpdHkwHhcNMTkwOTA1MDgxOTA1WhcNMjAwOTA0MDgxOTA1WjA6MQswCQYDVQQG\n
  EwJDTjELMAkGA1UECAwCQkoxETAPBgNVBAoMCFBlZXJzYWZlMQswCQYDVQQDDAJS\n
  QzBWMBAGByqGSM49AgEGBSuBBAAKA0IABDDn/J1WuyXWiTuj8xeuW88zsykb1j2z\n
  JlSjEyIvf9AggBfOJ3FHVCCwgJsfIAr6ZK23X5psdD0+LtNE5+ydEiOjfjB8MAkG\n
  A1UdEwQCMAAwLwYJYIZIAYb4QgENBCIWIENISU5BU1NMIENlcnRpZmljYXRpb24g\n
  QXV0aG9yaXR5MB0GA1UdDgQWBBSbvkh1rza59QJoFkctRh8GWYO8cTAfBgNVHSME\n
  GDAWgBRcHyP6yOEhMcLYN/aI/NJvwlRDMzANBgkqhkiG9w0BAQsFAAOCAQEABwBp\n
  wG/USPuPXIgQQAI1MLCjBwBm0JBLa/iR7ilebthsSMqPA3r5++HlsauyoG9C8w12\n
  GQeiNWeMKLtHiMK+nqdEMu3/Qu+pfJGho8pNp/xkKoHQKrhwS2nXExJk/TWyqks+\n
  gV4lVHLgSt0KfDp4qZE2XjSxOSY0bxg0z65ILgG6kulMMfexPj9mWAP+9jkzrarB\n
  Bnwndq1E8SH80HNC1Yjj4P9tuVZH1XBRd8dDgC7X8bwa8uq9Fv3x53LvzvkSUL7a\n
  EwC2ruHjaFuOmI1uidXFwLaQ7ufbXFGE5CwYhOamPbf2evUagNnVtCAJASmI1Ysy\n
  VVTCvkwclRIImtA3+Q==\n
  -----END CERTIFICATE-----
  ");

------

-------------------
memos
-------------------

.. code-block:: java

  public Ripple memos(JSONObject memosInfo);

用于给交易添加备忘录信息。


参数
----------


1. ``memosInfo`` - ``JSONObject`` : 备忘录信息，包含以下三个字段:

	- ``data`` - ``String`` : 16进制字符串，通常包含备忘录的内容；
	- ``format`` - ``String`` : [**可选**] 16进制字符串,备忘录内容的格式，例如 ``MIME 类型`` ；
	- ``type`` - ``String`` : [**可选**] 16进制字符串, 定义了此备忘录的格式（遵从协议 ``RFC 3986`` ）。


返回值
----------

``Ripple`` - Ripple对象,后面一般接submit进行连续操作,如示例。


示例

.. code-block:: java

    try{
        JSONObject item = new JSONObject();
        item.put("data",Util.toHexString("众享比特"));
        item.put("format",Util.toHexString("MIME 类型"));
        item.put("type",Util.toHexString("http://www.peersafe.cn/"));

        JSONObject obj =  c.pay("zcs4x6e64E3Jw59CPSPyZAtYmVuGxUM4gb","1000").memos(item).submit(Submit.SyncCond.validate_success);
        System.out.println(obj);

    }catch (Exception e){
        e.printStackTrace();
        Assert.fail();
    }

------



网关交易
===============

.. _account-set:

-------------------
accountSet 
-------------------

.. code-block:: java

  public Ripple accountSet(int nFlag, boolean bSet);
  public Ripple accountSet(String transferRate, String transferFeeMin, String transferFeeMax);

账户属性设置


参数
----------


1. ``nFlag`` - ``int``:                 一般情况下为8，表示asfDefaultRipple，详见 ``AccountSet Flags``
2. ``bSet`` - ``boolean``:             true:设置nFlag标志 ; false: 取消nFlag标志
3. ``transferRate`` - ``String``:      网关数字资产的转账费率，取值为1.0~2.0；
4. ``transferFeeMin`` - ``String``:    通过网关进行转账，网关要收取的最小手续费，10进制字符串数字，例如"1"
5. ``transferFeeMax`` - ``String``:    通过网关进行转账，网关要收取的最大手续费，10进制字符串数字，例如"10"


.. note::

	* 关于费率：可取消设置，min/max=0,rate=1.0为取消设置。
	* 每次设置都是重新设置，之前的设置会被替代。
	* min,max可只设置一个，但这时要同时设置rate。
	* 如果只设置rate，默认为取消设置min,max。
	* 如果只设置min与rate，max会被取消设置,同理只设置max与rate，min会被取消设置。

返回值
----------

``Ripple`` - Ripple对象,后面一般接submit进行连续操作,如示例。


示例
----------

.. code-block:: java

    //开启asfDefaultRipple
    JSONObject jsonObj = c.accountSet(8, true).submit(SyncCond.validate_success);
    System.out.print("set gateWay:" + jsonObj + "\ntrust gateWay ...\n");

    //设置网关手续费
    jsonObj = c.accountSet("1.005", "2", "3").submit(SyncCond.validate_success);
    System.out.print("set gateWay:" + jsonObj + "\ntrust gateWay ...\n");

------

.. _whitelist-set:

-------------------
whitelistSet 
-------------------

.. code-block:: java

  public Ripple whitelistSet(JSONArray whitelists, int setFlag);

账户属性设置

为网关设置转账不收取手续费用的账户地址集合，这些账户地址集合称为网关白名单。

参数
----------


1. ``whitelists`` - ``JSONArray``:     为网关添加或删除的白名单中的账户地址，可以同时添加或删除多个账户地址。
2. ``setFlag`` - ``int``:              设置白名单的标记，10标识添加白名单，11标识删除白名单。


.. note::

	* 白名单重复添加或删除会返回错误信息。

返回值
----------

``Ripple`` - Ripple对象，后面一般接submit进行连续操作，如示例。


示例
----------

.. code-block:: java

    //添加网关白名单
    JSONObject user = new JSONObject();
    user.put("User", "zLi3jhvjkJp32cKyNScQx6GXZaLoXuEJ2u");
    JSONObject whitelist = new JSONObject();
    whitelist.put("WhiteList", user);
    STObject object = STObject.fromJSONObject(whitelist);
    STArray arry = new STArray();
    arry.add(object);
				
    String tablestr = "{\"WhiteList\":{\"User\":\"" + sUser1+ "\"}}";
    JSONArray array =  Util.strToJSONArray(tablestr);
    jsonObj = c.whitelistSet(array, 10).submit(SyncCond.validate_success);
    System.out.print("set gateWay:" + jsonObj");

------

-------------------
trustSet
-------------------

.. code-block:: java

  public Ripple trustSet(String value, String sCurrency, String sIssuer)

信任网关，参数指定信任某个网关配置的数字资产额度。从而可分发该网关数字资产。使用时需要接submit提交交易，见示例。


参数
----------


1. ``value`` - ``String``:        信任额度
2. ``sCurrency`` - ``String``:    数字资产名称 ，例如"RMB"
3. ``sIssuer`` - ``String``:      数字资产的配置网关地址。



返回值
----------


``Ripple`` - Ripple对象,后面一般接submit进行连续操作,如示例。


示例


.. code-block:: java

    JSONObject jsonObj = c.trustSet("1000000000", "RMB", "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh").submit(SyncCond.validate_success);
    System.out.println(jsonObj);

------

.. _pay-数字资产:

----------------------------
pay(转账网关数字资产)
---------------------------

.. code-block:: java

   public  Ripple pay(String accountId, String value, String sCurrency, String sIssuer);

转发数字资产

.. note::
    这个函数有重载，除了可以转账系统数字资产外，还可以转账配置的数字资产：:ref:`pay(转账系统数字资产) <pay-系统数字资产>`


参数
----------


1. ``accountId``   - ``String``:  转账接受地址
2. ``value``       - ``String``:  转账数字资产的数量 最大值为:1e11.
3. ``sCurrency``   - ``String``:  数字资产名称 ，例如"RMB"
4. ``sIssuer``     - ``String``:  数字资产配置网关地址


返回值
----------


``Ripple`` - Ripple对象,后面一般接submit进行连续操作,如示例。


备注


网关要收取的手续费的计算公式如下

``transferRate``   - 费率，    为网关属性设置 :ref:`accountSet <account-set>` 函数中传入的 ``transferRate`` 

``transferFeeMin`` - 最小花费， 为网关属性设置 :ref:`accountSet <account-set>` 函数中传入的 ``transferFeeMin`` 

``transferFeeMax`` - 最大花费， 为网关属性设置 :ref:`accountSet <account-set>` 函数中传入的 ``transferFeeMax`` 

.. math::
    \begin{gather}
    fee   = 转账金额*(费率-1.0) \\
    交易花费=\begin{cases}
    最小花费, \quad fee<最小花费 \\
    fee,\quad 最小花费\leq fee\leq 最大花费 \\ 
    最大花费，\quad fee>最大花费
    \end{cases}
    \end{gather}


示例


.. code-block:: java

  // 给账户地址等于 z9VF7yQPLcKgUoHwMbzmQBjvPsyMy19ubs 的用户转账5RMB.
  JSONObject obj =  c.pay("z9VF7yQPLcKgUoHwMbzmQBjvPsyMy19ubs", "5", "RMB", "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh").submit(SyncCond.validate_success);
  System.out.println(obj);

  if(obj.has("error_message")){
    System.out.println(obj);
  }else {
    System.out.println( "status" + obj.getString("status"));
  } 

------

-------------------
getAccountBalance
-------------------

.. code-block:: java

  public JSONArray getAccountBalance(String address);
  public JSONObject getAccountBalance(String address, String gateWay, String currency);

查询数字资产余额和ZXC余额


参数
----------


1. ``address`` - ``String``:      账户地址
2. ``gateWay`` - ``String``:      数字资产的发行网关地址
3. ``currency`` - ``String``:     数字资产。



返回值
----------



示例


.. code-block:: java

    //获取该账户地址的所有数字资产余额和ZXC余额
    JSONArray jsonArry = c.getAccountBalance("z9VF7yQPLcKgUoHwMbzmQBjvPsyMy19ubs");
    System.out.println(jsonArry);

    JSONObject jsonObj = c.getAccountBalance("z9VF7yQPLcKgUoHwMbzmQBjvPsyMy19ubs", "zLi3jhvjkJp32cKyNScQx6GXZaLoXuEJ2u", "RMB");
    System.out.println(jsonObj);

------

表交易
==============

-------------------
createTable
-------------------

.. code-block:: java

  public Chainsql createTable(String name, List raw);
  public Chainsql createTable(String name, List rawList ,boolean confidential);
  public Chainsql createTable(String name, List raw,JSONObject operationRule);


建表。


参数
----------


1. ``name``   - ``String``: 所创建表名，创建表不支持自增型;
2. ``raw`` - ``List``: 创建表的字段名称必须为Json格式数据,详细格式及内容可参看  :ref:`建表raw字段说明 <create-table>`;例如：

.. code-block:: javascript

  {'field':'id','type':'int','length':11,'PK':1,'NN':1,'UQ':1}
  {'field':'name','type':'varchar','length':50,'default':null}
  {'field':'age','type':'int'}


3. ``confidential``  - ``boolean``:    表示创建的表是否为加密的表,true:创建加密表;如果不写,默认为false;
4. ``operationRule`` - ``JSONObject``: 行级控制规则，不能与confidential一起使用。格式及内容可查看 :ref:`行级控制规则 <recordLevel>` 。

.. _my-reference-chainsql:


返回值
----------


1. ``Chainsql`` - Chainsql对象，后面一般接submit函数进行连续操作,如示例。


示例


.. code-block:: java

  // 创建表 "dc_universe"
  JSONObject obj = c.createTable("dc_universe", c.array(
  "{'field':'id','type':'int','length':11,'PK':1,'NN':1,'UQ':1}",
  "{'field':'name','type':'varchar','length':50,'default':null}",
  "{'field':'age','type':'int'}"),
  false)
  .submit(SyncCond.db_success);

  if(obj.has("error_message")){
    System.out.println(obj);
  }else {
    System.out.println( "status" + obj.getString("status"));
  }

------

-------------------
renameTable
-------------------

.. code-block:: java

  public Chainsql renameTable(String oldName, String newName);

修改数据库中表的名字


参数
----------


1. ``oldName`` - ``String``:  旧的表名
2. ``newName`` - ``String``: 新的表名;两个名字都不能为空；


返回值
----------


``Chainsql`` -  :ref:`Chainsql <my-reference-chainsql>`.


示例


.. code-block:: java

  // 将表"dc_universe"改为"dc_name"
  JSONObject obj = c.renameTable("dc_universe", "dc_name").submit(SyncCond.db_success);
  
  if(obj.has("error_message")){
    System.out.println(obj);
  }else {
    JSONObject objRet = new JSONObject();
    objRet.put("status", obj.getString("status"));
    System.out.println( objRet);
  }


------

-------------------
dropTable
-------------------

.. code-block:: java

  public Chainsql dropTable(String tableName);

从数据库删除一个表。表和它的所有数据将被删除;


参数
----------


1. ``tableName``    - ``String``:  表名


返回值
----------


``Chainsql`` -  :ref:`Chainsql <my-reference-chainsql>`.


示例


.. code-block:: java

  JSONObject obj = c.dropTable("dc_universe").submit(SyncCond.db_success);
  if(obj.has("error_message")){
    System.out.println(obj);
  }else {
    System.out.println( "status" + obj.getString("status"));
  }

------

-------------------
table
-------------------

.. code-block:: java

  public Table table(String tableName);

创建一个table对象


参数
----------


1. ``tableName``    - ``String``:  表名


返回值
----------


.. _my-reference-table:

``Table`` - Table对象，一般后面接insert，get，submit等函数进行连续操作，如示例。


示例


.. code-block:: java

  String sTableName = "n12356";
  JSONObject obj = c.table(sTableName).insert(c.array("{id: 1, 'name': 'peera','age': 22}", "{id: 2, 'name': 'peerb','age': 21}"))
  .submit(SyncCond.db_success);

  if(obj.has("error_message")){
    System.out.println(obj);
  }else {
    System.out.println( "status" + obj.getString("status"));
  }


  JSONObject obj = c.table(sTableName)
  .get(c.array("{'id': 1}"))
  .update("{'age':52,'name':'Jack'}")
  .submit(SyncCond.db_success);

  if(obj.has("error_message")){
    System.out.println(obj);
  }else {
    System.out.println( "status" + obj.getString("status"));
  }

.. _InsertJava:

------

-------------------
insert
-------------------

.. code-block:: java

  public Table insert(List<String> raw);
  public Table insert(List<String> raw,String autoFillField);
  public Table insert(List<String> orgs,String autoFillField,String txsHashFillField);

向数据库中插入数据。


参数
----------

1. ``raw``    - ``List``:  raw类型必须是示例中json格式的数据类型，详细格式和内容可参看 :ref:`插入raw字段说明 <insert-table>` ;
2. ``autoFillField``  - ``String``: 插入操作支持将每次插入交易的哈希值作为字段信息同步插入到数据库中。使用该功能时，需要在建表时指定一个字段为存储交易哈希,并将该字段名作为参数传递给insert;
3. ``txsHashFillField``  - ``String``:  该参数的功能与 ``autoFillField`` 类似，可配合update实现存储历史哈希列表的功能，具体使用见 :ref:`update  <UpdateJava>` ; 

.. IMPORTANT::

  如果要用 ``txsHashFillField`` 功能，在插入时就需要用第三种重载，不然后面再对记录进行修改，交易hash会记录不全

返回值
----------


``Table`` - 见 :ref:`Table  <my-reference-table>`.


示例1

.. code-block:: java

  String sTableName = "n12356";
  // 向表sTableName中插入一条记录.
  JSONObject obj =  c.table(sTableName).insert(c.array("{id: 1, 'name': 'Jack','age': 22}", "{id: 2, 'name': 'Rose','age': 21}"))
  .submit(SyncCond.db_success);

  if(obj.has("error_message")){
    System.out.println(obj);
  }else {
    System.out.println( "status" + obj.getString("status"));
  }  

------

示例2

.. code-block:: java

  String sTableName = "n12356";
  // 向表sTableName中插入一条记录,并将该条交易的hash填充到该表的 tx_hash 字段中
  JSONObject obj =  c.table(sTableName).insert(c.array("{id: 1, 'name': 'Jack','age': 22}", "{id: 2, 'name': 'Rose','age': 21}"),"tx_hash")
  .submit(SyncCond.db_success);

  if(obj.has("error_message")){
    System.out.println(obj);
  }else {
    System.out.println( "status" + obj.getString("status"));
  }  


.. _UpdateJava:

------

-------------------
update
-------------------

.. code-block:: java

  public Table update(String raw);
  public Table update(String raw,String autoFillField);
  public Table update(String raw,String autoFillField,String txsHashFillField);

更新表中数据。

.. IMPORTANT::

  | 1. 更新之前需要调用table.get接口指定待修改内容，且必须指定，不能为空。详见示例
  | 2. 如果要用 ``txsHashFillField`` 功能，在插入时就需要向这个字段插入交易Hash


参数
----------

1. ``raw``  - ``List``:  raw类型必须都是示例中的json格式的数据类型，详细格式和内容可参看 :ref:`更新raw字段说明 <update-table>`;
2. ``autoFillField``  - ``String``: 更新操作支持将每次更新交易的哈希值作为字段信息同步更新到数据库中。使用该功能时，需要在建表时指定一个字段存储交易哈希,并将该字段名作为参数传递给update; 
3. ``txsHashFillField``  - ``String``: 该参数的功能与 ``autoFillField`` 类似,区别在于该参数指定的字段可存储历史交易哈希信息列表，将单条表记录的多次更新操作可汇集成一个hash值列表插入 ``txsHashFillField`` 字段中，并以 ``,`` 分割符分割哈希值，具体使用见示例3; 

返回值
----------

``Table`` - 见 :ref:`Table  <my-reference-table>`.


示例1

.. code-block:: java

  String sTableName = "n12356";
  // 更新 id 等于 1 的记录
  JSONObject obj = c.table(sTableName)
  .get(c.array("{'id': 1}"))
  .update("{'age':52,'name':'Jack'}")
  .submit(SyncCond.db_success);

  if(obj.has("error_message")){
    System.out.println(obj);
  }else {
    System.out.println( "status" + obj.getString("status"));
  }

------

示例2

.. code-block:: java

  String sTableName = "n12356";
  // 更新 id 等于 1 的记录, 并将该条交易的hash填充到该表的 tx_hash 字段中
  JSONObject obj = c.table(sTableName)
  .get(c.array("{'id': 1}"))
  .update("{'age':52,'name':'Jack'}","tx_hash")
  .submit(SyncCond.db_success);

  if(obj.has("error_message")){
    System.out.println(obj);
  }else {
    System.out.println( "status" + obj.getString("status"));
  }

------

示例3

.. code-block:: java

      String sHistory = "history_hash_example";
      List<String> arrTableField = Util.array("{'field':'id','type':'int','length':11,'PK':1,'NN':1,'UQ':1}",
              "{'field':'txn_hash','type':'text'}", "{'field':'age','type':'int'}");

      // 1、 建表时预留txn_hash 字段用以存于历史交易hash信息
      JSONObject obj;
      obj = c.createTable(sHistory,arrTableField,false).submit(Submit.SyncCond.db_success);
      System.out.println("create result:" + obj);

      // 2、 插表交易，并指定txn_hash 字段存储历史交易hash信息
      List<String> orgs = Util.array("{'id':1,'age': 1}");
      obj = c.table(sHistory).insert(orgs,"","txn_hash").submit(Submit.SyncCond.db_success);
      System.out.println("insert result:" + obj);

      // 3、 更新表交易，并指定txn_hash 字段存储历史交易hash信息
      List<String> arr1 = Util.array("{'id': 1}");
      for(int i=0;i<2;i++){
          String rule = "{'age':" + i + "}";
          obj = c.table(sHistory).get(arr1).update(rule,"","txn_hash").submit(Submit.SyncCond.db_success);
      }

      // 4、 查询历史交易hash信息，即查询表中字段txn_hash的信息
      obj = c.table(sHistory).get("{'id': 1}").submit();
     
      // txn_hash字段中存储了历史交易哈希列表信息，哈希值以,进行分割
      System.out.println("打印历史hash信息:" + obj);
      //{
      //  "final_result":true,
      //  "diff":0,
      //  "lines":[
      //    {
      //      "txn_hash":"C7F2C6D41693B878255610A1ED7A92F71A36A0892A8D6B4BD3C09C0721A7F56E, CF43F21CBA9B91B9AF51B6ED8D4CDF39C2F3E4C075EBB4DF560472BCE047E038,9A211F2F347FD35557B890FA6603F5E93BBE21F4BC79419F660C98B37096E8D0",
      //      "id":1,
      //     "age":1
      //    }
      //  ]
      //}
      
------

-------------------
delete
-------------------

.. code-block:: java

  public Table delete();

从表中删除对应条件的数据，需要与get函数配合使用。如果get条件为空，则删除所有数据。


参数
----------



返回值
----------


``Table`` - 见 :ref:`Table  <my-reference-table>`.


示例


.. code-block:: java

  String sTableName = "n12356";
  // 删除 id 等于 1 的记录.
  JSONObject obj =  c.table(sTableName)
  .get(c.array("{'id': 1}"))
  .delete()
  .submit(SyncCond.db_success);

  if(obj.has("error_message")){
    System.out.println(obj);
  }else {
    System.out.println( "status" + obj.getString("status"));
  }  

------

-------------------
beginTran
-------------------

.. code-block:: java

  public void beginTran();

开启事务.

-------------------
commit
-------------------

.. code-block:: java

  public JSONObject commit();
  public JSONObject commit(SyncCond cond);
  public JSONObject commit(Callback<?> cb);

提交事务;本次事务期间的所有操作都会打包提交到区块链网络。


参数
----------


1. ``cond`` - ``SyncCond``: 同步接口，参数为 枚举类型;
2. ``cb``   - ``Callback``: 异步接口，参数为 回调函数


返回值
----------



示例


.. code-block:: java

  String sTableName = "n12356";
  c.beginTran();

  c.table(sTableName).insert(c.array("{'name': 'Rose','age': 22}","{'name': 'Jack','age': 21}"));
  c.table("posts").get(c.array("{'id': 1}")).update("{'age':52,'name':'Rose'}");
  c.table(sTableName).get(c.array("{'id': 1}")).delete();

  // 1、
  System.out.println(c.commit());

  // 2、
  // c.commit(new Callback<JSONObject>() {
  //			@Override
  //			public void called(JSONObject args) {
  //				System.out.println(args);
  //			}
  //	});

  // 3、
  //System.out.println(c.commit(SyncCond.db_success));


输出

.. code-block:: json

  {
    "tx_hash": "D35F7FCD2467348EE7D781B507FC1262BC46AFDECEFDB39053F5AA1CE9AF1586",
    "status": "send_success"
  }

------

.. note:: 在事务开始和结束之间的insert，update，delete语句会包装在一个原子操作中执行，与数据库的事务类似，事务中执行的语句要么全部成功，要么全部失败。执行事务类型交易主要涉及两个api：beginTran,commit. beginTran开启事务，commit提交事务，事务中的操作全部执行成功事务才成功，有一个执行失败，则事务会自动回滚。在事务上下文中，不再支持单个语句的submit。


-------------------
grant
-------------------

.. code-block:: java

  public Chainsql grant(String name, String user,String flag);
  public Chainsql grant(String name, String user,String userPublicKey,String flag);

授权user用户操作表name的各项权限


参数
----------


1. ``name``    - ``String``:  表名
2. ``user``    - ``String``:  被授权账户地址 
3. ``flag``    - ``String``:  表操作规则.例如:"{insert:true,delete:false}" 表示user 账户可以执行插入操作，但是不能执行删除操作
4. ``userPublicKey``  - ``String``:  被授权账户公钥， **加密表** 授权需要用这个重载


返回值
----------


``Chainsql`` -  :ref:`Chainsql <my-reference-chainsql>`.


示例


.. code-block:: java

  String sTableName = "n12356";
  JSONObject obj = new JSONObject();
  obj = c.grant(sTableName, sNewAccountId, "{insert:true,update:true}")
          .submit(SyncCond.validate_success);
  System.out.println("grant result:" + obj.toString());

------

-------------------
setRestrict
-------------------

.. code-block:: java

  public void setRestrict(boolean falg);

设置是否使用严格模式，默认为非严格模式；在严格模式下，语句共识通过的条件是期望的快照HASH与预期一致


参数
----------


1. ``flag`` - ``boolean``: true:严格模式     false: 非严格模式


返回值
----------



示例


.. code-block:: java

    c.setRestrict(false);

------

.. _Java-setNeedVerify:

-------------------
setNeedVerify
-------------------

.. code-block:: java

  public void setNeedVerify(boolean flag);

设置是否需要认证


参数
----------


1. ``flag`` - ``boolean``: true: 需要认证  false: 不需要认证


返回值
----------



示例


.. code-block:: java

    c.setNeedVerify(false);

------

表查询
==============

.. note::
	现在查询无论数据库有多少内容，get接口一次最多返回200条结果，如果数据较多，可以结合 ``limit`` 接口做分页查询。

.. _get_Java:

-------------------
get
-------------------

.. code-block:: java

   public  Table get(List<String> args);

从数据库查询数据,后面可以进行其他操作，例如update、delete等;
通过指定查询的内容作为raw参数传入，raw的详细格式及内容可参看 :ref:`Raw字段详解 <查询Raw详解>`


参数
----------


1. ``args``    - ``List<String>``:  List 元素的格式是JSON类型，例如"{'name': 'aa'}";


返回值
----------


``Table`` - 见 :ref:`Table  <my-reference-table>`.


示例


.. code-block:: java

  String sTableName = "n12356";
  //查询 name 等于 hello 的记录.
  JSONObject obj  = c.table(sTableName).get(c.array("{'name': 'hello'}")).submit();

  System.out.println(obj);

输出

.. code-block:: json

  {
    "final_result": true,
    "diff": 0,
    "lines": [
      {
        "name": "hello",
        "id": 103,
        "age": 333
      },
      {
        "name": "hello",
        "id": 105,
        "age": 333
      },
      {
        "name": "hello",
        "id": 106,
        "age": 333
      }
    ]
  }

------

-------------------
limit
-------------------

.. code-block:: java

   public Table limit(String raw);

对数据库进行分页查询.返回对应条件的数据；必须与get配合使用;


参数
----------


1. ``raw``    - ``List``: 可参看 :ref:`Raw字段详解 <查询Raw详解>`


返回值
----------


``Table`` - 见 :ref:`Table  <my-reference-table>`.


示例


.. code-block:: java

  String sTableName = "n12356";
  //查询 name 等于 hello 的前10条记录
  JSONObject obj  = c.table(sTableName).get(c.array("{'name': 'hello'}")).limit("{index:0,total:10}").submit();
  System.out.println(obj);


输出

.. code-block:: json

  {
    "final_result": true,
    "diff": 0,
    "lines": [
      {
        "name": "hello",
        "id": 103,
        "age": 333
      },
      {
        "name": "hello",
        "id": 105,
        "age": 333
      },
      {
        "name": "hello",
        "id": 106,
        "age": 333
      }
    ]
  }  

------

-------------------
order
-------------------

.. code-block:: java

   public Table order(String raw);


参数
----------


1. ``raw``    - ``List<String>``:  可参看 :ref:`Raw字段详解 <查询Raw详解>`


返回值
----------


``Table`` - 见 :ref:`Table  <my-reference-table>`.


示例


.. code-block:: java

  String sTableName = "n12356";
  // 按 id 升序，name 的降序排序
  JSONObject obj = c.table(sTableName).get(c.array("{'name': 'hello'}")).order(c.array("{id:1}", "{name:-1}")).submit();
  System.out.println(obj);

输出

.. code-block:: json

  {
    "final_result": true,
    "diff": 0,
    "lines": [
      {
        "name": "hello",
        "id": 100,
        "age": 333
      },
      {
        "name": "hello",
        "id": 101,
        "age": 333
      }
    ]
  }


------

-------------------
withFields
-------------------

.. code-block:: java

  public Table withFields(String  orgs);

从数据库查询数据,并返回指定字段,必须与get配合使用;


参数
----------


1. ``orgs``    - ``String``:  orgs类型必须是示例中的json格式的数据类型;


返回值
----------


``Table`` - 见 :ref:`Table  <my-reference-table>`.


示例


.. code-block:: java

  String sTableName = "n12356";
  // 查询 name 等于 hello 的记录.取name以及id字段
  JSONObject obj  = c.table(sTableName).get(c.array("{'name': 'hello'}")).withFields("['name','id']").submit();
  System.out.println(obj);

输出

.. code-block:: json

  {
    "final_result": true,
    "diff": 0,
    "lines": [
      {
        "name": "hello",
        "id": 103
      },
      {
        "name": "hello",
        "id": 105
      },
      {
        "name": "hello",
        "id": 106
      }
    ]
  }


------

-------------------
getBySqlAdmin
-------------------

.. code-block:: java

   // 根据sql语句查询，admin接口，同步调用
   public JSONObject getBySqlAdmin(String sql);

   // 根据sql语句查询，admin接口，异步调用
   public void       getBySqlAdmin(String sql,Callback<JSONObject> cb);

直接传入SQL语句进行数据库查询操作。

.. note::

  * 因为直接操作数据库中的表，所以需要配合getTableNameInDB接口获取表在数据库中的真实表名。详见示例。
	* 本接口不做表权限判断，但是只能由节点配置文件中 **[port_ws_admin_local]** 的admin里所配置的ip才可以发起此接口的调用，否则调用失败。
	* 即nodejs接口运行所在的主机ip必须是配置在配置文件的 **[port_ws_admin_local]** 的admin里。
	* 因为在配置文件中配置为admin的ip认为是该节点的管理员。拥有对本节点已经同步表的查询权限。



参数
----------


1. ``sql``   - ``String``              :  标准sql语句
2. ``cb``    - ``Callback<JSONObject>``:  回调函数


返回值
----------


1. ``JSONObject`` :主要参数说明(格式见示例输出)   

      * ``final_result``   -  true，查询成功;false,查询失败。
      * ``diff``           -  当前区块序号与被查询表数据库同步到的区块序号的差值，如果有多个表，取最大差值。
      * ``lines``          -  返回查询的sql结果。
      * ``error_message``  -  错误返回时，显示错误信息。


示例


.. code-block:: java

  String rootAddress = "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh";
  String sTableName = "n12356";
  // select * from t_xxxxxxx
  c.getTableNameInDB(rootAddress, sTableName, new Callback<JSONObject>(){

  @Override
  public void called(JSONObject args) {
      System.out.println(args);
      if(args.has("nameInDB")) {
        String sql = "select * from t_" + args.getString("nameInDB");
        c.getBySqlAdmin(sql,new Callback<JSONObject>() {

          @Override
          public void called(JSONObject args) {
            System.out.println( args);
          }
          
        });
        
      }
    }
    
  });

成功

.. code-block:: json

    {
      "final_result":true,
      "lines":[
          {
          "name":"hello",
          "id":1,
          "age":333
          },
          {
          "name":"sss",
          "id":2,
          "age":444
          },
          {
          "name":"rrr",
          "id":3,
          "age":555
          }
      ]
    }

失败

.. code-block:: json

    {
        "error_message":"Bad credentials.",
        "request":{
          "id":2,
          "command":"r_get_sql_admin",
          "sql":"select * from t_27C10F5E2128840470021D82A40933CEC689FEC5"
        },
        "error_code":3,
        "id":2,
        "error":"forbidden",
        "type":"response",
        "status":"error"
    }

------

-------------------
getBySqlUser
-------------------

.. code-block:: java

    public JSONObject getBySqlUser(String sql);//同步接口
    public void       getBySqlUser(String sql,Callback<JSONObject> cb);// 异步接口

直接传入SQL语句进行数据库查询操作。

.. note::
  * 因为直接操作数据库中的表，所以需要配合getTableNameInDB接口获取表在数据库中的真实表名。详见示例。
  * 调用此接口前，as接口中设置的用户需要对SQL语句中的表有查询权限，才可以调用此接口。



参数
----------


1. ``sql``    - ``String``:  标准sql语句
2. ``cb``    - ``Callback<JSONObject>``:  回调函数


返回值
----------


1. ``JSONObject`` :主要参数说明(格式见示例输出)   

      * ``final_result``   -  true，查询成功;false,查询失败。
      * ``diff``           -  当前区块序号与被查询表数据库同步到的区块序号的差值，如果有多个表，取最大差值。
      * ``lines``          -  返回查询的sql结果。
      * ``error_message``  -  错误返回时，显示错误信息。


示例


.. code-block:: java

  String rootAddress = "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh";
  String sTableName = "n12356";
  JSONObject ret = c.getTableNameInDB(rootAddress, sTableName);
  if(ret.has("nameInDB")) {
    JSONObject obj = c.getBySqlUser("select * from t_" + ret.getString("nameInDB"));
    System.out.println( obj );
  }

成功

.. code-block:: json

    {
      "final_result":true,
      "lines":[
          {
          "name":"hello",
          "id":1,
          "age":333
          },
          {
          "name":"sss",
          "id":2,
          "age":444
          },
          {
          "name":"rrr",
          "id":3,
          "age":555
          }
      ]
    }

失败

.. code-block:: json

  {
    "error_message": "The user is unauthorized to the table.",
    "request": {
      "signature": "304402200BFB2A48BCBDCA39E10334ABE926CAB3AAF668DC34C0EC6FC16AB6D30F3906520220025A911F16530C60842CEBF6EF70BDFA053287427A007B6E8D5751D49BE7B36F",
      "tx_json": {
        "LedgerIndex": 33020,
        "Account": "zpMZ2H58HFPB5QTycMGWSXUeF47eA8jyd4",
        "Sql": "select * from t_A07D12AF1616383806669B128675E8C32A894265"
      },
      "id": 3,
      "publicKey": "02080EF3E62711D21A502DC6B6290E2819474AA7701271D82CDFF73F65AFEE7276",
      "signingData": "{\"LedgerIndex\":33020,\"Account\":\"zpMZ2H58HFPB5QTycMGWSXUeF47eA8jyd4\",\"Sql\":\"select * from t_A07D12AF1616383806669B128675E8C32A894265\"}",
      "command": "r_get_sql_user"
    },
    "final_result": true,
    "error_code": 77,
    "id": 3,
    "error": "tabUnauthorized",
    "type": "response",
    "status": "error"
  }


------------------------------------------------------------

智能合约接口
==============

.. toctree::
    :maxdepth: 2

    javaSmartContract
      
---------

多链接口
==============

.. _javaSchemaCreate:

-------------------
createSchema
-------------------

.. code-block:: java

    public Chainsql createSchema(JSONObject schemaInfo)

创建子链

参数
----------

参数为创建子链的信息，Json结构为：

1. ``SchemaName``       - ``String``:    子链名称;
2. ``WithState``        - ``String``:    是否继承主链状态;
3. ``AnchorLedgerHash`` - ``String``:     继承主链的区块Hash
4. ``Validators``       - ``JSONArray`` : 子链节点共识公钥列表
5. ``PeerList``         - ``JSONArray`` : 子链节点P2P连接方式列表

返回值
----------

``Chainsql`` - Chainsql对象，后面一般接submit函数进行连续操作,如示例。


示例


.. code-block:: java

  JSONObject schemaInfo = new JSONObject();
  schemaInfo.put("SchemaName","hello3");
  schemaInfo.put("WithState",false);
  schemaInfo.put("SchemaAdmin",rootAddress);

  List<String> validators = new ArrayList<String>();
  validators.add("03B31BE0888F679399A26A857BFFB1022376265C98A5236933FDB38C87F3E3EA6B");
  validators.add("027A642FCB1ADA4FB7346344515BED8E68FF346C142EAF2751DF1D57C09DE97FDD");
  validators.add("02FF0F655A0FD1E7D2876A8A03F3871F2E08BBAB3435053DCAC803D6E6A100A3A2");
  JSONArray validatorsJsonArray = new JSONArray(validators);
  schemaInfo.put("Validators",validatorsJsonArray);

  List<String> peerList = new ArrayList<String>();
  peerList.add("127.0.0.1:5431");
  peerList.add("127.0.0.1:5432");
  peerList.add("127.0.0.1:5436");
  JSONArray peerListJsonArray = new JSONArray(peerList);

  schemaInfo.put("PeerList",peerListJsonArray);
  schemaInfo.put("SchemaAdmin",rootAddress);
  try {
    JSONObject ret = c.createSchema(schemaInfo).submit(SyncCond.validate_success);
    System.out.println("创建子链结果："+ret);
    
  } catch (Exception e) {
    e.printStackTrace();
  }

.. _javaSchemaModify:

-------------------
modifySchema
-------------------
  
.. code-block:: java
  
  public Chainsql modifySchema(SchemaOpType type,JSONObject schemaInfo)
  

  修改子链

参数
----------

  参数为1为操作类型，可选值为：

  1. ``schema_add``
  2. ``schema_del``

  schemaInfo 结构为：
  
  1. ``SchemaID``         - ``String``:    子链ID;
  2. ``Validators``       - ``JSONArray`` : 要增删的子链节点共识公钥列表
  3. ``PeerList``         - ``JSONArray`` : 要增删的子链节点P2P连接方式列表
  
返回值
----------
  
  ``Chainsql`` - Chainsql对象，后面一般接submit函数进行连续操作,如示例。
  
  示例
  
  .. code-block:: java
  
    JSONObject schemaInfo = new JSONObject();
    schemaInfo.put("SchemaID", "59FA6AD350FFD235460C0455CA83461B31D6428150671EBEC093604AC75F0477");

    List<String> validators = new ArrayList<String>();
    validators.add("03830017E6B3B77DA7F1FABF461570C54B6F5B9B03554CFE26A5A2B4C3E22E79AF");
    JSONArray validatorsJsonArray = new JSONArray(validators);
    schemaInfo.put("Validators",validatorsJsonArray);

    List<String> peerList = new ArrayList<String>();
    peerList.add("127.0.0.1:5437");
    JSONArray peerListJsonArray = new JSONArray(peerList);
    schemaInfo.put("PeerList",peerListJsonArray);
    try {
      //增加节点
      JSONObject obj = c.modifySchema(SchemaOpType.schema_add, schemaInfo).submit(SyncCond.validate_success);
      System.out.println(obj);
    } catch (Exception e) {
      e.printStackTrace();
    }

.. _javagetSchemaList:

-------------------
getSchemaList
-------------------

.. code-block:: java
  
  public JSONObject getSchemaList(JSONObject params)

获取子链列表

参数
----------
  
  参数有两个可选项，均为过滤条件，不填则查询链上所有子链

  1. ``account``  - ``String``：  可选，建链账户地址
  2. ``running``  - ``bool``  :   可选，是否当前节点正运行中的子链
   
返回值
----------

| 返回值为Json数组，格式参 :ref:`rpc查询子链列表 <rpc查询子链列表>`
| 示例如下：

.. code-block:: java

  JSONObject option = new JSONObject();
  option.put("running",true);
  JSONObject list = c.getSchemaList(option);
  System.out.println("运行中的子链列表:" + list);

.. _javagetSchemaInfo:

-------------------
getSchemaInfo
-------------------

.. code-block:: java
  
  public JSONObject getSchemaInfo(String schemaID)

获取子链信息

参数
----------
  

  1. ``schemaID``  - ``String``：  子链ID
   
返回值
----------

| 返回值为Json数组，格式参考  :ref:`rpc查询子链信息 <rpc查询子链信息>`
| 示例如下：

.. code-block:: java

  String schemaID = "6BA63B86E5CE48283D03CC21D3BE5F4630CC6572CE7F54982E5AE687C998B7A3";
  JSONObject ret =  c.getSchemaInfo(schemaID);
  System.out.println("子链信息:" + ret);

.. _javasetSchema:

-------------------
setSchema
-------------------

.. code-block:: java
  
  public void setSchema(String schemaID)

设置要操作的子链ID，设置后所有的请求都针对子链进行


参数
----------
  

  1. ``schemaID``  - ``String``：  子链ID
    
返回值
----------

无

示例：

.. code-block:: java

  Chainsql c = new Chainsql();
  c.connect("ws://192.168.29.116:6006");
  c.setSchema("6BA63B86E5CE48283D03CC21D3BE5F4630CC6572CE7F54982E5AE687C998B7A3");
  JSONObject obj = c.getServerInfo();
  System.out.println(obj);


------------------------

订阅
==============

-------------------
subscribeTable
-------------------


.. code-block:: java

  public void subscribeTable(String name, String owner ,Callback<JSONObject> cb);

订阅某张表。该表相关的信息发生改变时，会通过回调函数返回改变内容。


参数
----------


1. ``name``    - ``String``:    表名;
2. ``owner``   - ``String``:    为表的所有者地址;
3. ``cb``      - ``Callback`` : 回调函数


返回值
----------



示例


.. code-block:: java

  String rootAddress = "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh";
  // 用户订阅TestName表信息，表的创建者为rootAddress
  c.event.subscribeTable("TestName", rootAddress,new Callback<JSONObject>() {
    @Override
    public void called(JSONObject args) {
      System.out.println(" table info :" + args);
    }
  });

------

-------------------
unsubcribeTable
-------------------

.. code-block:: java

  public void unsubscribeTable(String name, String owner,Callback<JSONObject> cb);

取消对表的订阅。


参数
----------


1. ``name``  - ``String``:  被订阅的数据库表名；
2. ``owner``      - ``String``:  被订阅的表拥有者地址；
3. ``cb``         - ``Callback``: 异步接口回调


返回值
----------



示例


.. code-block:: java

  String rootAddress = "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh";
  // 用户取消订阅TestName表
  c.event.unsubscribeTable("TestName", rootAddress, new Callback<JSONObject>() {
      @Override
      public void called(JSONObject args) {

        System.out.println(" unsubscribeTable info :" + args);
      }
  });

------

.. _javaAPI订阅交易:

-------------------
subscribeTx
-------------------

.. code-block:: java

   public void subscribeTx(String txid,Callback<?> cb);

订阅交易事件，提供交易的哈希值，就可以订阅此交易。


参数
----------


1. ``txid`` - ``String``:          被订阅的交易哈希值；
2. ``cb``   - ``callback``:  回调函数，为必填项，需用通过此函数接收后续订阅结果。


返回值
----------



示例


.. code-block:: java

  // 用户订阅交易Hash信息.
  c.event.subscribeTx("601781B50D7936370276287EAC3737D7A1D20281E2E73FCA31FE7563426C93B0", new Callback<JSONObject>() {

    @Override
    public void called(JSONObject args) {
      System.out.println("subscribeTx Info:" + args);
    }
  });

------

-------------------
unsubscribeTx
-------------------

.. code-block:: java

   public void unsubscribeTx(String txid,Callback<JSONObject> cb);

取消对交易的订阅


参数
----------


1. ``txid`` - ``String``:          被订阅的交易哈希值


返回值
----------



示例


.. code-block:: java

  // 取消订阅交易Hash信息.
  c.event.unsubscribeTx("601781B50D7936370276287EAC3737D7A1D20281E2E73FCA31FE7563426C93B0", new Callback<JSONObject>() {
    @Override
    public void called(JSONObject args) {
      System.out.println("unsubscribeTx Info:" + args);
    }
  });

---------------------------------------


.. _chainsql_pool:

并发支持
==============

1.5.2版本以后，新添加类 ``ChainsqlPool`` 支持并发操作，``ChainsqlPool`` 的大小默认为10。可支持创建多个Chainsql对象的连接池，如不符合需求，可以参考 `源码 <https://github.com/ChainSQL/java-chainsql-api/blob/master/chainsql/src/main/java/com/peersafe/chainsql/pool/>`_ 进行修改。

-------------
使用说明
-------------
 - ChainsqlUnit 资源使用完后，需调用 ``unlock`` 将资源返还给ChainsqlPool。
 - 请求 ``Chainsql`` 资源时，如果当前 ``ChainsqlPool`` 已无可用资源，这时 ``ChainsqlPool`` 会再创建新的 ``ChainsqlUnit`` 对象 ， 但是这个新创建的 ``ChainsqlUnit``  对象，调用 ``unlock`` 后 , 会自动与服务器节点断开连接。

.. warning::
    同一个账户不可以并发提交交易，只能在一个线程中串行发送交易，原因是账户在链上存在Sequence，只能顺序执行交易。

代码示例见 `ChainsqlPool示例 <https://github.com/ChainSQL/java-chainsql-api/blob/master/chainsql/src/test/java/com/peersafe/example/chainsql/TestChainsqlPool.javaa>`_。 

基本调用流程：

.. code-block:: java

  //初始化连接地址，设置并发数量
  ChainsqlPool.instance().init("ws://127.0.0.1:6006",10);

  //开启多线程
  for(int i=0; i<PublicVar.ThreadCount; i++) {
    new Thread(new InsertThread(i,mAccountList)).start();
  }

  //...
  //下面的操作都在线程中执行
  {
    //获取 ChainsqlUnit对象
    ChainsqlUnit unit = ChainsqlPool.instance().getChainsqlUnit();
    //获取chainsql对象
    Chainsql c = unit.getChainsql();

    //使用chainsql对象
    c.as(a.address, a.secret);
    c.use(PublicVar.rootAddress);
    JSONObject obj = c.table(PublicVar.mTableName).insert(Util.array("{'id':" + index + ",'age': 333,'name':'hello'}")).submit();

    //释放chainsql对象
    unit.unlock();
  }

**ChainsqlPool对外提供了如下接口：**

-------------------
init
-------------------

.. code-block:: java

   public void init(String url,int count);

初始化ChainsqlChainsqlPool

参数
----------

1. ``url``   - ``String``:       ChainSQL节点的Websocket 地址
2. ``count`` - ``int``:          创建的ChainSQL对象实例的个数

返回值
----------

示例

.. code-block:: java

		ChainsqlPool.instance().init("ws://127.0.0.1:6006",10);

------------------------------------------------------------------------------

-------------------
getChainsqlUnit
-------------------

.. code-block:: java

   public synchronized ChainsqlUnit getChainsqlUnit()

同步获取Chainsql池中可用的 ``ChainsqlUnit`` 资源。

参数
----------

返回值
----------

``ChainsqlUnit``:       ChainsqlUnit对象

示例

.. code-block:: java

		ChainsqlUnit unit = ChainsqlPool.instance().getChainsqlUnit();

------------------------------------------------

**ChainsqlUnit对外接口：**

-------------------
getChainsql
-------------------

.. code-block:: java

    public Chainsql getChainsql()

获取ChainsqlPool封装的Chainsql对象。


参数
----------

返回值
----------

``Chainsql`` - Chainsql对象

示例

.. code-block:: java

			ChainsqlUnit unit = ChainsqlPool.instance().getChainsqlUnit();
			Chainsql c = unit.getChainsql();

------------------------------------------------

-------------------
unlock
-------------------

.. code-block:: java

   public synchronized Chainsql lock();

解锁操作。``ChainsqlUnit`` 资源使用完后调用，将资源返还给ChainsqlPool。

参数
----------

返回值
----------

示例

.. code-block:: java

			ChainsqlUnit unit = ChainsqlPool.instance().getChainsqlUnit();		
			unit.unlock();

---------------------------------------


加解密
==============

-------------------
sign
-------------------
.. code-block:: javascript

    chainsql.sign(byte[] message,String secret)

| 使用账户私钥对字符串签名。

参数说明
-----------

1. ``message`` - ``byte[]`` : 被签名的原数据；
2. ``secret`` - ``String`` : 账户私钥。

返回值
-----------

``byte[]`` : 返回签名结果。

示例
-----------
.. code-block:: javascript

    String hello = "helloworld";
    byte[] signature = c.sign(hello.getBytes(), "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb");

------------------------------------------------------------------------------

-------------------
verify
-------------------
.. code-block:: javascript

	chainsql.verify((byte[] message,byte[] signature,String publicKey)

使用账户公钥验证签名正确性。

参数说明
-----------

1. ``message`` - ``byte[]`` : 被签名的原数据；
2. ``signature`` - ``byte[]`` : 签名结果；
3. ``publicKey`` - ``String`` : 账户公钥。

返回值
-----------

``bool`` : 返回验证的结果。

示例
-----------
.. code-block:: javascript

    String hello = "helloworld";
    c.verify(hello.getBytes(), signature, "cBQG8RQArjx1eTKFEAQXz2gS4utaDiEC9wmi7pfUPTi27VCchwgw")

------------------------------------------------------------------------------

-------------------
sign
-------------------
.. code-block:: javascript

	chainsql.sign(JSONObject tx,String secret)

| 使用账户私钥对交易签名。

参数说明
-----------

1. ``tx`` - ``JSONObject`` : 交易内容。
2. ``secret`` - ``String`` : 账户私钥。

返回值
-----------

``JSONObject`` : 返回签名结果。

示例
-----------
.. code-block:: javascript

    JSONObject obj = new JSONObject();
    JSONObject tx_json = new JSONObject();
    tx_json.put("Account", "rHb9CJAWyB4rj91VRWn96DkukG4bwdtyTh");
    tx_json.put("Amount", "10000000000");
    tx_json.put("Destination", "zpMZ2H58HFPB5QTycMGWSXUeF47eA8jyd4");
    tx_json.put("TransactionType", "Payment");
    tx_json.put("Sequence", 2);
    obj.put("tx_json", tx_json);
		
    JSONObject res = c.sign(obj, "snoPBrXtMeMyMHUVTgbuqAfg1SUTb");
    System.out.println("sign payment result:" + res);

------------------------------------------------------------------------------

-------------------
encrypt
-------------------
.. code-block:: javascript

	chainsql.encrypt(String plainText,List<String> listPublicKey)

| 可以使用多个账户公钥对字段级数据加密。

参数说明
-----------

1. ``plainText`` - ``String`` : 字段级数据；
2. ``listPublicKey`` - ``JsonArray`` : 账户公钥集合。

返回值
-----------

``String`` : 返回加密结果。

示例
-----------
.. code-block:: javascript

    String plainText = "hello world";
    List<String> pubList = new ArrayList<>();
    pubList.add("cBP7JPfSVPgqGfGXVJVw168sJU5HhQfPbvDRZyriyKNeYjYLVL8M");
    pubList.add("cBPaLRSCwtsJbz4Rq4K2NvoiDZWJyL2RnfdGv5CQ2UFWqyJ7ekHM");

    String cipherText = c.encrypt(plainText,pubList);
    System.out.println("加密后的密文为 : " + cipherText);

------------------------------------------------------------------------------


-------------------
decrypt
-------------------
.. code-block:: javascript

	chainsql.decrypt(String cipher,String secret)

| 使用私钥对字段级数据解密。

参数说明
-----------

1. ``cipher`` - ``String`` : 加密结果；
2. ``secret`` - ``String`` : 账户私钥。

返回值
-----------

``String`` : 返回解密结果。

示例
-----------
.. code-block:: javascript

    String plain = c.decrypt(cipherText,"xpvPjSRCtmQ3G99Pfu1VMDMd9ET3W");
    System.out.println("解密后的明文为 : " + plain);
    plain = c.decrypt(cipherText,"xnHAcvtn1eVLDskhxPKNrhTsYKqde");
    System.out.println("解密后的明文为 : " + plain);

------------------------------------------------------------------------------


-------------------
symEncrypt
-------------------
.. code-block:: javascript

	EncryptCommon.symEncrypt(byte[] plainBytes,byte[] password,boolean bSM)

| 使用密钥对明文进行加密。

参数说明
-----------

1. ``plainBytes`` - ``String`` : 加密明文；
2. ``password`` - ``String`` : 加密密钥；
3. ``bSM`` - ``boolean`` : 是否使用国密算法。如果使用国密算法，注意密钥是16位。

返回值
-----------

``byte[]`` : 加密结果。

示例
-----------
.. code-block:: javascript

    byte[] password = Util.getRandomBytes(16);
    byte[] rawBytes = EncryptCommon.symEncrypt("test".getBytes(),password, true);

------------------------------------------------------------------------------


-------------------
symDecrypt
-------------------
.. code-block:: javascript

	EncryptCommon.symDecrypt(byte[] cipherText, byte[] password, boolean bSM)

| 使用密钥对密文进行解密。

参数说明
-----------

1. ``cipherText`` - ``String`` : 解密密文；
2. ``password`` - ``String`` : 解密密钥；
3. ``bSM`` - ``boolean`` : 是否使用国密算法。如果使用国密算法，注意密钥是16位。

返回值
-----------

``byte[]`` : 返回解密后的明文。

示例
-----------
.. code-block:: javascript

    byte[] newBytes = EncryptCommon.symDecrypt(rawBytes, password, true);
    System.out.println("解密后的明文为 : " +  new String(newBytes));

------------------------------------------------------------------------------

-------------------
asymEncrypt
-------------------
.. code-block:: javascript

	EncryptCommon.asymEncrypt(byte[] plainBytes,byte[] publicKey)

| 使用公钥对明文进行加密。

参数说明
-----------

1. ``plainBytes`` - ``String`` : 加密明文；
2. ``publicKey`` - ``String`` : 加密公钥；

返回值
-----------

``byte[]`` : 返回加密结果。

示例
-----------
.. code-block:: javascript

    //SM
    byte [] pubBytes = getB58IdentiferCodecs().decode("pYvXDbsUUr5dpumrojYApjG8nLfFMXhu3aDvxq5oxEa4ZSeyjrMzisdPsYjfxyg9eN3ZJsNjtNENbzXPL89st39oiSp5yucU", B58IdentiferCodecs.VER_ACCOUNT_PUBLIC);
    byte[] rawBytes = EncryptCommon.asymEncrypt("test".getBytes(),pubBytes);

    //ecies
    IKeyPair keyPair = Seed.getKeyPair("xpvPjSRCtmQ3G99Pfu1VMDMd9ET3W");
    rawBytes = EncryptCommon.asymEncrypt("test".getBytes(),keyPair.canonicalPubBytes());

------------------------------------------------------------------------------

-------------------
asymDecrypt
-------------------
.. code-block:: javascript

	EncryptCommon.asymDecrypt(byte[] cipher,byte[] privateKey,boolean bSM)

| 使用私钥对密文进行解密。

参数说明
-----------

1. ``cipher`` - ``byte[]`` : 解密密文；
2. ``privateKey`` - ``byte[]`` : 解密私钥；
3. ``bSM`` - ``boolean`` : 是否使用国密算法。

返回值
-----------

``byte[]`` : 返回解密明文。

示例
-----------
.. code-block:: javascript

    //SM
    byte[] seedBytes   = getB58IdentiferCodecs().decodeAccountPrivate("pwRdHmA4cSUKKtFyo4m2vhiiz5g6ym58Noo9dTsUU97mARNjevj");
    byte[] newBytes = EncryptCommon.asymDecrypt(rawBytes, seedBytes, true);
    System.out.println("解密后的明文为 : " +  new String(newBytes));

    //ecies
    seedBytes   = getB58IdentiferCodecs().decodeFamilySeed("xpvPjSRCtmQ3G99Pfu1VMDMd9ET3W");
    newBytes = EncryptCommon.asymDecrypt(rawBytes, seedBytes, false);
    System.out.println("解密后的明文为 : " +  new String(newBytes));

