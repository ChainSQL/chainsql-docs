.. _remix.chainsql.net: http://remix.chainsql.net

在chainsql的nodejs接口中，我们提供了chainsql.contract对象，通过创建该对象，就可以开始与chainsql智能合约的交互了。

.. _contract-newObj:

合约示例
============

.. code-block:: javascript

    pragma solidity ^0.4.2;

    // Modified Greeter contract. Based on example at https://www.ethereum.org/greeter.

    contract mortal {
            /* Define variable owner of the type address*/
            address owner;
            
            

            /* this function is executed at initialization and sets the owner of the contract */
            constructor() public { owner = msg.sender; }

            /* Function to recover the funds on the contract */
            function kill() public { if (msg.sender == owner) selfdestruct(owner); }
    }

    contract greeter is mortal {
            /* define variable greeting of the type string */
            string greeting;
            
            
            function() external payable { }

            /* this runs when the contract is executed */
            constructor(string _greeting) public payable{
                    greeting = _greeting;
            }

            function newGreeting(string _greeting) public {
                    emit Modified(greeting, _greeting, greeting, _greeting);
                    greeting = _greeting;
            }
            
            function getMultiGreeting() public view returns(string,string){
                
                    return(greeting,greeting);
            }

            

            /* main function */
            function greet() public view returns (string) {
                    return greeting;
            }

            /* we include indexed events to demonstrate the difference that can be
            captured versus non-indexed */
            event Modified(
                            string indexed oldGreetingIdx, string indexed newGreetingIdx,
                            string oldGreeting, string newGreeting);
    }

------------------------------------------------------------------------------



创建合约对象
============
.. code-block:: javascript

	chainsql.contract(abi[, contractAddr]);

创建一个包含了所有在abi中定义的函数和事件的新合约对象。chainsql是一个chainsql-nodejs的对象。

参数说明
--------

1. ``abi`` - ``Array`` : 一个由Json对象组成的数组，每个Json对象都是solidity的函数或者事件,可通过 `remix.chainsql.net`_ 获取；
2. ``contractAddr`` - ``String`` : [**可选**]合约地址，在初次创建合约对象时，没有合约地址，可不填；合约已经存在，可直接利用合约地址，创建一个包含该合约函数和事件的合约对象。


返回值
--------

``Object`` : 包含合约函数和事件的智能合约对象。

示例
--------
.. code-block:: javascript

    //first create a contract object without contract address.
    const abi = '[{"constant":true,"inputs":[],"name":"getMultiGreeting","outputs":[{"name":"","type":"string"},{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[],"name":"kill","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"_greeting","type":"string"}],"name":"newGreeting","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"greet","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"inputs":[{"name":"_greeting","type":"string"}],"payable":true,"stateMutability":"payable","type":"constructor"},{"payable":true,"stateMutability":"payable","type":"fallback"},{"anonymous":false,"inputs":[{"indexed":true,"name":"oldGreetingIdx","type":"string"},{"indexed":true,"name":"newGreetingIdx","type":"string"},{"indexed":false,"name":"oldGreeting","type":"string"},{"indexed":false,"name":"newGreeting","type":"string"}],"name":"Modified","type":"event"}]';
    const contractObj = chainsql.contract(JSON.parse(abi));

    //create a contract object with contract address already in chainsql blockchain.
    const abi = '[{"constant":true,"inputs":[],"name":"getMultiGreeting","outputs":[{"name":"","type":"string"},{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[],"name":"kill","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"_greeting","type":"string"}],"name":"newGreeting","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"greet","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"inputs":[{"name":"_greeting","type":"string"}],"payable":true,"stateMutability":"payable","type":"constructor"},{"payable":true,"stateMutability":"payable","type":"fallback"},{"anonymous":false,"inputs":[{"indexed":true,"name":"oldGreetingIdx","type":"string"},{"indexed":true,"name":"newGreetingIdx","type":"string"},{"indexed":false,"name":"oldGreeting","type":"string"},{"indexed":false,"name":"newGreeting","type":"string"}],"name":"Modified","type":"event"}]';
    const contractObj = chainsql.contract(JSON.parse(abi), "zEyJTYupmFGo3tWCts8uyzmkkmjJAwgS2u");

