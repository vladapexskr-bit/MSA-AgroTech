### <a name="_b7urdng99y53_code"></a>**Название задачи: Разработка кодовой архитектуры системы (C4Code) — основное и альтернативное решения**
### <a name="_hjk0fkfyohdk_code"></a>**Автор: Владислав Гриша**
### <a name="_uanumrh8zrui_code"></a>**Дата: 01.10.2026**
### <a name="_3bfxc9a45514_code"></a>**Функциональные требования**

| **№** | **Действующие лица или системы**                            | **Use Case**                              | **Описание**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
|:-----:|:------------------------------------------------------------|:------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|   1   | `EventService`, `EventRepository`, `EventPublisher` | Получение карточки события | 1. `Controller` вызывает `EventService.getEventById(id)`.<br/>2. `EventService` вызывает `EventRepository.findById(id)`.<br/>3. `EventRepository` возвращает `Optional<Event>`.<br/>4. `EventService` маппит `Event` в `EventDto` через `mapToDto`.<br/>5. `Controller` возвращает `EventDto` в UI. |
|   2   | `EventService`, `EventRepository`, `EventPublisher` | Создание инцидента вручную | 1. `Controller` вызывает `EventService.createEvent(request)`.<br/>2. `EventService` валидирует `CreateEventRequest`.<br/>3. `EventService` сохраняет `Event` через `EventRepository.save(event)`.<br/>4. `EventService` публикует событие через `EventPublisher.publishEventCreated(event)`.<br/>5. `EventService` возвращает `EventDto` в `Controller`. |
|   3   | `AnimalService`, `AnimalRepository` | Получение списка животных по секции | 1. `Controller` вызывает `AnimalService.getAnimalsBySection(sectionId)`.<br/>2. `AnimalService` вызывает `AnimalRepository.findBySectionId(sectionId)`.<br/>3. `AnimalRepository` возвращает `List<Animal>`.<br/>4. `AnimalService` маппит `Animal` в `AnimalDto` через `mapToDto`.<br/>5. `Controller` возвращает `List<AnimalDto>` в UI. |
|   4   | `CameraService`, `VideoIngestClient` | Получение ссылки на видеопоток | 1. `Controller` вызывает `CameraService.getCameraStream(id)`.<br/>2. `CameraService` вызывает `VideoIngestClient.fetchStreamUrl(cameraId)`.<br/>3. `VideoIngestClient` вызывает REST API `videoIngest`.<br/>4. `VideoIngestClient` возвращает `StreamInfoDto`.<br/>5. `Controller` возвращает `StreamInfoDto` в UI. |
|   5   | `ErpService`, `ErpSystemClient` | Получение финансового отчёта | 1. `Controller` вызывает `ErpService.getFinanceReport(from, to)`.<br/>2. `ErpService` вызывает `ErpSystemClient.fetchFinanceReport(from, to)`.<br/>3. `ErpSystemClient` вызывает REST API `erpSystem`.<br/>4. `ErpService` маппит `ErpFinanceReport` в `FinanceReportDto`.<br/>5. `Controller` возвращает `FinanceReportDto` в UI. |
|   6   | `WebhookController`, `EventFilter`, `EventPublisher` (альтернатива) | Приём события от VMS | 1. VMS отправляет webhook на `WebhookController`.<br/>2. `WebhookController` парсит payload.<br/>3. `EventFilter` определяет приоритет события.<br/>4. `EventFilter` передаёт `FilteredEvent` в `EventPublisher`.<br/>5. `EventPublisher` публикует событие в `rabbitMQ`. |
|   7   | `coreApi`, `ClipClient`, VMS (альтернатива) | Запрос клипа по событию | 1. `coreApi` вызывает `ClipClient.fetchClip(eventId)`.<br/>2. `ClipClient` формирует REST-запрос к VMS.<br/>3. VMS возвращает URL клипа.<br/>4. `ClipClient` возвращает `ClipDto` в `coreApi`. |

### <a name="_u8xz25hbrgql_code"></a>**Нефункциональные требования**

| **№** | **Требование**                                                            |
|:-----:|:--------------------------------------------------------------------------|
|   1   | Система мониторинга должна обеспечивать высокую отказоустойчивость 95,95% |
|   2   | Система должна позволять добавлять новый функционал без изменения существующего |
|   3   | От момента фиксации нештатной ситуации видеоаналитикой до оповещения дежурного — не более 5 секунд |
|   4   | Система видеоаналитики должна обрабатывать видеопоток в реальном времени с задержкой в миллисекундах |
|   5   | Синхронные вызовы `Service → Repository` не должны блокироваться публикацией событий |
|   6   | Клиенты к внешним системам должны быть изолированы от бизнес-логики |
|   7   | Webhook от VMS должен обрабатываться за миллисекунды |
|   8   | Приоритизация событий должна настраиваться без пересборки |

### <a name="_qmphm5d6rvi3_code"></a>**Решение** ---> [C4Code — AnimalService](code_animalService.md)

