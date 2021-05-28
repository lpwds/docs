数据收发
=================================

.. toctree::
   :maxdepth: 2
   :caption: Contents:


=======================
1. 简介
=======================
LPWDS服务器程序使用redis pubsub模式作为其数据收发接口。

=======================
2. 节点信息读取
=======================
所有的节点信息都存储在redis数据里面， 获得节点列表的方法

.. code:: bash

    127.0.0.1:6379> smembers nodes
    1) "E2CB0730F0AE"
    2) "C68F2E7961D1"

读取节点信息的方法，例如读取"E2CB0730F0AE"节点信息

.. code:: bash

    127.0.0.1:6379> hgetall E2CB0730F0AE
    1) "type"
    2) "node"
    3) "state"
    4) "online"
    5) "rssi"
    6) "30"
    7) "base"
    8) "8B24B2713234"
    9) "timestamp"
    10) "169568"


=======================
2. 数据收发
=======================
节点的数据收发使用redis的pubsub模式来实现

为了支持二进制数据的发送， 数据的发送和接收使用的是HexString格式，
注意单次可发送或接收18个字节数据， 即HexString最大长度是36个字节

向节点发送数据，例如向节点"E2CB0730F0AE"发送数据

.. code:: bash

    127.0.0.1:6379> publish pipe.command E2CB0730F0AE010000006A30313233343536
    (integer) 1

    #E2CB0730F0AE010000006A30313233343536
    #E2CB0730F0AE： 节点模块的MAC地址
    #            01：数据类型：固定为01
    #              0000006A：发送消息的Profile，见节点模块
    #                      30313233343536：实际发送的内容


接收来自节点消息，接收来及节点的消息只需要订阅: pipe.message 即可

.. code:: bash

    127.0.0.1:6379> subscribe pipe.message
    Reading messages... (press Ctrl-C to quit)
    1) "subscribe"
    2) "pipe.message"
    3) (integer) 1

收到来及节点的消息会显示

.. code:: bash
    
    1) "message"
    2) "pipe.message"
    3) "C68F2E7961D1030000006E100101000256FEC800"
    #C68F2E7961D1030000006E100101000256FEC800
    #C68F2E7961D1 节点模块MAC地址
    #            03 消息类型 REDIS_NODE_API_DATA
    #              0000006E 消息的Profile
    #                      100101000256FEC800 消息内容

节点消息类型：

.. code:: c

    #define REDIS_NODE_API_CONNECT		0x01
    #define REDIS_NODE_API_DISCONNECT	        0x02
    #define REDIS_NODE_API_DATA			0x03
    #define REDIS_NODE_API_POLL			0x04
    #define REDIS_NODE_API_SEND_OK		0x05
    #define REDIS_NODE_API_ACK			0x06
    #define REDIS_NODE_API_ERROR		0x07
    #define REDIS_NODE_API_NACK			0x08


错误消息类型：

.. code:: c

    #define CASE_ERROR_BASE			                0

    #define CASE_ERROR_OK					(CASE_ERROR_BASE + 0x00)
    #define CASE_ERROR_NO_REQUEST				(CASE_ERROR_BASE + 0x01)
    #define CASE_ERROR_NO_DATA_RECVD			        (CASE_ERROR_BASE + 0x02)
    #define CASE_ERROR_DATA_REQUEST				(CASE_ERROR_BASE + 0x03)
    #define CASE_ERROR_SEND_BUSY				(CASE_ERROR_BASE + 0x04)
    #define CASE_ERROR_DATA_SLOT_ERROR			        (CASE_ERROR_BASE + 0x05)
    #define CASE_ERROR_DISCONNECT				(CASE_ERROR_BASE + 0x06)
    #define CASE_ERROR_NO_DEVICE				(CASE_ERROR_BASE + 0x07)

    #define CASE_ERROR_TX_BUSY					(CASE_ERROR_BASE + 0x40 + 0x00)
    #define CASE_ERROR_TX_TIMEOUT				(CASE_ERROR_BASE + 0x40 + 0x01)
    #define CASE_ERROR_NO_NODE					(CASE_ERROR_BASE + 0x40 + 0x02)