------------------------------------------------------------------------------

.. _contract-deploy:

合约部署
=========
.. code-block:: javascript

    contractObj.deploy(paramsNeeded).submit(options[, callback]);

通过deploy函数将合约部署到链上，返回结果可以通过指定callback函数或者直接使用then和catch接收promise的resolve和reject。

参数说明
--------

1. ``paramsNeeded`` - ``JsonObject`` : 由以下两个必选字段组成：

    * ``ContractData`` - ``String`` : 智能合约字节码，以0x开头。可通过 `remix.chainsql.net`_ 获取；
    * ``arguments`` - ``Array``: 智能合约构造函数的参数作为元素组成的数组，每个元素根据类型填写元素值，没有参数提供一个空数组[]。

2. ``options`` - ``JsonObject`` : 由以下两个字段组成：

    * ``ContractValue`` - ``String`` : [**可选**]如果合约构造函数为payable类型，则可使用该字段给合约转账，单位为drop；
    * ``Gas`` - ``Number`` : 执行合约部署所需要的手续费用。

3. ``callback`` - ``Function`` : [**可选**]回调函数，如果指定，则通过指定回调函数返回结果，否则需要通过then和catch接收promise结果。

返回值
--------

``JsonObject`` - 智能合约的部署成功或失败的结果。返回方式取决于是否指定回调函数。

1. 部署成功 - 指定回调函数，则通过回调函数返回，否则返回返回一个resolve的Promise对象。 ``JsonObject`` 包含以下字段：

	* ``status`` - ``String`` : 表示部署状态，成功则为固定值:"validate_success"；
	* ``tx_hash`` - ``String`` : 部署合约的交易hash值；
	* ``contractAddress`` - ``String`` : 部署成功的合约地址。
	
2. 部署失败 - 指定回调函数，则通过回调函数返回，否则返回返回一个reject的Promise对象。 ``JsonObject`` 内容依具体错误形式返回。

