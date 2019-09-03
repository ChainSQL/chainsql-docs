智能合约
###########################

ChainSQL目前支持Solidity智能合约形式。

- 支持Solidity合约到版本0.4.25。
- 智能合约支持数据库表相关操作
- 智能合约支持代币发行


`Solidity合约开发 <https://solidity.readthedocs.io/en/v0.4.25/>`_
*************************************************************************************

- `Solidity官方文档 <https://solidity.readthedocs.io/en/v0.4.25>`_
- `Remix在线IDE <http://remix.chainsql.net/>`_


.. _SmartContract_DB_Oper:

数据库操作智能合约开发
****************************************************

一. 简介
====================

通过扩展Solidity指令，在智能合约中支持Chainsql对表的操作。

二. 表操作的Solidity指令
==============================================

具体指令见 :ref:`表操作指令 <Table_sol_instruction>`

三. 示例
==============================================

本文通过一个示例，旨在指引用户如何实现自己的数据库操作合约,并通过JAVA API以及Node.js API实现对数据库操作合约的调用。

合约示例
++++++++++++++++++++++++++++++++++++++++

提供一个合约示例 ``solidity-TableTxs.sol``，合约中包括对表的增删改查等操作的指令，代码如下:

.. code-block:: javascript

    pragma solidity ^0.4.4;

    contract DBTest {
        /*
        * @param tableName eg: "test1"
        * @param raw eg: "[{\"field\":\"id\", \"type\" : \"int\", \"length\" : 11, \"PK\" : 1, \"NN\" : 1, \"UQ\" : 1}, { \"field\":\"account\", \"type\" : \"varchar\" }, { \"field\":\"age\", \"type\" : \"int\" }]"
        */
        function create(string tableName, string raw) public{
            msg.sender.create(tableName, raw);
        }
        /*
        * @param tableName eg: "test1"
        */
        function drop(string tableName) public{
            msg.sender.drop(tableName);
        }
        
        /*
        * @param owner table's owner'
        * @param tableName eg: "test1"
        * @param raw eg: "[{\"account\":\"zU42yDW3fzFjGWosdeVjVasyPsF4YHj224\", \"id\":0}, {\"account\":\"zU42yDW3fzFjGWosdeVjVasyPsF4YHj224\",   \"id\":1}, {\"account\":\"zU42yDW3fzFjGWosdeVjVasyPsF4YHj224\", \"id\":2}]"
        */
        function insert(address owner, string tableName, string raw) public{
            owner.insert(tableName, raw);
        }
        
        /*
        * @param tableName eg: "test1"
        * @param raw eg: "[{\"account\":\"zU42yDW3fzFjGWosdeVjVasyPsF4YHj224\", \"id\":0}, {\"account\":\"zU42yDW3fzFjGWosdeVjVasyPsF4YHj224\",   \"id\":1}, {\"account\":\"zU42yDW3fzFjGWosdeVjVasyPsF4YHj224\", \"id\":2}]"
        */
        function insert(string tableName, string raw) public {
            msg.sender.insert(tableName, raw);
        }
        
        /*
        * @param owner table's owner'
        * @param tableName "test1"
        * @param raw eg: "{\"id\":1}"
        */
        function deletex(address owner, string tableName, string raw) public {
            owner.deletex(tableName, raw);
        }
        
        /*
        * @param tableName "test1"
        * @param raw eg: "{\"id\":1}"
        */
        function deletex(string tableName, string raw) public {
            msg.sender.deletex(tableName, raw);
        }
        
        /*
        * @param owner table's owner'
        * @param tableName eg: "test1"
        * @param rawUpdate eg: "{\"age\":15}"
        * @param rawGet eg: "{\"id\": 2}"
        */
        function update(address owner, string tableName, string rawUpdate, string rawGet) public{
            owner.update(tableName, rawUpdate, rawGet);
        }
        
        /*
        * @param tableName eg: "test1"
        * @param rawUpdate eg: "{\"age\":15}"
        * @param rawGet eg: "{\"id\": 2}"
        */
        function update(string tableName, string rawUpdate, string rawGet) public{
            msg.sender.update(tableName, rawUpdate, rawGet);
        }
        
        /*
        * @param tableName eg: "test1"
        * @param tableNameNew eg: "testNew1"
        */
        function rename(string tableName, string tableNameNew) public{
            msg.sender.rename(tableName, tableNameNew);
        }
        
        /*
        * @param toWho ethereum address to be granted. need convert chainsql addr 2 ethereum addr .eg:  "0xzzzzzzzzzzzzzzzzzzzzBZbvji"
        * @param tableName eg: "test1"
        * @param raw eg: "{\"insert\":false, \"delete\":false}"
        */
        function grant(address toWho, string tableName, string raw) public{
            return msg.sender.grant(toWho, tableName, raw);
        }
        
        function sqlTransaction(string tableName) public{
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
        * @param raw eg: ""
        */
        function get(address owner, string tableName, string raw) public view returns(string) {
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
        * @param owner table's owner'
        * @param tableName eg: "test1"
        * @param raw eg: ""
        * @param field eg: "id"
        */
        function get(address owner, string tableName, string raw, string field) public view returns(string) {
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
        
        function concat(string _base, string _value) internal pure returns (string) {
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
 - 使用工具 ``solc`` 编译 `下载地址 <https://github.com/ChainSQL/solidity/releases/tag/v0.10.1>`_


工具 ``solc`` 使用示例

.. code-block:: bash

    # 编译sol文件，生成abi以及bin文件
    ./solc --abi -o ./ --overwrite TableTxs.sol
    ./solc --bin -o ./ --overwrite TableTxs.sol

--------------

JAVA API 的调用
++++++++++++++++++++++++++++++++++++++++

- 详细的调用流程见  :ref:`Java API智能合约调用 <JavaAPI_SmartContract_call>`
- 示例代码见  `JAVA 数据库操作合约调用示例 <https://github.com/ChainSQL/java-chainsql-api/blob/feature/contract/chainsql/src/test/java/com/peersafe/example/contract/TestContractTableTxs.java>`_
       
----------------

Node.js API的调用
++++++++++++++++++++++++++++++++++++++++

- 详细的调用流程见  :ref:`Node.js API智能合约调用 <contract-newObj>`
- 示例代码见  `Node.js 数据库操作合约调用示例 <https://github.com/ChainSQL/node-chainsql-api/blob/feature/dev-escrow/test/testContractTableTxs.js>`_


----------------

代币接口智能合约开发
****************************************************

一. 简介
====================

通过扩展Solidity指令，支持在智能合约中进行代币发行相关操作。

二. 代币相关的Solidity指令
==============================================

具体指令见 :ref:`代币相关指令 <Gateway_sol_instruction>`

三. 示例
==============================================

本文通过一个示例，旨在指引用户如何实现自己的代币操作合约,并通过JAVA以及Node.js实现对代币操作智能合约的调用。

合约示例
++++++++++++++++++++++++++++++++++++++++

提供一个合约示例 ``solidity-GatewayTxs.sol`` ，合约中包括代币操作的相关指令，代码如下:

.. code-block:: javascript

    pragma solidity ^0.4.17;
    contract GatewayTxsTest { 

        constructor () public {
        }
        
        function() external payable {  }
        
            
        /*
        *  设置网关相关属性
        * @param uFlag   一般情况下为8，表示asfDefaultRipple，详见https://developers.ripple.com/accountset.html#accountset-flags
        * @param bSet    true，开启uFlag；false 取消uFlag。
        */
        function accountSet(uint32 uFlag,bool bSet) public {
            msg.sender.accountSet(uFlag,bSet);
        }	
        
        /*
        *  设置网关交易费用
        * @param sRate    交易费率。范围为"1.0”- "2.0" 或者"0.0"
        * @param minFee   网关交易最小花费  字符串转成10进制数后， >=0
        * @param maxFee   网关交易最大花费	字符串转成10进制数后,  >=0
        * @ 备注 ,以下规则均在字符串转化为10进制数后进行运算
        
            1 sRate 为0或者1时，表示取消费率，但是此时的minFee必须等于maxFee。
            2 minFee 或者 maxFee为0 时，表示取消相应的最小，最大费用。
            3 minFee等于maxFee时， sRate 必为0或者1。
            4 除了minFee 或者 maxFee为0 时的情况时，minFee < maxFee。
            
        */
        function setTransferFee(string sRate,string minFee,string maxFee) public {
            
            msg.sender.setTransferFee(sRate,minFee,maxFee);
        }



        /*
        *   设置信任网关代币以及代币的额度
        * @param value           代币额度
        * @param sCurrency       代币名称
        * @param gateWay         信任网关地址
        */
        function trustSet(string value,string sCurrency,address gateWay) public {

            msg.sender.trustSet(value,sCurrency,gateWay);
        }

        /*
        *   设置信任网关代币以及代币的额度
        * @param contractAddr    合约地址
        * @param value           代币额度
        * @param sCurrency       代币名称
        * @param gateWay         信任网关地址
        */
        function trustSet(address contractAddr,string value,string sCurrency, address gateWay) public {

            contractAddr.trustSet(value,sCurrency,gateWay);
        }
        
        /*
        *   查询网关的信任代币额度
        * @param  sCurrency          代币名称
        * @param  power              查询参数.代币额度为100时，如果该参数为2，函数返回值为10000 = 100*10^2；代币额度为100.5时,如果该参数为1,函数返回值为1005 = 100.5*10^1  				
        * @param  gateWay            网关地址
        * @return -1:不存在网关代币信任关系; >=0 信任网关代币额度
        */
        function trustLimit(string sCurrency,uint64 power,address gateWay)
        public view returns(int256) {

            int256  ret =  (int256)(msg.sender.trustLimit(sCurrency,power,gateWay));
            
            return ret;
        }


        /*
        *   查询网关的信任代币额度
        * @param  contractAddr       合约地址
        * @param  sCurrency          代币名称
        * @param  power              查询参数.代币额度为100时，如果该参数为2，函数返回值为10000 = 100*10^2；代币额度为100.5时,如果该参数为1,函数返回值为1005 = 100.5*10^1  			
        * @param  gateWay            网关地址
        * @return -1:不存在网关代币信任关系; >=0 信任网关代币额度
        */
        function trustLimit(address contractAddr,string sCurrency,uint64 power,address gateWay)
        public view returns(int256) {
            // 合约地址也可查询网关信任代币信息
            
            int256  ret =  (int256)(contractAddr.trustLimit(sCurrency,power,gateWay));
            
            return ret;
        }	
        
        /*
        *   获取网关代币的余额
        * @param  sCurrency       代币名称
        * @param  power           查询参数.代币余额为100时，如果该参数为2，函数返回值为10000 = 100*10^2；代币余额为100.5时,如果该参数为1,函数返回值为1005 = 100.5*10^1  		
        * @param  gateWay         网关地址
        * @return -1:不存在该网关代币; >=0 网关代币的余额
        */
        function gatewayBalance(string sCurrency,uint64 power,address gateWay)   public view returns(int256) {

            int256  ret = (int256)(msg.sender.gatewayBalance(sCurrency,power,gateWay));
            return ret;
        }


        /*
        *   获取网关代币的余额
        * @param  contractAddr    合约地址
        * @param  sCurrency       代币名称
        * @param  power           查询参数.代币余额为100时，如果该参数为2，函数返回值为10000 = 100*10^2；代币余额为100.5时,如果该参数为1,函数返回值为1005 = 100.5*10^1  	
        * @param  gateWay         网关地址
        * @return -1:不存在该网关代币; >=0 网关代币的余额
        */
        function gatewayBalance(address contractAddr,string sCurrency,uint64 power,address gateWay) public view returns(int256)  {
            // 合约地址也可获取网关代币的余额
            
            int256  ret = (int256)(contractAddr.gatewayBalance(sCurrency,power,gateWay));
            return ret;
        }	
        
        
        
    /*
    *   转账代币
    * @param accountTo         转入账户
    * @param value             代币数量
    * @param sendMax           消耗代币的最大值，具体计算规则见http://docs.chainsql.net/interface/javaAPI.html#id84
    * @param sCurrency         代币名称
    * @param sGateway          网关地址
    */
        function pay(address accountTo,string value,string sendMax,
                            string sCurrency,address gateWay) public{


            msg.sender.pay(accountTo,value,sendMax,sCurrency,gateWay);
        }

        /*
        *   转账代币
        * @param contractAddr      合约地址
        * @param accountTo         转入账户
        * @param value             代币数量
        * @param sendMax           消耗代币的最大值，具体计算规则见http://docs.chainsql.net/interface/javaAPI.html#id84	
        * @param sCurrency         代币名称
        * @param gateWay           网关地址
        */
        function gatewayPay(address contractAddr,address accountTo,string value,string sendMax,
                            string sCurrency,address gateWay) public{
        

        contractAddr.pay(accountTo,value,sendMax,sCurrency,gateWay);
        }		
        
    }

sol文件的编译
++++++++++++++++++++++++++++++++++++++++

:ref:`合约编译 <Sol_compile>`


JAVA API 的调用
++++++++++++++++++++++++++++++++++++++++

- 详细的调用流程见  :ref:`Java API智能合约调用 <JavaAPI_SmartContract_call>`
- 示例代码见  `JAVA 代币发行示例 <https://github.com/ChainSQL/java-chainsql-api/blob/feature/contract/chainsql/src/test/java/com/peersafe/example/contract/TestContractGatewayTxs.java>`_


Node.js API 的调用
++++++++++++++++++++++++++++++++++++++++

- 详细的调用流程见  :ref:`Node.js智能合约调用 <contract-newObj>`
- 示例代码见  `Node.js 代币发行示例 <https://github.com/ChainSQL/node-chainsql-api/tree/feature/dev-escrow/test/testContractGatewayTxs.js>`_


