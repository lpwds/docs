节点模块
=================================

.. toctree::
   :maxdepth: 2
   :caption: Contents:


===============
1. 节点模块简介
===============
节点模块和基站模块、接入点模块不同的是：节点模块提供SDK用户可使用SDK进行SOC开发。
基站模块和接入点模块则是由厂家提供的标准品，不可进行定制开发。

节点模块可以使用厂家提供的协议栈和SDK进行SOC开发。
具体实现方式：

#. 协议栈烧录
    #. 协议栈的烧录必须使用厂家提供的协议栈烧录工具进行烧录。
    #. 烧录完协议栈之后方可进行SOC开发
#. SDK 有厂家提供用于SOC开发


===============
2. SDK开发文档
===============

1. 配置项

.. code:: c

    struct case_config_t
    {
        uint8_t  addr[6];    //节点模块的MAC地址
        uint16_t panid;     //节点需要连接的网络的PANID，如果设置成0xffff，则有节点模块通过接入点自动获取
        uint16_t extid;     //预留
        
        uint32_t poll_interval;     //内部参数
        uint32_t max_lose_cnt;      //内部参数
        uint32_t scan_interval;     //内部参数
        
        uint8_t key[16];    //节点需要连接的网络的KEY，节点模块可以通过接入点自动获取
        
        uint8_t  key2[16];       //用于鉴权的KEY
        uint8_t  iv2[16];        //用于鉴权的IV
        uint32_t token;         //用于鉴权的TOKEN
        uint32_t adv_mode;      //是否启用鉴权模式：0 不启用， 1 启用
        
        uint32_t multi_mode;    //内部参数
        uint32_t phy_mode;      //射频工作模式
        /*
            phy_mode&0xff => 射频工作模式
            #define PHY_MODE_1MBPS      1
            #define PHY_MODE_125KBPS    4
        */

        /*
            射频管脚使用
            switch((phy_mode >> 16) & 0xff) {
                case 0:
                    __tx_pin = 20;
                    __rx_pin = 19;
                    break;
                
                case 1:
                    __tx_pin = NRF_GPIO_PIN_MAP(1, 4);
                    __rx_pin = NRF_GPIO_PIN_MAP(1, 2);
                    break;
                
                default:
                    __tx_pin = 20;
                    __rx_pin = 19;
                    break;
            }
        */
    };

2. API详解

.. code:: c

    /*
        节点模块基础硬件初始化函数
    */
    uint32_t __svc(0) case_low_init(void);

    /*
        节点模块初始化函数， 必须再调用case_low_init()之后方可调用
    */
    uint32_t __svc(1) case_init(void);

    /*
        节点模块事件读取函数，应用层已经封装好，不建议用户直接使用
    */
    uint32_t __svc(2) case_evt_read(uint8_t *data);

    /*
        节点模块发送函数
        Profile: 需要发送的数据类型
        data: 需要发送的数据，单个数据长度最大18个字节
        len：需要发送数据的长度， 最大18
        ack：需要发送的数据使用需要网关应答， 0 不需要，1 需要
    */
    uint32_t __svc(3) case_write(uint32_t profile, uint8_t *data, uint8_t len, uint8_t ack);

    
    /*
        节点模块高速数据读取函数
        // 5A Len 00 00 xx xx xx xx
    */
    uint32_t __svc(4) case_evt_hs_read(uint8_t *data, uint32_t id);
    /*
        该函数暂未使用
    */
    uint32_t __svc(5) case_evt_hs_ack(uint8_t *data, uint8_t length, uint8_t ack);

    /*
        节点模块高速数据发送缓存区注册函数

        fifo：struct fifo{
                volatile unsigned int in;
                volatile unsigned int out;
                unsigned	 int mask;
                unsigned char *data;
            };  

        fifo大小： 256*N, N>=3
    */
    uint32_t __svc(6) case_evt_hs_ack_fifo_register(void *fifo);

    /*
        节点模块高速晶振自动函数， 
        on：0 高速晶振有协议栈自动控制，1 高速晶振始终工作
    */
    uint32_t __svc(7) case_clock_request(uint32_t on);

    /*
        节点模块配置读取函数
    */
    uint32_t __svc(8) case_config_read(void *config);

    /*
        节点模块配置更新函数
    */
    uint32_t __svc(9) case_config_update(void *config);

    /*
        节点模块RSSI读取函数
    */
    uint32_t __svc(10) case_rssi_read(void);

    /*
        节点模块协议栈版本读取函数
    */
    uint32_t __svc(11) case_version_read(uint8_t *version, uint32_t length);

3. 初始化

.. code:: c
    
    int main(void)
    {   
        
        /*
            节点模块的协议栈使用硬件列表：
            1. NRF_RADIO
            2. NRF_RTC0
            3. NRF_ECB

            用户使用协议栈时不得使用以上硬件
        */
        //节点模块基础硬件初始化
        case_low_init();
        //节点模块配置读取
        case_config_read(&case_info);

        //nRF52840 Normal
        //case_info.phy_mode = 0x00010001;
        
        //nRF52810 Normal
        //case_info.phy_mode = 0x00000001;
        //nRF52840 Long
        case_info.phy_mode = 0x00010004;
        //case_info.panid = 0xffff;
        //节点模块配置更新
        case_config_update(&case_info);
        //节点模块初始化
        case_init();
        //节点Profile初始化
        node_profile_init();
        
        while(1) {
            case_process();
            node_profile_process();
        }
        
    }


4. 开发模板

.. code:: c

    /*
    */
    #include "node_app.h"
    #include "node_auth.h"
    #include "node_uart.h"
    //#include "user_led.h"

    //节点模块收到消息后，LPWDS协议栈会调用该函数， 用户需在这里处理接收到的消息
    void node_app_on_cmd_handler(uint32_t profile, uint8_t *data, uint32_t length)
    {
        node_auth_cmd_process(profile, data, length);
        node_uart_on_cmd_handler(profile, data, length);
        //user_led_cmd_process(profile, data, length);
    }

    //节点模块的beacon周期调用函数， 只有在节点联网的状态下，该函数才会被调用
    void node_app_tick_process(void)
    {
        node_uart_tick_process();
    }

    void node_app_process(void)
    {
        node_uart_process();
    }

    //用户的初始化建议放在这里
    void node_app_init(void)
    {
        node_auth_init();
        node_uart_init();
        //user_led_init();
    }

