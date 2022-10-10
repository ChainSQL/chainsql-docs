.. _NodeAPI_entry:

===========
Node.js接口
===========

ChainSQL提供nodejs-API与节点进行交互，实现基础交易发送、链数据查询、数据库操作、智能合约操作等ChainSQL区块链交互操作。

环境准备
===========

Nodejs接口的使用，需要以下准备：

1. 安装nodejs环境
2. 安装chainsql的nodejs模块

.. _Node环境:

---------------
安装nodejs环境
---------------
.. _nodejs下载: https://nodejs.org/zh-cn/download/

 | 初次安装nodejs，可以到官方 `nodejs下载`_ 网址下载对应平台的最新版本安装包。
 | ChainSQL推荐使用nodejs的版本为 **v8.5.0及以上** 。可以通过命令 ``node -v`` 查看本机nodejs版本。

.. _require-chainsql:

-------------------------
安装chainsql的nodejs模块
-------------------------
在nodejs项目文件中使用 ``npm i chainsql`` 命令即可以将chainsql的nodejs模块安装到项目中，然后按照以下方式引入：

.. code-block:: javascript

	const ChainsqlAPI = require('chainsql');
	
	// 引入之后使用new创建全局chainsql对象，之后使用chainsql对象进行接口操作
	const chainsql = new ChainsqlAPI();

.. IMPORTANT::

    ChainsqlAPI不支持并发操作，如果需要并发发交易，需要在各并发任务中构造各自的ChainsqlAPI对象 。

---------------------------------------------
chainsql的nodejs模块对于多密码算法的支持方式
---------------------------------------------
nodejs的 :ref:`as <nodejsAS>` 确定nodejs-SDK以何种算法运行:

- 当as了一个secp256k1算法账户时，会使用sha哈希算法，此时能跟同样使用secp256k1的节点进行交互；
- 当as了一个国密算法账户时，会使用sm3哈希算法，此时能跟同样使用gmalg节点进行交互；
- SDK使用的算法必须和节点一致

版本变化
===========
nodejs接口在迭代过程中，有几次优化涉及接口使用方式的改变，包括以下方面：

1. **ChainSQL模块引入方式**

| 在 **0.6.57版本** 及之后的版本，引用chainsql使用 :ref:`最新方式 <require-chainsql>`；
| 在 **0.6.57版本** 之前，引用chainsql需要用如下方式：

.. code-block:: javascript

	const ChainsqlAPI = require('chainsql').ChainsqlAPI;

2. **pay接口调用方式**

| 在 **0.6.35版本** 及之后的版本中，:ref:`pay接口 <pay-introduce>` 统一按照现行交易提交方式，调用submit接口进行提交。
| 在 **0.6.35版本** 之前, pay接口不需要调用submit接口就可以发送pay操作。

3. **submit接口交易调用默认expect值**

| 在 **0.6.54版本** 及之后的版本中， 调用submit接口不传递 ``expect`` 参数的时候默认以"send_success"的方式进行交易提交；
| 在 **0.6.54版本** 之前的版本中，调用submit接口不传递 ``expect`` 参数的时候默认以"validate_success"的方式进行交易(非合约类)提交。

4. **get接口返回空值方式**

| 在 **ChainSQL节点0.30.3 release版本** 之前，调用get接口查询时，无数据返回时，返回一个 ``null``；
| **ChainSQL节点0.30.3 release版本** 无数据返回时，返回一个空的数组：``[]``

5. **useCert接口**

| 在 **0.6.56版本** 增加接口  :ref:`useCert <UseCertNodeJS>` 支持 :ref:`CA功能  <CAFeatures>` 

6. **generateAddress接口**

| 在 **0.70.1版本** 修改接口  :ref:`generateAddress <generateAddressNodeJS>` 支持国密算法

------------------------------------------------------------------------------



.. _Node.js返回值:

返回值格式
===========

ChainSQL的接口类型可以分为交易类和查询类，每种接口类型返回数据格式不同，下面就交易类和查询类进行说明。

----------------
交易类接口返回值
----------------

交易类接口是指提交内容需要经过ChainSQL区块链共识的。在nodejs的接口里主要通过submit接口去提交交易，返回值格式可以参见 :ref:`submit接口返回值 <submit-return>`。

----------------
查询类接口返回值
----------------

查询类接口返回值正常返回格式在各个接口有说明，错误返回值为一个 ``JsonObject`` 对象，具体包含以下字段：

	* ``name`` - ``String`` : 错误类型或者说错误码；
	* ``message`` - ``String`` : 详细错误说明。

------------------------------------------------------------------------------

基础接口
===========

.. _connect-api:

-----------
connect
-----------
.. code-block:: javascript

	chainsql.connect(wsAddr[, callback])

如果需要nodejs接口与节点进行交互，必须首先设置节点的websocket地址，与节点进行正常连接。

参数说明
-----------

1. ``wsAddr`` - ``String`` : 节点的websocket访问地址，格式为：``"ws://ip:port"`` 。
2. ``callback`` - ``Function`` : [**可选**]回调函数，如果指定，则通过指定回调函数返回结果，否则需要通过then和catch接收promise结果。

返回值
-----------

``JsonObject`` - 通过提供的websocket地址与节点连接的结果。返回方式取决于是否指定回调函数。

1. 连接成功 - 指定回调函数，则通过回调函数返回，否则返回一个resolve的Promise对象。 主要内容在返回值resObj的 ``resObj.api.connect`` 中，为一个 ``JsonObject``, 包括以下字段：
	
	* ``_fee_base`` - ``Number`` : 节点的基础交易手续费；
	* ``_ledgerVersion`` - ``Number`` : 当前节点区块高度；
	* ``_maxListeners`` - ``Number`` : 节点的最大连接数。
	* ``_url`` - ``String`` : 目前与节点连接使用的websocket地址
	
2. 部署失败 - 指定回调函数，则通过回调函数返回，否则返回一个reject的Promise对象。 ``JsonObject`` 主要包含以下字段：

	* ``message`` - ``String`` : 返回与指定ws地址链接失败："connect ECONNREFUSED wsAddr"；
	* ``name`` - ``String`` : 固定值："NotConnectedError"；

示例
-----------
.. code-block:: javascript

	// use the promise
	chainsql.connect("ws://127.0.0.1:6006").then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});

	// use the callback
	chainsql.connect("ws://101.201.40.123:5006", function(err, res) {
		err ? console.error(err) : console.log(res);
	});

------------------------------------------------------------------------------

.. _nodejsAS:

-----------
as
-----------
.. code-block:: javascript

	chainsql.as(user)

部分接口与节点进行交互操作前，需要指明一个全局的操作账户，这样避免在每次接口的操作中频繁的提供账户。再次调用该接口即可修改全局操作账户。

.. note::
	as接口必须在 :ref:`connect接口 <connect-api>` 调用之后使用。

参数说明
-----------
.. _user-Format:

1. ``user`` - ``JsonObject`` : chainsql账户，包含账户地址、账户私钥和账户公钥，其中账户公钥为可选字段。

.. code-block:: javascript

	const user = {
		secret: "xhFyeP6kFEtm......Rx7Zqu2xFvm",
		address: "zLtH4NFSqDFioq......Lf8xcyyw7VCf",
		publicKey: "cBPjhaYGqKMAShw......7Sd3JHKv5L3Uj2yVJfdV"
	};

返回值
-----------

None

示例
-----------
.. code-block:: javascript

	const user = {
		secret: "xhFyeP6kFEtm7KSyACRx7Zqu2xFvm",
		address: "zLtH4NFSqDFioq5zifriKKLf8xcyyw7VCf",
		publicKey: "cBPjhaYGqKMAShwbL7rnudzndBXWSKV57Sd3JHKv5L3Uj2yVJfdV"
	};
	chainsql.as(user);

------------------------------------------------------------------------------

-----------
use
-----------
.. code-block:: javascript

	chainsql.use(tableOwnerAddr)

use接口主要使用场景是针对ChainSQL的表操作，为其提供 **表的拥有者账户地址** 。

.. note::
	1. use接口必须在 :ref:`connect接口 <connect-api>` 调用之后使用。
	2. use接口和as接口的区别如下：

	- as接口的目的是设置全局操作账户GU，这个GU在表操作接口中扮演表的操作者，并且默认情况下也是表的拥有者（即表的创建者）。
	- 当as接口设置的账户不是即将操作的表的拥有者时，即可能通过表授权获得表的操作权限时，需要使用use接口设置表的拥有者地址。
	- use接口只需要提供账户地址即可，设置完之后，表的拥有者地址就确定了，再次使用use接口可以修改表拥有者地址。

参数说明
-----------

1. ``tableOwnerAddr`` - ``String`` : 账户地址

返回值
-----------

None

示例
-----------
.. code-block:: javascript

	const tableOwnerAddr = "zMpUjXckTSn6NacRq2rcGPbjADHBCPsURF"
	chainsql.use(tableOwnerAddr);

------------------------------------------------------------------------------

-----------
submit
-----------
.. code-block:: javascript

	chainsql.submit([param])

| 针对ChainSQL的交易类型的操作，需要使用submit接口执行提交上链操作，交易类型的操作是指需要进行区块链共识的操作。
| 还有一类ChainSQL的查询类操作，不需要使用submit接口，不需要进行区块链共识。 
| submit接口有使用前提，需要事先调用其他操作接口将交易主体构造，比如创建数据库表，需要调用createTable接口，然后调用submit接口，详细使用方法在具体接口处介绍。

.. note::
	在ChainSQL的查询类操作中，数据库查询接口[ :ref:`get接口 <get-API-node>` ]仍然需要submit是为了进行查询权限验证。

.. _submit-param:

参数说明
-----------

1. ``param`` - ``any`` : [**可选**]submit的参数是可选的，主要有以下3中情况：

	* ``JsonObject`` : 包含一个expect字段，表明期望交易发送之后达到某种预期结果就返回，可选值为``String``格式，可选结果如下：

		- "send_success" : 交易发送成功即返回结果；
		- "validate_success" ： 交易共识成功即返回结果；
		- "db_success" ： 涉及数据库交易，执行入库成功即返回结果。

	* ``Function`` : 回调函数，从该回调函数返回结果，expect以"send_success"为默认值进行交易提交。未指定回调函数则通过then和catch接收promise结果。
	* 不填写参数，则表明没有指定回调函数，然后expect以"send_success"为默认值进行交易提交。

.. _submit-return:

返回值
-----------

``JsonObject`` : 交易执行结果的返回值，如果指定回调函数，则通过回调函数返回，否则返回一个Promise的对象。

