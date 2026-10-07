### <a name="_b7urdng99y53"></a>**Название задачи: Разработка компонентной архитектуры системы**
### <a name="_hjk0fkfyohdk"></a>**Автор: Владислав Гриша**
### <a name="_uanumrh8zrui"></a>**Дата: 01.10.2026**
### <a name="_3bfxc9a45514"></a>**Функциональные требования**

| **№** | **Действующие лица или системы**                            | **Use Case**                              | **Описание**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
|:-----:|:------------------------------------------------------------|:------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|   1   | Дежурный, Веб-портал, `Controller`, `EventService`, `DataWarehouseRepository` | Просмотр карточки события | 1. UI обращается в `apiGateway`.<br/>2. `apiGateway` маршрутизирует в `Controller`.<br/>3. `Controller` вызывает `EventService.getEventById(id)`.<br/>4. `EventService` обращается к `DataWarehouseRepository.findById(id)`.<br/>5. `DataWarehouseRepository` читает данные из `dataWarehouse` (SQL).<br/>6. `EventService` преобразует `Event` в `EventDto`.<br/>7. `Controller` возвращает `EventDto` в UI. |
|   2   | Дежурный, Веб-портал, `Controller`, `EventService`, `EventPublisher` | Создание инцидента вручную | 1. Дежурный нажимает «Создать инцидент» в UI.<br/>2. UI обращается в `apiGateway`.<br/>3. `apiGateway` маршрутизирует в `Controller`.<br/>4. `Controller` вызывает `EventService.createEvent(request)`.<br/>5. `EventService` валидирует запрос.<br/>6. `EventService` сохраняет событие через `DataWarehouseRepository.save(event)`.<br/>7. `EventService` публикует событие через `EventPublisher.publishEventCreated(event)`.<br/>8. `EventPublisher` отправляет событие в `rabbitMQ` (AMQP).<br/>9. `Controller` возвращает `EventDto` в UI. |
|   3   | Зоотехник, Мобильное приложение, `Controller`, `CameraService`, `VideoIngestClient` | Просмотр списка камер | 1. UI обращается в `apiGateway`.<br/>2. `apiGateway` маршрутизирует в `Controller`.<br/>3. `Controller` вызывает `CameraService.getAllCameras()`.<br/>4. `CameraService` обращается к `VideoIngestClient.fetchAllCameras()`.<br/>5. `VideoIngestClient` вызывает `videoIngest` по REST API.<br/>6. `CameraService` преобразует `Camera` в `CameraDto`.<br/>7. `Controller` возвращает список `CameraDto` в UI. |
|   4   | Зоотехник, Мобильное приложение, `Controller`, `CameraService`, `VideoIngestClient` | Получение ссылки на видеопоток | 1. UI обращается в `apiGateway`.<br/>2. `apiGateway` маршрутизирует в `Controller`.<br/>3. `Controller` вызывает `CameraService.getCameraStream(id)`.<br/>4. `CameraService` запрашивает URL у `VideoIngestClient.fetchStreamUrl(id)`.<br/>5. `VideoIngestClient` возвращает `StreamInfoDto` с URL и протоколом.<br/>6. UI получает ссылку и открывает медиаканал.<br/>7. Видео идёт напрямую с камеры в UI, минуя `coreApi`. |
|   5   | Директор хозяйства, Веб-портал, `Controller`, `ErpService`, `ErpSystemClient` | Просмотр финансового отчёта | 1. Директор открывает отчёт в UI.<br/>2. UI обращается в `apiGateway`.<br/>3. `apiGateway` маршрутизирует в `Controller`.<br/>4. `Controller` вызывает `ErpService.getFinanceReport(from, to)`.<br/>5. `ErpService` запрашивает данные через `ErpSystemClient.fetchFinanceReport(from, to)`.<br/>6. `ErpSystemClient` вызывает `erpSystem` по REST API.<br/>7. `ErpService` преобразует `ErpFinanceReport` в `FinanceReportDto`.<br/>8. `Controller` возвращает `FinanceReportDto` в UI. |

### <a name="_u8xz25hbrgql"></a>**Нефункциональные требования**

| **№** | **Требование**                                                            |
|:-----:|:--------------------------------------------------------------------------|
|   1   | Система мониторинга должна обеспечивать высокую отказоустойчивость 95,95% |
|   2   | Система должна позволять добавлять новый функционал без изменения существующего |
|   3   | От момента фиксации нештатной ситуации видеоаналитикой до оповещения дежурного — не более 5 секунд |
|   4   | Система видеоаналитики должна обрабатывать видеопоток в реальном времени с задержкой в миллисекундах |
|   5   | Синхронные запросы UI не должны блокироваться асинхронными операциями |
|   6   | Публикация событий в брокер не должна задерживать ответ UI |

### <a name="_qmphm5d6rvi3"></a>**Решение** ---> [Компонентная диаграмма `coreApi`](./component-coreApi.md)

