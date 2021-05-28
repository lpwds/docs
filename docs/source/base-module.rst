基站模块
================================

.. toctree::
   :maxdepth: 2
   :caption: Contents:

=======================
1. 基站模块的主要功能：
=======================

#. 建立和维护低功耗连接
#. 为节点分配跳频表
#. 为节点分配时隙

基站模块是整个LPWDS网络中的核心设施。基站模块和LPWDS服务程序使用串口进行通信。
同时基站模块的配置也是通过LPWDS服务程序来来进行配置的。
LPWDS服务程序使用REDIS数据库作为其对外接口。任何对LPWDS的配置，包含LPWDS的服务程序，LPWDS基站，
LPWDS接入点和LPWDS节点均使用通过操作REDIS数据库来实现的。
REDIS数据库的使用方法见：https://www.runoob.com/redis/redis-tutorial.html


===================
2. 基站模块的配置项
===================
.. code:: c

   #define CELL_MAC_PROFILE_BASE_SET_SAVE          0x90
   #define CELL_MAC_PROFILE_BASE_SET_PANID         0x91
   #define CELL_MAC_PROFILE_BASE_SET_KEY           0x92
   //start 以下设置指令不建议用户直接使用。如有需求请联系厂商
   #define CELL_MAC_PROFILE_BASE_SET_SLOT1         0x93
   #define CELL_MAC_PROFILE_BASE_SET_SLOT2         0x94
   #define CELL_MAC_PROFILE_BASE_SET_MAC           0x95
   #define CELL_MAC_PROFILE_BASE_SET_POLL          0x96
   //end
   #define CELL_MAC_PROFILE_BASE_GET_TICK          0x97
   #define CELL_MAC_PROFILE_BASE_SET_BEACON        0x98
   #define CELL_MAC_PROFILE_BASE_GET_PANID         0x99
   #define CELL_MAC_PROFILE_BASE_GET_HW            0x9A
   #define CELL_MAC_PROFILE_BASE_GET_SW            0x9B
   #define CELL_MAC_PROFILE_BASE_GET_KEY           0x9C


===================
3. 基站模块配置简介
===================
基站模块的配置是通过REDIS数据库的pubsub来实现的。  

具体使用方法如下:

1. 配置基站块的指令下发：使用publish指令，topic：pipe.base.command
2. 基站模块的的消息返回：订阅REDIS的pipe.base.message来获取基站模块的返回消息


===================
4. 基站模块配置详解
===================

   以下实例均以‘C23380B93456’基站模块为例

1. 重启指令 0x82
   
.. code:: bash

   127.0.0.1:6379> publish pipe.base.command C23380B93456820000

   #该消息没有返回
   #指令末尾的0000 补偿长度使用无实际意义

2. 配置保存执行 0x90

   任何对基站模块的修改均不是立即生效的，其生效过程： 修改配置->保存配置->重启。 3个过程依次完成方可生效。

.. code:: bash

   127.0.0.1:6379> publish pipe.base.command C23380B93456900000

3. 获取PANID 0x99

.. code:: bash

   127.0.0.1:6379> publish pipe.base.command C23380B93456990000

   #redis-cli 订阅pipe.base.message返回：
   1) "pmessage"
   2) "*"
   3) "pipe.base.message"
   4) "99C23380B9345600123600F00040000891DF"
   
   #99C23380B9345600123600F00040000891DF 基站返回消息内容
   #99 消息指令
   #  C23380B93456 基站模块的mac地址
   #              00 消息返回结果： 00 代表OK，01代表ERROR
   #                1236 基站模块的PANID
   #                    00F0 基站模块的DS
   #                        0040 基站模块的RS
   #                             预留参数
   #                             

4. 设置PANID 0x91
   
.. code:: bash

   127.0.0.1:6379> publish pipe.base.command C23380B93456910248
   (integer) 2

   #redis-cli 订阅pipe.base.message返回：
   1) "pmessage"
   2) "*"
   3) "pipe.base.message"
   4) "91C23380B9345600DC48"

   #91C23380B9345600DC48 基站返回消息内容
   #91 消息指令
   #  C23380B93456 基站模块的mac地址
   #              00 消息返回结果： 00 代表OK，01代表ERROR
   #              

5. 设置KEY 0x92

.. code:: bash

   127.0.0.1:6379> publish pipe.base.command C23380B9345692000102030405060708090A0B0C0D0E0F
   (integer) 2

   #redis-cli 订阅pipe.base.message返回：
   1) "pmessage"
   2) "*"
   3) "pipe.base.message"
   4) "92C23380B9345600143D"

   #92C23380B9345600DC48 基站返回消息内容
   #92 消息指令
   #  C23380B93456 基站模块的mac地址
   #              00 消息返回结果： 00 代表OK，01代表ERROR
   #  

6. 获取硬件版本 0x9A

.. code:: bash

   127.0.0.1:6379> publish pipe.base.command C23380B934569A0000
   (integer) 2

   #redis-cli 订阅pipe.base.message返回：
   1) "pmessage"
   2) "*"
   3) "pipe.base.message"
   4) "9AC23380B93456004B4C4D5F4E524635323833325F56312C28"

   #9AC23380B93456004B4C4D5F4E524635323833325F56312C28 基站返回消息内容
   #9A 消息指令
   #  C23380B93456 基站模块的mac地址
   #              00 消息返回结果： 00 代表OK，01代表ERROR
   #                4B4C4D5F4E524635323833325F5631 >> KLM_NRF52832_V1


7. 获取固件版本 0x9B

.. code:: bash

   127.0.0.1:6379> publish pipe.base.command C23380B934569B0000
   (integer) 2

   #redis-cli 订阅pipe.base.message返回：
   1) "pmessage"
   2) "*"
   3) "pipe.base.message"
   4) "9BC23380B934560076302E352E354E8D1E"

   #9BC23380B934560076302E352E354E8D1E 基站返回消息内容
   #9B 消息指令
   #  C23380B93456 基站模块的mac地址
   #              00 消息返回结果： 00 代表OK，01代表ERROR
   #                76302E352E354E >> v0.5.5N


8. 获取KEY 0x9C

.. code:: bash

   127.0.0.1:6379> publish pipe.base.command A3399DCD5CE89C0000
   (integer) 2

   #redis-cli 订阅pipe.base.message返回：
   1) "pmessage"
   2) "*"
   3) "pipe.base.message"
   4) "9CA3399DCD5CE800000102030405060708090A0B0C0D0E0F00340B"

   #9CA3399DCD5CE800000102030405060708090A0B0C0D0E0F00340B 基站返回消息内容
   #9C 消息指令
   #  A3399DCD5CE8 基站模块的mac地址
   #              00 消息返回结果： 00 代表OK，01代表ERROR
   #                000102030405060708090A0B0C0D0E0F 基站模块密钥