1. 执行成功，则 ``JsonObject`` 中包含两个字段：

	* ``status`` - ``String`` : 为提交时expect后的设定值，如果没有，则默认为"send_success"；
	* ``tx_hash`` - ``String`` : 交易哈希值，通过该值可以在链上查询交易。
2. 执行失败，有以下几种情况：

	* 第一种交易共识前的字段信息有效性检测出错，``JsonObject`` 中主要包含以下字段：

		- ``name`` - ``String`` : 错误类型；
		- ``message`` - ``String`` : 错误具体描述。

	* 第二种交易提交之后共识出错，``JsonObject`` 中包含以下字段：

		- ``resultCode`` - ``String`` : 错误类型或者说错误码，可参考 :ref:`交易类错误码 <tx-errcode>`；
		- ``resultMessage`` - ``String`` : 错误具体描述。
	
	* 第三种交易提交共识后出错，主要是数据库入库操作中的错误，``JsonObject`` 中包含以下字段：

		- ``status`` 或者 ``error`` - ``String`` : 错误类型，详情见下面的表格
		- ``tx_hash`` - ``String`` : 交易哈希值。
		- ``error_message`` - ``String`` : [**可选**]在错误类型为"db_error"或者有 ``error`` 字段的时候，会额外附加错误信息。

  	* 入库相关错误说明见 :ref:`表交易错误码 <TableErrorCodes>` 

示例
-----------
.. code-block:: javascript

	const userAddr = "zMpUjXckTSn6NacRq2rcGPbjADHBCPsURF";

	//use the promise
	chainsql.pay(userAddr,"2000").submit({expect:'validate_success'}).then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});
	> if success, res:
	{
		status:"validate_success",
		tx_hash:"8626D36AE98E081DD1647A6058377DC83E79666E65B925927496913A1D71DB8A"
	}

	//use the callback
	chainsql.pay(userAddr,"2000").submit(function(err, res) {
		err ? console.error(err) : console.log(res);
	});

------------------------------------------------------------------------------

.. _pay-introduce:

---------------
pay（账户转账）
---------------
.. code-block:: javascript

	chainsql.pay(address, amount[, memos])

ChainSQL区块链中账户之间转账接口，支持系统数字资产与配置的数字资产。

.. note::

	**0.6.35** 版本之前，直接调用pay即可完成交易提交。
	**0.6.35** 版本及之后的版本，使用本接口只是构造转账交易，最后需要调用submit接口进行交易的提交。

参数说明
-----------

1. ``address`` - ``String`` : 接收转账方地址
2. ``amount`` - ``any`` : 转账数额，参数有两种提供格式：

	* ``String`` : 字符串格式，只能用于ChainSQL的系统数字资产ZXC，直接提供金额的字符串形式进行转账；
	* ``JsonObject`` : Json对象，主要针对网关配置的非系统数字资产，系统数字资产ZXC也可以用这种形式，包含以下三个字段：

		- ``value`` - ``String`` : 转账数额；
		- ``currency`` - ``String`` : 转账数字资产类型；
		- ``issuer`` - ``String`` : 该数字资产的配置网关地址。
3. ``memos`` - ``JSONArray`` : [**可选**]用于对这笔转账的说明，最后会随该笔转账交易记录到区块链上。``JSONArray`` 中的 ``JsonObject`` ，包含以下三个字段:

	- ``data`` - ``String`` : 任意的字符串，通常包含备忘录的内容；
	- ``format`` - ``String`` : [**可选**] 备忘录内容的格式，例如 ``MIME 类型`` ；
	- ``type`` - ``String`` : [**可选**] 一个唯一关系（根据RFC 3986）定义了此备忘录的格式。仅允许URL中允许的字符。

返回值
-----------

``JsonObject`` : 返回chainsql对象本身。

示例
-----------
.. code-block:: javascript

	const userAddr = "zMpUjXckTSn6NacRq2rcGPbjADHBCPsURF";

	var memos = [  
		{ 
			"data":"peersafe"      
		}
	];

	//use the promise
	chainsql.pay(userAddr, "2000",memos).submit({expect:'validate_success'}).then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});
		

------------------------------------------------------------------------------

.. _generateAddressNodeJS:

----------------
generateAddress
----------------
.. code-block:: javascript

	chainsql.generateAddress([secret][option])

生成一个ChainSQL账户，但是此账户未在链上有效，需要链上有效账户对新账户发起pay操作，新账户才有效。

参数说明
-----------

1. ``secret`` - ``String`` : [**可选**]参数为私钥，通过指定私钥，可返回基于该私钥的账户和公私钥对。该参数为空则生成一对新的公私钥及对应地址。

2.   ``option`` - ``JsonObject`` : [**可选**]参数为密码算法类型参数，通过指定密码算法的类别，可返回基于该密码算法的账户和公私钥对，具体使用参见示例。

返回值
-----------

1. ``JsonObject`` : 一个账户对象，主要包含字段如下：

	* ``address`` - ``String`` : 新账户地址，是原始十六进制的base58编码；
	* ``publicKey`` - ``String`` : 新账户公钥，是原始十六进制的base58编码；
	* ``secret`` - ``String`` : 新账户私钥，是原始十六进制的base58编码。

示例
-----------
.. code-block:: javascript

	let accountNew = chainsql.generateAddress();
	console.log(accountNew);
	>
	{
		address:"zwqrah4YEKCxLQM2oAG8Qm8p1KQ5dMB9tC"
		publicKey:"cB4uvqvj49hBjXT25aYYk91K9PwFn8A12wwQZq8WP5g2um9PJFSo"
		secret:"xnBUAtQZMEhDDtTtfjXhK1LE5yN6D"
	}

	// 使用示例
	let accountInfo = c.generateAddress({algorithm:"softGMAlg"});
	console.log(accountInfo)

	let accountInfoFromSecret = c.generateAddress({algorithm:"softGMAlg",secret:"p97evg5Rht7ZB7DbEpVqmV3yiSBMxR3pRBKJyLcRWt7SL5gEeBb"});
	console.log(accountInfoFromSecret)
	>
	{
		address:"zfQzzonsbmSsmfScGbaoX5LKHCh3bkmdfV"
		publicKey:"pYvGHGtwTpgVx6S6drWfdbBzSLsUqjxjN9ngUVC3uDMUz2P9dFpbmALeMyLQq5gPL1sYF9SmcYyMzw7qfF8kqSzoVku21KWK"
		secret:"p9qTtdZzAJTWzNbJF4HMVVUxfMEnYUDz9tygQf3GWtQbPtN6Bet"
	}
	{
		address:"zN7TwUjJ899xcvNXZkNJ8eFFv2VLKdESsj"
		publicKey:"pYvWhW4azFwanovo5MhL71j5PyTWSJi2NVurPYUrE9UYaSVLp29RhtxxQB7xeGvFmdjbtKRzBQ4g9bCW5hjBQSeb7LePMwFM"
		secret:"p97evg5Rht7ZB7DbEpVqmV3yiSBMxR3pRBKJyLcRWt7SL5gEeBb"
	}

------------------------------------------------------------------------------

---------------
getServerInfo
---------------
.. code-block:: javascript

	chainsql.getServerInfo([callback])

获取当前区块链基础信息

参数说明
-----------

1. ``callback`` - ``Function`` : [**可选**]回调函数，如果指定，则通过指定回调函数返回结果，否则返回一个promise对象。

返回值
-----------

1. ``JsonObject`` : 包含区块链基础信息，详细字段可在 :ref:`命令行server_info返回结果 <serverInfo-return>` 中查看， 主要字段介绍如下：

	* ``buildVersion`` - ``String`` : 节点程序版本
	* ``completeLedgers`` - ``String`` : 当前区块范围
	* ``peers`` - ``Number`` : peer节点数量
	* ``validationQuorum`` - ``Number`` : 完成共识最少验证节点个数

示例
-----------
.. _return-example:
.. code-block:: javascript

	//use the promise
	chainsql.getServerInfo().then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});

	//use the callback
	chainsql.getServerInfo(function(err, res) {
		err ? console.error(err) : console.log(res);
	});

------------------------------------------------------------------------------

---------------
getAccountInfo
---------------
.. code-block:: javascript

	chainsql.getAccountInfo(address[, callback])

从链上请求查询账户信息。

参数说明
-----------

1. ``address`` - ``String`` : 账户地址；
2. ``callback`` - ``Function`` : [**可选**]回调函数，如果指定，则通过指定回调函数返回结果，否则返回一个promise对象。

返回值
-----------

1. ``JsonObject`` : 包含账户基本信息。正常返回主要字段如下：

	* ``sequence`` - ``Number`` : 该账户交易次数；
	* ``zxcBalance`` - ``String`` : 账户ZXC系统数字资产的余额；
	* ``ownerCount`` - ``Number`` : 该账户在链上拥有的对象个数，如与其他账户建立的TrustLine，或者创建的一个表都作为一个对象。
	* ``previousAffectingTransactionID`` - ``String`` : 上一个对该账户有影响的交易哈希值；
	* ``previousAffectingTransactionLedgerVersion`` - ``Number`` : 上一个对该账户有影响的区块号。

.. note::
	返回值中的 ``previousAffectingTransactionID`` 里的交易不包括 TrustLine 和 挂单交易。

示例
-----------
.. code-block:: javascript

	try {
		chainsql.getAccountInfo("zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh").then(res => {
			console.log(res);
		}).catch(err => {
			console.error(err);
		});
	}catch(e){
		console.error(e);
	}
	>
	{
		ownerCount:35,
		previousAffectingTransactionID:"0099076BA13A03AEE5179A71F0A2E678DEA6891CAC6E05EC1656D6F857E071CB",
		previousAffectingTransactionLedgerVersion:4957964,
		sequence:2792,
		zxcBalance:"9974287188.189876",
	}

------------------------------------------------------------------------------

-----------
getLedger
-----------
.. code-block:: javascript

	chainsql.getLedger([[opts], [callback]])

返回指定区块的区块头信息，如果在opt中指定详细信息，则会额外返回所有交易信息或者账户状态。

参数说明
-----------