| **Компонент** | **Класс / Интерфейс** | **Тип** | **Назначение** |
|:--------------|:----------------------|:--------|:---------------|
| `AnimalService` | `AnimalService` | Interface | Контракт: методы работы с животными |
| `AnimalService` | `AnimalServiceImpl` | Class | Реализация бизнес-логики |
| `AnimalService` | `AnimalRepository` | Repository | Доступ к данным о животных |
| `AnimalService` | `Animal` | Entity | Сущность животного |
| `AnimalService` | `AnimalDto` | DTO | Данные для UI |
| `AnimalService` | `AnimalStatus` | Enum | Статусы животного |
| `AnimalService` | `TypeOfAnimal` | Enum | Типы животных |

### <a name="_qmphm5d6rvi3_code_2"></a>**Решение** ---> [C4Code — EventService](code_eventService.md)

| **Компонент** | **Класс / Интерфейс** | **Тип** | **Назначение** |
|:--------------|:----------------------|:--------|:---------------|
| `EventService` | `EventService` | Interface | Контракт: методы работы с событиями |
| `EventService` | `EventServiceImpl` | Class | Реализация бизнес-логики |
| `EventService` | `EventRepository` | Repository | Чтение и запись событий |
| `EventService` | `EventPublisher` | Publisher | Публикация событий в RabbitMQ |
| `EventService` | `Event` | Entity | Сущность события |
| `EventService` | `EventDto` | DTO | Данные для UI |
| `EventService` | `CreateEventRequest` | DTO | Запрос на создание события |
| `EventService` | `EventType` | Enum | Типы событий |
| `EventService` | `EventStatus` | Enum | Статусы события |

### <a name="_qmphm5d6rvi3_code_3"></a>**Решение** ---> [C4Code — CameraService](code_cameraService.md)

| **Компонент** | **Класс / Интерфейс** | **Тип** | **Назначение** |
|:--------------|:----------------------|:--------|:---------------|
| `CameraService` | `CameraService` | Interface | Контракт: методы работы с камерами |
| `CameraService` | `CameraServiceImpl` | Class | Реализация бизнес-логики |
| `CameraService` | `VideoIngestClient` | Client | REST-клиент к `videoIngest` |
| `CameraService` | `Camera` | Entity | Сущность камеры |
| `CameraService` | `CameraDto` | DTO | Данные для UI |
| `CameraService` | `StreamInfoDto` | DTO | Информация о медиапотоке |
| `CameraService` | `CameraType` | Enum | Типы камер |
| `CameraService` | `CameraStatus` | Enum | Статусы камер |

### <a name="_qmphm5d6rvi3_code_4"></a>**Решение** ---> [C4Code — ErpService](code_erpService.md)

| **Компонент** | **Класс / Интерфейс** | **Тип** | **Назначение** |
|:--------------|:----------------------|:--------|:---------------|
| `ErpService` | `ErpService` | Interface | Контракт: методы работы с ERP |
| `ErpService` | `ErpServiceImpl` | Class | Реализация бизнес-логики |
| `ErpService` | `ErpSystemClient` | Client | REST-клиент к `erpSystem` |
| `ErpService` | `ErpStockItem` | DTO | Модель ERP: остатки |
| `ErpService` | `ErpFinanceReport` | DTO | Модель ERP: финансовый отчёт |
| `ErpService` | `ErpEmployee` | DTO | Модель ERP: сотрудник |
| `ErpService` | `StockItemDto` | DTO | Ответ UI: остатки |
| `ErpService` | `FinanceReportDto` | DTO | Ответ UI: финансовый отчёт |
| `ErpService` | `EmployeeDto` | DTO | Ответ UI: сотрудник |
| `ErpService` | `SyncResultDto` | DTO | Результат синхронизации |


### <a name="_qmphm5d6rvi3_code_alt"></a>**Альтернатива** ---> [C4Code — VmsClient](code_alt.md)

| **Компонент** | **Класс / Интерфейс** | **Тип** | **Назначение** |
|:--------------|:----------------------|:--------|:---------------|
| `vmsClient` | `WebhookController` | Controller | Принимает webhook от VMS |
| `vmsClient` | `EventFilter` | Filter | Фильтрация и приоритизация событий |
| `vmsClient` | `EventPublisher` | Publisher | Публикация событий в RabbitMQ |
| `vmsClient` | `ClipClient` | Client | REST-клиент для запроса клипов |
| `vmsClient` | `VmsApiClient` | Client | REST-клиент для метаданных VMS |
| `vmsClient` | `VmsEvent` | DTO | Событие от VMS |
| `vmsClient` | `FilteredEvent` | DTO | Отфильтрованное событие |
| `vmsClient` | `ClipDto` | DTO | Информация о клипе |
| `vmsClient` | `CameraDto` | DTO | Метаданные камеры |
| `vmsClient` | `ClipRequest` | DTO | Запрос клипа |
| `vmsClient` | `EventPriority` | Enum | Приоритет события |
| `vmsClient` | `VmsEventType` | Enum | Типы событий VMS |

### <a name="_bjrr7veeh80c_code"></a>**Альтернативы внутри C4Code**

