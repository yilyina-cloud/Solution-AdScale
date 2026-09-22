Bidding Service
Назначение

Bidding Service выделяется из Auction Engine и отвечает за выполнение аукциона в RTB-потоке.

Сервис:

Получает кандидатов от Ad Selection Service.
Читает необходимые данные из RTB Cache.
Применяет правила аукциона.
Рассчитывает ставки.
Определяет победителя.
Возвращает результат в Delivery Service.

Сервис находится в критическом RTB-пути и масштабируется независимо.

Границы

Bidding Service отвечает только за логику аукциона.

Не входит в ответственность сервиса:

приём OpenRTB-запросов;
таргетинг и подбор кандидатов;
управление кампаниями;
финансовые операции;
запись кликов и показов;
аналитика.
API

Для внутреннего взаимодействия используется gRPC.

Основная операция:

CalculateBid(BidRequest) -> BidResult

BidRequest:

requestId
auctionId
placement
candidateIds[]

BidResult:

auctionId
status
winnerId
creativeId
bidPrice
currency

Статус результата:

WIN
NO_BID
ERROR
Данные

Bidding Service не имеет собственной transactional-БД и является stateless.

Для расчёта ставки используются данные из RTB Cache:

актуальные ставки;
состояние кампании;
бюджетный статус;
ограничения кампании.

Campaign DB остаётся source of truth.

Обновление данных:

Campaign Service
      ↓
Campaign DB
      ↓
Event Broker
      ↓
RTB Cache
      ↓
Bidding Service
Зависимости

Ad Selection Service → Bidding Service - gRPC.

Bidding Service -> RTB Cache — синхронное чтение.

Bidding Service -> Delivery Service — передача результата аукциона.

Прямого синхронного вызова Campaign Service или PostgreSQL в RTB hot path нет.

Масштабирование и отказоустойчивость

Bidding Service stateless, поэтому может горизонтально масштабироваться.

Если данные временно недоступны, используются:

кешированные данные;
fallback-правило;
NO_BID.

Для RTB предпочтительнее быстро вернуть NO_BID, чем превысить допустимое время ответа.
