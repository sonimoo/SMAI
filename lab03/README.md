# Лабораторная работа №3. Мониторинг ИТ-инфраструктуры с использованием Zabbix

 - **Калинкова София, I2302** 
 - **05.11.2025** 

## Цель работы

Ознакомление с системой Zabbix для мониторинга серверов, сервисов и сетевого оборудования.

## Задачи:

- Установить и настроить сервер Zabbix.
- Установить агент Zabbix на контролируемую машину.
- Добавить и настроить хост в системе.
- Просмотреть собираемые метрики и создать оповещения. 

## Необходимое оборудование и программное обеспечение:
- Две виртуальные машины (например, Ubuntu Server 22.04)
- Доступ в Интернет
- Zabbix Server + MySQL + Apache/Nginx
- Zabbix Agent 

## Ход работы:

## Подготовка 
Для выполнения лабораторной работы были созданы две виртуальные машины в VirtualBox на основе образа Ubuntu Server 22.04.
На каждой виртуальной машине был настроен тип сети NAT и добавлен второй сетевой адаптер. После этого вручную были заданы статические IP-адреса:
- Zabbix Server: 192.168.100.1
- Zabbix Agent: 192.168.100.2
Настройка сети позволила обеспечить корректное взаимодействие между сервером Zabbix и агентом для мониторинга.

### 1. Установка и настройка сервера Zabbix.

#### Обновление системы
На серверной ВМ (Ubuntu 22.04) выполняем обновление пакетов:

```
sudo apt update
sudo apt upgrade -y
```
![alt text](img/image.png)
![супер долго апгрейдывалось](img/image-1.png)

Обновление гарантирует установку актуальных версий пакетов и зависимостей перед установкой Zabbix.

#### Установка репозитория Zabbix
Для установки Zabbix на виртуальную машину с Ubuntu 22.04 был добавлен официальный репозиторий Zabbix версии 6.4.

Выполнены следующие команды:
```
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo apt update
```

- Первая команда скачивает пакет с настройкой репозитория Zabbix.
- Вторая команда устанавливает этот пакет и регистрирует репозиторий в системе.
- Третья команда обновляет список пакетов для последующей установки Zabbix.
![alt text](img/image-2.png)
![alt text](img/image-3.png)

#### Установка и настройка Zabbix Server

На виртуальной машине с `IP 192.168.100.1` был установлен `Zabbix Server` с веб-фронтендом для управления и мониторинга. Для этого выполнена команда:
```
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts -y
```
- Zabbix Server — основной компонент для сбора и обработки данных.
- Zabbix Frontend — веб-интерфейс для настройки хостов, триггеров и просмотра метрик.
- Конфигурация Apache и скрипты базы данных обеспечили корректную работу фронтенда и сервера.

На сервере настроена база данных MySQL для хранения всех метрик и триггеров.
![alt text](img/image-4.png)

### 2. Настройка базы данных и веб-интерфейса.

#### Установка MySQL (MariaDB) и создание базы для Zabbix

На виртуальной машине с `Zabbix Server` был установлен `MariaDB` и создана база данных для работы сервера:
```
sudo apt install mariadb-server -y
sudo mysql -uroot -p
```

![alt text](img/image-5.png)

