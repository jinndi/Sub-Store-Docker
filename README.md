# Sub-Store-Docker

### Sub-Store + Caddy Reverse Proxy, Docker Compose file

`Sub-Store` - это хайповый инструмент из КНР для управления прокси-подписками. Он поддерживает практически любые форматы, позволяет гибко фильтровать и преобразовывать конфигурации, а также выдавать результат в удобном виде для разных клиентов.

Вам больше не нужно доверять свои данные сомнительным сайтам - разверните собственный экземпляр панели и организуйте управление подписками с умом.

Официальный репозиторий: https://github.com/sub-store-org/Sub-Store

## Быстрый старт

1. Для начала установите на сервер Docker (вместе с Docker Compose) с помощью команды:

```bash
curl -sSL https://get.docker.com | sh
sudo usermod -aG docker $(whoami)
```

2. Ниже приведён пример файла `compose.yml` для развёртывания панели `Sub-Store` за реверс-прокси `Caddy` с автоматически продлеваемыми SSL-сертификатами:

```yaml
services:
  sub-store:
    image: xream/sub-store:http-meta
    container_name: sub-store
    restart: unless-stopped
    environment:
      # Укажите тут свои уникальный набор символов после косой черты:
      SUB_STORE_FRONTEND_BACKEND_PATH: /jfDud83kfDlo0kDewc
      # Длее ничего можно не менять.
      SUB_STORE_BACKEND_API_HOST: 0.0.0.0
      SUB_STORE_BACKEND_API_PORT: 3001
      SUB_STORE_BACKEND_MERGE: true
      # HTTP-META интерфейс, как правило, никаких изменений не требуется.
      PORT: 9876
      HOST: 127.0.0.1
    volumes:
      - sub_store:/opt/app/data
    networks:
      - sub_store_net

  caddy:
    image: ghcr.io/jinndi/caddy:latest
    restart: unless-stopped
    container_name: caddy-reverse
    environment:
      TZ: "Europe/Moscow"
      LOG_LEVEL: "info"
      # Укажите свой домен, или поддомен
      DOMAIN: "mysub.duckdns.org"
      # Укажите свой e-mail для ACME
      EMAIL: "mymail@gmail.com"
      # Тут ссылаемся на сервис sub-store с портом бэка+фронта
      PROXY_ROOT: "sub-store:3001"
    ports:
      # Эти порты должны быть доступны извне и открыты в файрволе (если он используется).
      - 80:80/tcp
      - 443:443/tcp
      - 443:443/udp
    volumes:
      - caddy_data:/data
      - caddy_config:/config
    networks:
      - sub_store_net
    cap_add:
      - NET_ADMIN

volumes:
  sub_store:
  caddy_data:
  caddy_config:

networks:
  sub_store_net:
```

Измените значения на свои согласно комментариям, затем скопируйте конфигурацию и на сервере по SSH создайте файл, например:

```bash
nano compose.yml
```

Вставьте содержимое сочетанием клавиш `Ctrl + Shift + V`, сохраните файл через `Ctrl + S` и выйдите из редактора сочетанием `Ctrl + X`.

Допустим, ваше доменное имя - `mysub.duckdns.org`. После запуска сервисов (команда будет приведена ниже) откройте панель в браузере:

```text
https://mysub.duckdns.org?api=https://mysub.duckdns.org/jfDud83kfDlo0kDewc
```

Либо просто перейдите по адресу: `https://mysub.duckdns.org` и укажите путь к Backend API вручную. В данном примере это: `/jfDud83kfDlo0kDewc`

Все доступные параметры и переменные окружения можно найти на странице Docker Hub: https://hub.docker.com/r/xream/sub-store

## Пример управления сервисами

* Запустить/перезагрузить:

```
docker compose up -d --force-recreate
```

* Остановить:

```
docker compose down --remove-orphans
```

* Выполнить обновление контейнеров на новые версии:

```
docker compose pull
```

* Полное удаление сервисов (образов):

```
docker compose down --rmi all
```
довавив ключ `-v` так же удалит данные (`volumes`)
