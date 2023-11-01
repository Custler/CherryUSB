Протокольный стек устройства
=========================

Протокольный стек устройства в основном отвечает за перечисление и загрузку драйверов. Перечисление здесь не рассматривается, а загрузка драйверов, или, точнее, загрузка драйверов интерфейсов, в основном осуществляется с помощью функции `usbd_add_interface`. Эта функция сохраняет переданный драйвер интерфейса в связном списке. Когда хост делает запрос к классу, можно обратиться к этому списку для доступа.
После вызова `usbd_desc_register` необходимо зарегистрировать интерфейсы и конечные точки. Запомните следующее:

- Сколько интерфейсов, столько раз и вызывается `usbd_add_interface`. В параметрах указываются соответствующие `xxx_init_intf`. Если нет поддерживаемых, создайте вручную и вставьте.
- Сколько конечных точек, столько раз и вызывается `usbd_add_endpoint`. При завершении прерывания будет вызван зарегистрированный обратный вызов для конечной точки.

CORE
-----------------

Структура конечной точки
""""""""""""""""""""""""""""""""""""

Структура конечной точки в основном используется для регистрации обратных вызовов при завершении прерываний для различных адресов конечных точек.

.. code-block:: C

    struct usbd_endpoint {
        uint8_t ep_addr;
        usbd_endpoint_callback ep_cb;
    };

- **list** Узел связного списка конечных точек
- **ep_addr** Адрес конечной точки (с направлением)
- **ep_cb** Обратный вызов при завершении прерывания для конечной точки.

.. note:: Вкратце: функция обратного вызова in эквивалентна функции обратного вызова при завершении отправки DMA; функция обратного вызова out эквивалентна функции обратного вызова при завершении приема DMA.

Структура интерфейса
""""""""""""""""""""""""""""""""""""

Структура интерфейса в основном используется для регистрации различных запросов для различных классов устройств, помимо стандартных запросов к устройству. Это включает в себя запросы к классу устройства, запросы от производителя и пользовательские запросы. А также обратные вызовы для уведомлений в протокольном стеке.

.. code-block:: C

    struct usbd_interface {
        usb_slist_t list;
        usbd_request_handler class_interface_handler;
        usbd_request_handler class_endpoint_handler;
        usbd_request_handler vendor_handler;
        usbd_notify_handler notify_handler;
        const uint8_t *hid_report_descriptor;
        uint32_t hid_report_descriptor_len;
        uint8_t intf_num;
    };

- **list** Узел связного списка интерфейсов
- **class_interface_handler** Обратный вызов для запросов настройки класса, получатель - интерфейс
- **class_endpoint_handler** Обратный вызов для запросов настройки класса, получатель - конечная точка
- **vendor_handler** Обратный вызов для запросов настройки от производителя
- **notify_handler** Обратный вызов для уведомлений о прерываниях и состояниях протокольного стека
- **hid_report_descriptor** Дескриптор отчета HID
- **hid_report_descriptor_len** Длина дескриптора отчета HID
- **intf_num** Текущий номер интерфейса
- **ep_list** Узел связного списка конечных точек

usbd_desc_register
""""""""""""""""""""""""""""""""""""

`usbd_desc_register` используется для регистрации дескрипторов USB. Типы дескрипторов включают: дескриптор устройства, дескриптор конфигурации (включая дескриптор конфигурации, дескриптор интерфейса, дескрипторы класса, дескрипторы конечных точек), дескрипторы строк, дескрипторы ограничений устройства.

.. code-block:: C

    void usbd_desc_register(const uint8_t *desc);

- **desc**  Дескриптор

usbd_msosv1_desc_register
""""""""""""""""""""""""""""""""""""

`usbd_msosv1_desc_register` используется для регистрации дескриптора WINUSB 1.0.

.. code-block:: C

    void usbd_msosv1_desc_register(struct usb_msosv1_descriptor *desc);

- **desc**  Дескриптор

usbd_msosv2_desc_register
""""""""""""""""""""""""""""""""""""

`usbd_msosv2_desc_register` используется для регистрации дескриптора WINUSB 2.0.

.. code-block:: C

    void usbd_msosv2_desc_register(struct usb_msosv2_descriptor *desc);

- **desc**  Дескриптор

usbd_bos_desc_register
""""""""""""""""""""""""""""""""""""

``usbd_bos_desc_register`` используется для регистрации дескриптора BOS. Для версий USB 2.1 и выше регистрация обязательна.

.. code-block:: C

    void usbd_bos_desc_register(struct usb_bos_descriptor *desc);

- **desc**  Дескриптор

usbd_add_interface
""""""""""""""""""""""""""""""""""""

``usbd_add_interface`` добавляет драйвер интерфейса. **Порядок добавления должен соответствовать порядку в дескрипторе**.

