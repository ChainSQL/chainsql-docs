智能合约
###########################

ChainSQL目前支持Solidity智能合约形式。

- 支持Solidity合约到版本0.8.5
- 智能合约支持数据库表相关操作
- 智能合约支持数字资产配置


`Solidity合约开发 <https://solidity.readthedocs.io/en/v0.8.5/>`_
*************************************************************************************

- `Solidity官方文档 <https://solidity.readthedocs.io/en/v0.8.5>`_
- `Remix在线IDE <http://remix.chainsql.net/>`_


----------------

预编译智能合约
****************************************************
除了以太坊支持的8个预编译合约指令，ChainSQL 还支持面向对象级别的预编译合约，目前支持的预编译合约有：

.. list-table::
    :align: left

    * - **合约名称**
      - **合约地址**
      - **描述**
    * - TablePrecompiled
      - 0x1001
      - 表操作预编译合约
    * - ToolsPrecompiled
      - 0x1002
      - 工具类预编译合约


----------------

.. _SmartContract_DB_Oper:

表操作预编译智能合约开发
****************************************************

一. 简介
====================

通过扩展Solidity指令，在智能合约中调用预编译合约支持Chainsql对表的操作。

二. 示例
==============================================

本文通过一个示例，旨在指引用户如何实现自己的数据库操作合约，并通过JAVA API以及Node.js API实现对数据库操作合约的调用。

预编译合约
++++++++++++++++++++++++++++++++++++++++

表相关功能的预编译合约solidity-PreCompiled.sol代码如下。

.. code-block:: javascript

    pragma solidity ^0.8.5;

    abstract contract TableOperation{
    function createTable(string memory tableName,string memory raw) public virtual;
	
    // 注：通过合约建表需要用户有对应的预留费用，默认配置下建一个表需要合约有1个ZXC（合约地址不需要账户基础预留费）
    function createByContract(string memory tableName,string memory raw) public virtual;
	
    function dropTable(string memory tableName) public virtual;
    
    function dropTableByContract(string memory tableName) public virtual;
    
    function grant(address destAddr,string memory tableName,string memory authRaw) public virtual;
    
    function grantByContract(address destAddr,string memory tableName,string memory authRaw) public virtual;
    
    function renameTable(string memory tableName,string memory tableNewName) public virtual;
    
    function renameTableByContract(string memory tableName,string memory tableNewName) public virtual;
    
    function insert(address owner, string memory tableName, string memory raw) public virtual;
    
    function insertWithHash(address owner, string memory tableName, string memory raw,string memory autoFillField) public virtual;
    
    function insertWithHashByContract(address owner, string memory tableName, string memory raw,string memory autoFillField) public virtual;
	
    function insertByContract(address owner, string memory tableName, string memory raw) public virtual;
	
    function update(address owner,string memory tableName,string memory raw,string memory updateRaw) public virtual;
	
    function updateByContract(address owner,string memory tableName,string memory raw,string memory updateRaw) public virtual;
    
    function update(address owner,string memory tableName,string memory raw) public virtual;

    function updateByContract(address owner,string memory tableName,string memory raw) public virtual;
    
    function deleteData(address owner,string memory tableName,string memory raw)public virtual;
	
    function deleteByContract(address owner,string memory tableName,string memory raw)public virtual;
	
    function addFields(string memory tableName,string memory raw)public virtual;
	
    function addFieldsByContract(string memory tableName,string memory raw)public virtual;
	
    function deleteFields(string memory tableName,string memory raw)public virtual;
	
    function deleteFieldsByContract(string memory tableName,string memory raw)public virtual;
	
    function modifyFields(string memory tableName,string memory raw)public virtual;
	
    function modifyFieldsByContract(string memory tableName,string memory raw)public virtual;
	
    function createIndex(string memory tableName,string memory raw)public virtual;
	
    function createIndexByContract(string memory tableName,string memory raw)public virtual;
	
    function deleteIndex(string memory tableName,string memory raw)public virtual;
	
    function deleteIndexByContract(string memory tableName,string memory raw)public virtual;
	
    function getDataHandle(address owner,string memory tableName,string memory raw)public view virtual returns(uint256);
	
    function getDataHandleByContract(address owner,string memory tableName,string memory raw)public view virtual returns(uint256);
    }



合约示例
++++++++++++++++++++++++++++++++++++++++

提供一个合约示例 ``solidity-PreCompiled-TableTxs.sol``，合约中包括通过调用预编译合约对表的增删改查以及在合约部署时创建表等操作的指令。合约中代码带有"ByContract"的方法是对合约地址表的操作。代码如下:

