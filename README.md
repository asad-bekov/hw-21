# Домашнее задание к занятию 5. «Практическое применение Docker»
*Асадбеков Асадбек*

---

### Задание 0: Проверка Docker Compose
- Удалён `docker-compose`
- Установлен современный `docker compose` v2.35.1+

*Убедитесь что у вас УСТАНОВЛЕН docker compose(без тире) версии не менее v2.24.X, для это выполните команду docker compose version*

*Описание*: Установлена современная версия Docker Compose (v2.35.1). Команда `docker compose version` показывает корректную информацию. Утилита `docker-compose` — это alias.

### Скриншот

![Docker Compose Version](https://github.com/asad-bekov/hw-21/raw/main/img/1.png)

## Задание 1: Dockerfile и Compose
- Создан файл `Dockerfile.python` на базе `python:3.9-slim`:

```yaml
FROM python:3.9-slim
WORKDIR /app
COPY . .
RUN pip install --no-cache-dir -r requirements.txt
CMD ["python", "not_tested_main.py"]
```

- Создан `.dockerignore`, чтобы не включать лишние файлы в образ:

```yaml
__pycache__/
*.pyc
.env
```
- Пример `compose.yaml`

```yaml
version: '3.8'

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile.python
    ports:
      - "5000:5000"
    environment:
      - DB_HOST=db
      - DB_PORT=3306
      - DB_USER=app
      - DB_PASSWORD=QwErTy1234
      - DB_NAME=virtd
    depends_on:
      - db

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root_pass_123
      MYSQL_DATABASE: virtd
      MYSQL_USER: app
      MYSQL_PASSWORD: QwErTy1234
    ports:
      - "3306:3306"
```

- *Описание*: Приложение собрано на основе `Dockerfile.python`, используется образ `python:3.9-slim`, зависимости установлены, выполнен запуск с `not_tested_main.py`.

### Скриншот

`Docker Compose`

![Скриншот Docker Compose](https://github.com/asad-bekov/hw-21/raw/main/img/2.png)


## Задание 2: Container Registry

- Зарегистрирован Container Registry в Yandex Cloud
- Настроена авторизация `docker` через `yc container registry configure-docker`
- Загружен образ приложения в реестр
- Выполнено сканирование на уязвимости (Docker Scout / Trivy)

### Скриншоты

- Аутентификация в реестре

![Скриншот Container Registry](https://github.com/asad-bekov/hw-21/raw/main/img/4.png)

- Тегирование и загрузка образа

![Скриншот Container Registry](https://github.com/asad-bekov/hw-21/raw/main/img/5.png)

- Сканирование уязвимостей (Trivy)

![Скриншот Container Registry](https://github.com/asad-bekov/hw-21/raw/main/img/4.1.png)

- Сканирование уязвимостей (Trivy - продолжение)

![Скриншот Container Registry](https://github.com/asad-bekov/hw-21/raw/main/img/4.2.png)

## Задание 3: Compose + Proxy + MySQL
- Создан `proxy.yaml` и объединён через `include`
- Настроены 3 сервиса: `web`, `db`, `reverse-proxy`
- Подключение к MySQL проверено через `docker exec`
- Получен SQL-ответ через `SELECT * FROM requests LIMIT 10`

### Скриншоты
- команда curl -L http://127.0.0.1:8090 возвращает в качестве ответа время и локальный IP-адрес

![команда curl -L http://127.0.0.1:8090 должна возвращать в качестве ответа время и локальный IP-адрес](https://github.com/asad-bekov/hw-21/raw/main/img/6.1.png)

- Скриншот MySQL-запроса
![Скриншот MySQL-запроса](https://github.com/asad-bekov/hw-21/raw/main/img/6.png)

## Задание 4: Развёртывание в Yandex Cloud
- Запущена ВМ (2 ГБ RAM)
- Подключение по SSH
- Bash-скрипт разворачивает проект в `/opt/shvirtd-example-python`
- Проверено HTTP-доступность через `curl` и внешний IP
 
SQL-запрос, подключение к базе данных внутри контейнера:

```sh
docker exec -it shvirtd-example-python-db-1 mysql -uapp -p
# Ввел пароль: QwErTy1234

mysql> USE virtd;
mysql> SELECT * FROM requests LIMIT 10;
```
Пример bash-скрипта для развёртывания (файл: deploy.sh):

```sh
#!/bin/bash

sudo apt update && sudo apt install -y docker.io docker-compose git

git clone https://github.com/asad-bekov/hw-21.git /opt/shvirtd-example-python

cd /opt/shvirtd-example-python

docker compose up -d
```

### Скриншоты

- SQL-запрос

![SQL-запрос](https://github.com/asad-bekov/hw-21/raw/main/img/2.1.png)

- [Ссылка на форк-репозиторий](https://github.com/asad-bekov/shvirtd-example-python)

## Задание 5: Резервное копирование
- Настроено резервное копирование через `schnitzler/mysqldump`
- Рабочий bash-скрипт сохраняет `.sql` в `/opt/backup`
- Настроен `crontab` на каждую минуту
- Получены успешные `.sql` резервные копии
---

- Структура каталогов

<pre><code>
/opt/
├── backup/
│   ├── backup_20250523_140101.sql
│   ├── backup_20250523_140201.sql
│   └── ...
└── shvirtd-example-python/
    └── backup.sh
</code></pre>

- Скрипт `backup.sh`

```sh
#!/bin/bash
DATE=$(date +"%Y%m%d_%H%M%S")
FILE="/opt/backup/backup_${DATE}.sql"

docker run --rm \
  --network shvirtd-example-python_backend \
  mysql:8.0 \
  mysqldump -h db -uapp -pQwErTy1234 virtd > "$FILE"

if [ $? -eq 0 ]; then
  echo "[+] Backup created: $FILE"
else
  echo "[!] Backup failed"
  rm -f "$FILE"
fi
```

### Скриншоты

- Вывод crontab -l

![Вывод crontab -l](https://github.com/asad-bekov/hw-21/raw/main/img/6.4.png)

- Список резервных копий

![Список резервных копий](https://github.com/asad-bekov/hw-21/raw/main/img/6.2.png)

- Содержимое одного дампа
![Содержимое SQL-дампа](https://github.com/asad-bekov/hw-21/raw/main/img/6.3.png)


## Задание 6: Извлечение бинарника Terraform
- Использован `docker save` и `dive` для поиска `/bin/terraform`
- Скопирован бинарник на хост и протестирован (`terraform version`)
- Также продемонстрирован способ с `docker cp`

### Скриншоты

- Скачивание образа с `Terraform`

![Скачивание образа с Terraform](https://github.com/asad-bekov/hw-21/raw/main/img/9.png)

- Просмотр слоев через dive, сохранение и распаковка образа

![Просмотр слоев через dive](https://github.com/asad-bekov/hw-21/raw/main/img/9.1.png)

- Проверка версии `Terraform`

![Проверка terraform version](https://github.com/asad-bekov/hw-21/raw/main/img/6.png)

## Задание 7: runC

- Сгенерирован `rootfs` и `config.json`
- Установлены переменные окружения
- Приложение успешно запущено через `sudo runc run my-python-app`
- Проверено: приложение отвечает на запросы, подключается к MySQL

### Скриншоты

- Запуск приложения с `runс` 

![Запуск runC-контейнера](https://github.com/asad-bekov/hw-21/raw/main/img/8.png)

- Проверка работы приложения

![Ответ от Flask-приложения](https://github.com/asad-bekov/hw-21/raw/main/img/8.1.png)


---

*примечание:* - некоторые скрины растянулись, так как возникли проблемы с графикой в VM
