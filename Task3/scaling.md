Масштабирование данных
1. Campaign Service

Хранилище: PostgreSQL.

Стратегия:

primary + read replicas;
запись выполняется в primary;
чтение для кабинета и служебных запросов можно направлять на replicas;
данные кампаний при дальнейшем росте можно шардировать по advertiserId или campaignId.

Для RTB Campaign DB напрямую не используется — Bidding читает данные из Redis.

2. Billing Service

Хранилище: PostgreSQL.

Стратегия:

primary + standby replica;
финансовые операции выполняются только через primary;
автоматический failover при отказе primary;
шардирование возможно по advertiserId при росте объёма.

Для финансов приоритетнее согласованность данных, чем масштабирование чтения.

3. Statistics Service

Хранилище: ClickHouse.

События impressions и clicks поступают через Kafka.

Данные распределяются между узлами по campaignId или advertiserId.

Репликация используется для отказоустойчивости.

Такое масштабирование подходит для большого количества append-only событий.

4. Analytics Service

Хранилище: отдельный ClickHouse.

Analytics не выполняет тяжёлые запросы к Campaign или Billing DB.

Данные поступают асинхронно через Kafka.

Для чтения используются реплики ClickHouse, что позволяет масштабировать отчётность независимо от записи событий.

5. Bidding Service

Собственной базы данных нет.

RTB Cache на Redis разворачивается в кластерном режиме с репликацией.

При росте нагрузки данные распределяются между узлами Redis.

6. CQRS

CQRS применяется для данных кампаний.

Write-модель:

Campaign Service -> PostgreSQL

Read-модель для RTB:

Campaign Service -> Kafka -> RTB Cache -> Bidding Service

Таким образом, операции изменения кампаний отделены от высокочастотного чтения в RTB.

Для аналитики также используется отдельная read-модель в ClickHouse.