.. code-block:: javascript

    pragma solidity ^0.8.5;

    contract DBTest {
    //合约部署时创建属于合约地址的表
    TableOperation op_;
    constructor(string memory tableName, string memory raw) payable{
        // TableOperation对象的初始化
        op_ = TableOperation(address(0x1001));
        op_.createByContract(tableName,raw);
    }

    fallback () payable external {}
    receive () payable external {}
    
    /*
    * @param tableName eg: "test1"
    * @param raw eg: "[{\"field\":\"id\", \"type\" : \"int\", \"length\" : 11, \"PK\" : 1, \"NN\" : 1, \"UQ\" : 1}, { \"field\":\"account\", \"type\" : \"varchar\" }, { \"field\":\"age\", \"type\" : \"int\" }]"
    */
    
    function create(string memory tableName, string memory raw) public{
        op_.createTable(tableName,raw);
    }
    function createByContract(string memory tableName, string memory raw) public{
        op_.createByContract(tableName,raw);
    }
    /*
    * @param tableName eg: "test1"
    */
    /*
    * @param owner table's owner'
    * @param tableName eg: "test1"
    * @param raw eg: "[{\"account\":\"zU42yDW3fzFjGWosdeVjVasyPsF4YHj224\", \"id\":0}, {\"account\":\"zU42yDW3fzFjGWosdeVjVasyPsF4YHj224\",   \"id\":1}, {\"account\":\"zU42yDW3fzFjGWosdeVjVasyPsF4YHj224\", \"id\":2}]"
    */
    function insert(address owner, string memory tableName, string memory raw) public{
        //owner.insert(tableName, raw);
        op_.insert(owner,tableName,raw);
    }

    function insertByContract(address owner, string memory tableName, string memory raw) public{
        //owner.insert(tableName, raw);
        op_.insertByContract(owner,tableName,raw);
    }

    function insertHash(address owner, string memory tableName, string memory raw,string memory autoFillField) public {
        op_.insertWithHash(owner,tableName,raw,autoFillField);
    }

    function insertHashByContract(address owner, string memory tableName, string memory raw,string memory autoFillField) public {
        op_.insertWithHashByContract(owner,tableName,raw,autoFillField);
    }
    /*
    * @param tableName eg: "test1"
    */
    function drop(string memory tableName) public{
        op_.dropTable(tableName);
    }
  
    function dropByContract(string memory tableName) public{
        op_.dropTableByContract(tableName);
    }
    /*
    * @param owner table's owner'
    * @param tableName "test1"
    * @param raw eg: "{\"id\":1}"
    */
    function deletex(address owner, string memory tableName, string memory raw) public {
        op_.deleteData(owner, tableName, raw);
    }

    function deletexByContract(address owner, string memory tableName, string memory raw) public {
        op_.deleteByContract(owner, tableName, raw);
    }

    /*
    * @param owner table's owner'
    * @param tableName eg: "test1"
    * @param rawUpdate eg: "{\"age\":15}"
    * @param rawGet eg: "{\"id\": 2}"
    */
    function update(address owner, string memory tableName, string memory rawUpdate, string memory rawGet) public{
        op_.update(owner, tableName, rawUpdate, rawGet);
    }

    function updateByContract(address owner, string memory tableName, string memory rawUpdate, string memory rawGet) public{
        op_.updateByContract(owner, tableName, rawUpdate, rawGet);
    }

    /*
    * @param owner table's owner'
    * @param tableName eg: "test1"
    * @param raw eg: "[{\"age\":15},{\"id\": 2}]"
    */
    function update(address owner, string memory tableName, string memory raw) public{
        op_.update(owner, tableName, raw);
    }

    function updateByContract(address owner, string memory tableName, string memory raw) public{
        op_.updateByContract(owner, tableName, raw);
    }


    /*
    * @param tableName eg: "test1"
    * @param tableNameNew eg: "testNew1"
    */
    function rename(string memory tableName, string memory tableNameNew) public{
        op_.renameTable(tableName, tableNameNew);
    }
    
    function renameByContract(string memory tableName, string memory tableNameNew) public{
        op_.renameTableByContract(tableName, tableNameNew);
    }

    /*
    * @param toWho ethereum address to be granted. need convert chainsql addr 2 ethereum addr .eg:  "0xzzzzzzzzzzzzzzzzzzzzBZbvji"
    * @param tableName eg: "test1"
    * @param raw eg: "{\"insert\":false, \"delete\":false}"
    */
    function grant(address toWho, string memory tableName, string memory raw) public{
        return op_.grant(toWho, tableName, raw);
    }
    function grantByContract(address toWho, string memory tableName, string memory raw) public{
        return op_.grantByContract(toWho, tableName, raw);
    }

    /* @param tableName eg: "test1"
     * @param raw [{\"field\":\"num\",\"type\":\"int\"}]
     */
    function addFields(string memory tableName, string memory raw) public{
        return op_.addFields(tableName, raw);
    }

   
    function addFieldsByContract(string memory tableName, string memory raw) public{
        return op_.addFieldsByContract(tableName, raw);
    }

    /* @param tableName eg: "test1"
     * @param raw [{\"field\":\"num\"}]
     */
    function deleteFields(string memory tableName, string memory raw) public{
        return op_.deleteFields(tableName, raw);
    }

    function deleteFieldsByContract(string memory tableName, string memory raw) public{
        return op_.deleteFieldsByContract(tableName, raw);
    }
    
    /*@param tableName eg: "test1"
    * @param raw [{\"field\":\"age\",\"type\":\"varchar\",\"length\":10,\"NN\":1}]
    */

    function modifyFields(string memory tableName, string memory raw) public{
        return op_.modifyFields(tableName, raw);
    }

    function modifyFieldsByContract(string memory tableName, string memory raw) public{
        return op_.modifyFieldsByContract(tableName, raw);
    }

    
    /*@param tableName eg: "test1"
    * @param raw [{\"index\":\"AcctLgrIndex\"},{\"field\":\"age\"},{\"field\":\"Account\"}]
    */
    function createIndex(string memory tableName, string memory raw) public{
        return op_.createIndex(tableName, raw);
    }

    function createIndexByContract(string memory tableName, string memory raw) public{
        return op_.createIndexByContract(tableName, raw);
    }

    /*@param tableName eg: "test1"
    * @param raw [{\"index\":\"AcctLgrIndex\"}]
    */
    function deleteIndex(string memory tableName, string memory raw) public{
        return op_.deleteIndex(tableName, raw);
    }

    function deleteIndexByContract(string memory tableName, string memory raw) public{
        return op_.deleteIndexByContract(tableName, raw);
    }


    function sqlTransaction(string memory tableName) public{
        db.beginTrans();
        msg.sender.create(tableName, "[{\"field\":\"id\", \"type\" : \"int\", \"length\" : 11, \"PK\" : 1, \"NN\" : 1, \"UQ\" : 1}, { \"field\":\"account\", \"type\" : \"varchar\" }, { \"field\":\"age\", \"type\" : \"int\" }]");
        msg.sender.insert(tableName, "[{\"account\":\"zU42yDW3fzFjGWosdeVjVasyPsF4YHj224\", \"id\":1}, {\"account\":\"zU42yDW3fzFjGWosdeVjVasyPsF4YHj224\",   \"id\":2}]");
        msg.sender.deletex(tableName, "{\"id\":1}");
        msg.sender.update(tableName, "{\"account\":\"id==2\"}", "{\"id\": 2}");
        db.commit();
    }

    /*
    * @param owner table's owner'
    * @param tableName eg: "test1"
    * @param raw eg: "[[],{\"$or\":[{\"id\":\"1\"}, {\"id\": \"2\"}]}]"
    */
    
    function get(address owner, string memory tableName, string memory raw) public view returns(string memory) {
        uint256 handle = op_.getDataHandle(owner, tableName, raw);
        require(handle != uint256(0), "Get table data failed,maybe user not authorized!");
        uint row = db.getRowSize(handle);
        uint col = db.getColSize(handle);
        bytes memory xxx = "";
        for(uint i=0; i<row; i++)
        {
            for(uint j=0; j<col; j++)
            {
                string memory y = (db.getValueByIndex(handle, i, j));
                xxx = bytes.concat(xxx,bytes(y));
                if(j != col - 1)
                    xxx = bytes.concat(xxx,", ");
            }
            xxx = bytes.concat(xxx,";\n");
        }
        return string(xxx);
    }
    
 
    function getByContract(address owner, string memory tableName, string memory raw)  public view returns(string memory) {
        uint256 handle = op_.getDataHandleByContract(owner, tableName, raw);
        require(handle != uint256(0), "Get table data failed,maybe user not authorized!");
        uint row = db.getRowSize(handle);
        uint col = db.getColSize(handle);
        bytes memory xxx = "";
        for(uint i=0; i<row; i++)
        {
            for(uint j=0; j<col; j++)
            {
                string memory y = (db.getValueByIndex(handle, i, j));
                xxx = bytes.concat(xxx,bytes(y));
                if(j != col - 1)
                    xxx = bytes.concat(xxx,", ");
            }
            xxx = bytes.concat(xxx,";\n");
        }
        return string(xxx);
    }
    /*
    * @param owner table's owner'
    * @param tableName eg: "test1"
    * @param raw eg: ""
    * @param field eg: "id"
    */

    function get(address owner, string memory tableName, string memory raw, string memory field) public view returns(string memory) {
        uint256 handle = op_.getDataHandle(owner, tableName, raw);
        require(handle != uint256(0), "Get table data failed,maybe user not authorized!");
        uint row = db.getRowSize(handle);
        bytes memory xxx = "";
        for(uint i=0; i<row; i++)
        {
            string memory y = (db.getValueByKey(handle, i, field));
            xxx = bytes.concat(xxx, bytes(y));
            xxx = bytes.concat(xxx, ";");
        }
        return string(xxx);
    }

    function getByContract(address owner, string memory tableName, string memory raw, string memory field) public view returns(string memory) {
        uint256 handle = op_.getDataHandleByContract(owner, tableName, raw);
        require(handle != uint256(0), "Get table data failed,maybe user not authorized!");
        uint row = db.getRowSize(handle);
        bytes memory xxx = "";
        for(uint i=0; i<row; i++)
        {
            string memory y = (db.getValueByKey(handle, i, field));
            xxx = bytes.concat(xxx, bytes(y));
            xxx = bytes.concat(xxx, ";");
        }
        return string(xxx);
    }}