1. ``opts`` - ``JsonObject`` : [**可选**]指定区块高度，定制返回内容。如果想默认获取最新区块信息，可以不填opt参数。可选字段如下：

	* ``includeAllData`` - ``Boolean`` : [**可选**]设置为True，如果又将includeState和(或)includeTransactions设置为True，会将详细交易和(或)详细账户状态返回；
	* ``includeState`` - ``Boolean`` : [**可选**]设置为True，会返回一个包含状态哈希值的数组；如果同时将includeAllData设置为True，则会返回一个包含状态详细信息的数组；
	* ``includeTransactions`` - ``Boolean`` : [**可选**]设置为True，会返回一个包含交易哈希值的数组；如果同时将includeAllData设置为True，则会返回一个包含交易详细信息的数组；
	* ``ledgerVersion`` - ``integer`` : [**可选**]将返回此指定区块高度的区块头信息。

2. ``callback`` - ``Function`` : [**可选**]回调函数，如果指定，则通过指定回调函数返回结果，否则返回一个promise对象。

返回值
-----------

1. ``JsonObject`` : 区块信息


示例
-----------
.. code-block:: javascript

	const opts = {
		ledgerVersion : 418,
		includeAllData : false,
		includeTransactions : true,
		includeState : false
	}
	chainsql.getLedger(opts, function(err, res) {
		err ? console.error(err) : console.log(res);
	});
	> res:
	{
		closeFlags:0
		closeTime:"2019-04-18T07:58:10.000Z"
		closeTimeResolution:10
		ledgerHash:"6FBC02023B7D2042EAABB717B0C1F764658C7DAB24F20AF609DACDD9A6EFAAEA"
		ledgerVersion:666
		parentCloseTime:"2019-04-18T07:58:02.000Z"
		parentLedgerHash:"8FFB1DF583D37283232A328D5F97909A350DD92BC54491D42A04F670D62997A5"
		stateHash:"4CA23AC3023743077A75E2B180CEC976D7F96D7DFCF4E62889B45BE2106A44C7"
		totalDrops:"99999999999999988"
		transactionHash:"8636D36AE86E081DD1647A6058395DC83E79666E65B925927496418A1D71DB8A"
	}

------------------------------------------------------------------------------

-----------------
getLedgerVersion
-----------------
.. code-block:: javascript

	chainsql.getLedgerVersion([callback])

获取最新区块高度（区块号）

参数说明
-----------

1. ``callback`` - ``Function`` : [**可选**]回调函数，如果指定，则通过指定回调函数返回结果，否则返回一个promise对象。

返回值
-----------

1. ``Number`` : 最新区块高度

示例
-----------
.. code-block:: javascript

	chainsql.getLedgerVersion(function(err, res) {
		err ? console.error(err) : console.log(res);
	});
	> 503

------------------------------------------------------------------------------

.. _jsledger_txs:

-----------------
getLedgerTxs
-----------------
.. code-block:: javascript

	chainsql.getLedgerTxs(ledger_index, include_success, include_failure)

获取区块中的成功、失败交易数，以及错误交易的hash及错误码。

参数说明
-----------

1. ``ledger_index`` - ``integer`` : 指定要获取的区块的区块高度。
2. ``include_success`` - ``Boolean`` : 是否要获取所有成功交易的哈希。
3. ``include_failure`` - ``Boolean`` : 是否要获取所有失败交易的哈希和错误码。

返回值
-----------

1. 可参考RPC中的\ :ref:`ledger_txs <rpcledger_txs>`\ 接口。

示例
-----------
.. code-block:: javascript

	var txs = await chainsql.getLedgerTxs(1000, true, true);
	console.log(txs);

.. warning::

  此API为\ :ref:`PoP共识版本 <PoP共识版本>`\ 新增API，只适用于PoP共识版本。

------------------------------------------------------------------------------

-----------------------
getAccountTransactions
-----------------------
.. code-block:: javascript

	chainsql.getAccountTransactions(address[, opts][, cb])

获取账户的交易信息

参数说明
-----------

1. ``address`` - ``String`` : 账户地址
2. ``opts`` - ``JsonObject`` : [**可选**]返回值限定值，可选字段如下：

	* ``counterparty`` - ``address`` : [**可选**]如果指定该选项，则节点只返回涉及指定网关的交易；
	* ``earliestFirst`` - ``boolean`` : [**可选**]如果为True，则节点将按最早的交易排前为顺序，默认是最新交易排前；
	* ``excludeFailures`` - ``boolean`` : [**可选**]如果为True，则节点只返回成功的交易；
	* ``initiated`` - ``boolean`` : [**可选**]如果为True，则节点只返回以第一个参数address作为交易发起者的交易，如果为False，则只返回address不是交易发起者的交易；
	* ``limit`` - ``integer`` : [**可选**]限定节点返回的交易个数；
	* ``maxLedgerVersion`` - ``integer`` : [**可选**]节点返回此指定区块之前区块中该账户的交易；
	* ``minLedgerVersion`` - ``integer`` : [**可选**]节点返回此指定区块之后区块中该账户的交易；
	* ``start`` - ``string`` : [**可选**]某个交易哈希值，如果指定，则从指定交易哈希的交易开始按序返回交易。不可以和maxLedgerVersion和minLedgerVersion一起使用；
	* ``types`` - ``TransactionTypes`` : [**可选**]节点只返回该交易类型的交易；

3. ``callback`` - ``Function`` : [**可选**]回调函数，如果指定，则通过指定回调函数返回结果，否则返回一个promise对象。。

返回值
-----------

1. ``Array`` : 返回一个包含交易信息的数组，每个交易信息的字段可参考getTransaction的返回值；如果指定回调函数，则通过回调函数返回，否则返回一个Promise对象。

示例
-----------
.. code-block:: javascript

	//use the callback
	const address = "zMpUjXckTSn6NacRq2rcGPbjADHBCPsURF";
	const opt = {limit:1};
	chainsql.getAccountTransactions(address, opt, function(err, res) {
		err ? console.error(err) : console.log(res);
	});
	>
	[
		{
			"type":"payment",
			"address":"zHyz3V6V3DZ2fYdb6AUc5WV4VKZP1pAEs9",
			"sequence":2791,
			"id":"0099076BA13A03AEE5179A71F0A2E678DEA6891CAC6E05EC1656D6F857E071CB",
			"specification":{
				"source":{
					"address":"zHyz3V6V3DZ2fYdb6AUc5WV4VKZP1pAEs9",
					"maxAmount":{"currency":"ZXX","value":"10000"}
				},
				"destination":{
					"address":"zob3V1NnCsFNYdGNod3auU5asmK1SkJs7",
					"amount":{"currency":"ZXX","value":"100"}
				}
			},
			"outcome":{
				"result":"tesSUCCESS",
				"timestamp":"2019-04-30T03:49:00.000Z",
				"fee":"0.000012",
				"balanceChanges":{
					"zHyz3V6V3DZ2fYdb6AUc5WV4VKZP1pAEs9":[{"currency":"ZXC","value":"-0.000012"},{"counterparty":"zob3V1NnCsFNYdGNod3auU5asmK1SkJs7","currency":"ZXX","value":"-100"}],
					"zob3V1NnCsFNYdGNod3auU5asmK1SkJs7":[{"counterparty":"zHyz3V6V3DZ2fYdb6AUc5WV4VKZP1pAEs9","currency":"ZXX","value":"100"}]
				},
				"orderbookChanges":{},
				"ledgerVersion":4957964,
				"indexInLedger":0,
				"deliveredAmount":{
					"currency":"ZXX",
					"value":"100",
					"counterparty":"zob3V1NnCsFNYdGNod3auU5asmK1SkJs7"
				}
			}
		}
	]

------------------------------------------------------------------------------

---------------
getTransaction
---------------
.. code-block:: javascript

	chainsql.getTransaction(txHash[,meta][,meta_chain][, callback])

获取指定交易哈希值的交易详情

参数说明
-----------

1. ``txHash`` - ``String`` : 交易的哈希值
2. ``meta`` - ``String`` : [**可选**]是否查询交易影响的链上状态信息
3. ``meta_chain`` - ``String`` : [**可选**]对于表交易是否查询修改表的上一条、下一条交易哈希
4. ``callback`` - ``Function`` : [**可选**]回调函数，如果指定，则通过指定回调函数返回结果，否则返回一个promise对象。

返回值
-----------

``JsonObject`` : 交易详情。

示例
-----------
.. code-block:: javascript

	//use the callback
	const txHash = "F1C0981FDFA67CB60A30BD6373029662EC0BBCF3FC50BB02C98812726369B3F9";
	var ret = await chainsql.getTransaction(txHash);
	console.log(ret)
	
	>
	{
		"type":"payment",
		"address":"zHyz3V6V3DZ2fYdb6AUc5WV4VKZP1pAEs9",
		"sequence":2791,
		"id":"0099076BA13A03AEE5179A71F0A2E678DEA6891CAC6E05EC1656D6F857E071CB",
		"specification":{
			"source":{
				"address":"zHyz3V6V3DZ2fYdb6AUc5WV4VKZP1pAEs9",
				"maxAmount":{"currency":"ZXX","value":"10000"}
			},
			"destination":{
				"address":"zob3V1NnCsFNYdGNod3auU5asmK1SkJs7",
				"amount":{"currency":"ZXX","value":"100"}
			}
		},
		"outcome":{
			"result":"tesSUCCESS",
			"timestamp":"2019-04-30T03:49:00.000Z",
			"fee":"0.000012",
			"balanceChanges":{
				"zHyz3V6V3DZ2fYdb6AUc5WV4VKZP1pAEs9":[{"currency":"ZXC","value":"-0.000012"},{"counterparty":"zob3V1NnCsFNYdGNod3auU5asmK1SkJs7","currency":"ZXX","value":"-100"}],
				"zob3V1NnCsFNYdGNod3auU5asmK1SkJs7":[{"counterparty":"zHyz3V6V3DZ2fYdb6AUc5WV4VKZP1pAEs9","currency":"ZXX","value":"100"}]
			},
			"orderbookChanges":{},
			"ledgerVersion":4957964,
			"indexInLedger":0,
			"deliveredAmount":{
				"currency":"ZXX",
				"value":"100",
				"counterparty":"zob3V1NnCsFNYdGNod3auU5asmK1SkJs7"
			}
		}
	}

------------------------------------------------------------------------------

-----------
sign
-----------
.. code-block:: javascript

	chainsql.sign(txJson, secret[, option])

交易签名接口，只能对交易进行签名。

参数说明
-----------

1. ``txJson`` - ``JsonObject`` : 交易对象，不同交易类型，结构不同。对chainsql的表及合约交易结构的说明可参考 :ref:`rpc接口 <rpc-tx>` 中每个接口的tx_json字段值；
2. ``secret`` - ``String`` : 签名者的私钥。
3. ``option`` - ``JsonObject`` : [**可选**]进行多方签名时，需要利用option参数提供签名账户地址，此时secret参数应为option中账户的私钥。此对象只包含一个字段：
	
	* ``signAs`` - ``String`` : 字符串，签名账户的地址。