示例
--------
.. code-block:: javascript

    
    // use the callback
    const abi = '[{"constant":true,"inputs":[],"name":"getMultiGreeting","outputs":[{"name":"","type":"string"},{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[],"name":"kill","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"_greeting","type":"string"}],"name":"newGreeting","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"greet","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"inputs":[{"name":"_greeting","type":"string"}],"payable":true,"stateMutability":"payable","type":"constructor"},{"payable":true,"stateMutability":"payable","type":"fallback"},{"anonymous":false,"inputs":[{"indexed":true,"name":"oldGreetingIdx","type":"string"},{"indexed":true,"name":"newGreetingIdx","type":"string"},{"indexed":false,"name":"oldGreeting","type":"string"},{"indexed":false,"name":"newGreeting","type":"string"}],"name":"Modified","type":"event"}]';    const contractObj = chainsql.contract(JSON.parse(abi));
    const deployBytecode = '0x60806040526040516109b13803806109b18339810180604052602081101561002657600080fd5b81019080805164010000000081111561003e57600080fd5b8281019050602081018481111561005457600080fd5b815185600182028301116401000000008211171561007157600080fd5b5050929190505050336000806101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff16021790555080600190805190602001906100cf9291906100d6565b505061017b565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f1061011757805160ff1916838001178555610145565b82800160010185558215610145579182015b82811115610144578251825591602001919060010190610129565b5b5090506101529190610156565b5090565b61017891905b8082111561017457600081600090555060010161015c565b5090565b90565b6108278061018a6000396000f300608060405260043610610062576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806305fe42491461006457806341c0e1b5146101605780634ac0d66e14610177578063cfae32171461023f575b005b34801561007057600080fd5b506100796102cf565b604051808060200180602001838103835285818151815260200191508051906020019080838360005b838110156100bd5780820151818401526020810190506100a2565b50505050905090810190601f1680156100ea5780820380516001836020036101000a031916815260200191505b50838103825284818151815260200191508051906020019080838360005b83811015610123578082015181840152602081019050610108565b50505050905090810190601f1680156101505780820380516001836020036101000a031916815260200191505b5094505050505060405180910390f35b34801561016c57600080fd5b50610175610415565b005b34801561018357600080fd5b5061023d6004803603602081101561019a57600080fd5b81019080803590602001906401000000008111156101b757600080fd5b8201836020820111156101c957600080fd5b803590602001918460018302840111640100000000831117156101eb57600080fd5b91908080601f016020809104026020016040519081016040528093929190818152602001838380828437600081840152601f19601f8201169050808301925050505050505091929192905050506104a6565b005b34801561024b57600080fd5b506102546106b4565b6040518080602001828103825283818151815260200191508051906020019080838360005b83811015610294578082015181840152602081019050610279565b50505050905090810190601f1680156102c15780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b606080600180818054600181600116156101000203166002900480601f01602080910402602001604051908101604052809291908181526020018280546001816001161561010002031660029004801561036a5780601f1061033f5761010080835404028352916020019161036a565b820191906000526020600020905b81548152906001019060200180831161034d57829003601f168201915b50505050509150808054600181600116156101000203166002900480601f0160208091040260200160405190810160405280929190818152602001828054600181600116156101000203166002900480156104065780601f106103db57610100808354040283529160200191610406565b820191906000526020600020905b8154815290600101906020018083116103e957829003601f168201915b50505050509050915091509091565b6000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff1614156104a4576000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16ff5b565b806040518082805190602001908083835b6020831015156104dc57805182526020820191506020810190506020830392506104b7565b6001836020036101000a0380198251168184511680821785525050505050509050019150506040518091039020600160405180828054600181600116156101000203166002900480156105665780601f10610544576101008083540402835291820191610566565b820191906000526020600020905b815481529060010190602001808311610552575b505091505060405180910390207f047dcd1aa8b77b0b943642129c767533eeacd700c7c1eab092b8ce05d2b2faf56001846040518080602001806020018381038352858181546001816001161561010002031660029004815260200191508054600181600116156101000203166002900480156106245780601f106105f957610100808354040283529160200191610624565b820191906000526020600020905b81548152906001019060200180831161060757829003601f168201915b5050838103825284818151815260200191508051906020019080838360005b8381101561065e578082015181840152602081019050610643565b50505050905090810190601f16801561068b5780820380516001836020036101000a031916815260200191505b5094505050505060405180910390a380600190805190602001906106b0929190610756565b5050565b606060018054600181600116156101000203166002900480601f01602080910402602001604051908101604052809291908181526020018280546001816001161561010002031660029004801561074c5780601f106107215761010080835404028352916020019161074c565b820191906000526020600020905b81548152906001019060200180831161072f57829003601f168201915b5050505050905090565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f1061079757805160ff19168380011785556107c5565b828001600101855582156107c5579182015b828111156107c45782518255916020019190600101906107a9565b5b5090506107d291906107d6565b5090565b6107f891905b808211156107f45760008160009055506001016107dc565b5090565b905600a165627a7a72305820ec5e59bd3a1929ffbf0765edb4eaf1e4d7589b32337d5fd1de23074fdef29f390029';
    contractObj.deploy({
        ContractData : deployBytecode,
        arguments:['Hello blockchain world!']
    }).submit({
        Gas: '5000000',
    }, function (err, res) {
        err ? console.log(err) : console.log(res);
    });
    >
    {
        status:"validate_success"
        tx_hash:"DD443076A8A4B02B6661261CCD456F2DC7F4031F12EC38EAD35E821782328318"
        contractAddress:"zPqMARn53PpN2fu8eScac4cEYW6b4w8ZH"
    }


    // use the promise
    const abi = '[{"constant":true,"inputs":[],"name":"getMultiGreeting","outputs":[{"name":"","type":"string"},{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[],"name":"kill","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"_greeting","type":"string"}],"name":"newGreeting","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"greet","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"inputs":[{"name":"_greeting","type":"string"}],"payable":true,"stateMutability":"payable","type":"constructor"},{"payable":true,"stateMutability":"payable","type":"fallback"},{"anonymous":false,"inputs":[{"indexed":true,"name":"oldGreetingIdx","type":"string"},{"indexed":true,"name":"newGreetingIdx","type":"string"},{"indexed":false,"name":"oldGreeting","type":"string"},{"indexed":false,"name":"newGreeting","type":"string"}],"name":"Modified","type":"event"}]';    const contractObj = chainsql.contract(JSON.parse(abi));
    const deployBytecode = '0x60806040526040516109b13803806109b18339810180604052602081101561002657600080fd5b81019080805164010000000081111561003e57600080fd5b8281019050602081018481111561005457600080fd5b815185600182028301116401000000008211171561007157600080fd5b5050929190505050336000806101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff16021790555080600190805190602001906100cf9291906100d6565b505061017b565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f1061011757805160ff1916838001178555610145565b82800160010185558215610145579182015b82811115610144578251825591602001919060010190610129565b5b5090506101529190610156565b5090565b61017891905b8082111561017457600081600090555060010161015c565b5090565b90565b6108278061018a6000396000f300608060405260043610610062576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806305fe42491461006457806341c0e1b5146101605780634ac0d66e14610177578063cfae32171461023f575b005b34801561007057600080fd5b506100796102cf565b604051808060200180602001838103835285818151815260200191508051906020019080838360005b838110156100bd5780820151818401526020810190506100a2565b50505050905090810190601f1680156100ea5780820380516001836020036101000a031916815260200191505b50838103825284818151815260200191508051906020019080838360005b83811015610123578082015181840152602081019050610108565b50505050905090810190601f1680156101505780820380516001836020036101000a031916815260200191505b5094505050505060405180910390f35b34801561016c57600080fd5b50610175610415565b005b34801561018357600080fd5b5061023d6004803603602081101561019a57600080fd5b81019080803590602001906401000000008111156101b757600080fd5b8201836020820111156101c957600080fd5b803590602001918460018302840111640100000000831117156101eb57600080fd5b91908080601f016020809104026020016040519081016040528093929190818152602001838380828437600081840152601f19601f8201169050808301925050505050505091929192905050506104a6565b005b34801561024b57600080fd5b506102546106b4565b6040518080602001828103825283818151815260200191508051906020019080838360005b83811015610294578082015181840152602081019050610279565b50505050905090810190601f1680156102c15780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b606080600180818054600181600116156101000203166002900480601f01602080910402602001604051908101604052809291908181526020018280546001816001161561010002031660029004801561036a5780601f1061033f5761010080835404028352916020019161036a565b820191906000526020600020905b81548152906001019060200180831161034d57829003601f168201915b50505050509150808054600181600116156101000203166002900480601f0160208091040260200160405190810160405280929190818152602001828054600181600116156101000203166002900480156104065780601f106103db57610100808354040283529160200191610406565b820191906000526020600020905b8154815290600101906020018083116103e957829003601f168201915b50505050509050915091509091565b6000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff1614156104a4576000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16ff5b565b806040518082805190602001908083835b6020831015156104dc57805182526020820191506020810190506020830392506104b7565b6001836020036101000a0380198251168184511680821785525050505050509050019150506040518091039020600160405180828054600181600116156101000203166002900480156105665780601f10610544576101008083540402835291820191610566565b820191906000526020600020905b815481529060010190602001808311610552575b505091505060405180910390207f047dcd1aa8b77b0b943642129c767533eeacd700c7c1eab092b8ce05d2b2faf56001846040518080602001806020018381038352858181546001816001161561010002031660029004815260200191508054600181600116156101000203166002900480156106245780601f106105f957610100808354040283529160200191610624565b820191906000526020600020905b81548152906001019060200180831161060757829003601f168201915b5050838103825284818151815260200191508051906020019080838360005b8381101561065e578082015181840152602081019050610643565b50505050905090810190601f16801561068b5780820380516001836020036101000a031916815260200191505b5094505050505060405180910390a380600190805190602001906106b0929190610756565b5050565b606060018054600181600116156101000203166002900480601f01602080910402602001604051908101604052809291908181526020018280546001816001161561010002031660029004801561074c5780601f106107215761010080835404028352916020019161074c565b820191906000526020600020905b81548152906001019060200180831161072f57829003601f168201915b5050505050905090565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f1061079757805160ff19168380011785556107c5565b828001600101855582156107c5579182015b828111156107c45782518255916020019190600101906107a9565b5b5090506107d291906107d6565b5090565b6107f891905b808211156107f45760008160009055506001016107dc565b5090565b905600a165627a7a72305820ec5e59bd3a1929ffbf0765edb4eaf1e4d7589b32337d5fd1de23074fdef29f390029';
    contractObj.deploy({
        ContractData : deployBytecode,
        arguments:['Hello blockchain world!']
    }).submit({
        Gas: '5000000',
    }).then(res => {
        console.log(res);
    }).catch(err => {
        console.error(err);
    });

