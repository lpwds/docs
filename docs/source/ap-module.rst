接入点模块
=================================

.. toctree::
   :maxdepth: 2
   :caption: Contents:
   
=======================
1. 接入点模块主要功能：
=======================

#. 负责节点入网的授权
#. 负责节点高速数据传输

======================
2. 基站模块的配置项
======================
.. code:: c

    #define PIPE_AP_UART_RESET_CMD                          0xC9
    #define PIPE_AP_UART_READ_HW                            0xCA
    #define PIPE_AP_UART_READ_SW                            0xCB
    #define PIPE_AP_UART_SAVE                               0xCC
    #define PIPE_AP_UART_SET_MAC                            0xCD


======================
3. 接入点模块配置简介
======================
    基站模块的配置是通过REDIS数据库的pubsub来实现的。  
    
具体使用方法如下:

1. 配置基站块的指令下发：使用publish指令，topic：pipe.ap.command
2. 基站模块的的消息返回：订阅REDIS的pipe.ap.message来获取基站模块的返回消息


======================
4. 接入点模块配置详解
======================

   以下实例均以‘3E7845A5F7A2’基站模块为例

1. 硬件版本读取 0xCA
   
.. code:: bash

    127.0.0.1:6379> publish pipe.ap.command 3E7845A5F7A2CA0000
    (integer) 2

    #redis-cli 订阅pipe.ap.message返回：
    1) "pmessage"
    2) "*"
    3) "pipe.ap.message"
    4) "CA3E7845A5F7A2004B4C4D5F4E524635323833325F56310F9B"

    #CA3E7845A5F7A2004B4C4D5F4E524635323833325F56310F9B 基站返回消息内容
    #CA 消息指令
    #  3E7845A5F7A2 接入点模块的mac地址
    #              00 消息返回结果： 00 代表OK，01代表ERROR
    #                4B4C4D5F4E524635323833325F5631 接入点硬件版本 => KLM_NRF52832_V1
    #                             
    #指令末尾的0000 补偿长度使用无实际意义