返回值
-----------

``JsonObject`` : 返回的签名结果，主要包含以下字段：

	* ``signedTransaction`` - ``String`` : 原交易序列化之后的十六进制；
	* ``id`` - ``String`` : 交易签名之后的十六进制签名值。

示例
-----------
.. code-block:: javascript

	const user = {
		address:"zwqrah4YEKCxLQM2oAG8Qm8p1KQ5dMB9tC",
		publicKey:"cB4uvqvj49hBjXT25aYYk91K9PwFn8A12wwQZq8WP5g2um9PJFSo",
		secret:"xnBUAtQZMEhDDtTtfjXhK1LE5yN6D"
	};

	let info = await chainsql.getAccountInfo(user.address);
	chainsql.getLedgerVersion( function(err,data){
		const payment = {
			"Account": "zwqrah4YEKCxLQM2oAG8Qm8p1KQ5dMB9tC",
			"Amount":"1000000000",
			"Destination": "zPBJ6Ai7JoQxYcYhF8gqsBDdhUVdaqQq2u",
			"TransactionType": "Payment",
			"Sequence": info.sequence,
			"LastLedgerSequence":data + 5,
			"Fee":"50"
		}
		 let signedRet = chainsql.sign(payment, user.secret);
		 console.log(signedRet);
	 }); 
	>
	{
  		"signedTransaction": "12000322800000002400000017201B0086955368400000000000000C732102F89EAEC7667B30F33D0687BBA86C3FE2A08CCA40A9186C5BDE2DAA6FA97A37D874473045022100BDE09A1F6670403F341C21A77CF35BA47E45CDE974096E1AA5FC39811D8269E702203D60291B9A27F1DCABA9CF5DED307B4F23223E0B6F156991DB601DFB9C41CE1C770A726970706C652E636F6D81145E7B112523F68D2F5E879DB4EAC51C66980503AF",
  		"id": "02BAE87F1996E3A23690A5BB7F0503BF71CCBA68F79805831B42ABAD5913BA86"
	}
	//multisigning
	const signer = {
		address:"zN9xbg1aPEP8aror3n7yr6s6JBGr7eEPeC",
		publicKey:"cBQV4TokXMD4fumjtUrBi1Jj8p6iAeK74F2SeHih7jnTECzXYdJU",
		secret:"xnrMj6Nyc3KYHn244KdMyaJn1wLUy"
	}
	const option = {
		signAs:"zN9xbg1aPEP8aror3n7yr6s6JBGr7eEPeC"
	}
	let signedRet = chainsql.sign(payment, signer.secret, option);

------------------------------------------------------------------------------

-----------------
getTableNameInDB
-----------------
.. code-block:: javascript

	chainsql.getTableNameInDB(address, tableName);

查询表在数据库中的记录的名字，在外定义的表名经过ChainSQL会进行格式转换，此接口查询转换之后的名字。

参数说明
-----------

1. ``address`` - ``String`` : 账户地址,表的拥有者；
2. ``tableName`` - ``String`` : 原始表名

返回值
-----------

``tableNameInDB`` - ``String`` : 通过Promise对象返回转换之后的表名。

示例
-----------
.. code-block:: javascript

	chainsql.getTableNameInDB(owner.address, sTableName).then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});
	>
	"B3E76AD6CAF7D483C5339787FF0257C58223C8CB"

------------------------------------------------------------------------------

-------------
getTableAuth
-------------
.. code-block:: javascript

	chainsql.getTableAuth(ownerAddr, tableName[, account])

参数说明
-----------

1. ``ownerAddr`` - ``String`` : 表的拥有者地址；
2. ``tableName`` - ``String`` : 表名；
3. ``account`` - ``Array`` : 查询指定账户的授权情况，当此参数不填时，返回指定表的所有授权情况。指定账户数组，只返回指定账户的授权情况。

返回值
-----------

``JsonObject`` : 由以下字段组成：

	* ``owner`` - ``String`` : 表的拥有者地址；
	* ``tablename`` - ``String`` : 表名；
	* ``users`` - ``Array`` : 由 ``JsonObject`` 组成的数组，每个 ``JsonObject`` 为某个账户的授权信息，具体字段如下：

		- ``account`` - ``String`` : 被授权账户地址；
		- ``authority`` - ``JsonObject`` : 具体被授予的操作权限，key为insert、delete、update、select，value为true或者false；

示例
-----------
.. code-block:: javascript

	chainsql.getTableAuth(owner.address,"b1",[]).then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});
	>
	{
		owner:"zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
		tablename:"tableTest",
		users:[
			{
				account:"zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
				authority:{
					delete:true,
					insert:true,
					select:true,
					update:true
				}
			},
			{
				account:"zLtH4NFSqDFioq5zifriKKLf8xcyyw7VCf",
				authority:{
					delete:false,
					insert:true,
					select:true,
					update:false
				}
			}
		]
	}

------------------------------------------------------------------------------

-----------------
getAccountTables
-----------------
.. code-block:: javascript

	chainsql.getAccountTables(address[, bGetDetailInfo])

查询账户名下创建的表。

参数说明
-----------

1. ``adress`` - ``String`` : 账户地址；
2. ``bGetDetailInfo`` - ``bGetDetailInfo`` : [**可选**]默认为false。为True时获取详细信息。

返回值
-----------

``JsonObject`` : key为tables， 值为一个数组，由账户下所有表信息组成，每个表信息为一个 ``JsonObject`` ，包含字段取决于bGetDetailInfo，为false，只包含ledger_index、nameInDB、tablename、tx_hash。为true，详细字段信息如下：

	* ``confidential`` - ``Boolean`` : 是否是加密表；
	* ``create_time`` - ``Number`` : 表创建时间，chainsql时间的秒数；
	* ``ledger_hash`` - ``String`` : 建表交易所在区块的区块哈希值；
	* ``ledger_index`` - ``Number`` : 建表交易所在区块的区块高度；
	* ``nameInDB`` - ``String`` : 转换之后的数据库表名；
	* ``raw`` - ``String`` : 建表交易raw的十六进制形式；
	* ``table_exist_inDB`` - ``Boolean`` : 表是否真正在数据库中；
	* ``tablename`` - ``String`` : 表名的可读形式；
	* ``tx_hash`` - ``String`` : 建表交易的哈希值。

示例
-----------
.. code-block:: javascript

	const ownerAddr = "zMpUjXckTSn6NacRq2rcGPbjADHBCPsURF";
	chainsql.getAccountTables(ownerAddr, true).then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});
	>
	{
		tables : [
			{
				confidential:true
				create_time:608977031
				ledger_hash:"8AC443126E74AE250CD438EF5414358D23F79CCA5A67A375AB19077FD24DBAB9"
				ledger_index:11657
				nameInDB:"B3E76AD6CAF7D483C5339787FF0257C58223C8CB"
				raw:"594EC5E54AB2E6EBF8B2AC61A3A8839D5B2C3D15EEDD0CFA10D0D5EC3FD9A0C519B8AA03F3C4D0EDC1F9C5CF89C20EE684B510F5B4D14D980BEB60511BCB8852ED68F2FD75A7F5AF46DACB4380A72D5E3D89A0F3A1CD39B47A2EACC3C384F6F2EA2307FD497966CCC9317AA58BD01C56D79BCA8FBF0997393D0AD88546B767F8750F1CF17DD4523DAF6659B6697C8D955D37970C1E8A040961DFA6DBEA8E13C9"
				table_exist_inDB:false
				tablename:"tableTest"
				tx_hash:"2282CC29603E5141EAC96B25FF4BF10E896103A06F7C055FCA90C9956B6C98F9"
			}
		]
	}

------------------------------------------------------------------------------

.. _UseCertNodeJS:

-----------------
useCert
-----------------

.. code-block:: javascript

  chainsql.useCert(userCert)

用于设置账户的CA证书信息。


参数说明
-----------

1. ``userCert``  - ``String``: 用户证书的文件内容.

返回值
-----------

示例
-----------

.. code-block:: javascript

	chainsql.useCert("-----BEGIN CERTIFICATE-----\n"+
		"MIIC2TCCAcGgAwIBAgICEEkwDQYJKoZIhvcNAQELBQAwczELMAkGA1UEBhMCQ04x\n"+
		"EDAOBgNVBAgMB1RpYW5qaW4xEDAOBgNVBAcMB1RpYW5qaW4xFTATBgNVBAoMDENI\n"+
		"SU5BU1NMIEluYzEpMCcGA1UEAwwgQ0hJTkFTU0wgQ2VydGlmaWNhdGlvbiBBdXRo\n"+
		"b3JpdHkwHhcNMTkwOTA1MDgxOTA1WhcNMjAwOTA0MDgxOTA1WjA6MQswCQYDVQQG\n"+
		"EwJDTjELMAkGA1UECAwCQkoxETAPBgNVBAoMCFBlZXJzYWZlMQswCQYDVQQDDAJS\n"+
		"QzBWMBAGByqGSM49AgEGBSuBBAAKA0IABDDn/J1WuyXWiTuj8xeuW88zsykb1j2z\n"+
		"JlSjEyIvf9AggBfOJ3FHVCCwgJsfIAr6ZK23X5psdD0+LtNE5+ydEiOjfjB8MAkG\n"+
		"A1UdEwQCMAAwLwYJYIZIAYb4QgENBCIWIENISU5BU1NMIENlcnRpZmljYXRpb24g\n"+
		"QXV0aG9yaXR5MB0GA1UdDgQWBBSbvkh1rza59QJoFkctRh8GWYO8cTAfBgNVHSME\n"+
		"GDAWgBRcHyP6yOEhMcLYN/aI/NJvwlRDMzANBgkqhkiG9w0BAQsFAAOCAQEABwBp\n"+
		"wG/USPuPXIgQQAI1MLCjBwBm0JBLa/iR7ilebthsSMqPA3r5++HlsauyoG9C8w12\n"+
		"GQeiNWeMKLtHiMK+nqdEMu3/Qu+pfJGho8pNp/xkKoHQKrhwS2nXExJk/TWyqks+\n"+
		"gV4lVHLgSt0KfDp4qZE2XjSxOSY0bxg0z65ILgG6kulMMfexPj9mWAP+9jkzrarB\n"+
		"Bnwndq1E8SH80HNC1Yjj4P9tuVZH1XBRd8dDgC7X8bwa8uq9Fv3x53LvzvkSUL7a\n"+
		"EwC2ruHjaFuOmI1uidXFwLaQ7ufbXFGE5CwYhOamPbf2evUagNnVtCAJASmI1Ysy\n"+
		"VVTCvkwclRIImtA3+Q==\n"+
		"-----END CERTIFICATE-----");

