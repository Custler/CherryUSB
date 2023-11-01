USB Эмуляция U-диска
=========================

Реализация на программном уровне
-------------------------------

Для просмотра полного кода обратитесь к `demo/msc_ram_template.c`

.. code-block:: C

    usbd_desc_register(msc_ram_descriptor);
    usbd_add_interface(usbd_msc_alloc_intf(MSC_OUT_EP, MSC_IN_EP));

    usbd_initialize();

- Вызов `msc_ram_init` конфигурирует дескриптор msc и инициализирует аппаратное обеспечение USB.
- Поскольку у msc есть 1 интерфейс, нам нужно вызвать `usbd_add_interface` один раз.
- Поток данных конечных точек в msc управляется стеком протоколов, поэтому не требуется регистрация пользовательских callback-функций для конечных точек. Аналогично, `usbd_configure_done_callback` также не требуется и может быть оставлен пустым.

.. code-block:: C

    void usbd_msc_get_cap(uint8_t lun, uint32_t *block_num, uint16_t *block_size)
    {
        *block_num = 1000; // Притворимся, что у нас так много буферов, хотя на самом деле их нет.
        *block_size = BLOCK_SIZE;
    }

    int usbd_msc_sector_read(uint32_t sector, uint8_t *buffer, uint32_t length)
    {
        return 0;
    }

    int usbd_msc_sector_write(uint32_t sector, uint8_t *buffer, uint32_t length)
    {
        return 0;
    }

- Реализация трех интерфейсов позволяет использовать msc. Операции чтения и записи выполняются в прерываниях, если не используется операционная система.
- `CONFIG_USBDEV_MSC_BLOCK_SIZE` может быть кратным 512. Изменение этого параметра может увеличить скорость чтения и записи msc, но также потребует больше оперативной памяти.

.. note:: MSC обычно используется совместно с RTOS, поскольку операции чтения и записи являются блокирующими. Помещение их в прерывания неприемлемо. `CONFIG_USBDEV_MSC_THREAD` позволяет включить управление через операционную систему.