.. note::

  * 调用前需要对TableOperation对象进行初始化。
  * 如果在部署合约时，创建属于合约地址的表，则需要添加"payable"关键字（部署合约并给合约地址转ZXC）。
  * 表查询方法不能用在合约交易中，只能供客户端查询使用。


合约文件的编译
++++++++++++++++++++++++++++++++++++++++

:ref:`合约编译 <Sol_compile>`

--------------

JAVA API 的调用
++++++++++++++++++++++++++++++++++++++++

- 详细的调用流程见  :ref:`Java API智能合约调用 <JavaAPI_SmartContract_call>`
- 示例代码见  `JAVA 合约中调用新增预编译合约接口示例 <https://github.com/ChainSQL/java-chainsql-api/blob/master/chainsql/src/test/java/com/peersafe/example/contract/TestPreCompiledContractTableTxs.java>`_
       
----------------

Node.js API的调用
++++++++++++++++++++++++++++++++++++++++

- 详细的调用流程见  :ref:`Node.js API智能合约调用 <contract-newObj>`
- 示例代码见  `Node.js 合约中调用新增预编译合约接口示例 <https://github.com/ChainSQL/node-chainsql-api/blob/master/test/testPreCompiledContractTableTxs.js>`_


----------------

数字资产接口智能合约开发
****************************************************

一. 简介
====================

通过扩展Solidity指令，支持在智能合约中进行数字资产配置相关操作。