------


---------------------------------
getTransactionResult
---------------------------------

.. code-block:: javascript

  chainsql.getTransactionResult(hash)

查询交易结果。


参数说明
-----------

1. ``hash``  - ``String``: 交易哈希.

返回值
-----------
可参考 \ :ref:`tx_result <cmdtx_result>`\  接口说明

示例
-----------

.. code-block:: javascript

	let rs = await c.getTransactionResult("3C6A343D64BDEF0E45C38AFCB8D9574391EEEA61397E5FFA2D61C18CDD635032");
	console.log( "tx_result: " , JSON.stringify( rs )) ;

------

网关交易
===========

-----------
accountSet
-----------
.. code-block:: javascript

	chainsql.accountSet(option)

账户属性设置。为交易类型，需要调用submit提交交易。

参数说明
-----------

1. ``option`` - ``JsonObject`` : 账户设置的可选属性，有一下几个：

	* ``enableRippling`` - ``Boolean`` : 是否开启网关的rippling功能，即信任同一网关同一数字资产的账户之间是否可以转账；
	* ``rate`` - ``Number`` : 信任同一网关同一数字资产的账户之间转账费率，取值为1.0~2.0；
	* ``min`` - ``Number`` : 根据rate计算出转账手续费后如果小于min值，则取min值；
	* ``max`` - ``Number`` : 根据rate计算出转账手续费后如果大于max值，则取max值

.. note::

	* 关于费率：可取消设置，min/max=0,rate=1.0为取消设置。
	* 每次设置都是重新设置，之前的设置会被替代。
	* min,max可只设置一个，但这时要同时设置rate。
	* 如果只设置min与rate，max会被取消设置,同理只设置max与rate，min会被取消设置。

返回值
-----------

``JsonObject`` : 返回chainsql对象本身。

示例
-----------
.. code-block:: javascript

	var opt = {
        enableRippling: true,
        rate: 1.002,
        min: 1,
        max: 1.5
    }
	chainsql.accountSet(opt).submit({ expect: "validate_success"}).then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});

------------------------------------------------------------------------------

-------------
whitelistSet
-------------
.. code-block:: javascript

	chainsql.addWhitelistSet(whiteLists)
	chainsql.delWhitelistSet(whiteLists)

账户属性设置。为交易类型，需要调用submit提交交易。

参数说明
-----------

   * ``whiteLists`` - ``JSONArray`` :  为网关添加或删除的白名单的地址，可以同时添加或删除多个白名单。


.. note::

	* 白名单重复添加或删除会返回错误信息。

返回值
-----------

``JsonObject`` : 返回chainsql对象本身。

示例
-----------
.. code-block:: javascript

	var whiteLists = [
		{
			user: "zKQwdkkzpUQC9haHFEe2EwUsKHvvwwPHsv"
		}
	]
	//添加白名单
	res = await c.addWhitelistSet(whiteLists).submit({ expect: 'validate_success' });
	//删除白名单
	res = await c.delWhitelistSet(whiteLists).submit({ expect: 'validate_success' });

------------------------------------------------------------------------------

-----------
trustSet
-----------
.. code-block:: javascript

	chainsql.trustSet(amount)

信任网关，参数指定信任某个网关的某数字资产数量。从而可以交易该数字资产。为交易类型，需要调用submit提交交易。

参数说明
-----------

1. ``amount`` - ``JsonObject`` : Json对象，主要针对网关配置的非系统数字资产，包含以下三个字段：

		* ``value`` - ``String`` : 转账数额；
		* ``currency`` - ``String`` : 转账数字资产种类；
		* ``issuer`` - ``String`` : 该数字资产的配置网关地址。

返回值
-----------

``JsonObject`` : 返回chainsql对象本身。

示例
-----------
.. code-block:: javascript

	const amount = {
        value: 20000,
        currency: "TES",
        issuer: "zxm9ptsc1H18Unmc9cThiRwLAYikat2Gp8"
    }

    chainsql.trustSet(amount).submit({ expect: 'validate_success' }).then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});

------------------------------------------------------------------------------

--------------------------
pay（网关数字资产转账）
--------------------------

网关配置数字资产转账和普通转账接口相同，只是在amount参数处有所不同，具体格式和用法可以参考 :ref:`pay <pay-introduce>`


表交易
===========

------------
createTable
------------
.. code-block:: javascript

	chainsql.createTable(tableName, raw[, option])

创建数据库表接口，交易发起者同时也为表的拥有者，即as接口指定的账户，此操作为交易类型，需要通过调用submit将该交易提交。

参数说明
-----------

1. ``tableName`` - ``String`` : 新建表的表名；
2. ``raw`` - ``Array`` : 对新建表的属性设定，详细格式及内容可参看  :ref:`建表raw字段说明 <create-table>`；
3. ``option`` - ``JsonObject`` : [**可选**]是否创建加密表及是否开启行及控制，字段如下：

	* ``confidential`` - ``Boolean`` : [**可选**]是否创建加密表，不传该值或者为false时，创建非加密表；
	* ``operationRule`` - ``JsonObject`` : [**可选**]行及控制规则，格式及内容可参看 :ref:`行级控制规则 <recordLevel>` 。

返回值
-----------

``JsonObject`` : 返回chainsql对象本身。

示例
-----------
.. code-block:: javascript

	const raw = [
		{'field':'id','type':'int','length':11,'PK':1,'NN':1},
		{'field':'name','type':'varchar','length':50,'default':""},
		{'field':'age','type':'int'}
	];
	const option = {
		confidential: true
	}

	chainsql.createTable("tableTest", raw, option).submit({expect:'db_success'}).then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});

------------------------------------------------------------------------------

------------
renameTable
------------
.. code-block:: javascript

	chainsql.renameTable(oldName, newName)

对已创建表进行重命名操作。为交易类型，需要调用submit提交交易。

参数说明
-----------

1. ``oldName`` - ``String`` : 表当前使用的名字；
2. ``newName`` - ``String`` : 新表名。

返回值
-----------

``JsonObject`` : 返回chainsql对象本身。

示例
-----------
.. code-block:: javascript

	const oldName = "tableTest";
	const newName = "chainsqlTable";
	chainsql.renameTable(oldName, newName).submit({expect:'db_success'}).then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});

------------------------------------------------------------------------------

-----------
dropTable
-----------
.. code-block:: javascript

	chainsql.dropTable(tableName)

执行删除表操作。为交易类型，需要调用submit提交交易。

参数说明
-----------

1. ``tableName`` - ``String`` : 欲删除表的表名。

返回值
-----------

``JsonObject`` : 返回chainsql对象本身。

示例
-----------
.. code-block:: javascript

	const tableName = "chainsqlTable";
	chainsql.dropTable(tableName).submit({expect:'db_success'}).then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});

------------------------------------------------------------------------------

-----------
table
-----------
.. code-block:: javascript

	chainsql.table(tableName)

构造表对象，用于对表的增删改查操作。比如insert操作，需要调用table接口获取table对象，然后调用insert接口。

参数说明
-----------

1. ``tableName`` - ``String`` : 表名。

返回值
-----------

``JsonObject`` : 创建的对应表名的表对象tableObj，用于调用表的增删改查操作。

示例
-----------
.. code-block:: javascript

	const tableName = "tableTest";
	const raw = [
		{'id':1,'age': 111,'name':'rrr'}
	];
	chainsql.table(tableName).insert(raw).submit({expect:'db_success'}).then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});

------------------------------------------------------------------------------

-----------
insert
-----------
.. code-block:: javascript

	tableObj.insert(raw[, txHashField][,txsHashField][,optFields])

| 对表进行插入操作，tableObj是由chainsql.table接口创建的。
| 此接口为交易类型，单独使用需要调用submit接口，事务中不需要。

参数说明
-----------

1. ``raw`` - ``Array`` : 插入操作的raw，详细格式和内容可参看 :ref:`插入raw字段说明 <insert-table>` ；
2. ``txHashField`` - ``String`` : [**可选**] 插入操作支持将每次执行插入交易的哈希值作为字段同步插入到数据库中，需要提前在建表的时候指定一个字段为存储交易哈希。
3. ``txsHashField`` - ``String`` : [**可选**] 插入操作支持将操作该记录的交易hash以逗号分隔插入到数据库中，需要提前在建表的时候指定一个字段为存储交易哈希。
4. ``optFields`` - ``String`` : [**可选**] 与 ``txHashField``、 ``txsHashField`` 类似的自动填充字段，可扩展，目前支持 ``ledgerSeqField`` 与 ``ledgerTimeField``，分别代表区块号与区块生成时间。

返回值
-----------

``JsonObject`` : 返回tableObj对象本身。

示例
-----------
.. code-block:: javascript

	const tableName = "tableTest";
	const raw = [
		{'id':1,'age': 111,'name':'rrr'}
	];
	chainsql.table(tableName).insert(raw).submit({expect:'db_success'}).then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});

	//自填充字段，注意，表结构中需要有txhash,txhashes,createTime,seq等字段
	var opt = {
		ledgerTimeField:"createTime",
		ledgerSeqField:"seq"
	}
	var rs = await c.table(sTableName).insert(raw,'txhash','txhashes',opt).submit({expect:'send_success'});
	console.log("testInsert ",rs);	

------------------------------------------------------------------------------

-----------
update
-----------
.. code-block:: javascript

	tableObj.update(raw[, txHashField][,txsHashField][,optFields])

| 对表内容进行更新操作，tableObj是由chainsql.table接口创建的。
| 此接口为交易类型，单独使用需要调用submit接口，事务中不需要。

.. warning::
	更新之前需要调用tableObj.get接口指定待修改内容，且必须指定，不能为空。

参数说明
-----------

1. ``raw`` - ``Array`` : 插入操作的raw，详细格式和内容可参看 :ref:`更新raw字段说明 <update-table>` 。
2. ``txHashField`` - ``String`` : [**可选**] 更新操作支持将每次执行更新交易的哈希值作为字段同步插入到数据库中，需要提前在建表的时候指定一个字段为存储交易哈希。
3. ``txsHashField`` - ``String`` : [**可选**] 更新操作支持将操作该记录的交易hash以逗号分隔插入到数据库中，需要提前在建表的时候指定一个字段为存储交易哈希。
4. ``optFields`` - ``String`` : [**可选**] 与 ``txHashField``、 ``txsHashField`` 类似的自动填充字段，可扩展，目前支持 ``ledgerSeqField`` 与 ``ledgerTimeField``，分别代表区块号与区块生成时间。

