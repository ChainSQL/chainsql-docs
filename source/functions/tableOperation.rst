数据库表操作
###########################

1 简介
*************************

ChainSQL 目前对数据库表操作的支持包括以下几个部分:

- 数据库表的创建

- 数据库表的插入

- 数据库表的更新

- 重命名数据库表

- 数据库表的删除

- 数据库表权限的授权

- 数据库表数据的查询

- 加密表的支持

- 智能合约对表操作的支持


------------

2 具体功能
*************************

本节通过一个示例，旨在指引用户如何实现自己对的数据库的表操作,并通过RPC,JAVA API以及Node.js API实现对数据库表的操作

数据库表的表名为 ``table_test`` ,表的结构如下:

.. list-table::

    * - **字段**
      - **类型**
      - **长度**
      - **是否为主键**
      - **描述**
    * - id
      - int
      - 11
      - 是
      - 用户唯一的id
    * - name
      - varchar
      - 50
      - 否
      - 用户的名字      
    * - age
      - int
      - 11
      - 否
      - 用户的年龄 

----------

2.1 数据库表的创建
=============================================

创建数据库表 ``table_test``

- RPC代码

.. code-block:: json

  {
      "method": "t_create",
      "params": [{
          "secret": "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb",
          "tx_json": {
              "TransactionType": "TableListSet",
              "Account": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
              "Tables": [
                  {
                      "Table": { "TableName": "table_test" }
                  }
              ],
              "OpType": 1,
              "Raw": [
                  {
                      "field": "id",
                      "type": "int",
                      "length": 11,
                      "PK": 1,
                      "NN": 1,
                      "UQ": 1
                  },
                  {
                      "field": "name",
                      "type": "varchar",
                      "length": 50
                  },
                  {
                      "field": "age",
                      "type": "int",
                      "length": 11
                  }
              ],
              "Confidential": false
          }
      }]
  }

-----------

- java代码

.. code-block:: java
  
    public void testCreateTable() {
      List<String> args = Util.array("{'field':'id','type':'int','length':11,'PK':1,'NN':1,'UQ':1}",
          "{'field':'name','type':'varchar','length':50,'default':null}", "{'field':'age','type':'int','length':11}");

      JSONObject obj;
      obj = c.createTable("table_test",args,false).submit(SyncCond.db_success);
      System.out.println("create result:" + obj);
    }

-------------------

- Node.js代码

.. code-block:: javascript

    var table_create = async function () {


          try{

              var tableRaw = [
                  { 'field': 'id', 'type': 'int','PK':1,'NN':1,'UQ':1 },
                  { 'field': 'name', 'type': 'varchar', 'length': 50 },
                  { 'field': 'age', 'type': 'int', 'length': 11 }
                ];
                          
                let ret = await  c.createTable("table_test", tableRaw).submit({expect:'db_success'});

                console.log("    table_create  :", ret);
      
          }
          catch (error) {
                  console.log("    table_create error : ", error);
          }
    };

----------------------

2.2 数据库表的插入
=============================================

插入以下数据到数据库表  ``table_test``

.. list-table::

    * - **id**
      - **name**
      - **age**
    * - 1
      - alice
      - 10

------------

- RPC代码

.. code-block:: json

  {
      "method": "r_insert",
      "params": [
          {
              "offline": false,
              "secret": "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb",
              "tx_json": {
                  "TransactionType": "SQLStatement",
                  "Account": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
                  "Owner": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
                  "Tables":[{
                      "Table":{
                          "TableName":"table_test"
                      }
                  }],
                  "Raw": [{"id":1, "name":"alice","age":10}],
                  "OpType": 6,
                  "AutoFillField":"TRACE_NO"
              }
          }
      ]
  }

-----------

- java代码

.. code-block:: java
  
    public void testinsert() {

        List<String> orgs = Util.array("{'id':1,'age': 10,'name':'alice'}");
        JSONObject obj;
        obj = c.table("table_test").insert(orgs).submit(SyncCond.db_success);
        System.out.println("insert result:" + obj);

      
    }

-------------------

- Node.js代码

.. code-block:: javascript

    var table_insert = async function () {

      try{

            const tableName = "table_test";
            const raw = [
                    {'id':1,'age': 10,'name':'alice'}
            ];
            
            let ret = await  c.table(tableName).insert(raw).submit({expect:'db_success'});

            console.log("    table_insert  :", ret);

      }
      catch (error) {
          console.log("    table_insert error : ", error);
      }
    }