- 对于普通账户，通过数字资产合约接口，只有自己作为交易发起方可以发起数字资产转账接口，并且只能转出自己持有的数字资产
- 对于合约账户，必须是合约在合约内可以发起数字资产转账，而不能是一个合约内通过编写函数调用另一个合约地址转账数字资产

二. 数字资产相关的Solidity指令
==============================================

- 说明：
    - 添加了合约中对网关设置，信任，转账网关数字资产，查询信任额度，查询网关数字资产余额功能的支持
    - 函数中涉及到给合约地址转账网关数字资产的操作，需要添加payable修饰符。
    - solidity本身没有提供获取合约地址的指令，需要通过接口传入。
    - 无信任关系时，查询信任额度，查询网关数字资产余额返回-1
    - 为支持查询浮点类型的值，trustLimit和gatewayBalance指令返回的是查询值。查询值和实际值的换算公式为:   查询值  = 实际值 * 10 ^(power) , power 为查询参数。详见相关函数注释。

网关的accoutSet属性设置
+++++++++++++++++++++++++++++++++++++

.. code-block:: javascript 

    /*
    *  设置网关相关属性
    * @param uFlag   一般情况下为8，表示asfDefaultRipple
    * @param bSet    true，开启uFlag；false 取消uFlag。
    */
    function accountSet(uint32 uFlag,bool bSet) public {
        msg.sender.accountSet(uFlag,bSet);
    }

设置网关交易费用
+++++++++++++++++++++++++++++++++++++

.. code-block:: javascript 

    /*
    *  设置网关交易费用
    * @param sRate    交易费率。范围为"1.0”- "2.0" 或者"0.0"
    * @param minFee   网关交易最小花费  字符串转成10进制数后， >=0
    * @param maxFee   网关交易最大花费	字符串转成10进制数后,  >=0
    * @ 
    *
    *    备注 ,以下规则均在字符串转化为10进制数后进行
    *
    *	 1 sRate 为0或者1时，表示取消费率，但是此时的minFee必须等于maxFee。
    *	 2 minFee 或者 maxFee为0 时，表示取消相应的最小，最大费用。
    *	 3 minFee等于maxFee时， sRate 必为0或者1。
    *	 4 除了minFee 或者 maxFee为0 时的情况时，minFee < maxFee。
    *	   
    */
    function setTransferFee(string sRate,string minFee,string maxFee) public {
        
        msg.sender.setTransferFee(sRate,minFee,maxFee);
    }





设置信任网关数字资产以及数字资产的额度
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

.. code-block:: javascript

    /*
    *   设置信任网关数字资产以及数字资产的额度
    * @param value           数字资产额度
    * @param sCurrency       数字资产名称
    * @param gateway         信任网关地址
    */
    function trustSet(string value,string sCurrency,address gateway) public {

        msg.sender.trustSet(value,sCurrency,gateway);
    }

    /*
    *   设置信任网关数字资产以及数字资产的额度
    * @param contractAddr    合约地址
    * @param value           数字资产额度
    * @param sCurrency       数字资产名称
    * @param gateway         信任网关地址
    */
    function trustSet(address contractAddr,string value,string sCurrency, address gateway) public {

        // 合约地址也可信任网关
        contractAddr.trustSet(value,sCurrency,gateway);
    }

查询网关的信任数字资产信息
+++++++++++++++++++++++++++++++++++++++

.. code-block:: javascript

    /*
    *   查询网关的信任数字资产额度.
    * @param  sCurrency          数字资产名称
    * @param  power              查询参数.数字资产额度为100时，如果该参数为2，函数返回值为10000 = 100*10^2；数字资产额度为100.5时,如果该参数为1，函数返回值为1005 = 100.5*10^1  
    * @param  gateway            网关地址
    * @return -1:不存在网关数字资产信任关系; >=0 信任网关数字资产查询额度
    */
    function trustLimit(string sCurrency,uint64 power,address gateway)
    public view returns(int256) {

        return msg.sender.trustLimit(sCurrency,power,gateway);
    }


    /*
    *   查询网关的信任数字资产信息.目前版本数字资产余额返回仅支持整数类型，下一版本会支持浮点类型。
    * @param  contractAddr       合约地址
    * @param  sCurrency          数字资产名称
    * @param  power              查询参数.数字资产额度为100时，如果该参数为2，函数返回值为10000 = 100*10^2；数字资产额度为100.5时,如果该参数为1，函数返回值为1005 = 100.5*10^1  
    * @param  gateWay            网关地址
    * @return -1:不存在网关数字资产信任关系; >=0 信任网关数字资产查询额度
    */
    function trustLimit(address contractAddr,string sCurrency,uint64 power,address gateway)
    public view returns(int256) {
        // 合约地址也可查询网关信任数字资产信息
        return contractAddr.trustLimit(sCurrency,power,gateway);

    }

查询网关数字资产余额
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

