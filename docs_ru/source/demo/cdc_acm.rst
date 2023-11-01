USB Виртуальный COM-порт (без функций UART)
============================================

USB виртуальный COM-порт реализуется с помощью класса USB CDC ACM и симулирует устройство VCP. Когда оно подключено к компьютеру, оно отображается как устройство последовательного порта. Отличие от модулей USB2TTL на рынке заключается в том, что виртуальный порт использует только USB и не взаимодействует с UART (периферийным устройством).

Реализация на программном уровне
-------------------------------

Для просмотра полного кода обратитесь к `demo/cdc_acm_template.c`

.. code-block:: C

    usbd_desc_register(cdc_descriptor);
    usbd_add_interface(usbd_cdc_acm_alloc_intf());
    usbd_add_interface(usbd_cdc_acm_alloc_intf());
    usbd_add_endpoint(&cdc_out_ep);
    usbd_add_endpoint(&cdc_in_ep);
    usbd_initialize();

- Вызов `cdc_acm_init` конфигурирует дескриптор cdc acm и инициализирует аппаратное обеспечение USB.
- Поскольку у cdc есть 2 интерфейса, нам нужно вызвать `usbd_add_interface` дважды.

.. code-block:: C

    void usbd_configure_done_callback(void)
    {
        /* установка первого OUT EP для чтения */
        usbd_ep_start_read(CDC_OUT_EP, read_buffer, 2048);
    }

    void usbd_cdc_acm_bulk_out(uint8_t ep, uint32_t nbytes)
    {
        USB_LOG_RAW("фактическая длина OUT: %d\r\n", nbytes);
        /* установка следующего OUT EP для чтения */
        usbd_ep_start_read(CDC_OUT_EP, read_buffer, 2048);
    }

    void usbd_cdc_acm_bulk_in(uint8_t ep, uint32_t nbytes)
    {
        USB_LOG_RAW("фактическая длина IN: %d\r\n", nbytes);

        if ((nbytes % CDC_MAX_MPS) == 0 && nbytes) {
            /* отправка ZLP */
            usbd_ep_start_write(CDC_IN_EP, NULL, 0);
        } else {
            ep_tx_busy_flag = false;
        }
    }

    void usbd_cdc_acm_set_dtr(uint8_t intf, bool dtr)
    {
        if (dtr) {
            dtr_enable = 1;
        } else {
            dtr_enable = 0;
        }
    }

    void cdc_acm_data_send_with_dtr_test(void)
    {
        if (dtr_enable) {
            memset(&write_buffer[10], 'a', 2038);
            ep_tx_busy_flag = true;
            usbd_ep_start_write(CDC_IN_EP, write_buffer, 2048);
            while (ep_tx_busy_flag) {
            }
        }
    }

- Функция `usbd_cdc_acm_set_dtr` вызывается при получении команды управления потоком от хоста, здесь мы используем dtr. При включении dtr начинается отправка.
- `usbd_configure_done_callback` - это callback функция, вызываемая после завершения перечисления. Поскольку у cdc acm есть OUT конечная точка, мы должны начать первый прием данных здесь. **Длина данных должна быть кратной максимальной длине пакета**.
- `usbd_cdc_acm_bulk_out` - это callback функция, вызываемая при завершении приема, здесь мы начинаем следующий прием.
- `usbd_cdc_acm_bulk_in` - это callback функция, вызываемая при завершении отправки, здесь мы проверяем, является ли длина отправки кратной максимальной длине пакета. Если это так, нужно отправить ZLP для обозначения окончания передачи.
- Вызов `usbd_ep_start_write` выполняет отправку. Обратите внимание, если возвращаемое значение меньше 0, нельзя выполнять следующий while.
