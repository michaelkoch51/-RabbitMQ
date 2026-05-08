# Домашнее задание к занятию «Очереди RabbitMQ»
Выполнил: Кочнев Михаил
# Задание 1. Установка RabbitMQ
![Установка RabbitMQ]()
# Задание 2. Отправка и получение сообщений
![запуск скриптов](https://github.com/user-attachments/assets/f9ce610c-f4dc-4d54-b8a2-2eceeba4d1a5)
![hello](https://github.com/user-attachments/assets/041f7212-fb96-459c-93a3-a30a223ebcbb)
# Задание 3. Подготовка HA кластера
![политика ha-all](https://github.com/user-attachments/assets/412066cb-c8fb-4c40-96d1-fa599e511dac)
![ha-all](https://github.com/user-attachments/assets/22c239a7-82ae-4148-b7f5-eb2f96b6e8ed)
![rabbitmqctl cluster_status](https://github.com/user-attachments/assets/63678db1-90a6-4c55-8814-b0c56baaf2d6)
![rabbitmqctl cluster_status](https://github.com/user-attachments/assets/ecc72cf1-ea9b-4150-a099-dbf89a3cbbc2)
![1](https://github.com/user-attachments/assets/6c73a9eb-1431-48ff-a9a6-fdc904302c21)
![2](https://github.com/user-attachments/assets/b5d171b4-1a9c-4155-867b-215ff366f7e0)
![3](https://github.com/user-attachments/assets/02d08852-d64d-4418-bdbe-859d195148f5)


Запущены две ноды RabbitMQ в Docker‑контейнерах: rmq01 и rmq02.

Вторая нода (rmq02) присоединена к кластеру первой ноды (rmq01) с помощью команд:

rabbitmqctl stop_app;

rabbitmqctl reset;

rabbitmqctl join_cluster rabbit@rmq01;

rabbitmqctl start_app.

Статус кластера проверен командой rabbitmqctl cluster_status — обе ноды присутствуют в разделах Disk Nodes и Running Nodes.

Результат: кластер успешно создан, обе ноды синхронизированы.

Настройка HA‑политики

Выполнена команда для создания политики высокой доступности:

bash
docker exec rabbitmq-cluster-test-rmq01-1 rabbitmqctl set_policy ha-all ".*" '{"ha-mode":"all"}' --apply-to queues
Параметры политики:

Сценарий: отключение первой ноды (rmq01) и работа через вторую ноду (rmq02).

Отключение rmq01:

bash
docker stop rabbitmq-cluster-test-rmq01-1
Отправка сообщения через rmq02:

в producer.py изменён порт на 5673 (порт второй ноды);

выполнено: python3 producer.py.

Получение сообщения через consumer.py:

в consumer.py также изменён порт на 5673;

запущено: python3 consumer.py;

получен вывод: [x] Received b'Hello, RabbitMQ Cluster!'.

Включение rmq01 обратно:

bash
docker start rabbitmq-cluster-test-rmq01-1
Проверка синхронизации:

через 1–2 минуты открыта страница Overview в веб‑интерфейсе — обе ноды активны;

проверена очередь hello — репликация восстановлена.

Вывод: кластер демонстрирует отказоустойчивость — при отказе одной ноды сообщения доступны через вторую.

Результаты тестирования
Параметр	Результат	Подтверждение
Создание кластера	Успешно	Вывод cluster_status (2 ноды)
HA‑политика	Применена	Поле "policy": "ha-all" в JSON, веб‑интерфейс
Репликация очереди	Работает	Поля "slave_nodes" и "synchronised_slave_nodes" в JSON
Отказоустойчивость	Подтверждена	Получение сообщения при отключённой rmq01
Восстановление после сбоя	Успешно	Обе ноды активны в Overview, репликация восстановлена