----------------------

2.3 数据库表的更新
=============================================

更新数据库表  ``table_test`` 数据, 规则为 当  ``id=1`` 时，设置 ``age=11`` 


.. code-block:: json

  {
      "method": "r_update",
      "params": [{
          "offline": false,
          "secret": "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb",
          "tx_json": {
              "TransactionType": "SQLStatement",
              "Account": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
              "Owner": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
              "Tables": [
                  {
                      "Table": { "TableName": "table_test" }
                  }
              ],
              "Raw": [
                {
                    "age":11
                  },
                  {
                      "id":1
                  }
              ],
              "OpType": 8
          }
      }]
  }

-----------

- java代码

.. code-block:: java
  
    public void testUpdateTable() {
      List<String> arr = Util.array("{'id': 1}");

      JSONObject obj;
      obj = c.table("table_test").get(arr).update("{'age':11}").submit(SyncCond.db_success);
      System.out.println("update result:" + obj);
    }

-------------------

- Node.js代码

.. code-block:: javascript


  var table_update = async function () {

    try{

        const tableName = "table_test";

        let ret = await   c.table(tableName).get({'id': 1}).update({'age':11}).submit({expect:'db_success'});

        console.log("    table_update  :", ret);

    }
    catch (error) {
            console.log("    table_update error : ", error);
    }
  }

----------------------

2.4 重命名数据库表
=============================================

重命名数据库表,表名  ``table_test`` 修改为  ``table_test_new``

.. code-block:: json

    {
        "method": "t_rename",
        "params": [{
            "offline": false,
            "secret": "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb",
            "tx_json": {
                "TransactionType": "TableListSet",
                "Account": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
                "Tables": [
                    {
                        "Table": {
                            "TableName": "table_test",
                            "TableNewName": "table_test_new"
                        }
                    }
                ],
                "OpType": 3
            }
        }]
    }


-----------

- java代码

.. code-block:: java
  
    public void testrename() {
      JSONObject obj;
      obj = c.renameTable("table_test", "table_test_new").submit(SyncCond.db_success);
      System.out.println("rename result:" + obj);
    }

-------------------

- Node.js代码

.. code-block:: javascript

    var table_rename = async function () {

        try{

            let ret = await c.renameTable("table_test", "table_test_new").submit({ expect: 'db_success' });
            console.log("    table_rename :", ret);

        }
        catch (error) {
                console.log("    renameTable ", error);
        }

    }


------------------

2.5 数据库表的删除
=============================================

删除 数据库表  ``table_test``

.. code-block:: json

    {
        "method": "t_drop",
        "params": [{
            "secret": "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb",
            "tx_json": {
                "TransactionType": "TableListSet",
                "Account": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
                "Tables": [
                    {
                        "Table": { "TableName": "table_test" }
                    }
                ],
                "OpType": 2
            }
        }]
    }

-----------

- java代码

.. code-block:: java
  
    public void testdrop() {
      JSONObject obj;
      obj = c.dropTable("table_test").submit(SyncCond.db_success);
      System.out.println("drop result:" + obj);
    }

-------------------

- Node.js代码

.. code-block:: javascript

  var table_drop = async function () {
      

      try{

          var sTableName= "table_test";

          let lll = await c.dropTable(sTableName).submit({ expect: 'db_success' });
          console.log("    dropTable", sTableName, lll);

      }
      catch (error) {
              console.log("    dropTable error : ", error);
      }

  }



2.6 数据库表权限的授权
=============================================

表的拥有者可以授予其他用户操作表的权限。权限包括 ``select`` ``insert`` ``update`` ``delete`` 等4宗。


- RPC代码

.. code-block:: json

  {
      "method": "t_grant",
      "params": [{
          "offline": false,
          "secret": "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb",
          "tx_json": {
              "TransactionType": "TableListSet",
              "Account": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
              "Tables": [
                  {
                      "Table": { "TableName": "table_test" }
                  }
              ],
              "OpType": 11,
              "User": "zpMZ2H58HFPB5QTycMGWSXUeF47eA8jyd4",
              "Raw": [
                  {
                      "select": true,
                      "insert": true,
                      "update": true,
                      "delete": true
                  }
              ]
          }
      }]
  }


-----------

- java代码

