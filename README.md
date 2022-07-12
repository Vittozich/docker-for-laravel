# Образец докера для работы с проектом Laravel, который находится в папке src

## Запуск

Чтобы не вызвало ошибок:

    mkdir src mysql mysql_test

Копировать .env.example в .env для докера

    cp .env.example .env

Сконфигурировать `.env` файл по своему усмотрению, назвать проект по-своему.

И уже после всего этого создать контейнеры

    docker-compose up -d --build

Далее можно использовать как этот образ, так и просто скопировать конфигурацию с необходимым `.env` и скопировать или установить laravel проект в папку `src` со своим `.git`, они не будут пересекаться.

Нужно учитывать конфликт портов.

# Containers:

- [php:8-fpm-alpine](https://registry.hub.docker.com/_/php) (build)
- [mysql:8](https://registry.hub.docker.com/_/mysql) (image)
- [nginx:stable-alpine](https://hub.docker.com/_/nginx) (image)
- [phpmyadmin/phpmyadmin](https://registry.hub.docker.com/r/phpmyadmin/phpmyadmin) (image)
- [composer:latest](https://registry.hub.docker.com/_/composer) (build as php but +)
- [library/node](https://registry.hub.docker.com/_/node) (image, npm)

# Команды:

## Создание и редактирование контейнеров

- `docker-compose up -d --build` - создание контейнера
(его запуск и по этой команде можно пересоздавать(после изменений))
- `docker-compose down` - удаляет контейнеры

## Доступы в контейнеры

По сути одинаковые действия, но реализация и работа разная:

- `docker exec -it имя_контейнера команда контейнера` - команда внутри работающего контейнера

- `docker-compose run --rm имя_контейнера команда контейнера` - команда, которая создает временный контейнер и выполняет команду в существующем контейнере, уничтожается.

### Быстрые команды:

- `docker exec -it php sh` - запуск консоли внутри контейнера
- `docker-compose run --rm npm -v` версия npm
- `docker-compose run --rm php -v` версия php
- `docker-compose run --rm artisan -V` версия laravel
- `docker-compose run --rm composer -V` версия composer

Команды внутри контейнера
- `ls -l` -список директорий по разрешению

Исполнители для работы:

- `docker-compose run --rm npm -i` установка пакета npm в папку `src`

При работе с разработкой пакетов и библиотек для laravel не добавлять в `src/composer.json` этот код (как советуют, это приведет к тому что симлинк не находится внутри контейнера): ещё раз НЕ ПРИМЕНЯТЬ ЭТОТ КОД, УБРАТЬ

    "options": {
        "symlink": true
    }

## По работе проекта:

- Для того чтобы собирать данные контейнеры для разных приложений в `.env` стоит назвать `APP_NAME`,
 чтобы для каждого проекта были свои контейнеры. Лучше называть с нижним подчеркиванием в начале
- Если нужно запустить разные сборки контейнеров одновременно - нужно поменять порты сервисов (контейнеров) во избежание конфликтов
- Возможности общего шаблона будут дополнены после проверки его на всех моих проектах (дополнительные либы для php и дополнительные контейнеры (типо phpmyadmin))
- Данный контейнер собирается, как группа контейнеров, которые описаны в `docker-compose.yml`
- Код фреймворка (Laravel) находится в папке `src` и `.gitignore` игнорирует то, что внутри, так как внутри данной папки - репозиторий `git` фреймворка (как проекта)
- В данной сборке используется `nginx`, но можно использовать и настроить `apache` -
механизм не будет отличаться, при обновлении файлов внутри `src` не нужно перезапускать контейнеры.
- ВАЖНО: все эти настройки - для локальной работы с проектом, для боевой версии немного другая конфигурация. (возможно, для каждого сервера своя)

## Конфигурация образа в отдельном файле:

- Вместо `image` в `docker-compose.yml` используется синтаксис `build` и внутри `build:` 2 переменные `context` - где этот файл искать (`./php`) и `dockerfile` - название докерфайла. (лучше это сделать в отдельной папке)


## Конфигурация mysql:

- База данных работает сначала с одной таблицей, с одним пользователем, который имеет доступ на чтение только одной базы
- При конфигурации базы данных в проекте необходимо указать начальную базу данных как в настройках `.env` проекта (в папке `src`)
 и изменить подключение с `localhost` или с `127.0.0.1` на название базы контейнера (в данном случае `mysql`)
 и это выглядит так:

        DB_HOST=mysql
        DB_USERNAME=mysql
        DB_PASSWORD=mysql

- для тестирования нужно в `phpunit.xml` указать `<env name="DB_HOST" value="mysql-tests"/>` в `<php>` тэге, что соответствует бд с тестовым контейнером. Не забыть указать `phpunit.xml` как конфигурационный файл для тестирования в phpstorm в настройках `Php/Test Frameworks`

- Если проект содержит несколько баз данных, их надо создать или копировать (если проект переносится на docker)
 и добавить права стандартному пользователю (в данном случае пользователю `mysql`)
- дополнительные параметры находятся в `./mysqlconf/my.cnf` - при их изменении достаточно просто перезапустить контейнер `mysql`
- долго не мог решить проблему того, что не используется файл `my.cnf` решение оказалось простое - ` command: --innodb_use_native_aio=0` в настройках контейнера
- `mysql -uroot -pmysql -hlocalhost -P3306 -e 'show global variables like "max_allowed_packet"';` - полезная команда выполняется внутри контейнера

## Конфигурация nginx:

- Создается отдельный файл конфигурации в папке `./nginx` с названием `default.conf`  -
 данные об этом указываются в настройке контейнера в `docker-compose.yml` в настройках nginx в `volumes`


## Работа с composer контейнером:

- После билда этот контейнер в состоянии отключенного.
- Выполнять команды для composer нужно с префиксом `--rm` - так как composer создает команду, по выполнении которой - команда должна быть удалена. 
Иначе будет создан ещё один контейнер. Пример - узнать версию `docker-compose run --rm composer -V`
- Данный контейнер в стандартной сборке использует совсем урезанную версию `php` с которой невозможны большинство моих проектов, поэтому я собираю билд.
- Важно понимать что `php`, который используется внутри стандартной сборки `composer` это не тот же самый, что используется в текущем контейнере `php`, ещё раз повторю, это причина почему я собираю билд.
- Внутри папки `./php/composer` есть файл `php.ini` он нужен для настроек всех образов `php` в данной группе контейнеров


## Работа с одновременно запущенными контейнерами, если они связаны

- не достаточно просто изменить порты и обращаться по имени контейнера вместо url
- Для создания моста между 2 группами контейнеров нужно создать новый `network`, это будет выглядеть так

      networks:
        laravel:
        app-shared:
          driver: bridge

- Данный `network` нужно указать в `networks` контейнеров `nginx` и `php`
- Для того чтобы использовать этот `network` в другом приложении необходимо узнать его точно имя,
оно указывается при создании контейнера, в моём случае это `laravel_example_container_app-shared` ,
можно проследить и предугадать как именно создается имя, но учитывая синтаксис (без точек и символов в верхнем регистре)
- Вот так это будет выглядеть в проекте `приемнике` запросов (не забыть присвоить

      networks:
          laravel:
          laravel_example_container_app-shared:
            external: true

---

# Проблемы с контейнером:

- Нет прав на запись в директории storage. Как решить? в создании образа docker нужно чтобы в директории с проектом устанавливать права `www-data` при формировании контейнера, если это не получается, пока обходиться временными костылями - заходить в контейнер php, затем присваивать роль с помощью команды `chown -R www-data storage/` Пока всё.