.. code-block:: javascript

    /*
    *   获取网关数字资产的余额
    * @param  sCurrency       数字资产名称
    * @param  power           查询参数.数字资产余额为100时，如果该参数为2，函数返回值为10000 = 100*10^2；数字资产余额为100.5时,如果该参数为1，函数返回值为1005 = 100.5*10^1
    * @param  gateway         网关地址
    * @return -1:不存在该网关数字资产; >=0 网关数字资产的查询余额
    */
    function gatewayBalance(string sCurrency,uint64 power,address gateway) public view returns(int256)  {

        return msg.sender.gatewayBalance(sCurrency,power,gateway);
    }


    /*
    *   获取网关数字资产的余额
    * @param  contractAddr    合约地址
    * @param  sCurrency       数字资产名称
    * @param  power           查询精度.例如实际数字资产余额为100时，如果该参数为2，函数返回值为10000 = 100*10^2；实际数字资产余额为100时，如果该参数为2，函数返回值为10000 = 100*10^2
    * @param  gateway         网关地址
    * @return -1:不存在该网关数字资产; >=0 网关数字资产的查询余额
    */
    function gatewayBalance(address contractAddr,string sCurrency,uint64 power,address gateway) public view  returns(int256) {
        // 合约地址也可获取网关数字资产的余额
        return contractAddr.gatewayBalance(sCurrency,power,gateway);
    }


数字资产转账接口
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

.. code-block:: javascript

    /*
    *   转账数字资产
    * @param accountTo         转入账户
    * @param value             数字资产数量
    * @param sendMax           消耗数字资产的最大值，具体计算规则见http://docs.chainsql.net/interface/javaAPI.html#id84    
    * @param sCurrency         数字资产名称
    * @param gateway           网关地址
    */
    function pay(address accountTo,string value,string sendMax,
                        string sCurrency,address gateway) public {
    
        msg.sender.pay(accountTo,value,sendMax,sCurrency,gateway);
    }

    /*
    *   转账数字资产
    * @param contractAddr      合约地址
    * @param accountTo         转入账户
    * @param value             数字资产数量
    * @param sendMax           消耗数字资产的最大值，具体计算规则见http://docs.chainsql.net/interface/javaAPI.html#id84        
    * @param sCurrency         数字资产名称
    * @param gateway           网关地址
    */
    function pay(address contractAddr,address accountTo,string value,string sendMax,string sCurrency,address gateway) public {
    
        // 合约地址也可转账数字资产
        contractAddr.pay(accountTo,value,sendMax,sCurrency,gateway);
    }	

三. 示例
==============================================

本文通过一个示例，旨在指引用户如何实现自己的数字资产操作合约，并通过JAVA以及Node.js实现对数字资产操作智能合约的调用。

合约示例
++++++++++++++++++++++++++++++++++++++++

提供一个合约示例 ``solidity-GatewayTxs.sol`` ，合约中包括数字资产操作的相关指令，代码如下:

.. code-block:: javascript

    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;

    contract GatewayTxsTest { 

        constructor () {
        }
    
    	fallback() external payable {  }
        receive() external payable {  }
    
        /*
        *  设置网关相关属性
        * @param uFlag   一般情况下为8，表示asfDefaultRipple，详见https://developers.ripple.com/accountset.html#accountset-flags
        * @param bSet    true，开启uFlag；false 取消uFlag。
        */
        function accountSet(uint32 uFlag,bool bSet) public {
            msg.sender.accountSet(uFlag,bSet);
        }	
    
        /*
        *  设置网关数字资产分发费用
        * @param sRate    网关数字资产分发费率。范围为"1.0”- "2.0" 或者"0.0"
        * @param minFee   网关网关数字资产分发最小花费  字符串转成10进制数后， >=0
        * @param maxFee   网关网关数字资产分发最大花费	字符串转成10进制数后，  >=0
    	* @ 备注 ,以下规则均在字符串转化为10进制数后进行运算
    
    		 1 sRate 为0或者1时，表示取消费率，但是此时的minFee必须等于maxFee。
    		 2 minFee 或者 maxFee为0 时，表示取消相应的最小，最大费用。
    		 3 minFee等于maxFee时， sRate 必为0或者1。
    		 4 除了minFee 或者 maxFee为0 时的情况时，minFee < maxFee。
    
        */
        function setTransferFee(string calldata sRate,string calldata minFee,string calldata maxFee) public {
            msg.sender.setTransferFee(sRate,minFee,maxFee);
        }

        /*
        *   设置信任网关数字资产以及数字资产的额度
        * @param value           数字资产额度
        * @param sCurrency       数字资产名称
        * @param gateWay         信任网关地址
        */
        function trustSet(string calldata value,string calldata sCurrency,address gateWay) public {
            msg.sender.trustSet(value,sCurrency,gateWay);
        }

        /*
        *   设置信任网关数字资产以及数字资产的额度
        * @param contractAddr    合约地址
        * @param value           数字资产额度
        * @param sCurrency       数字资产名称
        * @param gateWay         信任网关地址
        */
        function trustSet(address contractAddr,string calldata value,string calldata sCurrency, address gateWay) public {
            contractAddr.trustSet(value,sCurrency,gateWay);
        }
    
        /*
        *   查询网关的信任数字资产额度
        * @param  sCurrency          数字资产名称
    	* @param  power              查询参数.数字资产额度为100时，如果该参数为2，函数返回值为10000 = 100*10^2；数字资产额度为100.5时,如果该参数为1,函数返回值为1005 = 100.5*10^1  				
        * @param  gateWay            网关地址
        * @return -1:不存在网关数字资产信任关系; >=0 信任网关数字资产额度
        */
        function trustLimit(string calldata sCurrency,uint64 power,address gateWay)
        public view returns(int256) {
            int256  ret =  (int256)(msg.sender.trustLimit(sCurrency,power,gateWay));
    		return ret;
        }

        /*
        *   查询网关的信任数字资产额度
        * @param  contractAddr       合约地址
        * @param  sCurrency          数字资产名称
    	* @param  power              查询参数.数字资产额度为100时，如果该参数为2，函数返回值为10000 = 100*10^2；数字资产额度为100.5时,如果该参数为1,函数返回值为1005 = 100.5*10^1  			
        * @param  gateWay            网关地址
        * @return -1:不存在网关数字资产信任关系; >=0 信任网关数字资产额度
        */
        function trustLimit(address contractAddr,string calldata sCurrency,uint64 power,address gateWay)
        public view returns(int256) {
            // 合约地址也可查询网关信任数字资产信息
            int256  ret =  (int256)(contractAddr.trustLimit(sCurrency,power,gateWay));
    		return ret;
        }	
    
        /*
        *   获取网关数字资产的余额
        * @param  sCurrency       数字资产名称
    	* @param  power           查询参数.数字资产余额为100时，如果该参数为2，函数返回值为10000 = 100*10^2；数字资产余额为100.5时,如果该参数为1,函数返回值为1005 = 100.5*10^1  		
        * @param  gateWay         网关地址
        * @return -1:不存在该网关数字资产; >=0 网关数字资产的余额
        */
        function gatewayBalance(string calldata sCurrency,uint64 power,address gateWay)   public view returns(int256) {
            int256  ret = (int256)(msg.sender.gatewayBalance(sCurrency,power,gateWay));
    		return ret;
        }

        /*
        *   获取网关数字资产的余额
        * @param  contractAddr    合约地址
        * @param  sCurrency       数字资产名称
    	* @param  power           查询参数.数字资产余额为100时，如果该参数为2，函数返回值为10000 = 100*10^2；数字资产余额为100.5时,如果该参数为1,函数返回值为1005 = 100.5*10^1  	
        * @param  gateWay         网关地址
        * @return -1:不存在该网关数字资产; >=0 网关数字资产的余额
        */
        function gatewayBalance(address contractAddr,string calldata sCurrency,uint64 power,address gateWay) public view returns(int256)  {
            // 合约地址也可获取网关数字资产的余额
            int256  ret = (int256)(contractAddr.gatewayBalance(sCurrency,power,gateWay));
    		return ret;
        }	
    
      /*
      *   转账数字资产
      * @param accountTo         转入账户
      * @param value             数字资产数量
      * @param sendMax           消耗数字资产的最大值，具体计算规则见http://docs.chainsql.net/interface/javaAPI.html#id84
      * @param sCurrency         数字资产名称
      * @param sGateway          网关地址
      */
        function pay(address accountTo,string calldata value,string calldata sendMax,
                            string calldata sCurrency,address gateWay) public{
            msg.sender.pay(accountTo,value,sendMax,sCurrency,gateWay);
        }

        /*
        *   转账数字资产
        * @param contractAddr      合约地址
        * @param accountTo         转入账户
        * @param value             数字资产数量
        * @param sendMax           消耗数字资产的最大值，具体计算规则见http://docs.chainsql.net/interface/javaAPI.html#id84	
        * @param sCurrency         数字资产名称
        * @param gateWay           网关地址
        */
        function gatewayPay(address contractAddr,address accountTo,string calldata value,string calldata sendMax,
                            string calldata sCurrency,address gateWay) public{
    	   contractAddr.pay(accountTo,value,sendMax,sCurrency,gateWay);
        }		
    }