返回值
-----------

``JsonObject`` : 返回tableObj对象本身。

示例
-----------
.. code-block:: javascript

	const tableName = "tableTest";
	chainsql.table(tableName).get({'id': 1}).update({'age':200}).submit({expect:'db_success'}).then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});

	//自填充字段，注意，表结构中需要有txhash,txhashes,updateTime等字段
	var opt = {
		ledgerTimeField:"updateTime"
	}
	var rs = await c.table(sTableName).get({'id': 7}).update({'name':"28"},"txhash","txhashes",opt).submit({expect:'db_success'});
	console.log("testUpdate",rs);

------------------------------------------------------------------------------

-----------
delete
-----------
.. code-block:: javascript

	tableObj.delete()

| 对表内容进行删除操作，tableObj是由chainsql.table接口创建的。
| 此接口为交易类型，单独使用需要调用submit接口，事务中不需要。

.. warning::
	更新之前需要调用tableObj.get接口指定待删除内容，且必须指定，不能为空。即不可删除表的所有内容。

参数说明
-----------

None

返回值
-----------

``JsonObject`` : 返回tableObj对象本身。

示例
-----------
.. code-block:: javascript

	const tableName = "tableTest";
	chainsql.table(tableName).get({'id': 5}).delete().submit({expect:'db_success'}).then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});

------------------------------------------------------------------------------

-----------
beginTran
-----------
.. code-block:: javascript

	chainsql.beginTran()

开启事务操作。必须在事务操作的开始执行。

参数说明
-----------

None

返回值
-----------

None

示例
-----------
.. code-block:: javascript

	chainsql.beginTran();

------------------------------------------------------------------------------

-----------
commit
-----------
.. code-block:: javascript

	chainsql.commit([param])

| 事务完成的提交操作，在执行完beginTran()操作之后，就可以开始执行数据库操作，执行方法与普通数据库接口的操作一致，只是不用调用submit接口，而是最后统一调用commit接口进行提交，详细可参考下述示例。
| 事务目前只支持授权grant、插入insert、删除delete、更新update、断言assert五个接口。

参数说明
-----------

1. ``param`` : 同submit参数，可参考 :ref:`submit参数说明 <submit-param>`

返回值
-----------

同submit接口， 可参考 :ref:`submit返回值 <submit-return>`

示例
-----------
.. code-block:: javascript

	const tableName = "tableTest";

	chainsql.beginTran();
	chainsql.table(tableName).insert({ 'id':2, 'age': 222, 'name': 'id2' });
	chainsql.table(tableName).insert({ 'id':3, 'age': 555, 'name': 'id3' });
	chainsql.table(tableName).get({'id': 3}).update({'age':333})
	chainsql.table(tableName).get({ 'age': 333 }).update({ 'name': 'world' });
	chainsql.commit({expect: 'db_success'}).then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});

------------------------------------------------------------------------------

-----------
grant
-----------
.. code-block:: javascript

	chainsql.grant(tableName, userAddr, raw[, userPub])

ChainSQL数据库表权限管理，控制自己创建的表被其他用户访问的权限，权限包括增删改查四种。为交易类型，需要调用submit提交交易。

参数说明
-----------

1. ``tableName`` - ``String`` : 准备授权给user的表名；
2. ``userAddr`` - ``String`` : 被授权用户地址；对新建表的属性设定，详细格式及内容可参看 :ref:`授权raw字段说明 <grant-table>`
3. ``raw`` - ``JsonObject`` : 指定这次给user授予的权限，key为增删改查，value为True或者False，希望授予的操作置位True，否则为False；
4. ``userPub`` - ``String`` : [**可选**]针对加密表，必须添加user的公钥，格式为base58编码的。非加密表不需要填此参数。

返回值
-----------

``JsonObject`` : 返回chainsql对象本身。

示例
-----------
.. code-block:: javascript

	const tableName = "tableTest";
	const userAddr = "zxm9ptsc1H18Unmc9cThiRwLAYikat2Gp8";
	
	const raw = {select:true, insert:true, update:false, delete:false};
	chainsql.grant(tableName, userAddr, raw).submit({expect:'db_success'}).then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});

------------------------------------------------------------------------------

------------
setRestrict
------------
.. code-block:: javascript

	chainsql.setRestrict(mode)

设置ChainSQL严格模式的开关。

参数说明
-----------

1. ``mode`` - ``Boolean`` : 是否开启ChainSQL严格模式，true则开启，false则关闭。新建chainsql对象之后是默认关闭的。

返回值
-----------

None

示例
-----------
.. code-block:: javascript

	chainsql.setRestrict(true);

------------------------------------------------------------------------------

--------------
setNeedVerify
--------------
.. code-block:: javascript

	chainsql.setNeedVerify(isNeed)

设置ChainSQL是否开启数据库验证的开关，用于事务交易，在事务交易中对数据库操作在共识过程中进行实际数据库操作验证。

参数说明
-----------

1. ``isNeed`` - ``Boolean`` : 是否开启ChainSQL数据库验证的开关，true则开启，false则关闭。新建chainsql对象之后是默认开启的。

返回值
-----------

None

示例
-----------
.. code-block:: javascript

	chainsql.setNeedVerify(true);

------------------------------------------------------------------------------


表查询
===========

.. _get-API-node:

-----------
get
-----------
.. code-block:: javascript

	tableObj.get([raw])

| 对表内容进行查询操作，tableObj是由chainsql.table接口创建的。
| 此接口不属于交易类型，但是为了检查用户是否有查询权限，需要签名之后调用，因此需要调用submit接口。
| 此外ChainSQL还提供了三个查询限定接口，主要包括limit、order、withField。在对应接口下查看具体说明。

.. note::
	现在查询无论数据库有多少内容，get接口一次最多返回200条结果，如果数据较多，可以结合 :ref:`limit接口 <limit-API>` 做分页查询。

参数说明
-----------

1. ``raw`` - ``JsonObject`` : [**可选**]raw参数的详细格式及内容可参看 :ref:`查询raw字段说明 <查询Raw详解>` ，需要查询所有内容时，不传raw参数即可。

返回值
-----------

``JsonObject`` : 返回tableObj对象本身。

.. _get-return:

.. note::

	通过get接口调用submit提交查询请求之后，实际返回查询结果为一个 ``JsonObject`` ，具体包含字段如下：

	* ``diff`` - ``Number`` : 数据库内容与区块链上是否是最新的，当链上最新区块与数据库所同步到的区块差值小于3，则返回0，认为可接受；差值小于3会返回差值，提示数据库内容可能不是最新的；
	* ``lines`` - ``Array`` : 查询结果组成的数组，每个元素为数据库中的一行，类型为 ``JsonObject`` ，key为字段名，value为字段值。

示例
-----------
.. code-block:: javascript

	const tableName = "tableTest";
	const raw = {
		$and: [{id: {$gt: 1}}, {id: {$lt: 100}}]
	};
	chainsql.table(tableName).get(raw).submit().then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});

.. _limit-API:

-----------
limit
-----------
.. code-block:: javascript

	tableObj.limit(option)

| 对表内容进行查询操作的限定条件，tableObj是由chainsql.table接口创建的。
| 对数据库进行分页查询.返回对应条件的数据。
| 此接口单独使用没有意义，需要在get接口之后级联调用。并可以与order和withFields依次级联，但最后需要调用submit接口。

参数说明
-----------

1. ``option`` - ``JsonObject`` : 主要包括起始位置和总条数：

	* ``index`` - ``Number`` : 起始位置；
	* ``total`` - ``Number`` : 返回总条数。

返回值
-----------

``JsonObject`` : 返回tableObj对象本身。

示例
-----------
.. code-block:: javascript

	const tableName = "tableTest";
	const raw = [{'age': 16}];
	chainsql.table(tableName).get(raw).limit({index:0,total:10}).submit().then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});

-----------
order
-----------
.. code-block:: javascript

	tableObj.order(option1[, option2[, ...]])

| 对表内容进行查询操作的限定条件，tableObj是由chainsql.table接口创建的。
| 对查询的数据按指定字段进行排序；
| 此接口单独使用没有意义，需要在get接口之后级联调用。并可以与limit和withFields依次级联，但最后需要调用submit接口。

参数说明
-----------

1. ``option`` - ``JsonObject`` : 可以添加对多个字段的升降序控制，每个JsonObject的key为字段名，value为1表示升序，-1表示降序。

返回值
-----------

``JsonObject`` : 返回tableObj对象本身。

示例
-----------
.. code-block:: javascript

	const tableName = "tableTest";
	const raw = [{'id': 1}];
	chainsql.table(tableName).get(raw).order({id:1}, {name:-1}).submit().then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});

-----------
withFields
-----------
.. code-block:: javascript

	tableObj.withFields(feildArray)

| 对表内容进行查询操作的限定条件，tableObj是由chainsql.table接口创建的。
| 从数据库查询数据,通过参数指定返回字段。
| 此接口单独使用没有意义，需要在get接口之后级联调用。并可以与limit和order依次级联，但最后需要调用submit接口。

参数说明
-----------

1. ``feildArray`` - ``Array`` : 由限定的返回字段组成的数组，每个数组元素为 ``String`` 格式。数组元素可以是字段名，也可以是其他SQL语句，SQL作为参数可参考 :ref:`字段统计 <withField-introduce>` 。

返回值
-----------

``JsonObject`` : 返回tableObj对象本身。

示例
-----------
.. code-block:: javascript

	//查询表中记录数
	const tableName = "tableTest";
	const raw = [{'id': 1}];
	chainsql.table(tableName).get(raw).withFields(["COUNT(*) as count"]).submit().then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});

------------------------------------------------------------------------------

--------------
getBySqlAdmin
--------------
.. code-block:: javascript

	chainsql.getBySqlAdmin(sql)

直接传入SQL语句进行数据库查询操作，因为直接操作数据库中的表，所以需要配合getTableNameInDB接口获取表在数据库中的真实表名。

.. note::

	* 本接口不做表权限判断，但是只能由节点配置文件中 **[port_ws_admin_local]** 的admin里所配置的ip才可以发起此接口的调用，否则调用失败。
	* 即nodejs接口运行所在的主机ip必须是配置在配置文件的 **[port_ws_admin_local]** 的admin里。
	* 因为在配置文件中配置为admin的ip认为是该节点的管理员。拥有对本节点已经同步表的查询权限。

