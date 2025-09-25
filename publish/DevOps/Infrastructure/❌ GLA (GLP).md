# Кратенько
* GLA -- Grafana + Loki + Alloy, это свзяка является легковестным аналогом ELK, и легковестна она потому, что не индексирует ВЕСЬ текст логов, и не хранит их внутри себя. Здесь простая схема:
	_~~1. Promtail -- аналог Logstash, он собирает логи с сервисов на машинах, держит их в себе, если не может их отдать, и отправляет их в Loki~~
	1. Alloy -- это новый агент от Grafana Labs (ранее Promtail, Agent, OpenTelemetry Collector). Он тянет логи с сервисов и отдаёт их в локи
	2. Loki -- поисковик и агрегатор логов, он сжимает их, индексирует лейблы логов и хранит их в s3 хранилище (иногда и в самом себе, но чаще в s3)
	3. Grafana -- огромный гибкий сервис предоставляющий графики и алёрты а так же кучу всего, она делает запрос в Loki, он выдаёт ей логи, и всё ставновится топчик
* То есть схема следующая Alloy -> Loki -> Grafana

# TODO:
- [ ] Сравнение с ELK
- [ ] Описание преимуществ
- [ ] Вынести все инструменты в отдельные заметки
- [ ] Доописать Loki, и написать проблемы с которыми я столкнулся
# Подробно
## Описание:
* **GLA** — это стек для централизованного логирования и визуализации:
_~~1. **Promtail**
    - Агент, который собирает логи с узлов (файлы `/var/log/*.log` или другие источники).
    - Метки (`labels`) определяют, как логи будут индексироваться в Loki.
    - Отправляет логи в **Loki** через HTTP API (`/loki/api/v1/push`).
1. **[[❌ Alloy]]**
	- Агент, который собирает логи с узлов (файлы `/var/log/*.log` или другие источники).
    - Метки (`labels`) определяют, как логи будут индексироваться в Loki.
    - Отправляет логи в **Loki** через HTTP API (`/loki/api/v1/push`).
    - Был **Promtail** – агент, который только логи собирал и пихал в Loki.
	- Был **Grafana Agent** – более универсальный, мог собирать метрики, логи и чуть-чуть traces.
	- Были ещё всякие OpenTelemetry Collector-агенты.
	- В 2023–2024 Grafana Labs решили всё это **объединить в один инструмент** → появился **Alloy**.  
	**То есть:**
	- **Alloy = Promtail + Grafana Agent + часть OpenTelemetry Collector.**
	- Он официально позиционируется как **замена Promtail**.
2. **[[❌ Loki]]**
    - Система хранения и индексации логов.
    - Не полноценная [[❌ TSDB]], а оптимизированная под тексты логов с метаданными.
    - Поддерживает разные схемы хранения: `boltdb-shipper` (для S3/MinIO), `tsdb` (старый локальный вариант).
    - Содержит **compactor**, который управляет слиянием chunks и индексов.
3. **[[❌ Grafana]]**
    - Визуализирует и позволяет искать логи из Loki.
    - Настройка datasource → Loki (`http://loki:3100`).
4. **[[MinIO]] (S3-совместимый)**
    - Используется как **облачное хранилище** для chunks и индексов Loki.
    - Loki использует S3 API, даже если это локальный MinIO.
## Promtail
* **Базовый конфиг**:
```yaml
server:
http_listen_port: 9080
grpc_listen_port: 0
positions:
filename: /tmp/positions.yaml
clients:
- url: http://loki:3100/loki/api/v1/push
scrape_configs:
- job_name: system
	static_configs:
	- targets:
		- localhost
		labels:
		job: varlogs
		__path__: /var/log/*log
```
* positions.filename — где Promtail хранит, сколько прочитано логов.
* scrape_configs → указываем, какие файлы и какие метки использовать.
* Обычно разворачивается в mode: global на всех нодах Swarm, чтобы собирал логи со всех контейнеров/серверов.
# Источники:
```embed
title: "Loki + Grafana + Promtail: Quickstart with Docker Compose - Kubernetes Training"
image: "https://framerusercontent.com/images/RiOXTAbS2R1JQvgzJeepj80Cw.png"
description: "Build a complete Loki, Grafana, and Promtail logging system with Docker Compose"
url: "https://kubernetestraining.io/blog/loki-grafana-promtail-quickstart-with-docker-compose"
favicon: ""
aspectRatio: "56.25"
```

```embed
title: "   Loki Tutorial | Grafana Loki documentation   "
image: "https://grafana.com/meta-generator/Loki+Tutorial@@@loki@@@15.jpg"
description: "An expanded quick start tutorial taking you though core functions of the Loki stack."
url: "https://grafana.com/docs/loki/latest/get-started/quick-start/tutorial/"
favicon: ""
aspectRatio: "52.5"
```