------------------------------------------------------------------------------

.. _contract-submit:

更改合约内部状态调用
====================
.. code-block:: javascript

    contractObj.methods.function([params1[, params2, ...]]]).submit(options[, callback])

这种调用方式实际是以交易的形式发送到chainsql链上。然后执行合约的对应方法。并会对合约内部状态产生影响。function为合约的具体方法名。

参数说明
---------

1. ``params`` - ``any`` : 合约本身function的参数值，依据合约方法的参数个数和类型进行传递；
2. ``options`` - ``JsonObject`` : 由以下两个字段组成：

    * ``ContractValue`` - ``String`` : [**可选**]如果合约函数为payable类型，则可使用该字段给合约转账，单位为drop；
    * ``Gas`` - ``Number`` : 执行合约函数所需要的手续费用。

	.. _tx-expect:
    * ``expect`` - ``String`` : [**可选**]在chainsql中提供几种预期交易执行结果的返回，不指定则使用"send_success"，可选执行结果如下：

        - "send_success" : 交易发送成功即返回结果；
    	- "validate_success" ： 交易共识成功即返回结果；
    	- "db_success" ： 涉及数据库交易，执行入库成功即返回结果。
3. ``callback`` - ``Function`` : [**可选**]回调函数，如果指定，则通过指定回调函数返回结果，否则需要通过then和catch接收promise结果。