sol文件的编译
++++++++++++++++++++++++++++++++++++++++

:ref:`合约编译 <Sol_compile>`


JAVA API 的调用
++++++++++++++++++++++++++++++++++++++++

- 详细的调用流程见  :ref:`Java API智能合约调用 <JavaAPI_SmartContract_call>`
- 示例代码见  `JAVA 数字资产配置示例 <https://github.com/ChainSQL/java-chainsql-api/blob/master/chainsql/src/test/java/com/peersafe/example/contract/TestContractGatewayTxs.java>`_


Node.js API 的调用
++++++++++++++++++++++++++++++++++++++++

- 详细的调用流程见  :ref:`Node.js智能合约调用 <contract-newObj>`
- 示例代码见  `Node.js 数字资产配置示例 <https://github.com/ChainSQL/node-chainsql-api/tree/master/test/testContractGatewayTxs.js>`_


------------------------


数据库操作智能合约开发 **(已废弃)**
****************************************************

一. 简介
====================

通过扩展Solidity指令，在智能合约中支持Chainsql对表的操作。

二. 表操作的Solidity指令
==============================================

.. note::
    | ``owner`` 为address类型，表的拥有者地址。
    | ``raw`` 为字符串类型，非16进制，JSON格式。

创建表
+++++++++++++++++++++++++++++++++++++

.. code-block:: javascript

    owner.create("table_name", "create raw string");

    // example
    function createTable(string name, string raw) public {
        msg.sender.create(name, raw);
    }

插入
+++++++++++++++++++++++++++++++++++++

.. code-block:: javascript

    owner.insert("table_name", "insert raw string");

    // example
    function insertToTable(address owner, string name, string raw) public {
        owner.insert(name, raw);
    }

删除行
+++++++++++++++++++++++++++++++++++++

.. code-block:: javascript

    // delete参数代表删除条件
    owner.delete("table_name", "raw string");

    // example
    function deleteFromTable(address owner, string name, string raw) public {
        owner.delete(name, raw);
    }

修改
+++++++++++++++++++++++++++++++++++++

.. code-block:: javascript

    // update需要两个参数
    owner.update(table_name, "raw string", "get raw");

    // example
    function updateTable(address owner, string name, string getRaw, string updateRaw) public {
        owner.update(name, updateRaw, getRaw);
    }

