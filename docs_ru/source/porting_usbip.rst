USB IP: Ошибки и корректировки
==============================

В этом разделе основное внимание уделяется различиям и корректировкам в уже поддерживаемых USB IP у различных производителей. Дополнения приветствуются.

FSDEV
--------------------------

FSDEV поддерживает только устройства-подчиненные. Этот IP у разных производителей, как правило, основан на стандартных USB регистрах. 
Поэтому, при использовании, пользователям нужно изменить только `USBD_IRQHandler`, `USB_BASE` и `USB_NUM_BIDIR_ENDPOINTS`. 
Некоторые чипы также могут требовать конфигурации значения `PMA_ACCESS`, по умолчанию равного 2.

.. list-table::
    :widths: 30 20 30 30 30
    :header-rows: 1

    * - Чип
      - USBD_IRQHandler
      - USB_BASE
      - USB_NUM_BIDIR_ENDPOINTS
      - PMA_ACCESS
    * - STM32F0
      - USB_IRQHandler
      - 0x40005C00
      - 8
      - 1
    * - STM32F1
      - USB_LP_CAN1_RX0_IRQHandler
      - То же
      - То же
      - То же
    * - STM32F3
      - USB_LP_CAN_RX0_IRQHandler
      - То же
      - То же
      - 1 или 2
    * - STM32L0
      - USB_IRQHandler
      - То же
      - То же
      - 1
    * - STM32L1
      - USB_LP_IRQHandler
      - То же
      - То же
      - 2
    * - STM32L4
      - USB_IRQHandler
      - То же
      - То же
      - 1

MUSB
--------------------------

IP MUSB поддерживает как хосты, так и устройства-подчиненные. Mentor определил стандартный набор смещений регистров. 
Если ваше устройство соответствует этому стандарту, 
вам нужно будет изменить только `USBD_IRQHandler`, `USB_BASE`, `USB_NUM_BIDIR_ENDPOINTS` для устройства-подчиненного и `USBH_IRQHandler`, `USB_BASE`, `CONIFG_USB_MUSB_EP_NUM` для хоста. 
Если устройство не соответствует стандарту, вам потребуется реализовать следующие макросы смещений, как показано в примере:

.. code-block:: C

    #define MUSB_FADDR_OFFSET 0x00
    #define MUSB_POWER_OFFSET 0x01
    #define MUSB_TXIS_OFFSET  0x02
    #define MUSB_RXIS_OFFSET  0x04
    #define MUSB_TXIE_OFFSET  0x06
    #define MUSB_RXIE_OFFSET  0x08
    #define MUSB_IS_OFFSET    0x0A
    #define MUSB_IE_OFFSET    0x0B

    #define MUSB_EPIDX_OFFSET 0x0E

    #define MUSB_IND_TXMAP_OFFSET      0x10
    #define MUSB_IND_TXCSRL_OFFSET     0x12
    #define MUSB_IND_TXCSRH_OFFSET     0x13
    #define MUSB_IND_RXMAP_OFFSET      0x14
    #define MUSB_IND_RXCSRL_OFFSET     0x16
    #define MUSB_IND_RXCSRH_OFFSET     0x17
    #define MUSB_IND_RXCOUNT_OFFSET    0x18
    #define MUSB_IND_TXTYPE_OFFSET     0x1A
    #define MUSB_IND_TXINTERVAL_OFFSET 0x1B
    #define MUSB_IND_RXTYPE_OFFSET     0x1C
    #define MUSB_IND_RXINTERVAL_OFFSET 0x1D

    #define MUSB_FIFO_OFFSET 0x20

    #define MUSB_DEVCTL_OFFSET 0x60

    #define MUSB_TXFIFOSZ_OFFSET  0x62
    #define MUSB_RXFIFOSZ_OFFSET  0x63
    #define MUSB_TXFIFOADD_OFFSET 0x64
    #define MUSB_RXFIFOADD_OFFSET 0x66

    #define MUSB_TXFUNCADDR0_OFFSET 0x80
    #define MUSB_TXHUBADDR0_OFFSET  0x82
    #define MUSB_TXHUBPORT0_OFFSET  0x83
    #define MUSB_TXFUNCADDRx_OFFSET 0x88
    #define MUSB_TXHUBADDRx_OFFSET  0x8A
    #define MUSB_TXHUBPORTx_OFFSET  0x8B
    #define MUSB_RXFUNCADDRx_OFFSET 0x8C
    #define MUSB_RXHUBADDRx_OFFSET  0x8E
    #define MUSB_RXHUBPORTx_OFFSET  0x8F

    #define USB_TXADDR_BASE(ep_idx)    (USB_BASE + MUSB_TXFUNCADDR0_OFFSET + 0x8 * ep_idx)
    #define USB_TXHUBADDR_BASE(ep_idx) (USB_BASE + MUSB_TXFUNCADDR0_OFFSET + 0x8 * ep_idx + 2)
    #define USB_TXHUBPORT_BASE(ep_idx) (USB_BASE + MUSB_TXFUNCADDR0_OFFSET + 0x8 * ep_idx + 3)
    #define USB_RXADDR_BASE(ep_idx)    (USB_BASE + MUSB_TXFUNCADDR0_OFFSET + 0x8 * ep_idx + 4)
    #define USB_RXHUBADDR_BASE(ep_idx) (USB_BASE + MUSB_TXFUNCADDR0_OFFSET + 0x8 * ep_idx + 6)
    #define USB_RXHUBPORT_BASE(ep_idx) (USB_BASE + MUSB_TXFUNCADDR0_OFFSET + 0x8 * ep_idx + 7)