返回值
--------

``JsonObject`` : 合约函数执行成功或失败的结果。返回方式取决于是否指定回调函数。

1. 调用成功 - 指定回调函数，则通过回调函数返回，否则返回返回一个resolve的Promise对象。 ``JsonObject`` 包含以下字段：

	* ``status`` - ``String`` : 表示合约函数执行状态，其值由调用时的expect决定；
	* ``tx_hash`` - ``String`` : 合约函数的交易hash值。
	
2. 调用失败 - 指定回调函数，则通过回调函数返回，否则返回返回一个reject的Promise对象。 ``JsonObject`` 内容依具体错误形式返回。

示例
--------
.. code-block:: javascript

    // use the promise
    contractObj.methods.newGreeting('hello 1').submit({
        Gas: 500000,
        expect: "validate_success"
    }).then(res => {
        console.log(res);
    }).catch(err => {
        console.log(err);
    });

    // use the callback
    contractObj.methods.newGreeting('hello 1').submit({
        Gas: 500000,
        expect: "validate_success"
    },function (err, res) {
        err ? console.error(err) : console.log(res);
    });
    >
    {
        status:"validate_success"
        tx_hash:"F29FE3A0652162A480E591B92CB6982408FB4AFEB5BF645024D847E4218385BB"
    }

