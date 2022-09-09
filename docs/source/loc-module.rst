RSSI定位功能
=================================

.. toctree::
   :maxdepth: 2
   :caption: Contents:
   
=======================
1. LPWDS支持基于RSSI的定位功能：
=======================

#. LPWDS带定位功能的节点模块，会定期向外发送定位信标
#. LPWDS的的带定位功能的接入点模块负责接收带定位功能的节点模块

======================
2. 接入点模块配置详解
======================

   订阅REDIS的pipe.ap.message来获取基站模块的返回消息

1. 当LPWDS接入点模块接收到LPWDS节点的定位信标后会受到下面的消息
   
.. code:: bash

    1) "pmessage"
    2) "*"
    3) "pipe.ap.message"
    4) "CF3BB2DA26BE9F1AA3AE967E020F0E10"
    解析：
        CF3BB2DA26BE9F1AA3AE967E020F0E10
        CF 节点定位beacon消息
        3BB2DA26BE9F 接入点MAC地址
                    1AA3AE967E02 节点MAC地址
                                0F 节点的RSSI值，0x0F，换算成10进制是15