参数说明
-----------

1. ``sql`` - ``String`` : SQL语句，用于查询。

返回值
-----------

``JsonObject`` : 返回查询结果，格式与调用get接口之后在submit中获取的结果格式一致，但是没有diff字段。具体可参考 :ref:`get接口 <get-return>`

示例
-----------
.. code-block:: javascript

	const tableOwnerAddr = "zMpUjXckTSn6NacRq2rcGPbjADHBCPsURF";
	var tableName = "tabelTest";
	let tableNameInDB = await chainsql.getTableNameInDB(tableOwnerAddr, tableName);
	tableName = "t_" + tableNameInDB;

	chainsql.getBySqlAdmin("select * from " + tableName).then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});

------------------------------------------------------------------------------

-------------
getBySqlUser
-------------
.. code-block:: javascript

	chainsql.getBySqlUser(sql)

直接传入SQL语句进行数据库查询操作，因为直接操作数据库中的表，所有需要配合getTableNameInDB接口获取表在数据库中的真实表名。

.. note::
	* 调用此接口前as接口中设置的用户，对SQL语句中的表有查询权限，才可以调用此接口。

参数说明
-----------

1. ``sql`` - ``String`` : SQL语句，用于查询

返回值
-----------

``JsonObject`` : 返回查询结果，格式与调用get接口之后在submit中获取的结果格式一致，但是没有diff字段。可参考 :ref:`get接口 <get-return>`

示例
-----------
.. code-block:: javascript

	const user = {
		secret: "xx9rPirSmNSG6SDCsyDi8Du6x7qKc",
		address: "zxm9ptsc1H18Unmc9cThiRwLAYikat2Gp8",
		publicKey: "cBQh8rTr1FuQL97rMdtPZUhAPEN9airXqGwTw5bzrTVtcHMsjaLo"
	};
	chainsql.as(user);

	const tableOwnerAddr = "zMpUjXckTSn6NacRq2rcGPbjADHBCPsURF";
	var tableName = "tabelTest";
	let tableNameInDB = await chainsql.getTableNameInDB(tableOwnerAddr, tableName);
	tableName = "t_" + tableNameInDB;

	chainsql.getBySqlUser("select * from " + tableName).then(res => {
		console.log(res);
	}).catch(err => {
		console.error(err);
	});

------------------------------------------------------------------------------


智能合约接口
============

.. toctree::
	:maxdepth: 2

	nodejsSmartContract

------------------------------------------------------------------------------

多链接口
============

.. _nodecreateSchema:

------------------
createSchema
------------------

.. code-block:: javascript

    chainsql.createSchema(schemaInfo)

创建子链

参数说明
-------------

参数为创建子链的信息，Json结构为：

1. ``SchemaName``       - ``String``:    子链名称；
2. ``WithState``        - ``String``:    是否继承主链状态；
3. ``AnchorLedgerHash`` - ``String``:     继承主链的区块Hash；
4. ``Validators``       - ``JSONArray`` : 子链节点共识公钥列表；
5. ``PeerList``         - ``JSONArray`` : 子链节点P2P连接方式列表。

返回值
---------

``JsonObject`` : 返回chainsql对象本身。


示例
---------------

.. code-block:: javascript

	// 1、 不继承主链状态
	let schemaInfo = {
		SchemaName:"hello",
		WithState:false,
		SchemaAdmin:owner.address,
		Validators:[
			{
				Validator:{PublicKey:"0249DF94DCE166BC5097FA6A447C55DD378996A091CE0450B58BACDE258EB785B3"}
			},
			{
				Validator:{PublicKey:"033E10D1FF3DC55889DCC811B537E199DE09876A6B194989D5A1926AD8FBC0FBDB"}
			},
			{
				Validator:{PublicKey:"02AF9D493C879C114BE1084EB675983F3FC457874B299B45BFA34C597E28187647"}
			}
		],
		PeerList:[
			{
				Peer:{ Endpoint:"127.0.0.1:15125"}
			},
			{
				Peer:{ Endpoint:"127.0.0.1:25125"}
			},
			{
				Peer:{ Endpoint:"127.0.0.1:35125"}
			}
		]
	}

	let ret = await c.createSchema(schemaInfo).submit({expect:'validate_success'})
	console.log("创建不继承状态子链:" + JSON.stringify(ret))

.. _nodemodifySchema:

---------------------
modifySchema
---------------------
  
.. code-block:: javascript
  
  chainsql.modifySchema(schemaInfo)
  

  修改子链

参数说明
-------------

  schemaInfo 结构为：
  
  1. ``SchemaID``         - ``String``:    子链ID；
  2. ``Validators``       - ``JSONArray`` : 要增删的子链节点共识公钥列表；
  3. ``PeerList``         - ``JSONArray`` : 要增删的子链节点P2P连接方式列表；
  4. ``ModifyType``		  - ``String``:	   修改子链类型，可取值为 ``schema_add`` 或 ``schema_del``。
  
返回值
------------
  
``JsonObject`` : 返回chainsql对象本身。
  
示例
-------------
  
.. code-block:: javascript
  
  let schemaInfo = {
	SchemaID:"6BA63B86E5CE48283D03CC21D3BE5F4630CC6572CE7F54982E5AE687C998B7A3",
	Validators:[
		{
			Validator:{PublicKey:"02BD87A95F549ECF607D6AE3AEC4C95D0BFF0F49309B4E7A9F15B842EB62A8ED1B"}
		}
	],
	PeerList:[
		{
			Peer:{ Endpoint:"192.168.29.108:5125"}
		}
	]
  }

  let ret = await c.modifySchema(schemaInfo).submit({expect:'validate_success'})
  console.log("修改子链结果：" , ret)

.. _nodedeleteSchema:

---------------------
deleteSchema
---------------------
  
.. code-block:: javascript
  
  chainsql.deleteSchema(schemaInfo)
  

  修改子链

参数说明
-------------

  schemaInfo 结构为：
  
  1. ``SchemaID``         - ``String``:    子链ID；。
  
返回值
------------
  
``JsonObject`` : 返回chainsql对象本身。
  
示例
-------------
  
.. code-block:: javascript
  
  let schemaInfo = {
	SchemaID:"6BA63B86E5CE48283D03CC21D3BE5F4630CC6572CE7F54982E5AE687C998B7A3"
  }

  let ret = await c.deleteSchema(schemaInfo).submit({expect:'validate_success'})
  console.log("删除子链结果：" , ret)

.. _nodegetSchemaList:

----------------------
getSchemaList
----------------------

.. code-block:: javascript
  
  chainsql.getSchemaList(option)

获取子链列表

参数说明
-------------
  
  参数有两个可选项，均为过滤条件，不填则查询链上所有子链

  1. ``account``  - ``String``：  可选，建链账户地址；
  2. ``running``  - ``bool``  :   可选，是否当前节点正运行中的子链。
   
返回值
---------------

| 返回值为Json数组，格式参 :ref:`rpc查询子链列表 <rpc查询子链列表>`

示例
-----------

.. code-block:: javascript

	var option = {
		"running":true
	}
	let ret = await c.getSchemaList()
	console.log("getSchemaList:" + JSON.stringify(ret))

.. _nodegetSchemaInfo:

---------------------------
getSchemaInfo
---------------------------

.. code-block:: javascript
  
  chainsql.getSchemaInfo(schemaID)

获取子链信息

参数说明
-------------

  1. ``schemaID``  - ``String``：  子链ID。
   
返回值
------------

| 返回值为Json数组，格式参考  :ref:`rpc查询子链信息 <rpc查询子链信息>`

示例
-----------

.. code-block:: javascript

	let schemaID = "6BA63B86E5CE48283D03CC21D3BE5F4630CC6572CE7F54982E5AE687C998B7A3"
	let ret = await c.getSchemaInfo(schemaID)
	console.log("getSchemaInfo:" + JSON.stringify(ret))

.. _nodesetSchema:

----------------------
setSchema
----------------------

.. code-block:: javascript
  
  chainsql.setSchema(schemaID)

设置要操作的子链ID，设置后所有的请求都针对子链进行


参数说明
-------------
  

  1. ``schemaID``  - ``String``：  子链ID。
    
返回值
------------

无

示例
-----------

.. code-block:: javascript

	await c.setSchema("AB34D23A49AD8A07D1F906135B71E8FEC8F94EE1CF41D36C1860C2CA79FB71BF")
	ret = await c.getServerInfo();
	console.log("子链server_info:" + JSON.stringify(ret))


.. note::

	因为切换子链会重新进行区块订阅，需要在setSchema同步返回后继续后面的子链操作

------------------------------------------------------------------------------


订阅
===========

---------------
subscribeTable
---------------
.. code-block:: javascript

	chainsql.event.subscribeTable(owner, tableName, callback)

| 订阅表操作的事件，提供表名和表的拥有者，就可以订阅此表。
| 普通表不用授权即可订阅。对于加密表需要被授权，即被grant之后，才可以看到交易明文，否则看到数据库操作部分为密文。

参数说明
-----------

1. ``owner`` - ``String`` : 被订阅的表拥有者地址；
2. ``tableName`` - ``String`` : 被订阅的数据库表名；
3. ``callback`` - ``Function`` : 回调函数，为必填项，需用通过此函数接收后续订阅结果。

返回值
-----------

``Promise`` : 返回一个promise对象，指示是否订阅成功。

示例
-----------
.. code-block:: javascript

	const tableOwnerAddr = "zMpUjXckTSn6NacRq2rcGPbjADHBCPsURF";
	const tableName = "tabelTest";
	chainsql.event.subscribeTable(tableOwnerAddr, tableName, function(err, data) {
		err ? console.error(err) : console.log(data)
	}).then(res => {
		console.log("subTable success.");
	}).catch(error => {
		console.error("subTable error:" + error);
	});

------------------------------------------------------------------------------

----------------
unsubcribeTable
----------------
.. code-block:: javascript

	chainsql.event.unsubcribeTable(owner, tableName)

取消对表的订阅。

参数说明
-----------

1. ``owner`` - ``String`` : 被订阅的表拥有者地址；
2. ``tableName`` - ``String`` : 被订阅的数据库表名；

返回值
-----------

``Promise`` : 返回一个promise对象，指示是否取消订阅成功。