В MySQL были выполнены команды:
```sql
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'zabbixpass';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

Создана база zabbix с кодировкой `utf8mb4`.

Создан пользователь `zabbix` с паролем `zabbixpass` и предоставлены все права на базу.

![alt text](img/image-6.png)

#### Импорт структуры базы

![alt text](img/image-7.png)

в Zabbix 6.4 структура скриптов изменилась — теперь вся база для сервера создаётся из server.sql.gz.

Импортируем базу

Для сервера:
```
sudo zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql -uzabbix -pzabbixpass zabbix
```
Проверяем, что таблицы появились:

```
mysql -uzabbix -pzabbixpass -D zabbix -e "SHOW TABLES;"
```

появились таблицы `hosts, items, triggers` и т.д. 

#### Запуск Zabbix Server

После импорта базы выполняем запуск сервера и проверку состояния:
```
sudo systemctl restart zabbix-server
sudo systemctl status zabbix-server
```

Сервер `active (running)`.

#### Настройка веб-интерфейса

Веб-фронтенд Zabbix доступен по адресу:

 http://<IP сервера>/zabbix

![alt text](img/image-8.png)
Проверено, что Apache2 установлен и работает корректно.

### 3. Установка Zabbix Agent на другой машине.

#### Подготовка агента 

На виртуальной машине с агентом выполняется обновление пакетов:

```
sudo apt update
sudo apt upgrade -y
```
![alt text](img/image-9.png)

#### Добавляем репозиторий Zabbix

Для установки `Zabbix Agent` на виртуальную машину был добавлен официальный репозиторий `Zabbix` версии 6.4. Выполнены команды:
```
sudo apt install wget -y
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo apt update
```

- wget устанавливается для скачивания пакетов.
- Загружается пакет с репозиторием Zabbix и устанавливается через dpkg.
- Обновляется список пакетов для последующей установки агента.

![alt text](img/image-10.png)
![alt text](img/image-11.png)

#### Установка Zabbix Agent

На агентской машине был установлен `Zabbix Agent` с помощью команды:

```
sudo apt install zabbix-agent -y
```

Агент обеспечивает сбор метрик (CPU, RAM, диск, сеть) с данной машины.

После установки агент можно настроить и запустить для взаимодействия с Zabbix Server.
![alt text](img/image-12.png)

#### Настройка Zabbix Agent

После установки на агентской машине необходимо настроить конфигурационный файл агента:
```
sudo nano /etc/zabbix/zabbix_agentd.conf
```

В файле нужно найти и изменить следующие строки (убрать символ #, если он присутствует):
```
Server=192.168.100.1
ServerActive=192.168.100.1
Hostname=ZabbixAgent
```

- Server — IP-адрес Zabbix Server, которому агент будет отправлять данные.
- ServerActive — IP-адрес сервера для активного режима агента (агент сам шлёт данные серверу).
- Hostname — имя хоста, которое будет отображаться в веб-интерфейсе Zabbix.

После сохранения файла агент готов к запуску и подключению к серверу.

![alt text](img/image-13.png)
![alt text](img/image-14.png)

#### Запуск и включение Zabbix Agent

После настройки конфигурационного файла агент запускается и включается для автоматического старта при загрузке системы:

```
sudo systemctl restart zabbix-agent
sudo systemctl enable zabbix-agent
sudo systemctl status zabbix-agent
```

- restart — перезапуск агента после внесённых изменений в конфигурацию.
- enable — включение автозапуска при старте системы.
- status — проверка текущего состояния агента.

![alt text](img/image-15.png)

статус агента active (running), что подтверждает корректную работу и подключение к серверу Zabbix.

#### Импорт начальной схемы и данных базы Zabbix

Для корректной работы `Zabbix Server` необходимо создать все таблицы и начальные данные в базе данных. Для этого на сервере выполняется команда:
```
sudo zcat /usr/share/doc/zabbix-sql-scripts/mysql/create.sql.gz | mysql -uzabbix -p zabbix
```

Файл `create.sql.gz` содержит структуру базы данных (таблицы hosts, items, triggers и др.) и начальные записи, необходимые для работы сервера.

Подключение к базе происходит под пользователем zabbix с паролем `zabbixpass`.


! Без выполнения этого шага сервер `Zabbix` не сможет функционировать, так как ему просто нечего хранить и обрабатывать.
После импорта базы можно запускать сервер и открывать веб-фронтенд:
```
sudo systemctl restart zabbix-server
sudo systemctl status zabbix-server
```

Сервер должен быть active (running), а веб-интерфейс доступен по адресу:
http://<IP сервера>/zabbix
![alt text](img/image-16.png)
![alt text](img/image-17.png)

Welcome (Приветствие)
Выбираем язык интерфейса (по умолчанию English).

![alt text](img/image-18.png)
Configure DB connection (Настройка подключения к базе)

- DB type: MySQL
- Database host: localhost
- Database name: zabbix
- User: zabbix
- Password: zabbixpass

Нажимаем Test connection для проверки соединения с базой.Если соединение успешно, нажимаем Next step.

![alt text](img/image-19.png)
![alt text](img/image-20.png)
Check of pre-requisites (Проверка требований)

Zabbix проверяет наличие необходимых PHP-расширений, права доступа и конфигурацию сервера.

Если все требования выполнены (зелёные галочки), нажимаем Next step.

![alt text](img/image-21.png)
На этом шаге веб-фронтенд завершает установку и создаёт необходимые внутренние файлы конфигурации.

После успешной установки появляется сообщение о готовности Zabbix к работе. веб-фронтенд готов к использованию: добавлению хостов, созданию триггеров и панелей мониторинга.

![alt text](img/image-22.png)

! На самом деле В `Zabbix` по умолчанию есть стандартный аккаунт (поэтому используем его):
- Username: Admin 
- Password: zabbix

![alt text](img/image-23.png)

### 4. Добавление хоста в веб-интерфейсе Zabbix.

Для мониторинга агентской машины необходимо добавить её как хост в веб-интерфейсе `Zabbix`:

Переходим к управлению хостами

В боковом меню выбираем: `Data collection → Hosts`

## Создаём новый хост

![alt text](img/image-24.png)

Нажимаем кнопку Create host (она в правом верхнем углу:D).

Заполняем поля:
- Host name: ZabbixAgent (имя хоста, которое указано в конфигурации агента)
- Visible name: можно оставить такое же или придумать короткое имя
- Groups: добавляем хост в группу, например Linux servers
- Interfaces: добавляем интерфейс типа Agent и указываем IP-адрес агента 192.168.100.2
- Привязываем шаблон мониторинга
- В разделе Templates выбираем, например, шаблон Template OS Linux by Zabbix agent, чтобы автоматически подключить стандартные метрики CPU, RAM, диск, сеть и др.

- Сохраняем хост: Нажимаем Add или Save.

![alt text](img/image-25.png)

Новый хост появится в списке и начнёт собирать данные через Zabbix Agent.
После добавления можно проверить состояние:
В списке хостов должно отображаться Enabled и Мониторинг активен.

![alt text](img/image-26.png)

Метрики начинают поступать через несколько минут (CPU, RAM, диск, сеть и т.д.).

### 5. Проверка сбора данных (CPU, RAM, сеть).

После добавления хоста можно проверить, что Zabbix Agent собирает метрики:
В веб-интерфейсе переходим в: Monitoring → Latest data

Выбираем нужный хост (аgent-vm) и смотрим список метрик (items), которые начали собираться.

Примеры метрик, собираемых с Linux-хоста:

**все вместе:**
![alt text](img/image-27.png)


**cpu:**
- system.cpu.util[,user] — загрузка CPU пользователем
- system.cpu.util[,system] — загрузка CPU системой
- system.cpu.util[,idle] — время простоя CPU
![alt text](img/image-28.png)

**Memory (RAM):**

- vm.memory.size[total] — всего памяти
- vm.memory.size[available] — доступно памяти
- vm.memory.utilization — процент использования памяти
![alt text](img/image-29.png)

**Network (Сеть):**

- net.if.in["enp0s3"] — входящий трафик
- net.if.out["enp0s3"] — исходящий трафик
- net.if.in["enp0s3",errors] — ошибки входящего трафика

![alt text](img/image-30.png)

**Storage (Диск):**

- vfs.fs.size[/,used] — занятoе место
- vfs.fs.size[/,free] — свободное место
- vfs.fs.size[/,pused] — процент использования

![alt text](img/image-31.png)

### 6. Настройка оповещения (триггер + действие).

*Цель*: получать уведомления, когда какая-то метрика выходит за пределы нормы, например, загрузка CPU слишком высокая.

#### Создание триггера

В веб-интерфейсе Zabbix переходим: Configuration → Hosts

Выбираем нужный хост (agent-vm) → вкладка Triggers → Create trigger.

Заполняем поля:

- Name: High CPU Load
- Severity: High
- Expression:
![alt text](img/image-32.png)
Это означает: если свободный CPU < 20%, триггер сработает.

![alt text](img/image-33.png)

что-то не так, поэтому выбираем Add рядом с Expression

![alt text](img/image-35.png)

![alt text](img/image-36.png)

Теперь Expression: `last(/agent-vm/system.cpu.util[,idle])=0`

`но!:` эта формула срабатывает только если CPU idle ровно 0%, что в реальной системе встречается крайне редко. Для тестов лучше использовать небольшие значения, например <20, но что-то не сложилось, оставим как есть.

Нажимаем `Add` или `Save`, чтобы триггер появился в списке.

#### Создание действия (Action) для триггера
Переходим: `Configuration → Actions → Trigger actions → Create trigger action`

Заполняем поля:

- Name: Notify on High CPU
- Conditions:
    - Trigger = High CPU Load (определяет, при каком триггере будет выполняться действие)

![alt text](img/image-37.png)
![alt text](img/image-38.png)
![alt text](img/image-39.png)

#### теперь переходим в раздел Operations:

- Operation: Send message
- Recipients: выбираем пользователя или группу (например, Zabbix administrators)
- Step duration: 0 (для бесконечного повторения)
- Сохраняем действие.

![alt text](img/image-40.png)
![alt text](img/image-41.png)
![alt text](img/image-42.png)
![alt text](img/image-43.png)

Теперь при срабатывании триггера Zabbix будет автоматически отправлять уведомления выбранным пользователям.

### 7. Создание персональной панели мониторинга (dashboard). 

Для удобного наблюдения за состоянием хоста agent-vm создаём персональную панель:

#### Открытие панели

В веб-интерфейсе Zabbix переходим: `Dashboard → Edit dashboard`

![alt text](img/image-44.png)

#### Добавление виджетов

- Нажимаем Add widget (кнопка с + или «Add widget»).
- Выбираем тип виджета из списка.

Виджеты, добавленные на панель:

**График CPU**

- Type: Graph
- Name: CPU Idle
- Host: agent-vm
- Item: system.cpu.util[,idle]
- Refresh interval: 30s

Нажимаем `Add`
![alt text](img/image-45.png)
![alt text](img/image-46.png)

**Состояние триггеров**

- Type: Data overview / Triggers
- Name: Triggers overview
- Hosts: agent-vm
- Show only High severity

Нажимаем `Add`
![alt text](img/image-47.png)

#### Сохранение панели

После добавления всех виджетов нажимаем `Save dashboard`.

Панель обновляется автоматически и позволяет следить за состоянием хоста agent-vm в реальном времени.

![alt text](img/image-48.png)

## Контрольные вопросы:

**1. Какова роль основных компонентов Zabbix (Server, Agent, Proxy, Database, Frontend)? **

*Zabbix Server* — центральный компонент, который собирает и обрабатывает данные от агентов, хранит их в базе и управляет триггерами и действиями.

*Zabbix Agent* — установлен на хостах, собирает данные о метриках (CPU, RAM, сеть и др.) и отправляет их серверу.

*Zabbix Proxy *— промежуточный компонент, который собирает данные с хостов и передаёт их серверу, используется для распределённого мониторинга.

*Database* — хранит все данные о хостах, метриках, триггерах и действиях.

*Frontend* — веб-интерфейс Zabbix, через который настраиваются хосты, триггеры, действия и просматриваются метрики и отчёты.

**2. Как осуществляется связь между Zabbix Server и Zabbix Agent? **

Агент устанавливается на контролируемый хост и может работать в двух режимах:
- Passive: сервер запрашивает данные у агента.
- Active: агент сам отправляет данные на сервер.

Связь происходит через TCP-порт (по умолчанию 10050 для агента, 10051 для сервера).

**3. Что такое «items» и «triggers» в Zabbix?**

*Items* — это отдельные метрики, которые собирает Zabbix Agent или сервер (например, использование CPU, свободная память, сетевой трафик).

*Triggers* — условия или правила на основе значений Items, которые определяют, когда возникает проблема (например, триггер «High CPU Load», когда CPU idle < 20%).

## Вывод

В ходе лабораторной работы была установлена и настроена система Zabbix для мониторинга серверов и сетевого оборудования. Студенты научились устанавливать Zabbix Server с базой данных MySQL и веб-интерфейсом, а также Zabbix Agent на контролируемой машине, добавлять хосты и просматривать собираемые метрики (CPU, память, диск, сеть).

Были созданы триггеры для уведомлений о превышении пороговых значений, настроены действия (actions) для автоматической отправки сообщений пользователям, а также разработана персональная панель мониторинга (Dashboard) для наглядного отслеживания состояния хоста в реальном времени. Лабораторная работа продемонстрировала практическое применение Zabbix для контроля ИТ-инфраструктуры.