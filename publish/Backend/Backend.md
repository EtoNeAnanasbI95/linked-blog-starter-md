# Table of content
* CSharp
* Golang
* Patterns
# Future roadmap and rules notes
1. Язык 
	1. Его особенности
	2. Его библиотеки
2. Инфраструктурные инструменты
	1. Базы данных
		1. Реляционные
		2. Не реляционные
		3. Колоночные
		4. Time-Series базы
	2. Брокеры сообщений
		1. Apache Kafka
		2. RabbitMQ
		3. Nats
	3. Балансировщики рагрузки
		1.  Подумать :)
	4. Кэши 
		1. Виды кэшей:
			1. In-memory: Redis, Memcached.
			2. Распределённые: Hazelcast, Apache Ignite.
			3. Кэш на уровне приложения: Spring Cache, Guava Cache.
			4. CDN: Cloudflare, Akamai, AWS CloudFront.
	5. Контейнеризация и оркестрация
	    - Контейнеры: Docker, Podman.
	    - Оркестрация: Kubernetes, Docker Swarm
	    - Основы: pods, deployments, services, ingress.
	    - Инструменты: Helm, Kustomize, ArgoCD.
	6. Мониторинг и логирование
	    - Мониторинг: Prometheus, Grafana, Zabbix.
	    - Логирование: ELK Stack (Elasticsearch, Logstash, Kibana), Loki.
	    - Метрики: latency, throughput, error rates.
	    - Трассировка: Jaeger, Zipkin, OpenTelemetry.
3. Sys dis
	- **Архитектурные паттерны**
	    - Монолит vs Микросервисы vs Serverless.
	    - Event-Driven Architecture.
	    - CQRS (Command Query Responsibility Segregation).
	    - Domain-Driven Design (DDD): bounded contexts, aggregates, entities.
	- **Масштабируемость**
	    - Вертикальное vs горизонтальное масштабирование.
	    - Шардинг, партиционирование, репликация.
	    - Load balancing и распределённые системы.
	- **Интеграция**
	    - API Design:
	        - REST: HATEOAS, versioning, OpenAPI/Swagger.
	        - GraphQL: schema, resolvers, subscriptions.
	        - gRPC: protobuf, streaming.
	    - Асинхронные интеграции: брокеры сообщений, event sourcing.
	- **Безопасность**
	    - Аутентификация: JWT, OAuth2, OpenID Connect.
	    - Авторизация: RBAC, ABAC.
	    - Шифрование: TLS, данные в покое (encryption at rest).
	    - Защита: OWASP Top 10, SQL-инъекции, XSS, CSRF.
	- **Производительность**
	    - Оптимизация БД: индексы, денормализация, кэширование.
	    - Оптимизация API: batching, pagination, compression (Gzip).
	    - Профилирование: JMeter, New Relic, YourKit.
	- **Принципы проектирования**
	    - SOLID
	    - Идемпотентность, иммутабельность