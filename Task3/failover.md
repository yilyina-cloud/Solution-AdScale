Отказоустойчивость данных

RPO и RTO

Bidding / Redis
RPO: данные восстанавливаются из source of truth.
RTO: до 5 минут.
Стратегия: Redis Cluster, replicas, cache warming.

Campaign Service
RPO: до 5 минут.
RTO: до 15 минут.
Стратегия: PostgreSQL replica + backup.

Billing Service
RPO: близкий к 0.
RTO: до 10 минут.
Стратегия: PostgreSQL synchronous replica + backup.

Statistics Service
RPO: до 5 минут.
RTO: до 30 минут.
Стратегия: Kafka replay + репликация ClickHouse.

Analytics Service
RPO: до 15 минут.
RTO: до 60 минут.
Стратегия: Kafka replay + репликация ClickHouse.

Для Billing требования строже, так как потеря финансовых транзакций недопустима.

Резервное копирование

Для PostgreSQL используются:

- регулярные full backup;
- WAL-архивирование;
- point-in-time recovery;
- хранение резервных копий отдельно от production.

Для ClickHouse используются репликация между узлами и периодические snapshots.

Kafka хранит события с репликацией и позволяет повторно обработать поток после восстановления consumer.

Redis не является source of truth. При потере кеша он восстанавливается из Campaign DB, Billing DB и событий Kafka.

Failover

При отказе основной PostgreSQL выполняется переключение на replica.

Для Billing используется синхронная replica, чтобы минимизировать риск потери финансовых данных.

Redis разворачивается в кластерном режиме с replicas.

Kafka и ClickHouse разворачиваются как кластеры с репликацией между узлами.

Соответствие 12-factor

Сервисы следуют основным принципам 12-factor:

- конфигурация хранится вне кода;
- сервисы stateless;
- PostgreSQL, Kafka и Redis рассматриваются как backing services;
- экземпляры сервисов могут запускаться и останавливаться независимо;
- логи передаются в централизованную систему;
- для development, test и production используется одинаковый подход к сборке и deployment.