.. code-block:: C

    void usbd_add_interface(struct usbd_interface *intf);

- **intf**  Драйвер интерфейса, обычно получается из функции `xxx_init_intf` различных классов

usbd_add_endpoint
""""""""""""""""""""""""""""""""""""

``usbd_add_endpoint`` добавляет функцию обратного вызова для завершения прерывания по конечной точке.

.. code-block:: C

    void usbd_add_endpoint(struct usbd_endpoint *ep);;

- **ep**  Дескриптор конечной точки

usbd_initialize
""""""""""""""""""""""""""""""""""""

``usbd_initialize`` используется для инициализации конфигурации регистров устройства USB, часов USB, прерываний и т.д. Обратите внимание, что эту функцию необходимо вызывать последней из всех перечисленных API. **Если используется ОС, выполнение должно происходить в потоке**.

.. code-block:: C

    int usbd_initialize(void);

usbd_event_handler
""""""""""""""""""""""""""""""""""""

``usbd_event_handler`` — это функция обратного вызова для прерываний или некоторых состояний протокольного стека. Большинство IP поддерживают только USBD_EVENT_RESET и USBD_EVENT_CONFIGURED.

.. code-block:: C

    void usbd_event_handler(uint8_t event);

CDC ACM
-----------------

usbd_cdc_acm_init_intf
""""""""""""""""""""""""""""""""""""

``usbd_cdc_acm_init_intf`` используется для инициализации интерфейса класса USB CDC ACM и реализации соответствующих функций этого интерфейса.

- ``cdc_acm_class_interface_request_handler`` используется для обработки запросов настройки класса USB CDC ACM.
- ``cdc_notify_handler`` используется для обработки других прерываний USB CDC.

.. code-block:: C

    struct usbd_interface *usbd_cdc_acm_init_intf(struct usbd_interface *intf);

- **return**  Дескриптор интерфейса

usbd_cdc_acm_set_line_coding
""""""""""""""""""""""""""""""""""""

``usbd_cdc_acm_set_line_coding`` используется для конфигурации последовательного порта. Если используется только USB, и не используется последовательный порт, этот интерфейс не требуется реализовывать пользователем, используется значение по умолчанию.

.. code-block:: C

    void usbd_cdc_acm_set_line_coding(uint8_t intf, struct cdc_line_coding *line_coding);

- **intf** Номер управляющего интерфейса
- **line_coding** Конфигурация последовательного порта

usbd_cdc_acm_get_line_coding
""""""""""""""""""""""""""""""""""""

``usbd_cdc_acm_get_line_coding`` используется для получения конфигурации последовательного порта. Если используется только USB, и не используется последовательный порт, этот интерфейс не требуется реализовывать пользователем, используется значение по умолчанию.

.. code-block:: C

    void usbd_cdc_acm_get_line_coding(uint8_t intf, struct cdc_line_coding *line_coding);

- **intf** Номер управляющего интерфейса
- **line_coding** Конфигурация последовательного порта

usbd_cdc_acm_set_dtr
""""""""""""""""""""""""""""""""""""

``usbd_cdc_acm_set_dtr`` используется для управления сигналом DTR последовательного порта. Если используется только USB, и не используется последовательный порт, этот интерфейс не требуется реализовывать пользователем, используется значение по умолчанию.

.. code-block:: C

    void usbd_cdc_acm_set_dtr(uint8_t intf, bool dtr);

- **intf** Номер управляющего интерфейса
- **dtr** Значение dtr: 1 означает низкий уровень сигнала, 0 — высокий

usbd_cdc_acm_set_rts
""""""""""""""""""""""""""""""""""""

``usbd_cdc_acm_set_rts`` используется для управления сигналом RTS последовательного порта. Если используется только USB, и не используется последовательный порт, этот интерфейс не требуется реализовывать пользователем, используется значение по умолчанию.

.. code-block:: C

    void usbd_cdc_acm_set_rts(uint8_t intf, bool rts);

- **intf** Номер управляющего интерфейса
- **rts** Значение rts: 1 означает низкий уровень сигнала, 0 — высокий

CDC_ACM_DESCRIPTOR_INIT
""""""""""""""""""""""""""""""""""""

``CDC_ACM_DESCRIPTOR_INIT`` настраивает требуемые для cdc acm дескрипторы и параметры для удобства пользователя. Общая длина равна `CDC_ACM_DESCRIPTOR_LEN`.

.. code-block:: C

    CDC_ACM_DESCRIPTOR_INIT(bFirstInterface, int_ep, out_ep, in_ep, str_idx);