| **Контейнер** | **Компонент** | **Тип** | **Назначение** |
|:--------------|:--------------|:--------|:---------------|
| `coreApi` | `Controller` | Controller | Принимает HTTP-запросы от `apiGateway` |
| `coreApi` | `AnimalService` | Service | Бизнес-логика по животным |
| `coreApi` | `CameraService` | Service | Бизнес-логика по камерам |
| `coreApi` | `EventService` | Service | Бизнес-логика по событиям и инцидентам |
| `coreApi` | `ErpService` | Service | Бизнес-логика по интеграции с ERP |
| `coreApi` | `DataWarehouseRepository` | Repository | Чтение данных из Data Warehouse |
| `coreApi` | `VideoIngestClient` | Client | REST-клиент к Video Ingest |
| `coreApi` | `ErpSystemClient` | Client | REST-клиент к ERP-системе |
| `coreApi` | `EventPublisher` | Publisher | Публикация событий в RabbitMQ |
| `aiAdapter` | `FrameConsumer` | Consumer | Читает кадры из RabbitMQ |
| `aiAdapter` | `FrameProcessor` | Processor | Подготовка кадров (ресайз, нормализация) |
| `aiAdapter` | `AiClient` | Client | gRPC-клиент к внешнему AI |
| `aiAdapter` | `ResultHandler` | Handler | Обработка результатов распознавания |
| `aiAdapter` | `AiEventPublisher` | Publisher | Публикация событий от AI в RabbitMQ |

### <a name="_bjrr7veeh80c"></a>**Альтернативы** ---→ [Компонентная диаграмма `vmsClient`](./component-vmsClient.md)

| **№** | **Альтернатива** | **Недостатки**                         |
|:-----:|:-----------------|:---------------------------------------|
|   1   | Единый `CoreService` без разделения по доменам | God Object, сложно тестировать и расширять |
|   2   | Все внешние вызовы из `Controller` | Смешение слоёв, нет изоляции интеграций |
|   3   | `AiClient` как отдельный контейнер | Лишний сетевой хоп внутри `agroSystem` |

**Недостатки, ограничения, риски**

| **№** | **Решение**                                                       | **Недостатки**                                                                    | **Ограничения**                                                               | **Риски**                                                                                                                                                                        |
|:-----:|:------------------------------------------------------------------|:----------------------------------------------------------------------------------|:------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|   1   | `Controller` (Spring MVC)                                         | Точка входа всех запросов                                                         | Зависит от `apiGateway`                                                       | При падении `Controller` — UI недоступен                                                                                                                                         |
|   2   | `AnimalService` / `CameraService` / `EventService` / `ErpService` | Дублирование кода маппинга DTO                                                    | Зависят от `DataWarehouseRepository`, клиентов и `EventPublisher`             | При добавлении домена — нужен новый сервис                                                                                                                                       |
|   3   | `DataWarehouseRepository` (Spring Data JDBC)                      | Не поддерживает JPA-транзакции                                                    | Зависит от ClickHouse                                                          | При недоступности `dataWarehouse` — UI не работает                                                                                                                               |
|   4   | `VideoIngestClient` (Spring WebClient)                            | Зависит от стабильности `videoIngest`                                             | Ограничен rate-limit `videoIngest`                                            | При недоступности `videoIngest` — нет данных о камерах                                                                                                                           |
|   5   | `ErpSystemClient` (Spring WebClient)                              | Зависит от стабильности ERP                                                       | Ограничен API ERP (1С)                                                        | При недоступности ERP — нет финансовых данных                                                                                                                                    |
|   6   | `EventPublisher` (Spring AMQP)                                    | Публикация событий в брокер                                                       | Зависит от `rabbitMQ`                                                          | При перегрузке брокера — события в DLQ                                                                                                                                           |
|   7   | `FrameConsumer` (Go, AMQP)                                        | Зависит от стабильности брокера                                                   | Пропускная способность RabbitMQ                                                | При падении брокера — кадры не читаются                                                                                                                                          |
|   8   | `FrameProcessor` (Go)                                             | CPU-нагрузка на подготовку кадров                                                 | Ограничен ресурсами edge                                                      | При перегрузке — задержка в обработке                                                                                                                                            |
|   9   | `AiClient` (Go, gRPC)                                             | Зависит от внешнего AI                                                            | Ограничен квотами AI-сервиса                                                   | При падении `aiTracking` — события не формируются                                                                                                                                |
|   10  | `ResultHandler` (Go)                                              | Зависит от формата ответа AI                                                      | Ограничен схемой ответа                                                        | При изменении API AI — требуется правка                                                                                     |
|   11  | `AiEventPublisher` (Go, AMQP)                                     | Дублирует `EventPublisher` из `coreApi`                                           | Зависит от `rabbitMQ`                                                          | При падении — события AI не публикуются                                                                                                                                          |