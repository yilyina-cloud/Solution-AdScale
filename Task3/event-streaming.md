Потоковая обработка событий

Для асинхронной обработки используется Kafka.

Топики

| Topic                 | Назначение |

| ad.impressions        | События показов |
| ad.clicks             | События кликов |
| campaign.changed      | Изменения кампаний и таргетинга |
| bid-settings.changed  | Изменения ставок |
| budget.changed        | Изменения доступного рекламного бюджета |

Формат событий

Используется Avro.

Причины выбора:

- компактный бинарный формат;
- контроль схемы;
- поддержка эволюции схем;
- меньший объём сообщений по сравнению с JSON.

Общие поля:

eventId  
eventType  
timestamp

Остальные поля зависят от типа события.

Например:

impression/click — campaignId, advertiserId, userId, placementId;

campaign.changed — campaignId, advertiserId, status;

budget.changed — advertiserId, availableBudget.

Consumer Groups

Statistics Service обрабатывает impressions и clicks.

Analytics Service получает рекламные события своей consumer group.

RTB Cache Updater обрабатывает изменения кампаний, ставок и бюджета и обновляет Redis.

Хранение

Impressions и clicks хранятся в Kafka 7 дней.

События изменений кампаний, ставок и бюджета — 3 дня.

После обработки долговременное хранение выполняется в соответствующих сервисных хранилищах.

Для impressions, clicks и изменений кампаний ключ партиционирования — campaignId.

Для budget.changed используется advertiserId.
