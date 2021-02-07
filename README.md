Deployment приложений (php, ruby, c#) средствами Docker.
---
Что такое Docker?
--
Wiki: Docker — программное обеспечение для автоматизации развёртывания и управления приложениями в средах с поддержкой контейнеризации. Позволяет «упаковать» приложение со всем его окружением и зависимостями в контейнер, который может быть перенесён на любую Linux-систему с поддержкой cgroups в ядре, а также предоставляет среду по управлению контейнерами.

Самый лучший способ познакомиться с Docker — выполнить команду 

`docker run -dp 80:80 docker/getting-started`

и открыть в браузере адрес http://localhost. Даже минимального уровня 
английского достаточно для того, чтобы понять, что там написано.

Создание окружения для разработки
--
Php - см. docker-compose.yml

C# - ПКМ->Добавить->Поддержка Docker

Запаковка приложения в контейнер и размещение в корпоративном реестре
--
Основное место хранения контейнеров - hub.docker.com. 
Там же есть контейнер, в котором можно хранить образы контейнеров:

`docker run -dp 5000:5000 --name registry registry:2`

Сборка образа производится командой:

`docker build -t php-test-app:1.0 .`

где php-test-app - имя образа, 
1.0 - имя тэга, 
. - текущая директория. Сделать новый образ на основе уже существующего можно командой

`docker image tag php-test-app localhost:5000/php-test-app:1.0`

где localhost:5000 - url, на котором висит локальный registry.
Таким образом, url развернутого registry включается в название образа.
Разумеется, название с url можно указать сразу при создании образа:

`docker build -t localhost:5000/php-test-app:1.0 .`

Публикация контейнеров производится командой:

`docker push localhost:5000/php-test-app:1.0`

Проверить, что образ успешно опубликован можно командами

`docker image rm localhost:5000/php-test-app:1.0 && docker pull localhost:5000/php-test-app:1.0`

Ввод в эксплуатацию приложения новой версии на живом сервере (ручной деплоймент)
--
Для ручного обновления версии приложения с использованием Docker нужно, чтобы на сервере приложение 
работало внутри Docker. Один из вариантов - использовать docker-compose.yml подобный тому, который 
используется для запуска приложения на локальном компьютере, но в качестве источника будет указан 
образ с запакованным приложением:

`image: localhost:5000/php-test-app:1.0`, см docker-compose-live.yml

Разумеется, вместо localhost:5000 будет другой URL, по которому доступен локальный Registry.
Приложение в таком случае запускается командой

Для запуска новой версии приложения при такой схеме нужно выполнить несколько команд:
- Собрать и выложить новую версию контейнера с приложением:
  `docker build -t localhost:5000/php-test-app:1.1 . && docker push localhost:5000/php-test-app:1.1`
- обновить версию контейнера в файле docker-compose.yml на сервере
- Перезапустить docker-compose: `docker-compose up -d`

Выполнение последней команды вызовет скачивание новой версии контейнера с приложением, 
пересоздание и запуск контейнеров.

Настройка автоматического деплоймента средствами Gitlab-CI
--
В случае использования средств CI/CD можно автоматизировать вышеописанные действия, написав для
этого сценарий:


Так же, в случае использования CI/CD есть возможность обновления сервера, где приложение работает
без использования Docker. В этом случае схема выглядит примерно так: скачивается новая версия образа, 
запускается контейнер, внутри контейнера выполняется rsync нужной директории, при необходимости 
запускаются скрипты миграций и прочее:
