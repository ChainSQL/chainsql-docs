.. _CAFeatures:

CA功能
###########################

1 简介
*************************

1.1 基本概念
=============================================

    CA是 ``Certificate Authority`` 的缩写，也叫“证书授权中心”。

- 它是负责管理和签发证书的第三方机构，作用是检查证书持有者身份的合法性，并签发证书，以防证书被伪造或篡改。

- 一般来说，CA必须是所有行业和所有公众都信任的、认可的，就像负责颁发身份证的公安局、负责发放行驶证、驾驶证的车管所。

    CA证书，顾名思义，就是CA颁发的证书。

- CA证书也就我们常说的数字证书，包含证书拥有者的身份信息，CA机构的签名，公钥和私钥。身份信息用于证明证书持有者的身份；CA签名用于保证身份的真实性；公钥和私钥用于通信过程中加解密，从而保证通讯信息的安全性。

- CA是权威可信的第三方机构，是“发证机关”。CA证书是CA发的“证件”，用于证明自身身份，就像身份证和驾驶证。

------------

2 功能介绍
*************************


主要参考 ``HyperLedger-Fabric`` 对交易准入的设计，在ChainSQL中实现对于 CA证书的签发及校验。

- CA证书的签发
- CA证书的校验

2.1 CA证书的签发
=============================================

CA证书的签发主要涉及到以下主体：

- CA工具：根据ChainSQL的账户seed生成CA证书请求
- 客户端：通过CA工具生成CA证书请求文件
- CA签发机构: 生成根证书以及签发ChainSQL账户证书
- 节点：配置根证书


在 ``0.30.5`` 版本中，CA工具集成到了ChainSQL节点程序中，CA签发机构的功能通过  ``OPENSSL``  实现。 

CA证书签发的时序图如下:

    .. image:: ../../images/caSignature.png
        :width: 541px
        :alt: ChainSQL Framework
        :align: center

-----------

2.1 CA证书的校验
=============================================

CA 证书的校验，是通过在ChainSQL API发送的交易请求中带上CA证书的信息，通过Chainsql节点程序实现证书的校验。

节点对交易中CA证书的校验流程如下图:

    .. image:: ../../images/caVerify.png
        :width: 990px
        :alt: ChainSQL Framework
        :align: center

-------------------


3 具体使用
*************************
包括以下几个部分

- 生成自签名根证书
- 证书请求文件生成
- 签发用户证书
- 节点配置文件
-  API接口

3.1 生成自签名根证书
=============================================
- 通过 ``OpenSSL`` 生成根证书以及签名证书，详细文档参见 `openssl 生成根证书以及签名证书 <https://blog.csdn.net/xiangguiwang/article/details/80333728/>`_ 

.. code-block:: bash

    # 1 生成CA私钥
    openssl ecparam -genkey -name secp256k1 -out key.pem

    # 2 生成CA自签名根证书
    openssl req  -new -x509 -days 365 -key key.pem -out ca.cert


3.2 证书请求文件生成
=============================================

- 使用账户的私钥生成证书请求文件 ``x509Req.csr``

.. code-block:: bash

    ./chainsqld_classic --conf="chainsqld.cfg" gen_csr "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb" "CN BJ BJ Peersafe RC"

3.3 签发用户证书
=============================================
- 用户将证书请求文件 ``x509Req`` 发送给根CA，然后在根CA目录下执行以下命令签署用户证书 ``userCert.cert``

.. code-block:: bash

    # 签署用户证书
    openssl x509 -req -in x509Req.csr -out userCert.cert -CA ca.cert -CAkey key.pem -CAcreateserial

------------------------


3.4 节点配置文件
=============================================

 - 新增加根证书配置项  ``[x509_crt_path]`` ，该选项表示X509 根证书文件路径

.. code-block:: bash
d
    # X509 根证书文件路径
    [x509_crt_path]
    ./ca1.cert
    ./ca2.cert

- 配置CA证书服务器公钥 ``[ca_certs_keys]`` 以及地址 ``[ca_certs_sites]``

.. code-block:: bash

    [ca_certs_keys]
    029d1f40fc569fff2a76417008d98936a04417db0758c8ab123dee6dbd08d79398

    [ca_certs_sites]
    http://192.168.29.112:8081/

------------------------

3.5 API接口
=============================================

 - 增加接口  ``useCert``，用于设置账户的CA证书信息

 .. code-block:: javascript

    // JAVA  API
    /**
    * @param userCert  用户证书的文件内容
    */
    public void useCert(String userCert) {
        this.connection.userCert = userCert;

    }

    // NodeJS API 
    /**
    * @param userCert  用户证书的文件内容
    */
    ChainsqlAPI.prototype.useCert = function (userCert) {
        this.connect.useCert(userCert);
    }


