# Курсовая по Docker

# 🚀 DockerCW – Трекер привычек с Telegram-напоминаниями + CI/CD

**DockerCW** — Django-приложение для трекинга привычек с интеграцией Telegram-бота.  
Проект контейнеризирован (Docker + Docker Compose) и снабжен пайплайном CI/CD на **GitHub Actions** с автодеплоем на удалённый сервер (Docker).

---

## 🌐 Продакшн-адрес

Приложение развернуто по адресу: **http://51.250.33.202/**  
> Если адрес изменится — обнови этот блок и переменную `ALLOWED_HOSTS` в настройках.

---

## 🧱 Архитектура

- `web` — Django-приложение (API).
- `db` — PostgreSQL.
- `redis` — брокер для фоновых задач/кэша.
- `nginx` — обратный прокси (раздача статики/медиа и проксирование в `web`).
- **Docker Compose** связывает все сервисы.

---

## 🔐 Переменные окружения (`.env` пример)

Создай файл `.env` в корне проекта:

```env
# Django
SECRET_KEY=change_me
DEBUG=False
ALLOWED_HOSTS=51.250.33.202,localhost,127.0.0.1

# Postgres
POSTGRES_DB=postgres
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
DB_HOST=db
DB_PORT=5432

# Telegram
TELEGRAM_TOKEN=your_telegram_token

# Дополнительно, при необходимости
# CSRF_TRUSTED_ORIGINS=http://51.250.33.202,http://localhost


# Локальный запуск(Dev)
1. Клонировать репозиторий:
git clone https://github.com/Artemteben/DockerCW.git
cd DockerCW
2. Создать .env (см. выше).
3. Поднять контейнеры:
docker compose up -d --build
4. Применить миграции и создать суперпользователя:
docker compose exec web python manage.py migrate
docker compose exec web python manage.py createsuperuser
5. Проверить работу:
API: http://localhost/ (или http://127.0.0.1/)
Nginx проксирует на web; статика собирается внутри контейнеров.
6. Полезно:
docker compose logs -f web    # логи Django
docker compose logs -f nginx  # логи Nginx
docker compose ps             # статус сервисов
# Подготовка удалённого сервера
1. Обновить систему:
sudo apt update && sudo apt -y upgrade
2. Установить Docker и Compose (plugin):
sudo apt -y install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list >/dev/null
sudo apt update
sudo apt -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker $USER
# выйди и зайди заново (или `newgrp docker`)
3. (опционально) Создать пользователя деплоя:
sudo adduser deploy
sudo usermod -aG docker deploy
4. Открыть порты:
sudo apt -y install ufw
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw --force enable
5. Подготовить директорию приложения:
sudo mkdir -p /opt/apps/DockerCW
sudo chown -R $USER:$USER /opt/apps/DockerCW
Клонировать репозиторий (git clone) на сервер в /opt/apps/DockerCW и создать .env с прод-секретами.
🤖 CI/CD на GitHub Actions
Секреты репозитория (Settings → Secrets and variables → Actions → New repository secret)

Добавь следующие secrets:

SSH_HOST — IP/домен сервера (например, 51.250.33.202)

SSH_USER — пользователь на сервере (например, deploy или текущий)

SSH_KEY — приватный SSH-ключ (PEM/ed25519) для доступа на сервер (без пароля)

SSH_PORT — порт SSH (например, 22)

SERVER_APP_DIR — путь до проекта на сервере (/opt/apps/DockerCW)

Для публикации образа в GitHub Container Registry (GHCR) можно использовать встроенный GITHUB_TOKEN. Образ будет доступен как ghcr.io/<owner>/dockercw:tag.

Убедись, что пакет/образ отмечен как public в GHCR (или выполняй на сервере docker login ghcr.io в шаге деплоя).

🚢 Ручной деплой (один раз) на сервере

Скопировать проект (или git clone) в /opt/apps/DockerCW.

Создать .env с прод-секретами.

Создать docker-compose.prod.yml (см. выше).

Первый запуск:
docker compose -f docker-compose.prod.yml up -d
docker compose -f docker-compose.prod.yml exec web python manage.py migrate --noinput
docker compose -f docker-compose.prod.yml exec web python manage.py createsuperuser
Дальше деплой будет происходить автоматически по пушу в main (GitHub Actions).

🧪 Тестирование локально (через Docker)
docker compose run --rm web python manage.py test -v 2

## Автор

[Артём Тебен](https://github.com/Artemteben)