.. _contract-call:

读取合约内部状态调用
====================
::

    contractObj.methods.function([params1[, params2[, ...]]]).call([callback])

这种调用方式只是读取合约内部某个变量状态，非交易，不会对合约内部状态产生影响。function为合约的具体方法名。

参数说明
---------

1. ``params`` - ``any`` : 合约本身function的参数值，依据合约方法的参数个数和类型进行传递；
2. ``callback`` - ``Function`` : [**可选**]回调函数，如果指定，则通过指定回调函数返回结果，否则需要通过then和catch接收promise结果。

返回值
--------

返回值由合约本身的函数规定的返回值个数及类型决定，个数为1时，直接返回该值，个数大于1时，构造为一个JsonObject返回。返回方式取决于是否指定回调函数。

1. 调用成功时，指定回调函数，则通过回调函数返回，否则返回返回一个resolve的Promise对象；
2. 调用失败时，指定回调函数，则通过回调函数返回，否则返回返回一个reject的Promise对象。依具体错误形式返回。

示例
--------
.. code-block:: javascript

    // return only one value
    contractObj.methods.greet().call((err, str) => {
        err ? console.log("greet err:", err) : console.log(str);
    });
    >
    hello 1

    // return more than one value
    contractObj.methods.getMultiGreeting().call((err, str) => {
        err ? console.log("getMultiGreeting err:", err) : console.log(str);
    });
    >
    {0: "hello 1", 1: "hello 1"}

------------------------------------------------------------------------------

.. _contract-fallback:

合约fallback函数调用
====================
::

	chainsql.payToContract(contractAddr, contractValue, gas).submit(options)

当合约定义了fallback函数时，可通过payToContract接口直接向合约转账。如果未定义则调用出错。

参数说明
---------

1. ``contractAddr`` - ``String`` : 接收转账的合约地址；
2. ``contractValue`` - ``Number`` : 转账数额；
3. ``gas`` - ``Number`` : 执行转账交易的手续费用；
4. ``options`` - ``JsonObject`` : 指定交易执行到何种状态返回，默认为"send_success", 具体可参考 :ref:`交易expect <tx-expect>`。

返回值
--------

``Promise`` : 根据调用时的expect值，返回对应的执行状态。成功返回一个resolve的Promise对象，失败返回一个reject的Promise对象。

示例
--------
.. code-block:: javascript

    const contractAddr = "zEyJTYupmFGo3tWCts8uyzmkkmjJAwgS2u";
    chainsql.payToContract(contractAddr, 2000, 30000000).submit({
        expect: "validate_success"
    }).then(res => {
        console.log(res);
    }).catch(err => {
        console.log(err);
    });
    >
    {
        status:"validate_success",
        tx_hash:"92A7E277BB4229DAEC71A2D9D8C282FB307E328E8FC05C4BE29D20240A5F9E13"
    }

------------------------------------------------------------------------------

.. _contract-encode:

获取合约函数编码值
==================
.. code-block:: javascript

    contractObj.methods.function([params1[, params2, ...]]]).encodeABI()

将合约函数包括参数在内进行编码，得到contract data，或者称为inputdata。可以直接用于合约函数调用，或者在其他合约中作为参数传递，或者使用chainsql的rpc接口调用合约。

参数说明
---------

1. ``params`` - ``any`` : 合约本身function的参数值，依据合约方法的参数个数和类型进行传递；

返回值
--------

``String`` : 进行合约调用时可以使用的函数编码值，contract data。

示例
--------
.. code-block:: javascript

    let funInputData = contractObj.methods.greet().encodeABI();
    console.log(funInputData);
    > "0xcfae3217"

