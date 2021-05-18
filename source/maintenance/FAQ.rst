
############
FAQ
############

Api 调用问题
**********************

1. 两笔挂单，分别是买单与卖单，兑换比例一致，但未成交
    原因：网关未开启 

2. Insufficient reserve to create offer
    原因：挂单时Zxc余额不满足保留费用要求，需要最终满足::
    
        ZxcBalance >= AccountReserve(5) + AccountObjectCount*1

3. Fee of xxx exceeds 
    原因：不在交易中填写Fee会自己去算，而负载高时，算出来的Fee比较大，>100时，会报这个错误。所以建议自己在交易中填上Fee的值。

4. 对表进行操作，无论什么交易都是db_timeout
    原因：表的某个区块没有同步到，由于某种原因被跳过了，然后后面的交易，PreviousTxnLgrSeq总是无法与当前的对上
    目前暂未发现导致的根本原因，只能把LedgerSeq重新置为TxnLedgerSeq:

    .. code-block:: sql

        update SyncTableState as t1,(select * from SyncTableState where 
        Owner='zDmdwbqbtcxPcj9ytzuKyQLPQi2DuWXmgk' and deleted=0) as t2 set 
        t1.LedgerSeq=t2.TxnLedgerSeq where t1.TableNameInDB=t2.TableNameInDB;

5. Insufficient reserve to create offer   
    众享币不足  需要预留对象+5  查询对象 accountObject

6. Auth for unclaimed account needs correct master key 
    账户与签名私钥不一致

7. Current user doesn\'t have this auth   
    用户没有信任这个货币

8. Ledger sequence too high / tefMAX_LEDGER
    这个是因为发送交易的时候，用API发送交易时，会在交易中带一个LastLedgerSequence，当交易发送至节点时，发到节点的时候，发现当前区块号已经大于LastLedgerSequence，就会报这个错误。
    引起这个错误的原因可能是：

    1. 一般是客户端调试导致发送时间太长
    2. node.js api客户端执行了同步操作，并且同步操作的时间太长，导致订阅的ledgerVersion得不到更新，在准备交易时取到的是旧的ledgerVersion，虽然最后对ledgerVersion+3作为 LastLedgerSequence的值，但是到节点那里仍然发生tefMAX_LEDGER错误

9. This sequence number has already past.
    这个是因为用API发送交易时，会在交易中带一个交易账户的Sequence,当交易发送至节点时，发现账户的Sequence已经大于交易中带的Sequence时，就会报这个错误。
    引起这个错误的原因可能是：

    1. 同一个账户，没有等上一次的交易成功返回后，继续发送一个新的交易。例如同一个账户准备发送两个交易，发送第一个交易时，Sequence为2；没有等第一个交易成功返回时，继续发送第二个的交易，此时Sequence依然为2。当第二个交易发送到节点时，此时由于第一个交易已经处理成功，账户的Sequence变为了3，而第二个交易中带的Sequence还为2，此时就会出现 **This sequence number has already past** 的错误。

    

Chainsql节点问题
**********************

1. 客户端频繁发请求，过一段时间就发不了了，直接丢包
    输出如下：

    .. code-block:: console

        2018-Dec-05 02:45:23 Resource:WRN Consumer entry 114.242.47.14 dropped with balance 525166 at or above drop threshold 15000
        2018-Dec-05 02:45:23 Resource:WRN Consumer entry 114.242.47.14 dropped with balance 525260 at or above drop threshold 15000
        2018-Dec-05 02:45:23 Resource:WRN Consumer entry 114.242.47.14 dropped with balance 525354 at or above drop threshold 15000
        2018-Dec-05 02:45:23 Resource:WRN Consumer entry 114.242.47.14 dropped with balance 525447 at or above drop threshold 15000

    这是因为ChainSQL本身不允许一个ip频繁发请求，认为这是在攻击，如果一个已知ip要这样做，需要把它放到节点的admin列表中。

    .. code-block:: ini

        [port_ws_public]
        port = 5006
        ip = 0.0.0.0
        protocol = ws
        admin = 101.201.40.124,59.110.154.242,114.242.47.102

    然后重启节点