| **№** | **Альтернатива** | **Недостатки**                         |
|:-----:|:-----------------|:---------------------------------------|
|   1   | Единый `CoreService` вместо доменных сервисов | God Object, сложно тестировать и расширять |
|   2   | Прямые вызовы клиентов из `Controller` | Смешение слоёв, нет изоляции интеграций |
|   3   | `EventPublisher` внутри `Controller` | Нарушение единственной ответственности |
|   4   | DTO = Entity (без маппинга) | Утечка внутренней модели в UI, сложно менять БД |
|   5   | Единый `VmsAdapter` вместо разделения на компоненты | Смешение ответственности, сложно тестировать |
|   6   | Polling VMS вместо webhook | Высокая задержка, лишняя нагрузка |
|   7   | Прямые вызовы VMS из `coreApi` | Утечка вендорской логики в бизнес-слой |
|   8   | Без приоритизации событий | Перегрузка `notifications` при массовых событиях |

**Недостатки, ограничения, риски**

| **№** | **Решение**                                                       | **Недостатки**                                                                    | **Ограничения**                                                               | **Риски**                                                                         |
|:-----:|:------------------------------------------------------------------|:----------------------------------------------------------------------------------|:------------------------------------------------------------------------------|:----------------------------------------------------------------------------------|
|   1   | `AnimalService` / `AnimalServiceImpl`                             | Дублирование методов интерфейса и реализации                                      | Зависит от `AnimalRepository`                                                 | При изменении контракта — править и интерфейс, и реализацию                       |
|   2   | `AnimalRepository` (Spring Data JDBC)                             | Не поддерживает JPA-транзакции                                                    | Зависит от ClickHouse                                                          | При изменении схемы БД — править запросы                                          |
|   3   | `Animal`, `AnimalDto`                                             | Дублирование полей, ручной маппинг                                                | DTO должен быть сериализуемым                                                  | При добавлении поля — править в двух местах                                       |
|   4   | `EventService` / `EventServiceImpl`                               | `createEvent` смешивает запись и публикацию                                       | Зависит от `EventRepository` и `EventPublisher`                                | При падении `EventPublisher` — событие сохранено, но не опубликовано              |
|   5   | `EventPublisher` (Spring AMQP)                                    | Публикация в брокер не гарантирует доставку                                       | Зависит от `rabbitMQ`                                                          | При потере сообщения — событие не обработается (митигация: DLQ)                   |
|   6   | `VideoIngestClient` (Spring WebClient)                            | Зависит от стабильности `videoIngest`                                             | Ограничен rate-limit внешнего API                                              | При падении `videoIngest` — нет данных о камерах                                  |
|   7   | `ErpSystemClient` (Spring WebClient)                              | Зависит от стабильности ERP (1С)                                                  | Ограничен API ERP                                                              | При изменении API 1С — правка клиента                                             |
|   8   | `AiClient` (gRPC)                                                 | Стрим требует стабильного соединения                                              | Ограничен квотами AI-сервиса                                                   | При разрыве стрима — потеря кадров                                                |
|   9   | `AiResult`, `AiCallbackHandler`                                   | Зависит от формата ответа AI                                                      | Ограничен схемой ответа                                                        | При изменении API AI — правка обработчика                                         |
|   10  | `WebhookController` (Go HTTP)                                     | Зависит от стабильности webhook VMS                                               | Ограничен форматом payload VMS                                                | При изменении формата webhook — правка парсинга                                   |
|   11  | `EventFilter` (Go)                                                | Сложная логика приоритизации                                                      | Зависит от конфигурации                                                       | При ошибке конфигурации — события обрабатываются неверно                          |
|   12  | `EventPublisher` (Go AMQP)                                        | Дублирует роль `EventPublisher` в других контейнерах                              | Зависит от `rabbitMQ`                                                          | При падении брокера — события не публикуются                                      |
|   13  | `ClipClient` (Go HTTP)                                            | Зависит от стабильности VMS                                                       | Ограничен rate-limit VMS                                                      | При недоступности VMS — нет клипов                                                |
|   14  | `VmsApiClient` (Go HTTP)                                          | Дублирование интеграции с `ClipClient`                                            | Ограничен API VMS                                                             | При изменении API — правка обоих клиентов                                         |
|   15  | `VmsEvent`, `FilteredEvent`                                       | Дублирование полей                                                                | Формат диктуется VMS                                                          | При изменении формата — правка маппинга                                           |
|   16  | `EventPriority`, `VmsEventType`                                   | Захардкоженные значения                                                           | Требует пересборки при добавлении                                               | При появлении нового типа события — правка enum                                   |
|   17  | Webhook-интеграция с VMS                                          | Зависимость от стабильности сети                                                  | Ограничена пропускной способностью 4G                                         | При обрыве связи — потеря webhook (митигация: буферизация)                        |
|   18  | Приоритизация событий                                             | Сложность отладки                                                                 | Требует настройки для каждой фермы                                            | При неверной настройке — критические события обрабатываются как низкоприоритетные |