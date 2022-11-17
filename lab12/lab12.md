# Лабораторная работа №12
## Что такое docker-compose и зачем его применять?

`Docker-compose` - это средство, разработанное для помощи в определении запуска docker образов. Все что можно выполниять с помощью команды `docker` (для запуска/остановки контейнера) можно упаковать в один файл и, с помощью 2-х команд, запускать и останавливать контейнеры.

Основным компонентом docker-compose является `docker-compose.yaml` файл. В данном файле необходимо описать какие контейнеры нужно запустить, какие переменные в них необходимо передать, в какой сети будут запускаться контейнеры и тд.

---
## Небольшое сравнение "docker-compose up" с "docker run"

Рассмотрим небольшой пример `docker-compose.yaml` файла:

```yaml
version : "3"

services:
  mongo:
    container_name: mongo
    image: mongo:latest
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: root
      MONGO_INITDB_DATABASE: test-db
    ports:
     - 27017:27017
    volumes:
     - ./mongodb-data/:/data/db
```
В данном примере описан простой `docker-compose` файл (в формате `yaml`) для запуска контейнера с базой данных [mongodb](https://docs.mongodb.com/). Создайте в пустой папке файл `docker-compose.yaml` и скопируйте содержимое из примера в данный файл. Для использования файла для запуска контейнера введите команду в консоле:
```bash
cd /путь/до/файла/

docker-compose up -d
# если не получится, попробуйте использовать sudo:
# sudo docker-compose up -d
```
Проверку запуска можно выполнить с помощью команд:
```bash
docker ps
# если не получится, попробуйте использовать sudo:
# sudo docker ps
```
Остановим контейнер, для этого воспользуемся командой:
```bash
cd /путь/до/файла/

docker-compose down
# если не получится, попробуйте использовать sudo:
# sudo docker-compose down
```

С помощью `docker run` можно выполнить все тоже самое. Команда для запуска будет выглядеть следующим образом (**команда для примера, запускать ее не нужно!!!**):
```bash
docker run -d \
    --name mongo \
    --restart always \
    -e MONGO_INITDB_ROOT_USERNAME=root \
    -e MONGO_INITDB_ROOT_PASSWORD=root \
    -e MONGO_INITDB_DATABASE=test-db \
    -p 27017:27017\
     -v ~/path/to/mongodb-data/:/data/db \
    mongo:latest
```
Команда выглядит громоздкой. Запомнить ее очень сложно, а еще сложнее отследить через месяц после запуска контейнера, что и с какими параметрами было запущено. Поэтому голый `docker run` почти никогда не используют. Вместо него используют "оболочки" работующие поверх, такие как `docker-compose`.

>Теперь, зная команды запуска и остановки контейнеров через `docker-compose`, вы можете самостоятельно проверить данный пример.

---
## Компоненты docker-compose файла

Рассмотрим каждую строчку `docker-compose.yaml` файла в отдельности (для получения подробной информации, а также, для изучения дополнительных возможностей `docker-compose`, советую обратиться к [официальному мануалу](https://docs.docker.com/compose/compose-file/compose-file-v3/)).

---
### Version
`version : "3"` - это версия самого приложения `docker-compose` На текущий момент версия 3 является самой последней актуальной версией `docker-compose`.

---
### Services
`services:` - это заголовок, в котором указывается перечисление контейнеров для запуска. Например, имеется задача по установки 2-х баз данных `mongo`, тогда `docker-compose.yaml` файл может выглядеть следующим образом:

```yaml
version : "3"

services:
  mongo1:
    image: mongo:latest
    restart: always
  mongo2:
    image: mongo:latest
    restart: always
```
Данный `docker-compose.yaml` файл запустит 2 экземпляра базы `mongo`. Ключи `mongo1` и `mongo2` - это названия сервисов (название может быть любым). Под каждым сервисом описываются настройки для запуска своего контейнера. Контейнеры могут быть любыми (не обязательно 2 экземпляра одного и того же приложения):

```yaml
version : "3"

services:
  mongo:
    image: mongo:latest
    restart: always
  mysql:
    image: mysql:latest
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: example
```

В этом примере запускаются 2 контейнера с 2 разными БД (`mongo` и `mysql`).

Стоит также уточнить, что оба контейнера никак друг с другом не связаны. `docker-compose` не связывает контейнеры, он служит только для запуска и их остановки.

---
### Image
`image` - это название образа, который будет использоваться для запуска контейнера.

На прошлой лабораторной работе мы использовали `Dockerfile` для создания образа. Для сборки образов использовали команду:

```bash
cd /путь/до/Dockerfile/

docker build -t <my_image_name> ./
```

`<my_image_name>` - это название образа, который мы получили после его создания (название может быть любым). Данное название передается в `image` в `docker-compose.yaml` файле:

```yaml
version : "3"

services:
  mongo:
    image: <my_image_name>
    restart: always
```

---
### container_name
`container_name` - это параметр, который переопределяет название запущенного контейнера (это не название образа, это название контейнера, который будет запущен, значение можно задать любое). Данное название работает как некая ссылка на запущенный контейнер (на ровне с `id` запущенного контейнера).

Данную ссылку используют во многих местах. Например, на прошлой лабораторной работе, для проверки логов запущенного контейнера приходилось использовать `id` с помощью команды:
```bash
docker logs <container_id>
```
Теперь, при явном указании `container_name`, обращаться к запущенному контейнеру можно через его имя:
```bash
docker logs mongo
```

Также, стоит отметить, что данное название должно быть индивидуальным для каждого запущенного контейнера.

Для проверки названия запущенного образа, необходимо выполнить команду:
```bash
docker ps
```
---
### restart
`restart` - политика перезапуска контейнера. Она необходима для указания правила в случаях ошибки выполнения программы внутри контейнера. Например, если программа завершилась с ошибкой, при политике `restart: always` контейнер перезапустится.

Существует 4 политики перезапуска, о данных политиках можно подробно почитать в официальной [документации](https://docs.docker.com/compose/compose-file/compose-file-v3/#restart).

Рассмотрим пример (где запуск программы в `docker` может завершиться с ошибкой):
```yaml
version : "3"

services:
  mysql:
    container_name: mysql
    image: mysql:latest
    restart: always
```

Данное приложение запущено с политикой `restart: always` и имеет ошибку. Проверим состояние контейнера:
```bash
docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS                                  PORTS     NAMES
13a4c94d9f65   mysql:latest   "docker-entrypoint.s…"   4 seconds ago   Restarting (1) Less than a second ago             mysql
```
В состоянии под колонкой `STATUS` указано, что контейнер постоянно перезагружается.

Проверим логи приложения:
```log
docker logs mysql
2021-10-02 13:21:42+00:00 [ERROR] [Entrypoint]: Database is uninitialized and password option is not specified
    You need to specify one of the following:
    - MYSQL_ROOT_PASSWORD
    - MYSQL_ALLOW_EMPTY_PASSWORD
    - MYSQL_RANDOM_ROOT_PASSWORD
```

Приложение указывает на то, что не был передан пароль через переменные окружения.
Остановим контейнер, и добавим переменную:

```yaml
version : "3"

services:
  mysql:
    image: mysql:latest
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: example
```

---
### Environment

`environment` указывает на переменные окружения, которые необходимо передать в контейнер, при его запуске. Например, можно передать переменную `TZ`, или любую другую переменную, которая используется вашим приложением.

Например, при запуске контейнера с БД `mongo`, можно передать логин/пароль пользователя и название БД:
```yaml
version : "3"

services:
  mongo:
    container_name: mongo
    image: mongo:latest
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: root
      MONGO_INITDB_DATABASE: test-db
```

---
### Volumes
`volumes` - указывает на массив монтированных директорий, файлов. Так как в примере запускается БД, необходимо сохранять данные во вне контейнера (ведь, если контейнер остановится, данные будут уничтожены вместе с ним):
```yaml
version : "3"

services:
  mongo:
    container_name: mongo
    image: mongo:latest
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: root
      MONGO_INITDB_DATABASE: test-db
    volumes:
     - ./mongodb-data/:/data/db
```
`./mongodb-data/:/data/db` указывает на то, что необходимо монтировать папку `./mongodb-data/` на лольной машине в папку `/data/db` внутри контейнера. Таким образом можно передавать различные файлы внутрь программы (например, конфиги приложения), а также сохранять полученные файлы во время работы контейнера (данные БД).


---
### Ports

`ports` - указывает правила проброса портов изнутри контейнеера наружу.

В докере по-умолчанию имеется своя подсеть. При создании контейнера, каждому из них выдается свой адрес в этой подсети (взаимодействие идет через NAT). Контейнер может взаимодействовать с внешней сетью через общий шлюз докер подсети (например, скачивать внутрь себя приложения). Но из вне (за NAT-ом) мы не может взаимодействовать с внутренними приложениями контейнера (обращаться к приложению внутри докера по определенному порту). Для этого используется проброс портов через NAT, который указывается через переменную ports:
```yaml
version : "3"

services:
  mongo:
    container_name: mongo
    image: mongo:latest
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: root
      MONGO_INITDB_DATABASE: test-db
    ports:
     - 9090:27017
```

Таким образом, проброс для порта 27017 выглядит так:
```
0.0.0.0:9090->(адрес выданный контейнеру из подсети docker):27017
```

Таким образом, обращаясь к адресу `0.0.0.0:9090` из локальной консоли вы будете обращаться к приложению, работающему по адресу `0.0.0.0:27017` внутри контейнера.

Для изучение сетей в docker будет выделено отдельное занятие.


---
## Задание

1.  Запустить программу написанную на прошлой лабораторной работе через `docker-compose`. Задание считается выполненным, если:
    - имеется `docker-compose.yaml` файл;
    - запуск и остановка контейнера осуществляются с помощью команд `docker-compose up -d` и `docker-compose down`;
    - С помощью команды `docker logs <id/name>`отображаются логи запущенного контейнера.

2.  Изучить приложения `prometheus` и `grafana`. Написать `docker-compose.yaml` для их заупска. Задание считается выполненным, если:
    - имеется `docker-compoes.yaml` файл;
    - запуск и остановка контейнера осуществляются с помощью команд `docker-compose up -d` и `docker-compose down`;
    - Приложения открываются через Web UI (браузер).
