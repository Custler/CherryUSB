USB Двухканальный микрофон и динамик
============================

Реализация программного обеспечения
------------

Подробный код можно найти в `demo/audio_v1_mic_speaker_multichan_template.c`

.. code-block:: C

    usbd_desc_register(audio_descriptor);
    usbd_add_interface(usbd_audio_alloc_intf());
    usbd_add_interface(usbd_audio_alloc_intf());
    usbd_add_interface(usbd_audio_alloc_intf());
    usbd_add_endpoint(&audio_in_ep);
    usbd_add_endpoint(&audio_out_ep);

    usbd_audio_add_entity(0x02, AUDIO_CONTROL_FEATURE_UNIT);
    usbd_audio_add_entity(0x05, AUDIO_CONTROL_FEATURE_UNIT);

    usbd_initialize();

- Вызов функции `audio_init` для конфигурации дескриптора аудио и инициализации аппаратного обеспечения USB.
- Поскольку для микрофона, динамика и управления требуется 3 интерфейса, нам нужно вызвать функцию `usbd_add_interface` три раза.
- В дескрипторе по умолчанию включены управление громкостью и отключение звука, поэтому нужно зарегистрировать соответствующие сущности, используя `usbd_audio_add_entity`.

.. code-block:: C

    void usbd_audio_open(uint8_t intf)
    {
    }
    void usbd_audio_close(uint8_t intf)
    {
    }

- Когда мы открываем значок громкости на ПК или интерфейс музыкального плеера или микрофона, эти два интерфейса будут вызваны для запуска или остановки передачи данных.

.. code-block:: C

    usbd_ep_start_write(AUDIO_IN_EP, write_buffer, 2048);

- Поскольку в аудиопротоколе нет протоколов прикладного уровня, передаются только исходные данные аудио, поэтому достаточно просто вызвать `usbd_ep_start_write`. После завершения отправки будет вызван прерывание завершения.
- Поскольку динамик требует использования выходного конечного пункта, необходимо запустить первый прием в `usbd_configure_done_callback`. Конечно, если нет возможности принять данные, можно не запускать и запустить, когда это будет необходимо.