- **bFirstInterface** обозначает смещение первого интерфейса cdc acm среди всех интерфейсов
- **int_ep** обозначает адрес прерывающей конечной точки (с направлением)
- **out_ep** обозначает адрес конечной точки bulk out (с направлением)
- **in_ep** обозначает адрес конечной точки bulk in (с направлением)
- **str_idx** идентификатор строки, соответствующей управляющему интерфейсу

HID
-----------------

usbd_hid_init_intf
""""""""""""""""""""""""""""""""""""

``usbd_hid_init_intf`` используется для инициализации интерфейса класса USB HID и реализации соответствующих функций этого интерфейса:

- ``hid_class_interface_request_handler`` используется для обработки запросов настройки класса USB HID.
- ``hid_notify_handler`` используется для обработки других прерываний USB HID.

.. code-block:: C

    struct usbd_interface *usbd_hid_init_intf(struct usbd_interface *intf, const uint8_t *desc, uint32_t desc_len);

- **desc** дескриптор отчета
- **desc_len** длина дескриптора отчета

MSC
-----------------

usbd_msc_init_intf
""""""""""""""""""""""""""""""""""""

``usbd_msc_init_intf`` используется для инициализации интерфейса класса MSC и реализации соответствующих функций этого интерфейса. Также регистрирует функции обратного вызова для конечных точек. (Поскольку протокол msc bot фиксирован, его не нужно реализовывать, поэтому функции обратного вызова для конечных точек не требуют реализации пользователем).

- ``msc_storage_class_interface_request_handler`` используется для обработки прерывающих запросов настройки USB MSC.
- ``msc_storage_notify_handler`` используется для реализации других прерываний USB MSC.

.. code-block:: C

    struct usbd_interface *usbd_msc_init_intf(struct usbd_interface *intf, const uint8_t out_ep, const uint8_t in_ep);

- **out_ep** адрес конечной точки out
- **in_ep** адрес конечной точки in

usbd_msc_get_cap
""""""""""""""""""""""""""""""""""""

``usbd_msc_get_cap`` используется для получения информации о lun, количестве секторов и размере каждого сектора. Эту функцию необходимо реализовать пользователю.

.. code-block:: C

    void usbd_msc_get_cap(uint8_t lun, uint32_t *block_num, uint16_t *block_size);

- **lun** логическая единица хранения, пока не используется, поддерживается только одна
- **block_num** количество секторов хранения
- **block_size** размер сектора хранения

usbd_msc_sector_read
""""""""""""""""""""""""""""""""""""

``usbd_msc_sector_read`` используется для чтения данных, начиная с определенного сектора хранилища. Эту функцию необходимо реализовать пользователю.

.. code-block:: C

    int usbd_msc_sector_read(uint32_t sector, uint8_t *buffer, uint32_t length);

- **sector** смещение сектора
- **buffer** указатель на буфер для хранения прочитанных данных
- **length** длина чтения, в настоящее время равна размеру одного сектора

usbd_msc_sector_write
""""""""""""""""""""""""""""""""""""

``usbd_msc_sector_write`` используется для записи данных, начиная с определенного сектора хранилища. Эту функцию необходимо реализовать пользователю.

.. code-block:: C

    int usbd_msc_sector_write(uint32_t sector, uint8_t *buffer, uint32_t length);

- **sector** смещение сектора
- **buffer** указатель на буфер с данными для записи
- **length** длина записи, в настоящее время равна размеру одного сектора

UAC
-----------------

usbd_audio_init_intf
""""""""""""""""""""""""""""""""""""

``usbd_audio_init_intf`` используется для инициализации интерфейса класса USB Audio и реализации соответствующих функций этого интерфейса:

- ``audio_class_interface_request_handler`` используется для обработки прерывающих запросов настройки USB Audio на уровне интерфейса.
- ``audio_class_endpoint_request_handler`` используется для обработки прерывающих запросов настройки USB Audio на уровне конечной точки.
- ``audio_notify_handler`` используется для реализации других прерываний USB Audio.

.. code-block:: C

    struct usbd_interface *usbd_audio_init_intf(struct usbd_interface *intf);

- **class** дескриптор класса
- **intf** дескриптор интерфейса

usbd_audio_open
""""""""""""""""""""""""""""""""""""

``usbd_audio_open`` используется для запуска передачи аудиоданных.

.. code-block:: C

    void usbd_audio_open(uint8_t intf);

- **intf** номер интерфейса для запуска

usbd_audio_close
""""""""""""""""""""""""""""""""""""

``usbd_audio_close`` используется для остановки передачи аудиоданных.

.. code-block:: C

    void usbd_audio_close(uint8_t intf);

- **intf** номер интерфейса для остановки

usbd_audio_add_entity
""""""""""""""""""""""""""""""""""""

``usbd_audio_add_entity`` используется для добавления элементов управления, таких как feature unit или clock source.