Специфические значения макросов для различных чипов-подчиненных:

.. list-table::
    :widths: 30 30 30 30
    :header-rows: 1

    * - Чип
      - USBD_IRQHandler
      - USB_BASE
      - USB_NUM_BIDIR_ENDPOINTS
    * - ES32F3xx
      - USB_INT_Handler
      - 0x40086400
      - 5
    * - MSP432Ex
      - То же
      - 0x40050000
      - То же
    * - F1C100S
      - USB_INT_Handler
      - 0x01c13000
      - 4

Специфические значения макросов для различных хост-чипов:

.. list-table::
    :widths: 30 30 30 30
    :header-rows: 1

    * - Чип
      - USBH_IRQHandler
      - USB_BASE
      - CONIFG_USB_MUSB_EP_NUM
    * - ES32F3xx
      - USB_INT_Handler
      - 0x40086400
      - 5
    * - MSP432Ex
      - То же
      - 0x40050000
      - То же
    * - F1C100S
      - USB_INT_Handler
      - 0x01c13000
      - 4

DWC2
--------------------------

DWC2 IP поддерживает и хосты, и устройства-подчиненные. **synopsys** определил стандартный набор смещений регистров. 
Большинство производителей используют эти стандартные смещения, поэтому для устройств-подчиненных 
потребуется изменить только `USBD_IRQHandler`, `USB_BASE`, `USB_NUM_BIDIR_ENDPOINTS`, а для хостов — `USBH_IRQHandler`, `USB_BASE`.

Кроме того, следует обратить внимание на VBUS SENSING, который также может влиять на нормальную инициализацию USB. 
Как его изменить, можно посмотреть здесь: `GD32 dwc2 драйвер и его совместимость с GCCFG_NOVBUSSENS регистром в STM32 <https://github.com/sakumisu/CherryUSB/issues/64>`_.

.. caution:: Поддержка хостов ограничивается dwc2 IP с высокоскоростными возможностями, так как он поддерживает режим DMA. 
Если производитель приобрел IP без поддержки DMA, его использовать невозможно.

Специфические значения макросов для различных чипов-подчиненных:

.. list-table::
    :widths: 30 30 30 30
    :header-rows: 1

    * - Чип
      - USBH_IRQHandler
      - USB_BASE
      - USB_NUM_BIDIR_ENDPOINTS
    * - STM32, кроме H7
      - OTG_FS_IRQHandler/OTG_HS_IRQHandler
      - 0x50000000UL/0x40040000UL
      - 5
    * - STM32 H7
      - То же
      - 0x40080000UL/0x40040000UL
      - 9

Специфические значения макросов для различных хост-чипов:

.. list-table::
    :widths: 30 30 30 30
    :header-rows: 1

    * - Чип
      - USBH_IRQHandler
      - USB_BASE
      - CONFIG_USB_DWC2_PIPE_NUM
    * - Все серии STM32
      - OTG_HS_IRQHandler
      - 0x40040000UL
      - 12

EHCI
--------------------------

EHCI — это стандартный интерфейс хост-контроллера, разработанный Intel. 
Все производители должны реализовать в EHCI определенные регистры и их функции. Макросы для конфигурации EHCI следующие:

.. code-block:: C

  //Базовый адрес регистра возможностей хост-контроллера
  #define CONFIG_USB_EHCI_HCCR_BASE (0xxx)
  //Базовый адрес операционного регистра хост-контроллера
  #define CONFIG_USB_EHCI_HCOR_BASE (0xxx)
  //Включить ли вывод информации о конфигурации EHCI
  #define CONFIG_USB_EHCI_INFO_ENABLE
  //Отключить ли зарезервированные регистры, по умолчанию 9 двойных слов
  #define CONFIG_USB_ECHI_HCOR_RESERVED_DISABLE
  //Включить ли bit0 в регистре configflag
  #define CONFIG_USB_EHCI_CONFIGFLAG
  //Включить ли бит питания порта
  #define CONFIG_USB_EHCI_PORT_POWER

Поскольку EHCI — это только хост-контроллер, обычно он используется вместе с устройством-контроллером и контроллером OTG. 
Скорость обычно определяется в регистрах OTG, поэтому пользователям нужно реализовать функцию `usbh_get_port_speed`.
