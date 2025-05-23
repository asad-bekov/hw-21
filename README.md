# Домашнее задание к занятию 5. «Практическое применение Docker»
*Асадбеков Асадбек*

---

## Выполненные задания

### Задание 0: Проверка Docker Compose
- Удалён `docker-compose`
- Установлен современный `docker compose` v2.35.1+

### Задание 1: Dockerfile и Compose
- Собран образ Python-приложения на базе `python:3.9-slim`
- Создан `Dockerfile.python`, `.dockerignore`, `.gitignore`, `compose.yaml`
- Протестирована работа приложения
- Использован `not_tested_main.py` (замена нестабильному `main.py`)

### Задание 2: Container Registry
- Зарегистрирован Container Registry в Yandex Cloud
- Настроена авторизация `docker` через `yc container registry configure-docker`
- Загружен образ приложения в реестр
- Выполнено сканирование на уязвимости (Docker Scout / Trivy)

### Задание 3: Compose + Proxy + MySQL
- Создан `proxy.yaml` и объединён через `include`
- Настроены 3 сервиса: `web`, `db`, `reverse-proxy`
- Подключение к MySQL проверено через `docker exec`
- Получен SQL-ответ через `SELECT * FROM requests LIMIT 10`

### Задание 4: Развёртывание в Yandex Cloud
- Запущена ВМ (2 ГБ RAM)
- Подключение по SSH
- Bash-скрипт разворачивает проект в `/opt/shvirtd-example-python`
- Проверено HTTP-доступность через `curl` и внешний IP

### Задание 5: Резервное копирование
- Настроено резервное копирование через `schnitzler/mysqldump`
- Рабочий bash-скрипт сохраняет `.sql` в `/opt/backup`
- Настроен `crontab` на каждую минуту
- Получены успешные `.sql` резервные копии

### Задание 6: Извлечение бинарника Terraform
- Использован `docker save` и `dive` для поиска `/bin/terraform`
- Скопирован бинарник на хост и протестирован (`terraform version`)
- Также продемонстрирован способ с `docker cp`

### Задание 7: runC
- Сгенерирован `rootfs` и `config.json`
- Установлены переменные окружения
- Приложение успешно запущено через `sudo runc run my-python-app`
- Проверено: приложение отвечает на запросы, подключается к MySQL

---

## Скриншоты

Kлючевые этапы выполнения задания:

### Задание 1-2
![Docker Compose и запуск](https://github.com/asad-bekov/hw-21/raw/main/img/1.png)
![Проверка подключения к БД](https://github.com/asad-bekov/hw-21/raw/main/img/1.1.png)

### Задание 3
![Состояние контейнеров](https://github.com/asad-bekov/hw-21/raw/main/img/2.png)
![Ответ сервиса](https://github.com/asad-bekov/hw-21/raw/main/img/2.1.png)

### Задание 4
![Доступ через внешний IP](https://github.com/asad-bekov/hw-21/raw/main/img/3.png)
![SQL-результат запроса](https://github.com/asad-bekov/hw-21/raw/main/img/3.1.png)

### Задание 5
![Резервные копии в директории /opt/backup](https://github.com/asad-bekov/hw-21/raw/main/img/4.png)

### Задание 6
![Извлечение Terraform](https://github.com/asad-bekov/hw-21/raw/main/img/5.png)
![Проверка terraform version](https://github.com/asad-bekov/hw-21/raw/main/img/6.png)

### Задание 7
![Запуск runC-контейнера](https://github.com/asad-bekov/hw-21/raw/main/img/7.png)
![Ответ от Flask-приложения](https://github.com/asad-bekov/hw-21/raw/main/img/8.png)

---

