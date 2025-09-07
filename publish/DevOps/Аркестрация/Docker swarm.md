# Термины
- **Node (нода)**: Один сервер в кластере. Бывает **manager** (управляет кластером) и **worker** (выполняет задачи).
- **Service**: Абстракция, описывающая контейнер(ы) — образ, порты, количество реплик (копий).
- **Task**: Конкретный экземпляр контейнера, запущенный на ноде.
- **Stack**: Набор сервисов, описанных в одном файле (обычно docker-compose.yml), для развертывания приложения
# Описание
* Принцип тот же что и в кубере. есть менеджер нода, есть воркер ноды. С менеджер ноды осуществляется управление всем кластером
## Пример инициализации менеджер ноды
* На **первом сервере** (будет manager) инициализируй кластер:
```bash
docker swarm init --advertise-addr <IP_первого_сервера>
```
Пример вывода:
```bash
Swarm initialized: current node is now a manager.  To add a worker to this swarm, run the following command:  docker swarm join --token <токен> <IP_первого_сервера>:2377
```
* Чтоб получить токен второй раз, если понадобится, надо написать
```shell
docker swarm join-token manager
```
## Пример инициализации worker ноды
* Потом просто с каждой worker ноды привязываемся в менежер ноде
- На **других серверах** (worker'ы) выполни команду docker swarm join:
```bash
docker swarm join --token <токен> <IP_первого_сервера>:2377
```
## Пример того, что можно делать с менеджер ноды
- Проверка статус кластера с manager'а:
	Вывод список нод (manager и worker'ы).
```bash
docker node ls
```
## Пример развёртки MinIO кластера
- **Создай файл конфигурации** (minio-stack.yml) на manager'е:
	```yaml
	version: '3.8'
	services:
	minio:
	image: quay.io/minio/minio:latest
	command: server /data{1...4} --console-address ":9001"
	environment:
	  - MINIO_ROOT_USER=admin
	  - MINIO_ROOT_PASSWORD=strongpassword
	ports:
	  - "9000:9000"
	  - "9001:9001"
	volumes:
	  - minio-data:/data
	networks:
	  - minio-net
	deploy:
	  mode: global
	  placement:
		constraints:
		  - node.role == worker
	  restart_policy:
		condition: on-failure
	volumes:
	minio-data:
	driver: local
	networks:
	minio-net:
	driver: overlay
	```
    
    **Объяснение:**
    
    - command: server /data{1...4} — запускает MinIO в распределенном режиме с 4 нодами.
    - mode: global — запускает по одному экземпляру MinIO на каждой worker-ноде.
    - volumes: minio-data — локальный том для хранения данных.
    - networks: minio-net — создает сеть overlay для общения контейнеров.
- **Разверни стек**:
	```bash
docker stack deploy -c minio-stack.yml minio
	```
- **Проверь статус сервисов**:
	```bash
docker stack ps minio
	```
    Увидишь, что MinIO запустился на worker'ах.
    
**И вот таких стэков можно создавать столько, сколько надо. Грубо говоря, стэк это новая абстракция надо компоузом**