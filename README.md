# Task 8 — CI/CD
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Docker Compose](https://img.shields.io/badge/Docker_Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=for-the-badge&logo=springboot&logoColor=white)
![Angular](https://img.shields.io/badge/Angular-DD0031?style=for-the-badge&logo=angular&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-336791?style=for-the-badge&logo=postgresql&logoColor=white)

---

В данном задании реализован CI/CD пайплайн для приложения. 

---

## Репозитории

Backend: 
https://github.com/Aston-DevOps-Course/kanban-backend

Frontend: 
https://github.com/Aston-DevOps-Course/kanban-frontend

---

# Видео-презентация

Ссылка на видео-презентацию работы проекта:
https://cloud.triniss.ru/s/bAEaKsWcdnnM3nP

---
# Выполненные задачи по данному проекту (6-8)

## Task 6

### Docker

- Установлен Docker и Docker Compose
- Созданы Dockerfile для backend и frontend приложений
- Реализованы multi-stage сборки
- Использованы best practices:
  - минимальные базовые образы
  - разделение build/runtime stages
  - кеширование зависимостей
  - `.dockerignore`
  - запуск только необходимых артефактов

### Docker Compose

Реализован запуск:
- PostgreSQL
- Backend
- 2 экземпляров frontend
- Nginx load balancer

Frontend работает через балансировщик Nginx.

---

## Task 7

### Reverse Proxy

Настроен Nginx:

- `http://app.local/` → frontend
- `http://app.local/api/` → backend

### HTTPS

Реализована работа HTTPS через self-signed SSL сертификаты.

### Правильная очередность старта контейнеров

Использованы:
- `depends_on`
- `healthcheck`

### Централизованный сбор логов

Добавлены:
- Loki
- Promtail

Логи frontend и backend контейнеров централизованно собираются и доступны через Grafana.

### Централизованный сбор метрик

Добавлены:
- Prometheus
- cAdvisor

Собираются:
- CPU
- RAM
- network
- container metrics

### Grafana

Настроены дашборды:
- контейнеров
- frontend/backend сервисов
- Docker metrics
- логов приложений

---

# CI/CD Pipeline

Для backend и frontend репозиториев реализован CI/CD pipeline через GitHub Actions.

## Pipeline включает:

### CI Stage

При каждом `git push` в ветку `main` автоматически выполняется:

- checkout репозитория
- Docker build
- Docker image tagging
- push образа в Docker Hub

### CD Stage

После успешного push образа:

- GitHub Actions подключается к Proxmox host по SSH
- выполняется `pct exec` внутри LXC контейнера
- внутри контейнера выполняется:
  - `docker compose pull`
  - `docker compose up -d`

В результате обновление backend/frontend происходит автоматически после push изменений в репозиторий.

---

# Структура инфраструктуры

```text
Internet
   ↓
GitHub Actions
   ↓
Docker Hub
   ↓
Proxmox Host
   ↓
LXC Container
   ↓
Docker Compose Stack
```

# Запуск проекта

```bash
mkdir -p /opt/kanban
cd /opt/kanban

git clone https://github.com/Aston-DevOps-Course/kanban-backend.git
git clone https://github.com/Aston-DevOps-Course/kanban-frontend.git
git clone https://github.com/Aston-DevOps-Course/kanban-main.git task7```

## Запуск инфраструктуры

```bash
cd /opt/kanban/task7
docker compose up -d
```

---

# Доступ к сервисам

| Сервис      | Адрес                                          |
| ----------- | ---------------------------------------------- |
| Frontend    | [https://app.local](https://app.local)         |
| Backend API | [https://app.local/api](https://app.local/api) |
| Grafana     | http://SERVER_IP:3000                          |
| Prometheus  | http://SERVER_IP:9090                          |
| Loki        | http://SERVER_IP:3100                          |

---

# Grafana

## Вход

```text
login: admin
password: admin
```

---

# Автоматическое обновление приложения

После внесения изменений в backend/frontend код:

```bash
git add .
git commit -m "update"
git push
```

GitHub Actions автоматически:

1. собирает Docker image
2. публикует image в Docker Hub
3. выполняет deployment внутри LXC контейнера
4. обновляет контейнеры через Docker Compose

### backend.yml:
```yaml
name: Backend CI/CD

on:
  push:
    branches: [main]

jobs:

  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Login Docker Hub
        run: echo "${{ secrets.DOCKER_PASS }}" | docker login -u "${{ secrets.DOCKER_USER }}" --password-stdin

      - name: Build image
        run: docker build -t ${{ secrets.DOCKER_USER }}/kanban-backend:latest .

      - name: Push image
        run: docker push ${{ secrets.DOCKER_USER }}/kanban-backend:latest


  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest

    steps:
      - name: Deploy via Proxmox → LXC
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.PROXMOX_HOST }}
          username: root
          key: ${{ secrets.SSH_KEY }}

          script: |
            pct exec 140 -- bash -c "
              cd /opt/kanban/task7 &&
              docker compose pull &&
              docker compose up -d
            "
```

### frontend.yml

```yaml
name: Frontend CI/CD

on:
  push:
    branches: [main]

jobs:

  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Login Docker Hub
        run: echo "${{ secrets.DOCKER_PASS }}" | docker login -u "${{ secrets.DOCKER_USER }}" --password-stdin

      - name: Build image
        run: docker build -t ${{ secrets.DOCKER_USER }}/kanban-frontend:latest .

      - name: Push image
        run: docker push ${{ secrets.DOCKER_USER }}/kanban-frontend:latest


  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest

    steps:
      - name: Deploy via Proxmox → LXC
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.PROXMOX_HOST }}
          username: root
          key: ${{ secrets.SSH_KEY }}

          script: |
            pct exec 140 -- bash -c "
              cd /opt/kanban/task7 &&
              docker compose pull &&
              docker compose up -d
            "
```