.. code-block:: C

    void usbd_audio_add_entity(uint8_t entity_id, uint16_t bDescriptorSubtype);

- **entity_id** идентификатор добавляемого элемента управления
- **bDescriptorSubtype** подтип дескриптора для entity_id

usbd_audio_set_mute
""""""""""""""""""""""""""""""""""""

``usbd_audio_set_mute`` используется для установки режима "Тишина".

.. code-block:: C

    void usbd_audio_set_mute(uint8_t ch, uint8_t enable);

- **ch** канал для установки режима "Тишина"
- **enable** 1 для активации режима "Тишина", 0 для деактивации

usbd_audio_set_volume
""""""""""""""""""""""""""""""""""""

``usbd_audio_set_volume`` используется для установки громкости.

.. code-block:: C

    void usbd_audio_set_volume(uint8_t ch, float dB);

- **ch** канал для установки громкости
- **dB** уровень громкости в децибелах, диапазон для UAC1.0 от -127 до +127 dB, для UAC2.0 от 0 до 256 dB

usbd_audio_set_sampling_freq
""""""""""""""""""""""""""""""""""""

``usbd_audio_set_sampling_freq`` используется для установки частоты дискретизации аудио на устройстве.

.. code-block:: C

    void usbd_audio_set_sampling_freq(uint8_t ep_ch, uint32_t sampling_freq);

- **ch** конечная точка или канал для установки частоты дискретизации, для UAC1.0 это конечная точка, для UAC2.0 это канал
- **dB** устанавливаемая частота дискретизации

usbd_audio_get_sampling_freq_table
""""""""""""""""""""""""""""""""""""

``usbd_audio_get_sampling_freq_table`` используется для получения списка поддерживаемых частот дискретизации. Если функция не реализована, используется список частот по умолчанию.

.. code-block:: C

    void usbd_audio_get_sampling_freq_table(uint8_t **sampling_freq_table);

- **sampling_freq_table** адрес таблицы частот дискретизации, формат согласно таблице частот по умолчанию

usbd_audio_set_pitch
""""""""""""""""""""""""""""""""""""

``usbd_audio_set_pitch`` используется для установки высоты тона аудио, функция доступна только в UAC1.0.

.. code-block:: C

    void usbd_audio_set_pitch(uint8_t ep, bool enable);

- **ep** конечная точка для установки высоты тона
- **enable** активация или деактивация установки высоты тона

UVC
-----------------

usbd_video_init_intf
""""""""""""""""""""""""""""""""""""
``usbd_video_init_intf`` используется для инициализации интерфейса класса USB Video и реализации соответствующих функций этого интерфейса:

- ``video_class_interface_request_handler`` используется для обработки прерывающих запросов настройки USB Video.
- ``video_notify_handler`` используется для реализации других прерываний USB Video.

.. code-block:: C

    struct usbd_interface *usbd_video_init_intf(struct usbd_interface *intf,
                                             uint32_t dwFrameInterval,
                                             uint32_t dwMaxVideoFrameSize,
                                             uint32_t dwMaxPayloadTransferSize);

- **class** дескриптор класса
- **intf** дескриптор интерфейса

usbd_video_open
""""""""""""""""""""""""""""""""""""

``usbd_video_open`` используется для запуска передачи видеоданных.

.. code-block:: C

    void usbd_video_open(uint8_t intf);

- **intf** номер интерфейса для запуска

usbd_video_close
""""""""""""""""""""""""""""""""""""

``usbd_video_close`` используется для остановки передачи видеоданных.

.. code-block:: C

    void usbd_video_open(uint8_t intf);

- **intf** номер интерфейса для остановки

usbd_video_mjpeg_payload_fill
""""""""""""""""""""""""""""""""""""

``usbd_video_mjpeg_payload_fill`` используется для заполнения нового буфера данными MJPEG. Данные MJPEG разделяются на фреймы, размер каждого из которых контролируется ``dwMaxPayloadTransferSize``, и к ним добавляется заголовок, текущий размер которого составляет 2 байта. За структуру заголовка отвечает ``struct video_mjpeg_payload_header``.

.. code-block:: C

    uint32_t usbd_video_mjpeg_payload_fill(uint8_t *input, uint32_t input_len, uint8_t *output, uint32_t *out_len);

- **input** пакет данных в формате MJPEG, начинающийся с FFD8 и заканчивающийся FFD9
- **input_len** размер пакета данных MJPEG
- **output** выходной буфер
- **out_len** фактический размер отправляемых данных
- **return** возвращает количество фреймов, которые USB должен отправить в соответствии с ``dwMaxPayloadTransferSize``

DFU
-----------------

PRINTER
-----------------

MTP
-----------------