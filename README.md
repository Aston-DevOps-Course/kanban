# Task 7 — Reverse Proxy, HTTPS, Logging, Monitoring
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Docker Compose](https://img.shields.io/badge/Docker_Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=for-the-badge&logo=springboot&logoColor=white)
![Angular](https://img.shields.io/badge/Angular-DD0031?style=for-the-badge&logo=angular&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-336791?style=for-the-badge&logo=postgresql&logoColor=white)

---

В данном задании реализована инфраструктура для Kanban-приложения с использованием Docker Compose.

Состав системы:

- Spring Boot backend
- Angular frontend
- Nginx reverse proxy
- HTTPS
- Load balancing frontend-контейнеров
- Централизованный сбор логов
- Централизованный сбор метрик
- Grafana dashboards


```text id="l7b6o8"
task7/
├── docker-compose.yml
├── nginx/
│   └── default.conf
├── certs/
├── monitoring/
│   └── prometheus.yml
├── logging/
│   └── promtail-config.yml
└── README.md
```

---

## Репозитории

Backend: 
https://github.com/Aston-DevOps-Course/kanban-backend

Frontend: 
https://github.com/Aston-DevOps-Course/kanban-frontend

---

#  Настройка app.local

## Linux/macOS

Открыть:

```bash id="qvq0v4"
sudo nano /etc/hosts
```

## Windows

Открыть файл:

```text id="ylzjvj"
C:\Windows\System32\drivers\etc\hosts
```

В обоих вариантах обавить строку:

```text id="4epp7h"
127.0.0.1 app.local
```

---

# Генерация HTTPS сертификата

Из папки `task7` выполнить:

```bash id="okfn6j"
mkdir certs
```

```bash id="t8j9sm"
openssl req -x509 -nodes -days 365 \
-newkey rsa:2048 \
-keyout certs/app.local.key \
-out certs/app.local.crt
```

---

# Запуск проекта

Из папки `task7`:

```bash id="f0s8yb"
docker compose up --build
```

---

# Доступ к сервисам

## Frontend

```text id="lkrvsz"
https://app.local
```

## Backend API

```text id="y7qv8w"
https://app.local/api/
```

## Grafana

```text id="4x7o5k"
http://localhost:3000
```

## Prometheus

```text id="jfjmr8"
http://localhost:9090
```

## cAdvisor

```text id="d6mmtg"
http://localhost:8088
```

---

# Load Balancing

Frontend работает в двух экземплярах:

* frontend1
* frontend2

Nginx распределяет запросы между контейнерами через upstream.

---

# HTTPS

Nginx настроен на HTTPS:

* HTTP автоматически перенаправляется на HTTPS
* используются self-signed сертификаты

---

# Очередность запуска контейнеров

1. PostgreSQL
2. Backend
3. Frontend контейнеры
4. Nginx
5. Monitoring stack

---

# Централизованный сбор логов

## Используемые сервисы

* Loki
* Promtail

## Как работает

1. Docker контейнеры записывают логи
2. Promtail считывает Docker logs
3. Loki хранит логи
4. Grafana отображает логи

---

# Централизованный сбор метрик

## Используемые сервисы

* cAdvisor
* Prometheus
* Grafana

## Как работает

1. cAdvisor собирает метрики контейнеров
2. Prometheus periodically scrape metrics
3. Grafana визуализирует метрики

---

# Настройка Grafana

## Первый вход

Открыть:

```text id="0rffaw"
http://localhost:3000
```

Стандартные данные:

```text id="4lv8qf"
login: admin
password: admin
```

После первого входа Grafana предложит сменить пароль.

---

# Добавление Prometheus datasource

## Перейти:

```text id="knud14"
Connections → Data sources → Add data source
```

Выбрать:

```text id="od3e4o"
Prometheus
```

URL:

```text id="myq58x"
http://prometheus:9090
```

Нажать:

```text id="zrb7pw"
Save & Test
```

---

# Добавление Loki datasource

## Перейти:

```text id="6c05fw"
Connections → Data sources → Add data source
```

Выбрать:

```text id="ep6qk7"
Loki
```

URL:

```text id="7utdwy"
http://loki:3100
```

Нажать:

```text id="3oy2sd"
Save & Test
```

---

# Создание Dashboard для метрик

## Перейти:

```text id="89r9yh"
Dashboards → New Dashboard
```

## Нажать:

```text id="4b4lph"
Add Visualization
```

## Выбрать datasource:

```text id="7p8z5f"
Prometheus
```

---

# Примеры метрик

## CPU usage контейнеров

```text id="brtxkz"
rate(container_cpu_usage_seconds_total[1m])
```

## Memory usage

```text id="jlwmxh"
container_memory_usage_bytes
```

## Network traffic

```text id="l6fg4h"
rate(container_network_receive_bytes_total[1m])
```

## Container filesystem usage

```text id="2rjgwt"
container_fs_usage_bytes
```

---

# Создание Dashboard для логов

## Создать новую visualization

Datasource:

```text id="9bx6fh"
Loki
```

---

# Примеры Loki запросов

## Все логи

```text id="s8k9bl"
{job="docker"}
```

## Логи backend

```text id="7vtgtm"
{container="kanban-app"}
```

## Логи frontend

```text id="m7r87w"
{container="frontend1"}
```

---

# Использованные best practices

## Docker

* multi-stage builds
* lightweight images
* isolated containers
* named volumes

## Infrastructure

* reverse proxy
* HTTPS
* load balancing
* centralized logging
* centralized monitoring

## Monitoring

* metrics separation
* log aggregation
* observability stack

---

# Итог:

Реализована production-like инфраструктура:

* reverse proxy
* HTTPS
* frontend load balancing
* централизованные логи
* централизованные метрики
* monitoring stack
* Grafana dashboards
* orchestration через Docker Compose

```