查询
+++++++++++++++++++++++++++++++++++++

 * 查询返回一个句柄，需要自定义一个类型，如handle（或者直接使用uint256）。
 * handle不可作为函数返回值返回（只能作为临时对象使用），也不能作为成员变量使用（作为成员变量使用，跨交易时，会获取不到内容）。
 * 可根据查询得到的句柄去获取查询结果中的字段值。
 * 提供遍历方法，可根据句柄遍历查询结果。

.. code-block:: javascript

    uint256 handle = owner.get(tableName, raw);
    uint row = db.getRowSize(handle);
    uint col = db.getColSize(handle);
    string memory xxx;
    for (uint i = 0; i < row; i++)
    {
        for (uint j = 0; j < col; j++)
        {
            string memory y1 = (db.getValueByIndex(handle, i, j));
            string memory y2 = (db.getValueByKey(handle, i, field));
        }
    }

事务相关
+++++++++++++++++++++++++++++++++++++

 * 增加两个指令beginTrans()、commit()，指令之间的部分组成事务。
 * 两个指令之间的操作逐行执行。

.. code-block:: javascript

    db.beginTrans();
    owner.insert(name.raw);
    uint256 handle = owner.get(name, getRaw);
    if (db.getRowSize(handle) > 0) {
        owner.update(name, updateRaw, getRaw);
    }

    ...
    // every op is alone

    db.commit();

授权
+++++++++++++++++++++++++++++++++++++

 * 必须由表的拥有者发起。

.. code-block:: javascript

    owner.grant(user_address, table_name, "grant_raw");

    // example
    function grantTable(string name, address user, string raw) public {
        msg.sender.grant(user, name, raw);
    }

删除表
+++++++++++++++++++++++++++++++++++++

 * 必须由表的拥有者发起。

.. code-block:: javascript

    owner.drop("table_name");

    // example
    function dropTable(string name) public {
        msg.sender.drop(name);
    }

重命名表
+++++++++++++++++++++++++++++++++++++

 * 必须由表的拥有者发起

.. code-block:: javascript

    owner.rename("table_name", "new_name");

    //example
    function renameTable(string name,string newName) public {
        msg.sender.rename(name, newName);
    }

三. 示例
==============================================

本文通过一个示例，旨在指引用户如何实现自己的数据库操作合约，并通过JAVA API以及Node.js API实现对数据库操作合约的调用。

合约示例
++++++++++++++++++++++++++++++++++++++++

提供一个合约示例 ``solidity-TableTxs.sol``，合约中包括对表的增删改查等操作的指令，代码如下:

