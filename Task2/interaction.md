Взаимодействие сервисов
1. RTB critical path

Основной поток:

DSP
 ↓ HTTPS / OpenRTB
RTB Gateway
 ↓ gRPC
Ad Selection Service
 ↓ gRPC
Bidding Service
 ↓ gRPC
Delivery Service

Для внутренних вызовов RTB используется gRPC.

Причины выбора:

низкие накладные расходы;
быстрый бинарный протокол;
строгие контракты;
подходит для частых service-to-service вызовов с жёстким latency.

REST в RTB critical path не используется.

2. Campaign Service

Для управления кампаниями используется REST:

Advertiser Dashboard
 ↓ REST
API Gateway
 ↓ REST
Campaign Service

REST подходит для CRUD-операций рекламного кабинета и не находится в критичном RTB-потоке.

Bidding Service не вызывает Campaign Service при обработке каждого аукциона.

Изменения кампаний передаются асинхронно:

Campaign Service
 ↓
Campaign DB
 ↓
Kafka
 ↓
RTB Cache

Bidding Service читает актуальную read-модель из RTB Cache.

3. Интеграция с DSP

Внешняя интеграция выполняется по HTTPS / OpenRTB.

RTB Gateway:

принимает bid requests;
валидирует запрос;
выполняет аутентификацию;
преобразует DSP-specific формат во внутреннюю модель;
возвращает bid response.

Это позволяет изолировать Bidding Service от особенностей конкретных DSP.

4. Потоковые события

Для impressions и clicks используется Kafka.

Delivery Service
 ↓
Kafka
 ├─→ Statistics Service
 └─→ Analytics Service

Kafka выбран потому, что эти события:

поступают непрерывным потоком;
не требуют обработки до завершения RTB-запроса;
должны выдерживать пики нагрузки;
могут обрабатываться несколькими consumers независимо.

Таким образом, статистика и аналитика не увеличивают latency RTB.

5. Используемые протоколы
Взаимодействие	Протокол
DSP -> RTB Gateway	HTTPS / OpenRTB
RTB Gateway -> Ad Selection	gRPC
Ad Selection ->Bidding	gRPC
Bidding -> Delivery	gRPC
Dashboard -> API Gateway	HTTPS / REST
API Gateway -> Campaign / Billing / Analytics	REST
Campaign -> события изменений	Kafka
Impressions / clicks	Kafka
Bidding -> RTB Cache	Redis protocol