示例
-----------
.. code-block:: javascript

	const tableOwnerAddr = "zMpUjXckTSn6NacRq2rcGPbjADHBCPsURF";
	const tableName = "tabelTest";
	chainsql.event.unsubscribeTable(tableOwnerAddr, tableName).then(res => {
		console.log("unsubTable success.");
	}).catch(error => {
		console.error("unsubTable error:" + error);
	});

------------------------------------------------------------------------------

.. _nodeAPI订阅交易:

-----------
subscribeTx
-----------
.. code-block:: javascript

	chainsql.event.subscribeTx(txHash, callback)

订阅交易事件，提供交易的哈希值，就可以订阅此交易。

参数说明
-----------

1. ``txHash`` - ``String`` : 被订阅的交易哈希值；
2. ``callback`` - ``Function`` : 回调函数，为必填项，需用通过此函数接收后续订阅结果。

返回值
-----------

``Promise`` : 返回一个promise对象，指示是否订阅成功。

示例
-----------
.. code-block:: javascript

	let txHash = "02BAE87F1996E3A23690A5BB7F0503BF71CCBA68F79805831B42ABAD5913BA86";
	chainsql.event.subscribeTx(txHash, function(err, data) {
		err ? console.error(err) : console.log(data)
	}).then(res => {
		console.log("subTx success.");
	}).catch(error => {
		console.error("subTx error:" + error);
	});

------------------------------------------------------------------------------

--------------
unsubscribeTx
--------------
.. code-block:: javascript

	chainsql.event.unsubscribeTx(txHash, callback)

取消对交易的订阅。

参数说明
-----------

1. ``txHash`` - ``String`` : 被订阅的交易哈希值。

返回值
-----------

``Promise`` : 返回一个promise对象，指示是否订阅成功。

示例
-----------
.. code-block:: javascript

	let txHash = "02BAE87F1996E3A23690A5BB7F0503BF71CCBA68F79805831B42ABAD5913BA86";
	chainsql.event.unsubscribeTx(txHash).then(res => {
		console.log("unsubTx success.");
	}).catch(error => {
		console.error("unsubTx error:" + error);
	});	

加解密
===========

---------------
signFromString
---------------
.. code-block:: javascript

	chainsql.signFromString(hexMsg,secret)

| 使用账户私钥对字符串签名。

参数说明
-----------

1. ``hexMsg`` - ``String`` : 被签名的原数据；
2. ``secret`` - ``String`` : 账户私钥。

返回值
-----------

``String`` : 返回签名结果。

示例
-----------
.. code-block:: javascript

    var secret = "xnoz9Le8yENN7U3fWxoeMymnT31XD";
    var hexMsg = Buffer.from("hello world").toString('hex');
    var signature = c.signFromString(hexMsg,secret);

------------------------------------------------------------------------------

----------------
verify
----------------
.. code-block:: javascript

	chainsql.verify(hexStr,signature,publicKey)

使用账户公钥验证签名正确性。

参数说明
-----------

1. ``hexStr`` - ``String``    : 被签名的原数据；
2. ``signature`` - ``String`` : 签名结果；
3. ``publicKey`` - ``String`` : 账户公钥。

返回值
-----------

``bool`` : 返回验证的结果。

示例
-----------
.. code-block:: javascript

    var secret = "xnoz9Le8yENN7U3fWxoeMymnT31XD";
    var wallet =  c.generateAddress(secret);

    console.log(wallet);
    var hexStr = Buffer.from("hello world").toString('hex');

    var signature = "3045022100BDC5E1154B68B6A9FFD7F7CA36CF3B79D0BF0EDF186D09D460E537EAB9BEB31002204F54BCE76918B4F7415319E62B19A2B1F7200234F049FBD524C61EB5DD4965AA";

    var bVerifyOK  = c.verify(hexStr,signature,wallet.publicKey);

------------------------------------------------------------------------------


---------------
sign
---------------
.. code-block:: javascript

	chainsql.sign(payment,secret)

| 使用账户私钥对交易签名。

参数说明
-----------

1. ``payment`` - ``JsonObject`` : 交易内容；
2. ``secret`` - ``String`` : 账户私钥。

返回值
-----------

``String`` : 返回签名结果。

示例
-----------
.. code-block:: javascript

    let info = await c.api.getAccountInfo("rHb9CJAWyB4rj91VRWn96DkukG4bwdtyTh");
    console.log(info);
    c.getLedgerVersion(function(err,data){
    var payment = {
        "Account": "rHb9CJAWyB4rj91VRWn96DkukG4bwdtyTh",
        "Amount":"1000000000",
        "Destination": "rBuLBiHmssAMHWQMnEN7nXQXaVj7vhAv6Q",
        "TransactionType": "Payment",
        "Sequence": info.sequence,
        "LastLedgerSequence":data + 5,
        "Fee":"50"
    }
    let signedRet = c.sign(payment,user.secret);
    console.log(signedRet);
    c.api.submit(signedRet.signedTransaction).then(function(data){
        console.log(data);
    });

------------------------------------------------------------------------------


---------------
encryptText
---------------
.. code-block:: javascript

	chainsql.encryptText(rawData,listPublic)

| 可以使用多个账户公钥对字段级数据加密。

参数说明
-----------

1. ``rawData`` - ``String`` : 字段级数据；
2. ``listPublic`` - ``JsonArray`` : 账户公钥集合。

返回值
-----------

``String`` : 返回加密结果。

示例
-----------
.. code-block:: javascript

	var listPublic = ["cBP7JPfSVPgqGfGXVJVw168sJU5HhQfPbvDRZyriyKNeYjYLVL8M", "cBPaLRSCwtsJbz4Rq4K2NvoiDZWJyL2RnfdGv5CQ2UFWqyJ7ekHM"];
	var cip = await chainsql.encryptText("test",listPublic);
	console.log("cipher:" + cip);

------------------------------------------------------------------------------



---------------
decryptText
---------------
.. code-block:: javascript

	chainsql.decryptText(cip,secret)

| 使用私钥对字段级数据解密。

参数说明
-----------

1. ``cip`` - ``String`` : 加密结果；
2. ``secret`` - ``String`` : 账户私钥。

返回值
-----------

``String`` : 返回解密结果。

示例
-----------
.. code-block:: javascript

	var text = await chainsql.decryptText(cip,"xpvPjSRCtmQ3G99Pfu1VMDMd9ET3W");
	console.log("plain text:" + text);
	var text2 = await chainsql.decryptText(cip,"xnHAcvtn1eVLDskhxPKNrhTsYKqde");
	console.log("plain text2:" + text2);

------------------------------------------------------------------------------


---------------
symEncrypt
---------------
.. code-block:: javascript

	chainsql.symEncrypt(symKey, plaintext, algType)

| 使用密钥对明文进行加密。

参数说明
-----------

1. ``symKey`` - ``String`` : 加密密钥；
2. ``plaintext`` - ``String`` : 加密明文；
3. ``algType`` - ``String`` : 加密算法类型："aes"、"gmAlg"、"softGMAlg"，默认加密算法是"aes"。

返回值
-----------

``String`` : 加密结果。

示例
-----------
.. code-block:: javascript

	//softGMAlg
	var symCipher = chainsql.symEncrypt("abcd","hello,world", "softGMAlg");
	console.log(symCipher);

	//aes
	var symCipher = chainsql.symEncrypt("abcd","hello,world", "aes");
	console.log(symCipher);

------------------------------------------------------------------------------


---------------
symDecrypt
---------------
.. code-block:: javascript

	chainsql.symDecrypt(symKey, encryptedHex, algType)

| 使用密钥对密文进行解密。

参数说明
-----------

1. ``symKey`` - ``String`` : 解密密钥；
2. ``encryptedHex`` - ``String`` : 解密密文；
3. ``algType`` - ``String`` : 加密算法类型："aes"、"gmAlg"、"softGMAlg"，默认加密算法是"aes"。

返回值
-----------

``String`` : 返回解密后的明文。

示例
-----------
.. code-block:: javascript

	//softGMAlg
	var plainhex = chainsql.symDecrypt("abcd",symCipher, "softGMAlg");
	var symDecrypted = utils.arrayToUtf8(utils.hexToArray(plainhex));
	console.log(symDecrypted);
	
	//aes
	var symDecrypted = chainsql.symDecrypt("abcd",symCipher, "aes");
	console.log(symDecrypted);

------------------------------------------------------------------------------


---------------
asymEncrypt
---------------
.. code-block:: javascript

	chainsql.asymEncrypt(plaintext, publicKey, algType)

| 使用公钥对明文进行加密。

参数说明
-----------

1. ``plaintext`` - ``String`` : 加密明文；
2. ``publicKey`` - ``String`` : 加密公钥；
3. ``algType`` - ``String`` : 加密算法类型："ecies"、"gmAlg"、"softGMAlg"，默认加密算法是"ecies"。

返回值
-----------

``String`` : 返回加密结果。

示例
-----------
.. code-block:: javascript

	//softGMAlg
	var symCipher = chainsql.asymEncrypt("hello,world","pYvXDbsUUr5dpumrojYApjG8nLfFMXhu3aDvxq5oxEa4ZSeyjrMzisdPsYjfxyg9eN3ZJsNjtNENbzXPL89st39oiSp5yucU", "softGMAlg");
	console.log(symCipher);

	//ecies
	symCipher = chainsql.asymEncrypt("hello,world","cB4MLVsyn5MnoYHhApEyGtPCuEf9PAGDopmpB7yFwTbhUtzrjRRT");
	console.log(symCipher);

------------------------------------------------------------------------------


---------------
asymDecrypt
---------------
.. code-block:: javascript

	chainsql.asymDecrypt(encryptedHex, privateKey, algType)

| 使用私钥对密文进行解密。

参数说明
-----------

1. ``encryptedHex`` - ``String`` : 解密密文；
2. ``privateKey`` - ``String`` : 解密私钥；
3. ``algType`` - ``String`` : 加密算法类型："ecies"、"gmAlg"、"softGMAlg"，默认加密算法是"ecies"。

返回值
-----------

``String`` : 返回解密明文。

示例
-----------
.. code-block:: javascript

	//softGMAlg
	var symDecrypted = chainsql.asymDecrypt(symCipher,"pwRdHmA4cSUKKtFyo4m2vhiiz5g6ym58Noo9dTsUU97mARNjevj", "softGMAlg");
	console.log(symDecrypted.toString());

	//ecies
	symDecrypted = chainsql.asymDecrypt(symCipher,"xpvPjSRCtmQ3G99Pfu1VMDMd9ET3W");
	console.log(symDecrypted.toString());