------------------------------------------------------------------------------

.. _contract-event:

合约事件监听
=============
.. code-block:: javascript

    contractObj.events.eventFunc([callback]);

参数说明
---------

1. ``callback`` - ``Function`` : [**可选**]回调函数，如果指定，则通过指定回调函数返回结果，否则需要通过then和catch接收promise结果。

返回值
--------

返回值包含合约事件指定的监听内容，返回方式由是否指定回调函数决定。

1. 正常监听：指定回调函数，则通过回调函数返回，否则返回返回一个resolve的Promise对象。具体返回内容包括：
``JsonObject`` : 返回事件内容，具体包含以下字段：

    * ``ContractAddress`` - ``String`` : 合约地址；
    * ``event`` - ``String`` : 事件函数名称；
    * ``raw`` - ``JsonObject`` :  事件返回原始十六进制数据，包括data和topic两个字段；
    * ``returnValues`` - ``JsonObject`` :  按事件定义的返回值顺序以及返回值变量名，给出可读形式的返回值；
    * ``signature`` - ``String`` : 事件函数签名；
    * ``type`` - ``String`` : 类型，固定值为"contract_event"。

2. 监听异常：指定回调函数，则通过回调函数返回，否则返回返回一个reject的Promise对象。依具体错误形式返回。

示例
--------
.. code-block:: javascript

    contractObj.events.Modified((err, res) => {
        err ? console.log(err) : console.log(JSON.stringify(res));
    });
    >
    {
        "ContractAddress": "zfKAaSvH5QLm25bn2VGv2fbYfuthFdVxeP",
        "type": "contract_event",
        "returnValues": {
            "0": "0x67bc65b2694b00787b2c2fb7ac32f828b5c1f01c49107edabd382a1dc3a5f86c",
            "1": "0x67bc65b2694b00787b2c2fb7ac32f828b5c1f01c49107edabd382a1dc3a5f86c",
            "2": "hello 1",
            "3": "hello 1",
            "oldGreetingIdx": "0x67bc65b2694b00787b2c2fb7ac32f828b5c1f01c49107edabd382a1dc3a5f86c",
            "newGreetingIdx": "0x67bc65b2694b00787b2c2fb7ac32f828b5c1f01c49107edabd382a1dc3a5f86c",
            "oldGreeting": "hello 1",
            "newGreeting": "hello 1"
        },
        "event": "Modified",
        "signature": "0x047dcd1aa8b77b0b943642129c767533eeacd700c7c1eab092b8ce05d2b2faf5",
        "raw": {
            "data": "0x00000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000000000000000000000000000000000080000000000000000000000000000000000000000000000000000000000000000768656C6C6F203200000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000768656C6C6F203200000000000000000000000000000000000000000000000000",
            "topics": [
                "0x047dcd1aa8b77b0b943642129c767533eeacd700c7c1eab092b8ce05d2b2faf5",
                "0x67bc65b2694b00787b2c2fb7ac32f828b5c1f01c49107edabd382a1dc3a5f86c",
                "0x67bc65b2694b00787b2c2fb7ac32f828b5c1f01c49107edabd382a1dc3a5f86c"
            ]
        }
    }



合约中调用新增的表相关指令
=================================================

`数据库合约示例 <https://github.com/ChainSQL/java-chainsql-api/blob/master/chainsql/src/main/resources/solidity/table/solidity-TableTxs.sol>`_ 


`测试用例 <https://github.com/ChainSQL/node-chainsql-api/blob/master/test/testContractTableTxs.js>`_


合约中调用新增的代币接口相关指令
=================================================

`代币合约示例 <https://github.com/ChainSQL/java-chainsql-api/blob/master/chainsql/src/main/resources/solidity/gateway/solidity-GatewayTxs.sol>`_ 


`node.js测试用例 <https://github.com/ChainSQL/node-chainsql-api/tree/master/test/testContractGatewayTxs.js>`_