.. code-block:: java
  
    public void grant() {
      JSONObject obj = new JSONObject();
      obj = c.grant("table_test", "zpMZ2H58HFPB5QTycMGWSXUeF47eA8jyd4", "{select:true,insert:true,update:true,delete:true}")
            .submit(SyncCond.validate_success);
      System.out.println("grant result:" + obj.toString());
    }

-------------------

- Node.js代码

.. code-block:: javascript

    var table_grant = async function () {

        try{

            var flag = { select:true,insert:true,update:true,delete:true };
            let ret = await c.grant("table_test", "zpMZ2H58HFPB5QTycMGWSXUeF47eA8jyd4", flag).submit({ expect: 'db_success' });
            console.log("    table_grant :", ret);

        }
        catch (error) {
                console.log("    table_grant error : ", error);
        }
    }


-----------------

2.7 数据库表数据的查询
=============================================

查询数据库的表数据。

- RPC代码

数据库查询的RPC代码, 具体见 :ref:`RPC <RPC_DB_Search>`

-----------

- java代码

.. code-block:: java
  
    String sTableName = "table_test";
    //查询 name 等于 alice 的记录.
    JSONObject obj  = c.table(sTableName).get(c.array("{'name': 'alice'}")).submit();

    System.out.println(obj);

-------------------

- Node.js代码

.. code-block:: javascript

  var table_get = async function () {

        try{
            const tableName = "table_test"; 
            //查询 name 等于 alice 的记录
            var raw =  [
                [],{"name":"alice"}];
              
            let ret = await  c.table(tableName).get(raw).submit();

            console.log("    table_get  :", ret);
    
        }
        catch (error) {
            console.log("    table_get error : ", error);
        }


  }


-----------------


2.8 加密表的支持
=============================================

设计文档参见  :ref:`Raw 加密 <Raw_Confidential>`


- RPC代码

.. code-block:: json

  {
      "method": "t_create",
      "params": [{
          "secret": "xnoPBzXtMeMyMHUVTgbuqAfg1SUTb",
          "tx_json": {
              "TransactionType": "TableListSet",
              "Account": "zHb9CJAWyB4zj91VRWn96DkukG4bwdtyTh",
              "Tables": [
                  {
                      "Table": { "TableName": "table_test" }
                  }
              ],
              "OpType": 1,
              "Raw": [
                  {
                      "field": "id",
                      "type": "int",
                      "length": 11,
                      "PK": 1,
                      "NN": 1,
                      "UQ": 1
                  },
                  {
                      "field": "name",
                      "type": "varchar",
                      "length": 50
                  },
                  {
                      "field": "age",
                      "type": "int",
                      "length": 11
                  }
              ],
              "Confidential": true
          }
      }]
  }


-----------

- java代码

.. code-block:: java
  
    public void testCreateConfidentialTable() {
      List<String> args = Util.array("{'field':'id','type':'int','length':11,'PK':1,'NN':1,'UQ':1}",
          "{'field':'name','type':'varchar','length':50,'default':null}", "{'field':'age','type':'int','length':11}");

      JSONObject obj;
      obj = c.createTable("table_test",args,true).submit(SyncCond.db_success);
      System.out.println("create result:" + obj);
    }

-------------------

- Node.js代码

.. code-block:: javascript

  var table_create_confidential = async function () {

    try{

        var tableRaw = [
            { 'field': 'id', 'type': 'int','PK':1,'NN':1,'UQ':1 },
            { 'field': 'name', 'type': 'varchar', 'length': 50 },
            { 'field': 'age', 'type': 'int', 'length': 11 }
          ];
          
          var option = {
              confidential: true
          };
          
          let ret = await  c.createTable("table_test", tableRaw,option).submit({expect:'db_success'});

          console.log("    table_create_confidential  :", ret);

    }
    catch (error) {
            console.log("    table_create_confidential error : ", error);
    }

  }

----------------------

2.9 智能合约对表操作的支持
=============================================

 :ref:`表操作智能合约支持 <SmartContract_DB_Oper>`


3 完整的代码示例
*************************

- `node.js 数据库表操作 <https://github.com/ChainSQL/node-chainsql-api/blob/master/test/testTable.js/>`_

- `JAVA 数据库表操作  <https://github.com/ChainSQL/java-chainsql-api/blob/master/chainsql/src/test/java/com/peersafe/example/chainsql/TestChainsql.java/>`_