2. 启动节点报：make db backends error,please check cfg!
    数据库连接失败，请检查配置文件中数据库(sync_db)的配置

3. cfg文件中配置项不起作用
    注释只能写在开头，中间写注释会导致配置项不起作用（#）

4. 如何升级chainsql节点
    一般升级chainsql节点只需要挨个节点替换重启即可，步骤如下：

    1. 停掉一个正在运行的节点（先用 ``./chainsqld stop`` 命令，如果停不掉再用 ``kill`` 命令杀进程）
    2. 替换新的chainsqld可执行程序
    3. 启动chainsqld进程
    4. 查看 ``server_info``，直到 ``completed_ledgers`` 正常出块
    5. 依次对所有节点执行1-4过程

5. 节点全部挂掉，找不到原因
    | 使用secureCRT或者Xshell连接服务器，退出时，直接关闭对话窗口，会将nohup后台运行的进程杀死。
    | 应该使用 ``exit`` 命令退出 ssh 工具终端

6. peers命令看不到其它节点，节点日志报：Clock for is off by ...
    节点间时间不一致导致的，一般发生在内网，有两种解决方式：
    
    1. 手动将时间调整到一致（相关不超过20秒即可）
    2. 在[sntp_servers]中配置内网的时间服务器地址（推荐用这一种方式）

7. 执行./chainsqld peers 命令报 internal error 403
    这是因为peers命令是一个admin权限的命令，节点配置文件中的http协议配置中，admin肯定不包含本机，解决方法：

    1. 在http协议配置中admin配置加上127.0.0.1
    2. 将admin配置为0.0.0.0（表示所有调用http命令的ip都是admin，不推荐这种做法）

    .. code-block:: ini

        [port_rpc_admin_local]
        port = 5006
        ip = 0.0.0.0
        admin = 127.0.0.1
        protocol = http

8. peers命令看不到其它节点，配置没问题，telnet peer端口能通，不是问题6的情况
    可能原因：

    1. ``db/peerfinder.sqlite`` 与 ``db/wallet.db`` 会缓存之前的连接ip，会影响节点发现，将这两个文件删除再重启

9. 节点启动时突然退出，日志最后几行没有错误信息
    在日志中查找有没有 ``FTL`` 字样的信息，如：

    .. code-block:: console

        2019-Dec-19 03:21:07 Application:FTL Invalid seed specified in [validation_seed]
        2019-Dec-19 03:21:07 JobQueue:NFO Auto-tuning to 6 validation/transaction/proposal threads.

    ``FTL`` 是 ``fatal`` 的缩写，上面的日志说明是 ``[validation_seed]`` 字段配置有问题导致 ``fatal`` 级别错误。发出 ``fatal`` 错误信号后节点过一会儿自动退出

表同步问题
**********************

1. 表同步后，中文显示乱码。
    可能原因：

    1. 后端数据库配置的字符集不是 ``utf8`` ，确认后端数据库的字符集配置。

    .. code-block:: console

        mysql> show variables like '%character%';
        +--------------------------+----------------------------------------+
        | Variable_name            | Value                                  |
        +--------------------------+----------------------------------------+
        | character_set_client     | utf8                                   |
        | character_set_connection | utf8                                   |
        | character_set_database   | utf8                                   |
        | character_set_filesystem | binary                                 |
        | character_set_results    | utf8                                   |
        | character_set_server     | utf8                                   |
        | character_set_system     | utf8                                   |
        | character_sets_dir       | E:\mysql-5.7.24-winx64\share\charsets\ |
        +--------------------------+----------------------------------------+

    2. 节点配置文件中 ``sync_db`` 配置单元中没有配置 ``charset`` 或 ``charset`` 配置项不是 ``utf8`` 。

    .. code-block:: ini

        [sync_db]
        type=mysql
        #type=sqlite
        host=localhost
        port=3306
        user=root
        pass=123456
        db=wc
        charset=utf8
        #first_storage=0