.. code-block:: javascript

    // SPDX-License-Identifier: MIT
    pragma solidity ^0.8.0;

    contract DBTest {
        /*
        * @param tableName eg: "test1"
        * @param raw eg: "[{\"field\":\"id\", \"type\" : \"int\", \"length\" : 11, \"PK\" : 1, \"NN\" : 1, \"UQ\" : 1}, { \"field\":\"account\", \"type\" : \"varchar\" }, { \"field\":\"age\", \"type\" : \"int\" }]"
        */
    	function create(string calldata tableName, string calldata raw) public{
    		msg.sender.create(tableName, raw);
    	}
    	/*
    	* @param tableName eg: "test1"
    	*/
    	function drop(string calldata tableName) public{
    	    msg.sender.drop(tableName);
    	}
    
    	/*
    	* @param owner table's owner
    	* @param tableName eg: "test1"
    	* @param raw eg: "[{\"account\":\"zU42yDW3fzFjGWosdeVjVasyPsF4YHj224\", \"id\":0}, {\"account\":\"zU42yDW3fzFjGWosdeVjVasyPsF4YHj224\",   \"id\":1}, {\"account\":\"zU42yDW3fzFjGWosdeVjVasyPsF4YHj224\", \"id\":2}]"
    	*/
    	function insert(address owner, string calldata tableName, string calldata raw) public{
    	    owner.insert(tableName, raw);
    	}
    
    	/*
    	* @param tableName eg: "test1"
    	* @param raw eg: "[{\"account\":\"zU42yDW3fzFjGWosdeVjVasyPsF4YHj224\", \"id\":0}, {\"account\":\"zU42yDW3fzFjGWosdeVjVasyPsF4YHj224\",   \"id\":1}, {\"account\":\"zU42yDW3fzFjGWosdeVjVasyPsF4YHj224\", \"id\":2}]"
    	*/
    	function insert(string calldata tableName, string calldata raw) public {
    	    msg.sender.insert(tableName, raw);
    	}
    
    	/*
    	* @param owner table's owner
    	* @param tableName "test1"
    	* @param raw eg: "{\"id\":1}"
    	*/
    	function deletex(address owner, string calldata tableName, string calldata raw) public {
    	    owner.deletex(tableName, raw);
    	}
    
    	/*
    	* @param tableName "test1"
    	* @param raw eg: "{\"id\":1}"
    	*/
    	function deletex(string calldata tableName, string calldata raw) public {
    	    msg.sender.deletex(tableName, raw);
    	}
    
    	/*
    	* @param owner table's owner
    	* @param tableName eg: "test1"
    	* @param rawUpdate eg: "{\"age\":15}"
    	* @param rawGet eg: "{\"id\": 2}"
    	*/
    	function update(address owner, string calldata tableName, string calldata rawUpdate, string calldata rawGet) public{
    	    owner.update(tableName, rawUpdate, rawGet);
    	}
    
    	/*
    	* @param tableName eg: "test1"
    	* @param rawUpdate eg: "{\"age\":15}"
    	* @param rawGet eg: "{\"id\": 2}"
    	*/
    	function update(string calldata tableName, string calldata rawUpdate, string calldata rawGet) public{
    	    msg.sender.update(tableName, rawUpdate, rawGet);
    	}
    
    	/*
    	* @param tableName eg: "test1"
    	* @param tableNameNew eg: "testNew1"
    	*/
    	function rename(string calldata tableName, string calldata tableNameNew) public{
    	    msg.sender.rename(tableName, tableNameNew);
    	}
    
    	/*
    	* @param toWho ethereum address to be granted. need convert chainsql addr 2 ethereum addr .eg:  "0xzzzzzzzzzzzzzzzzzzzzBZbvji"
    	* @param tableName eg: "test1"
    	* @param raw eg: "{\"insert\":false, \"delete\":false}"
    	*/
    	function grant(address toWho, string calldata tableName, string calldata raw) public{
    	    return msg.sender.grant(toWho, tableName, raw);
    	}
    
    	function sqlTransaction(string calldata tableName) public{
    	    db.beginTrans();
    	    msg.sender.create(tableName, "[{\"field\":\"id\", \"type\" : \"int\", \"length\" : 11, \"PK\" : 1, \"NN\" : 1, \"UQ\" : 1}, { \"field\":\"account\", \"type\" : \"varchar\" }, { \"field\":\"age\", \"type\" : \"int\" }]");
            msg.sender.insert(tableName, "[{\"account\":\"zU42yDW3fzFjGWosdeVjVasyPsF4YHj224\", \"id\":1}, {\"account\":\"zU42yDW3fzFjGWosdeVjVasyPsF4YHj224\",   \"id\":2}]");
            msg.sender.deletex(tableName, "{\"id\":1}");
            msg.sender.update(tableName, "{\"account\":\"id==2\"}", "{\"id\": 2}");
            db.commit();
    	}

        /*
    	* @param owner table's owner
    	* @param tableName eg: "test1"
    	* @param raw eg: ""
        */
        function get(address owner, string calldata tableName, string calldata raw) public view 
        returns(string memory) {
            uint256 handle = owner.get(tableName, raw);
    		require(handle != uint256(0), "Get table data failed,maybe user not authorized!");
            uint row = db.getRowSize(handle);
            uint col = db.getColSize(handle);
            string memory xxx;
            for(uint i=0; i<row; i++)
            {
                for(uint j=0; j<col; j++)
                {
                    string memory y = (db.getValueByIndex(handle, i, j));
                    xxx = concat(xxx, y);
    				if(j != col - 1)
                    	xxx = concat(xxx, ", ");
                }
                xxx = concat(xxx, ";\n");
            }
            return xxx;
        }
            /*
    	* @param owner table's owner
    	* @param tableName eg: "test1"
    	* @param raw eg: ""
    	* @param field eg: "id"
        */
        function get(address owner, string calldata tableName, string calldata raw, string calldata field) 
        public view returns(string memory) {
            uint256 handle = owner.get(tableName, raw);
    		require(handle != uint256(0), "Get table data failed,maybe user not authorized!");
            uint row = db.getRowSize(handle);
            string memory xxx;
            for(uint i=0; i<row; i++)
            {
                string memory y = (db.getValueByKey(handle, i, field));
                xxx = concat(xxx, y);
                xxx = concat(xxx, ";");
            }
            return xxx;
        }

        function concat(string memory _base, string memory _value) internal pure 
        returns (string memory) {
            bytes memory _baseBytes = bytes(_base);
            bytes memory _valueBytes = bytes(_value);

            string memory _tmpValue = new string(_baseBytes.length + _valueBytes.length);
            bytes memory _newValue = bytes(_tmpValue);

            uint j = 0;
            for(uint i=0; i<_baseBytes.length; i++) {
                _newValue[j++] = _baseBytes[i];
            }

            for(uint i=0; i<_valueBytes.length; i++) {
                _newValue[j++] = _valueBytes[i];
            }

            return string(_newValue);
        }
    }

.. _Sol_compile:

合约文件的编译
++++++++++++++++++++++++++++++++++++++++

通过以下2种方式编译合约sol文件，生成abi以及bin文件。

 - 使用Remix在线IDE编译 `Remix在线IDE <http://remix.chainsql.net/>`_  `使用手册 <https://remix-ide.readthedocs.io/en/stable/>`_
 - 使用工具 ``solc`` 编译 `下载地址 <https://github.com/ChainSQL/solidity/releases/tag/v0.8.5>`_


工具 ``solc`` 使用示例

.. code-block:: bash

    # 编译sol文件，生成abi以及bin文件
    ./solc --abi -o ./ --overwrite TableTxs.sol
    ./solc --bin -o ./ --overwrite TableTxs.sol

--------------

JAVA API 的调用
++++++++++++++++++++++++++++++++++++++++

- 详细的调用流程见  :ref:`Java API智能合约调用 <JavaAPI_SmartContract_call>`
- 示例代码见  `JAVA 数据库操作合约调用示例 <https://github.com/ChainSQL/java-chainsql-api/blob/master/chainsql/src/test/java/com/peersafe/example/contract/TestContractTableTxs.java>`_
       
----------------

Node.js API的调用
++++++++++++++++++++++++++++++++++++++++

- 详细的调用流程见  :ref:`Node.js API智能合约调用 <contract-newObj>`
- 示例代码见  `Node.js 数据库操作合约调用示例 <https://github.com/ChainSQL/node-chainsql-api/blob/master/test/testContractTableTxs.js>`_
