<a id="top"></a>
<img src="https://raw.githubusercontent.com/protsenkovi/docker-curriculum/master/images/logo.png" alt="docker logo">

*Научитесь легко создавать и разворачивать распределённые приложения в облаке с помощью Docker.*

*На основе статьи https://prakhar.me/docker-curriculum/ и её перевода https://habrahabr.ru/post/310460/.*

<a href="#top" class="top" id="getting-started">Top</a>

# Часть 2

<a href="#top" class="top" id="table-of-contents">Top</a>
## Оглавление

    - [3 Многоконтейнерные окружения](#multi-container)
        - [3.1 Приложение для поиска фургонов c едой в Сан-Франциско (SF Food Trucks)](#foodtrucks)
        - [3.2 Сетевая инфраструктура Docker](#docker-network)
        - [3.3 Docker Compose](#docker-compose)
        - [3.4 AWS Elastic Container Service](#aws-ecs)
    - [4 Заключение](#wrap-up)
        - [4.1 Что дальше?](#next-steps)
        - [4.2 Обратная связь](#feedback)
    - [Источники](#references)

------------------------------

<a href="#table-of-contents" class="top" id="preface">Top</a>
<a id="multi-container"></a>

## 3.0 Многоконтейнерные окружения

В прошлый раз мы увидели, как легко и просто запускать приложения с помощью Docker. Мы начали с простого статического сайта, а потом запустили Flask-приложение. Оба варианта можно было запускать локально или в облаке, несколькими командами. И то, и другое приложения работали в **одном контейнере**.

Если у вас был опыт управления сервисами в эксплуатируемом конечными пользователями приложении, то вы знаете, что современные приложения не такие простые. Как правило всегда используется база данных или другой тип постоянного хранилища. Системы [Redis](http://redis.io/) и [Memcached](http://memcached.org/) стали практически обязательной частью архитектуры веб-приложения. Поэтому, в этом разделе мы научимся "докеризировать" приложения, которым требуется несколько запущенных сервисов.

В частности, вы увидим, как запускать и управлять **многоконтейнерными** Docker-окружениями. Почему нужно несколько контейнеров, спросите вы? Ну, одна из главных идей Docker в том, что он предоставляет изоляцию. Идея совмещения процесса и его зависимостей в одной песочнице (называемой контейнером) и делает Docker мощным инструментом.

Когда монолитное приложение становится слишком сложным для поддержки в распределённой среде мудрым решением является произвести его декомпозицию на компоненты-сервисы. Хорошей идеей является содержание **сервисов** в отдельных контейнерах. Разным компонентам скорее всего потребуются разные ресурсы, и необходимость в новых ресурсах может возникать в разной степени. Помещая компоненты в отдельные контейнеры, мы можем выделять наиболее подходящий тип ресурсов для каждой части приложения. Это также созвучно со всем [микросервисным](http://martinfowler.com/articles/microservices.html) движением. Это одна из причин, по которой Docker (и любая другая технология контейнеризации) находится на [передовой](https://medium.com/aws-activate-startup-blog/using-containers-to-build-a-microservices-architecture-6e1b8bacb7d1#.xl3wryr5z) современных микросервисных архитектур. 

<a id="foodtrucks"></a>
### 3.1 Приложение для поиска фургонов c едой в Сан-Франциско

Приложение, которое мы переведём в Docker, называется SF Food Trucks. Приложение создавалось с целью сделать что-то похожее на реально эксплуатируемое приложение и приносещее пользу, но не слишком сложное для данного пособия. Вот что получилось.

<img src="https://raw.githubusercontent.com/prakhar1989/FoodTrucks/master/shot.png" alt="sf food trucks">

Серверная часть написана на Python (Flask framework), а для поиска используется [Elasticsearch](https://www.elastic.co/products/elasticsearch). Как и всё остальное в этом обучении, код проекта находится на [Github](http://github.com/prakhar1989/FoodTrucks). Мы используем это приложение, чтобы научиться запускать и разворачивать многоконтейнерное окружение.

Скачайте код проекта на хост машину командой `git clone http://github.com/prakhar1989/FoodTrucks`. 

Теперь, когда вы воодушевлены (надеюсь), давайте подумаем, как будет выглядеть этот процесс. В нашем приложении есть бэкенд на Flask и сервис Elasticsearch. Естественным образом можно разделить приложение на два контейнера: один для Flask, другой для Elasticsearch (ES). Если приложение станет популярным, мы сможем масштабировать приложение добавлением новых контейнеров для тех компонентов, которые станут узким местом. 


Отлично, значит нам нужно два контейнера. Это не сложно, правда? Мы уже создавали Flask-контейнер в прошлый раз. А для Elasticsearch... давайте посмотрим, есть ли что-нибудь в репозитории:

```
$ docker search elasticsearch
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
elasticsearch                     Elasticsearch is a powerful open source se...   2183      [OK]
itzg/elasticsearch                Provides an easily configurable Elasticsea...   51                   [OK]
nshou/elasticsearch-kibana        Elasticsearch-5.2.2 Kibana-5.2.2                24                   [OK]
barnybug/elasticsearch            Latest Elasticsearch 1.7.2 and previous re...   15                   [OK]
digitalwonderland/elasticsearch   Latest Elasticsearch with Marvel & Kibana       14                   [OK]
```

Quite unsurprisingly, there exists an officially supported [image](https://hub.docker.com/_/elasticsearch/) for Elasticsearch. To get ES running, we can simply use `docker run` and have a single-node ES container running locally within no time.

Не удивительно, но существует официальный [образ](https://hub.docker.com/_/elasticsearch/) для Elasticsearch. Чтобы запустит ES, нужно всего лишь выполнить `docker run`, и вскоре у нас будет локальный, работающий контейнер с одним узлом ES.

> Замечание. Если контейнер не запускается, попробуйте подключиться в интерактивном режиме (с ключом -it) и выяснить причину ошибки. Одной из причин может быть нехватка памяти для Elasticsearch. 
> Для версии старше 5 требуется минимум 2 Гб. Проблема также может быть в хост машине. Увеличьте количество доступных кусочков памяти процессу выполнением команды `sysctl -w vm.max_map_count=262144`. Обратите внимание, что это модифицирует свойство системы, в которой может выполняться множество других контейнеров.


```
$ docker run -dp 9200:9200 elasticsearch
d582e031a005f41eea704cdc6b21e62e7a8a42021297ce7ce123b945ae3d3763

$ curl 0.0.0.0:9200
{
  "name" : "js54ey7",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "iDFG-pTSRDaknSNSuRnhOg",
  "version" : {
    "number" : "5.3.0",
    "build_hash" : "3adb13b",
    "build_date" : "2017-03-23T03:31:50.652Z",
    "build_snapshot" : false,
    "lucene_version" : "6.4.1"
  },
  "tagline" : "You Know, for Search"
}

```

Заодно давайте запустим контейнер с Flask. Но вначале нужен `Dockerfile`. В прошлой секции мы использовали образ `python:3-onbuild` в качестве базового. Однако, в этом раз, кроме установки зависимостей через `pip`, нам нужно, чтобы приложение генерировало Javascript файл. Для этого потребуется Nodejs. В связи с появлением дополнительных файлов для работы контейнера нам нужно построить новый образ. Начнем с базового образа `ubuntu`.

> Замечание. Если оказывается, что существующий образ не подходит для вашей задачи, то спокойно создавайте свой образ на основе другого базового образа. В большинстве случаев, для образов на Docker Hub можно найти соответствующий `Dockerfile` на Github. Почитайте существующие докерфайлы — это один из лучших способов научиться делать свои образы.


Наш  [Dockerfile](https://github.com/prakhar1989/FoodTrucks/blob/master/Dockerfile) для Flask-приложения выглядит следующим образом:
```
# start from base
FROM ubuntu:14.04
MAINTAINER Prakhar Srivastav <prakhar@prakhar.me>

# install system-wide deps for python and node
RUN apt-get -yqq update
RUN apt-get -yqq install python-pip python-dev
RUN apt-get -yqq install nodejs npm
RUN ln -s /usr/bin/nodejs /usr/bin/node

# copy our application code
ADD flask-app /opt/flask-app
WORKDIR /opt/flask-app

# fetch app specific deps
RUN npm install
RUN npm run build
RUN pip install -r requirements.txt

# expose port
EXPOSE 5000

# start app
CMD [ "python", "./app.py" ]
```

Тут много всего нового. Вначале указан базовый образ [Ubuntu LTS](https://wiki.ubuntu.com/LTS), потом используется пакетный менеджер `apt-get` для установки зависимостей, в частности: Python и Node. Флаг `yqq` нужен для игнорирования вывода и автоматического выбора "Yes" во всех диалогах. Также создается символическая ссылка для бинарного файла `node`. Это нужно для решения проблем обратной совместимости.

Потом мы используем команду `ADD` для копирования приложения в нужную директорию в контейнере — `/opt/flask-app`. Здесь будет находиться весь наш код. Мы также устанавливаем эту директорию в качестве рабочей, так что следующие команды будут выполняться в контексте этой локации. Теперь, когда наши системные зависимости установлены, пора установить зависимости уровня приложения. Начнем с Node, установки пакетов из npm и запуска команды сборки, как указано в нашем `package.json` [файле](https://github.com/prakhar1989/FoodTrucks/blob/master/flask-app/package.json#L7-L9). В конце устанавливаем пакеты Python, открываем порт и определяем запуск приложения с помощь `CMD`, как в предыдущем разделе.

Наконец, можно собрать образ и запустить контейнер.
```
$ docker build -t <имя пользователя Docker>/<имя образа>.
```

При первом запуске нужно будет больше времени, так как клиент Докера будет скачивать образ `ubuntu`, запускать все команды и готовить образ. Повторный запуск `docker build` после последующих изменений будет практически моментальным. Давайте попробуем запустить приложение.

```
$ docker run -P <имя пользователя Docker>/<имя образа>
Unable to connect to ES. Retying in 5 secs...
Unable to connect to ES. Retying in 5 secs...
Unable to connect to ES. Retying in 5 secs...
Out of retries. Bailing out...
```

Упс! Наше приложение не смогло запуститься, потому что оно не может подключиться к Elasticsearch. Как сообщить одному контейнеру о другом и как заставить их взаимодействовать друг с другом? Ответ в следующей секции.

<a id="docker-network"></a>
### 3.2 Сетевая инфраструктура Docker

Перед тем, как обсудить возможности Docker для решения описанной задачи, давайте посмотрим на возможные варианты обхода проблемы. Думаю, это поможет нам оценить удобство той функциональности, которую мы вскоре изучим.

Ладно, давайте запустим `docker ps`. Что тут у нас:

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                              NAMES
c31bf3beb299        elasticsearch       "/docker-entrypoin..."   2 hours ago         Up 2 hours          0.0.0.0:9200->9200/tcp, 9300/tcp   tender_wilson
```

Итак, у нас есть контейнер ES по адресу и порту `0.0.0.0:9200`, и мы можем напрямую обращаться к нему. Если можно было бы сообщить нашему приложению подключаться к этому адресу, то оно сможет общаться с ES, верно? Давайте взглянем на [код на Питоне](https://github.com/prakhar1989/FoodTrucks/blob/master/flask-app/app.py#L7), туда, где описано подключение.

So we have one ES container running on `0.0.0.0:9200` port which we can directly access. If we can tell our Flask app to connect to this URL, it should be able to connect and talk to ES, right? Let's dig into our [Python code](https://github.com/prakhar1989/FoodTrucks/blob/master/flask-app/app.py#L7) and see how the connection details are defined.

```
es = Elasticsearch(host='es')
```

Для того, чтобы это заработало, нужно сообщить Flask контейнеру, что контейнер ES запущен на хосте 0.0.0.0 (порт по умолчанию 9200), и всё заработает, да? К сожалению, нет, потому что IP 0.0.0.0 это адрес для доступа к контейнеру с `хост-машины` (с виртуальной машины boot2docker или вашей основной машины). Другой контейнер не сможет обратиться по этому адресу. Ладно, если не этот адрес, то какой тогда адрес нужно использовать для работы с контейнером ES? Рад, что вы спросили.

Это хороший момент, чтобы изучить работу сети в Docker. После установки, Docker автоматически создает три сети:

```
$ docker network ls
NETWORK ID          NAME                DRIVER
075b9f628ccc        none                null
be0f7178486c        host                host
8022115322ec        bridge              bridge
```

Сеть **bridge** — это сеть, в которой контейнеры запущены по умолчанию. Это значит, что когда я запускаю контейнер ES, он работает в bridge сети. Чтобы удостовериться, давайте проверим:


```
$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "8022115322ec80613421b0282e7ee158ec41e16f565a3e86fa53496105deb2d7",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Config": [
                {
                    "Subnet": "172.17.0.0/16"
                }
            ]
        },
        "Containers": {
            "e931ab24dedc1640cddf6286d08f115a83897c88223058305460d7bd793c1947": {
                "EndpointID": "66965e83bf7171daeb8652b39590b1f8c23d066ded16522daeb0128c9c25c189",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        }
    }
]
```

Видно, что контейнер `e931ab24dedc` находится в секции `Containers`. Также виден IP-адрес, выданный этому контейнеру — `172.17.0.2`. Именно этот адрес мы и искали? Давайте проверим: запустим Flask-приложение и попробуем обратиться к нему по IP:

```
$ docker run -it --rm <имя пользователя Docker>/<имя образа> bash
root@35180ccc206a:/opt/flask-app# curl 172.17.0.2:9200
bash: curl: command not found
root@35180ccc206a:/opt/flask-app# apt-get -yqq install curl
root@35180ccc206a:/opt/flask-app# curl 172.17.0.2:9200
{
  "name" : "Jane Foster",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "2.1.1",
    "build_hash" : "40e2c53a6b6c2972b3d13846e450e66f4375bd71",
    "build_timestamp" : "2015-12-15T13:05:55Z",
    "build_snapshot" : false,
    "lucene_version" : "5.3.1"
  },
  "tagline" : "You Know, for Search"
}
root@35180ccc206a:/opt/flask-app# exit
```

Cейчас всё должно стать понятно. Мы запустили контейнер в интерактивном режиме с процессом bash. Флаг --rm нужен для удобства, благодаря нему контейнер автоматически удаляется после выхода. Мы попробуем curl, но нужно сначала установить его. После этого можно удостовериться, что по адресу `172.17.0.2:9200` на самом деле можно обращаться к ES! Супер!

Не смотря на то, что мы нашли способ наладить связь между контейнерами, существует несколько проблем с этим подходом:

1. Придется добавлять записи в файл `/etc/hosts` (локальный DNS) внутри Flask-контейнера, чтобы приложение понимало, что имя хоста `es` означает `172.17.0.2`. Если IP-адрес меняется, то придется вручную менять запись.

2. Так как сеть *bridge* используется всеми контейнерами по умолчанию, этот метод **не безопасен**.

Но есть хорошие новости: в Docker есть отличное решение этой проблемы. Docker позволяет создавать собственные изолированные сети. Это решение также помогает справиться с проблемой `/etc/hosts`, сейчас увидим как.

Во-первых, давайте создадим свою сеть. 

```
$ docker network create foodtrucks
1a3386375797001999732cb4c4e97b88172d983b08cd0addfcb161eed0c18d89

$ docker network ls
NETWORK ID          NAME                DRIVER
1a3386375797        foodtrucks          bridge
8022115322ec        bridge              bridge
075b9f628ccc        none                null
be0f7178486c        host                host
```

Команда `network create` создаёт новую сеть *bridge*. Нами сейчас нужен именно этот тип сети. Существуют и другие типы, о которых вы можете прочитать в [официальной документации](https://docs.docker.com/engine/userguide/networking/dockernetworks/).

Теперь у нас есть сеть. Можно запустить наши контейнеры внутри сети с помощью флага `--net`. Давайте так и сделаем, но сначала остановим контейнер с ElasticSearch, который был запущен в сети bridge по умолчанию.


```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                              NAMES
e931ab24dedc        elasticsearch       "/docker-entrypoint.s"   4 hours ago         Up 4 hours          0.0.0.0:9200->9200/tcp, 9300/tcp   cocky_spence

$ docker stop e931ab24dedc
e931ab24dedc

$ docker run -dp 9200:9200 --net foodtrucks --name es elasticsearch
2c0b96f9b8030f038e40abea44c2d17b0a8edda1354a08166c33e6d351d0c651

$ docker network inspect foodtrucks
[
    {
        "Name": "foodtrucks",
        "Id": "1a3386375797001999732cb4c4e97b88172d983b08cd0addfcb161eed0c18d89",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Config": [
                {}
            ]
        },
        "Containers": {
            "2c0b96f9b8030f038e40abea44c2d17b0a8edda1354a08166c33e6d351d0c651": {
                "EndpointID": "15eabc7989ef78952fb577d0013243dae5199e8f5c55f1661606077d5b78e72a",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {}
    }
]
```

Мы сделали то же, что и раньше, но на этот раз дали контейнеру название `es`. Перед тем, как запускать контейнер с приложением, давайте проверим что происходит, когда запуск происходит в сети.

```
$ docker run -it --rm --net foodtrucks <имя пользователя Docker>/<имя образа> bash
root@53af252b771a:/opt/flask-app# cat /etc/hosts
172.18.0.3	53af252b771a
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.18.0.2	es
172.18.0.2	es.foodtrucks

root@53af252b771a:/opt/flask-app# curl es:9200
bash: curl: command not found
root@53af252b771a:/opt/flask-app# apt-get -yqq install curl
root@53af252b771a:/opt/flask-app# curl es:9200
{
  "name" : "Doctor Leery",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "2.1.1",
    "build_hash" : "40e2c53a6b6c2972b3d13846e450e66f4375bd71",
    "build_timestamp" : "2015-12-15T13:05:55Z",
    "build_snapshot" : false,
    "lucene_version" : "5.3.1"
  },
  "tagline" : "You Know, for Search"
}
root@53af252b771a:/opt/flask-app# ls
app.py  node_modules  package.json  requirements.txt  static  templates  webpack.config.js
root@53af252b771a:/opt/flask-app# python app.py
Index not found...
Loading data in elasticsearch ...
Total trucks loaded:  733
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
root@53af252b771a:/opt/flask-app# exit
```

Ура! Работает! Магическим образом Docker теперь разрешает имя es в нужный ip, и поэтому `es:9200` можно использовать в приложении — этот адрес корректно направляет запросы в контейнер ES. Отлично! Давайте теперь запустим Flask-контейнер по-настоящему:

```
$ docker run -d --net foodtrucks -p 5000:5000 --name foodtrucks-web <имя пользователя Docker>/<имя образа>
2a1b77e066e646686f669bab4759ec1611db359362a031667cacbe45c3ddb413

$ docker ps
CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                              NAMES
2a1b77e066e6        prakhar1989/foodtrucks-web   "python ./app.py"        2 seconds ago       Up 1 seconds        0.0.0.0:5000->5000/tcp             foodtrucks-web
2c0b96f9b803        elasticsearch                "/docker-entrypoint.s"   21 minutes ago      Up 21 minutes       0.0.0.0:9200->9200/tcp, 9300/tcp   es

$ curl -I 0.0.0.0:5000
HTTP/1.0 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 3697
Server: Werkzeug/0.11.2 Python/2.7.6
Date: Sun, 10 Jan 2016 23:58:53 GMT
```

Зайдите на [http://0.0.0.0:5000](http://0.0.0.0:5000) (или на ip виртуальной машны вместо 0.0.0.0), и увидите приложение в работе. Опять же, может показаться, что было много работы, но на самом деле мы ввели всего 4 команды чтобы с нуля дойти до работающего приложения. Все команды, которые мы проделали собраны в [bash скрипте](https://github.com/prakhar1989/FoodTrucks/blob/master/setup-docker.sh).

```
#!/bin/bash

# build the flask container
docker build -t <имя пользователя Docker>/<имя образа> .

# create the network
docker network create foodtrucks

# start the ES container
docker run -d --net foodtrucks -p 9200:9200 -p 9300:9300 --name es elasticsearch

# start the flask app container
docker run -d --net foodtrucks -p 5000:5000 --name foodtrucks-web prakhar1989/foodtrucks-web
```

Теперь представьте, что хотите поделиться приложением с другом. Или хотите запустить на сервере, где установлен Docker. Можно запустить всю систему с помощью одной команды!

```
$ git clone https://github.com/prakhar1989/FoodTrucks
$ cd FoodTrucks
$ ./setup-docker.sh
```

Вот и все! По-моему, это невероятно крутой и мощный способ распространять и запускать приложения!

<a id="docker-links"></a>
##### Docker Links

Перед тем, как завершить этот раздел, стоит отметить, что `docker network` это относительно новая фича, она входит в релиз Docker 1.9 [release](https://blog.docker.com/2015/11/docker-1-9-production-ready-swarm-multi-host-networking/). До того, как появился `network`, ссылки были допустимым способом настройки взаимодействия между контейнерами. В соответствии с официальной документацией [docs](https://docs.docker.com/engine/userguide/networking/default_network/dockerlinks/), `linking` вскоре будет переведены в статус deprecated. Если вам попадется туториал или статья, где используется `link` для соединения контейнеров, то просто не забывайте использовать вместо этого network (на момент публикации перевода links является legacy, — прим. пер.)


<a id="docker-compose"></a>
### 3.3 Docker Compose

До этого момента мы изучали клиент Docker. Но в Docker экосистеме есть несколько других инструментов с открытым исходным кодом, которые хорошо взаимодействуют с Docker. Некоторые из них это:

1. [Docker Machine](https://docs.docker.com/machine/) позволяет создавать Докер-хосты на своем компьютере, облачном провайдере или внутри дата-центра.
2. [Docker Compose](https://docs.docker.com/compose/) — инструмент для определения и запуска много-контейнерных приложений.
3. [Docker Swarm](https://docs.docker.com/swarm/) — решение для кластеризации.

В этом разделе мы поговорим об одном из этих инструментов — Docker Compose, и узнаем, как он может упростить работу с несколькими контейнерами.

У Docker Compose довольно интересная предыстория. Примерно три года назад компания OrchardUp запустила инструмент под названием Fig. Идея была в том, чтобы создавать изолированные рабочие окружения с помощью Docker. Проект очень хорошо восприняли на Hacker News [Hacker News](https://news.ycombinator.com/item?id=7132044).

[Первый комментарий](https://news.ycombinator.com/item?id=7133449) на самом деле неплохо объясняет, зачем нужен Fig и что он делает:

> На самом деле, смысл Docker в следующем: запускать процессы. Сегодня у Docker есть неплохое API для запуска процессов: создание общих между контейнерами (иными словами, запущенными образами) разделов или директорий (shared volumes), перенаправление портов с хост-машины в контейнер, вывод логов, и так далее. Но больше ничего: Докер сейчас работает только на уровне процессов.

> Не смотря на то, что в нём содержатся некоторые возможности оркестрации несколькими контейнерами для создания единого "приложения", в Docker нет ничего, что помогало бы с управлением такими группами контейнеров как одной сущностью. И вот зачем нужен инструмент вроде Fig: чтобы обращаться с группой контейнеров как с единой сущностью. Чтобы думать о "запуске приложений" (иными словами, "запуске оркестрированного кластера контейнеров") вместо "запуска контейнеров".

Оказалось, что многие пользователи Docker согласны с такими мыслями. Постепенно, Fig набрал популярность, Docker Inc. заметили, купили компанию и назвали проект Docker Compose.

Итак, зачем используется *Compose*? Это инструмент для простого определения и запуска многоконтейнерных Docker приложений. В нем есть файл `docker-compose.yml`, и с его помощью можно одной командой поднять приложение с набором сервисов.

Давайте посмотрим, сможем ли мы создать файл `docker-compose.yml` для нашего приложения SF-Foodtrucks и проверим, способен ли он на то, что обещает.

The first step, however, is to install Docker Compose. If you're running Windows or Mac, Docker Compose is already installed as it comes in the Docker Toolbox. Linux users can easily get their hands on Docker Compose by following the [instructions](https://docs.docker.com/compose/install/) on the docs. Since Compose is written in Python, you can also simply do `pip install docker-compose`. Test your installation with -

Однако вначале нужно установить Docker Compose. Есть у вас Windows или Mac, то Docker Compose уже установлен — он идет в комплекте с Docker Toolbox. На Linux можно установить Docker Compose следуя простым [инструкциям](https://docs.docker.com/compose/install/) на сайте документации. Compose написан на Python, поэтому можно сделать просто `pip install docker-compose`. Проверить работоспособность так:

```
$ docker-compose version
docker-compose version 1.12.0, build b31ff33
docker-py version: 2.2.1
CPython version: 2.7.13
OpenSSL version: OpenSSL 1.0.1t  3 May 2016

```

Теперь можно перейти к следующему шагу, то есть к созданию файла `docker-compose.yml`. Синтаксис `yml`-файлов очень простой, и в репозитории уже есть пример, который мы будем использовать

```
version: "2"
services:
  es:
    image: elasticsearch
  web:
    image: <имя пользователя Docker>/<имя образа>
    command: python app.py
    ports:
      - "5000:5000"
    volumes:
      - .:/code
```

Давайте разберём это подробнее. На родительском уровне мы задали название неймспейса для наших сервисов: `es` и `web`. К каждому сервису можно добавить дополнительные параметры, среди которых `image` — обязательный. Для `es` мы указываем доступный на Docker Hub образ `elasticsearch`. Для Flask-приложения — тот образ, который мы создали самостоятельно в начале этого раздела.

С помощью других параметров вроде `command` и `ports` можно предоставить информацию о контейнере. Параметр `volumes` отвечает за точку монтирования, где будет находиться код в контейнере `web`. Это опциональный параметр, он полезен, если нужно обращаться к логам и так далее. Подробнее о параметрах и возможных значениях можно прочитать в [документации](https://docs.docker.com/compose/compose-file).

> Замечание. Нужно находиться в директории с файлом `docker-compose.yml` чтобы запускать большую часть команд Compose.

Отлично! Файл готов, давайте посмотрим на `docker-compose` в действии. Но вначале нужно удостовериться, что порты свободны. Так что если у вас запущены контейнеры Flask и ES, то пора их остановить:

```
$ docker stop $(docker ps -q)
39a2f5df14ef
2a1b77e066e6
```

Теперь можно запускать `docker-compose`. Перейдите в директорию с приложением Foodtrucks и выполните команду `docker-compose up`.

```
$ docker-compose up
Creating network "foodtrucks_default" with the default driver
Creating foodtrucks_es_1
Creating foodtrucks_web_1
Attaching to foodtrucks_es_1, foodtrucks_web_1
es_1  | [2016-01-11 03:43:50,300][INFO ][node                     ] [Comet] version[2.1.1], pid[1], build[40e2c53/2015-12-15T13:05:55Z]
es_1  | [2016-01-11 03:43:50,307][INFO ][node                     ] [Comet] initializing ...
es_1  | [2016-01-11 03:43:50,366][INFO ][plugins                  ] [Comet] loaded [], sites []
es_1  | [2016-01-11 03:43:50,421][INFO ][env                      ] [Comet] using [1] data paths, mounts [[/usr/share/elasticsearch/data (/dev/sda1)]], net usable_space [16gb], net total_space [18.1gb], spins? [possibly], types [ext4]
es_1  | [2016-01-11 03:43:52,626][INFO ][node                     ] [Comet] initialized
es_1  | [2016-01-11 03:43:52,632][INFO ][node                     ] [Comet] starting ...
es_1  | [2016-01-11 03:43:52,703][WARN ][common.network           ] [Comet] publish address: {0.0.0.0} is a wildcard address, falling back to first non-loopback: {172.17.0.2}
es_1  | [2016-01-11 03:43:52,704][INFO ][transport                ] [Comet] publish_address {172.17.0.2:9300}, bound_addresses {[::]:9300}
es_1  | [2016-01-11 03:43:52,721][INFO ][discovery                ] [Comet] elasticsearch/cEk4s7pdQ-evRc9MqS2wqw
es_1  | [2016-01-11 03:43:55,785][INFO ][cluster.service          ] [Comet] new_master {Comet}{cEk4s7pdQ-evRc9MqS2wqw}{172.17.0.2}{172.17.0.2:9300}, reason: zen-disco-join(elected_as_master, [0] joins received)
es_1  | [2016-01-11 03:43:55,818][WARN ][common.network           ] [Comet] publish address: {0.0.0.0} is a wildcard address, falling back to first non-loopback: {172.17.0.2}
es_1  | [2016-01-11 03:43:55,819][INFO ][http                     ] [Comet] publish_address {172.17.0.2:9200}, bound_addresses {[::]:9200}
es_1  | [2016-01-11 03:43:55,819][INFO ][node                     ] [Comet] started
es_1  | [2016-01-11 03:43:55,826][INFO ][gateway                  ] [Comet] recovered [0] indices into cluster_state
es_1  | [2016-01-11 03:44:01,825][INFO ][cluster.metadata         ] [Comet] [sfdata] creating index, cause [auto(index api)], templates [], shards [5]/[1], mappings [truck]
es_1  | [2016-01-11 03:44:02,373][INFO ][cluster.metadata         ] [Comet] [sfdata] update_mapping [truck]
es_1  | [2016-01-11 03:44:02,510][INFO ][cluster.metadata         ] [Comet] [sfdata] update_mapping [truck]
es_1  | [2016-01-11 03:44:02,593][INFO ][cluster.metadata         ] [Comet] [sfdata] update_mapping [truck]
es_1  | [2016-01-11 03:44:02,708][INFO ][cluster.metadata         ] [Comet] [sfdata] update_mapping [truck]
es_1  | [2016-01-11 03:44:03,047][INFO ][cluster.metadata         ] [Comet] [sfdata] update_mapping [truck]
web_1 |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```

Перейдите по IP чтобы увидеть приложение. Круто, да? Всего лишь пара строк конфигурации и несколько Докер-контейнеров работают в унисон. Давайте остановим сервисы и перезапустим в detached mode:


```
web_1 |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
Killing foodtrucks_web_1 ... done
Killing foodtrucks_es_1 ... done

$ docker-compose up -d
Starting foodtrucks_es_1
Starting foodtrucks_web_1

$ docker-compose ps
      Name                    Command               State           Ports
----------------------------------------------------------------------------------
foodtrucks_es_1    /docker-entrypoint.sh elas ...   Up      9200/tcp, 9300/tcp
foodtrucks_web_1   python app.py                    Up      0.0.0.0:5000->5000/tcp
```

Не удивительно, но оба контейнера успешно запущены. Откуда берутся имена? Их Compose придумал сам. Но что насчет сети? Его *Compose* тоже делаем сам? Хороший вопрос, давайте выясним.

Для начала, остановим запущенные сервисы. Их всегда можно вернуть одной командой:

```
$ docker-compose stop
Stopping foodtrucks_web_1 ... done
Stopping foodtrucks_es_1 ... done
```

Заодно, давайте удалим сеть `foodtrucks`, которую создали в прошлый раз. Эта сеть нам не потребуется, потому что Compose автоматически сделает все за нас.

```
$ docker network rm foodtrucks
$ docker network ls
NETWORK ID          NAME                DRIVER
4eec273c054e        bridge              bridge
9347ae8783bd        none                null
54df57d7f493        host                host
```

Класс! Теперь в этом чистом состоянии можно проверить, способен ли *Compose* на волшебство.

```
$ docker-compose up -d
Recreating foodtrucks_es_1
Recreating foodtrucks_web_1
$ docker ps
CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                    NAMES
f50bb33a3242        prakhar1989/foodtrucks-web   "python app.py"          14 seconds ago      Up 13 seconds       0.0.0.0:5000->5000/tcp   foodtrucks_web_1
e299ceeb4caa        elasticsearch                "/docker-entrypoint.s"   14 seconds ago      Up 14 seconds       9200/tcp, 9300/tcp       foodtrucks_es_1
```

Пока все хорошо. Проверим, создались ли какие-нибудь сети:


```
$ docker network ls
NETWORK ID          NAME                 DRIVER
0c8b474a9241        bridge               bridge              
293a141faac3        foodtrucks_default   bridge              
b44db703cd69        host                 host                
0474c9517805        none                 null  
```

Видно, что *Compose* самостоятельно создал сеть `foodtrucks_default` и подсоединил оба сервиса в эту сеть, так, чтобы они могли общаться друг с другом. Каждый контейнер для сервиса подключен к сети, и оба контейнера доступны другим контейнерам в сети. Они доступны по `hostname`, который совпадает с названием контейнера. Давайте проверим, находится ли эта информация в `/etc/hosts`.


```
$ docker ps
CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                    NAMES
bb72dcebd379        prakhar1989/foodtrucks-web   "python app.py"          20 hours ago        Up 19 hours         0.0.0.0:5000->5000/tcp   foodtrucks_web_1
3338fc79be4b        elasticsearch                "/docker-entrypoint.s"   20 hours ago        Up 19 hours         9200/tcp, 9300/tcp       foodtrucks_es_1

$ docker exec -it bb72dcebd379 bash
root@bb72dcebd379:/opt/flask-app# cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.18.0.2	bb72dcebd379
```

Упс! Оказывается, файл понятия не имеет о `es`. Как же наше приложение работает? Давайте попингуем его по названию хоста:

```
root@bb72dcebd379:/opt/flask-app# ping es
PING es (172.18.0.3) 56(84) bytes of data.
64 bytes from foodtrucks_es_1.foodtrucks_default (172.18.0.3): icmp_seq=1 ttl=64 time=0.049 ms
64 bytes from foodtrucks_es_1.foodtrucks_default (172.18.0.3): icmp_seq=2 ttl=64 time=0.064 ms
^C
--- es ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 999ms
rtt min/avg/max/mdev = 0.049/0.056/0.064/0.010 ms
```

Вуаля! Работает! Каким-то магическим образом контейнер смог сделать пинг хоста `es`. Оказывается, Docker 1.10 добавили новую сетевую систему, которая производит обнаружение сервисов через DNS-сервер. Если интересно, то почитайте подробнее о [предложении](https://github.com/docker/libnetwork/issues/767) and [release notes](https://blog.docker.com/2016/02/docker-1-10/) и release notes.

На этом наш тур по Docker Compose завершен. С этим инструментом можно ставить сервисы на паузу, запускать отдельные команды в контейнере и даже масштабировать систему, то есть увеличивать количество контейнеров. Также советую изучать некоторые [другие примеры](https://docs.docker.com/compose/overview/#common-use-cases) использования Docker Compose.

Надеюсь, проделанные нами действия продемонстрировали как на самом деле просто управлять многоконтейнерной средой с *Compose*. В последнем разделе мы развернём всё на AWS!


<a id="aws-ecs"></a>
### 3.4 AWS Elastic Container Service

В прошлом разделе мы использовали `docker-compose` чтобы запустить наше приложение локально одной командой: `docker-compose up`. Теперь, когда приложение работает, мы хотим показать его миру, заполучить юзеров, поднять кучу денег и купить большой дом в Майами. Последние три шага выходят за пределы этого пособия, так что займемся выяснением деталей о развёртывании многоконтейнерного приложения в облаке AWS.

Если вы дочитали до этого места, то скорее всего убедились, что Docker — довольно полезная технология. И вы не одиноки. Облачные провайдеры заметили взрывной рост популярности Докера и стали добавлять поддержку в свои сервисы. Сегодня, Docker-приложения можно разворачивать на AWS,  [Azure](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-docker-vm-extension/), [Rackspace](http://blog.rackspace.com/docker-with-the-rackspace-open-cloud/), [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-use-the-digitalocean-docker-application) и много других. Мы уже умеем разворачивать приложение с одним контейнером на Elastic Beanstalk, а в этом разделе мы изучим [Elastic Container Service (or ECS)](https://aws.amazon.com/ecs/).

AWS ECS — это масштабируемый и гибкий сервис по управлению контейнерами, и он поддерживает Docker. С его помощью можно управлять кластером на EC2 через простой API. В Beanstalk настройки по умолчанию адекватны для большого количества задач, но ECS позволяет настроить каждый аспект окружения по вашим потребностям. По этой причине ECS — не самый лучший выбор для начало обучения.

К счастью, у ECS есть удобный инструмент командной строки ([CLI](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI.html)) с поддержкой Docker Compose и автоматической провизией на ECS! Так как у нас уже есть рабочий файл `docker-compose.yml`, настройка и запуск на AWS должна быть достаточно легкой. Начнем!

The first step is to install the CLI. As of this writing, the CLI is not supported on Windows. Instructions to install the CLI on both Mac and Linux are explained very clearly in the [official docs](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_installation.html). Go ahead, install the CLI and when you are done, verify the install by running

В начале нужно установить CLI. На момент написания этого пособия CLI-утилита не доступна на Windows. Инструкции по установке CLI на Mac и Linux хорошо описаны на сайте с [официальной документацией](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_installation.html). Установите утилиту, а потом проверьте её работоспособность.


```
$ ecs-cli --version
ecs-cli version 0.1.0 (*cbdc2d5)
```
Следующий шаг — задание пары ключей для авторизации на инстансах. Зайдите на страницу EC2 Console и создайте новый keypair. Скачайте файл и держите его в безопасном месте. Еще один момент — имя региона. В примере ниже ключ назван именем `ecs` и регион `us-east-1`.  В последующем повествовании это будет подразумеваться.

<img src="images/keypair.png" alt="keypair.png" />

Теперь настройте CLI.

```
$ ecs-cli configure --region us-east-1 --cluster foodtrucks
INFO[0000] Saved ECS CLI configuration for cluster (foodtrucks)
```

Команда `configure` с именем региона, в котором хотим разместить наш кластер, и названием кластера. Нужно указать тот же регион, что использовался при создании ключей. Если у вас не настроен [AWS CLI](https://aws.amazon.com/cli/), то следуйте [руководству](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-set-up.html), которое подробно описывает все шаги.

Следующий шаг позволяет утилите создавать шаблон [CloudFormation](https://aws.amazon.com/cloudformation/).

```
$ ecs-cli up --keypair ecs --capability-iam --size 2 --instance-type t2.micro
INFO[0000] Created cluster                               cluster=foodtrucks
INFO[0001] Waiting for your cluster resources to be created
INFO[0001] Cloudformation stack status                   stackStatus=CREATE_IN_PROGRESS
INFO[0061] Cloudformation stack status                   stackStatus=CREATE_IN_PROGRESS
INFO[0122] Cloudformation stack status                   stackStatus=CREATE_IN_PROGRESS
INFO[0182] Cloudformation stack status                   stackStatus=CREATE_IN_PROGRESS
INFO[0242] Cloudformation stack status                   stackStatus=CREATE_IN_PROGRESS
```

Здесь мы указываем названия ключей, которые мы скачали (в моем случае `ecs`), количество инстансов (`--size`) и тип инстансов, на которых хотим запускать контейнеры. Флаг `--capability-iam` говорит утилите, что мы понимаем, что эта команда может создать ресурсы IAM.

В последнем шаге мы используем файл `docker-compose.yml`. Требуется небольшое изменение, так что вместо модификации файла, давайте сделаем копию и назовем ее `aws-compose.yml`. Содержание этого [файла](https://github.com/prakhar1989/FoodTrucks/blob/master/aws-compose.yml) (после изменений):

```
es:
  image: elasticsearch
  cpu_shares: 100
  mem_limit: 262144000
web:
  image: prakhar1989/foodtrucks-web
  cpu_shares: 100
  mem_limit: 262144000
  ports:
    - "80:5000"
  links:
    - es
```

The only changes we made from the original `docker-compose.yml` are of providing the `mem_limit` and `cpu_shares` values for each container. We also got rid of the `version` and the `services` key, since AWS doesn't yet support [version 2](https://docs.docker.com/compose/compose-file/#version-2) of Compose file format. Since our apps will run on `t2.micro` instances, we allocate 250mb of memory. Another thing we need to do before we move onto the next step is to publish our image on Docker Hub. As of this writing, ecs-cli **does not** support the `build` command - which is [supported](https://docs.docker.com/compose/compose-file/#build) perfectly by Docker Compose.

Единственные отличия от оригинального файла `docker-compose.yml` это параметры `mem_limit` и `cpu_shares` для каждого контейнера.

Также, мы убрали `version` и `services`, так как AWS еще не поддерживает [версию 2](https://docs.docker.com/compose/compose-file/#version-2) файлового формата Compose. Так как наше приложение будет работать на инстансах типа `t2.micro`, мы задали 250 мегабайт памяти. Теперь нам нужно опубликовать образ на Docker Hub. На момент написания этого пособия, ecs-cli **не поддерживает** команду build. Но Docker Compose [поддерживает](https://docs.docker.com/compose/compose-file/#build) ее без проблем.


```
$ docker push prakhar1989/foodtrucks-web
```

Красота! Давайте запустим финальную команду и развернём приложение на ECS!

```
$ ecs-cli compose --file aws-compose.yml up
INFO[0000] Using ECS task definition                     TaskDefinition=ecscompose-foodtrucks:2
INFO[0000] Starting container...                         container=845e2368-170d-44a7-bf9f-84c7fcd9ae29/es
INFO[0000] Starting container...                         container=845e2368-170d-44a7-bf9f-84c7fcd9ae29/web
INFO[0000] Describe ECS container status                 container=845e2368-170d-44a7-bf9f-84c7fcd9ae29/web desiredStatus=RUNNING lastStatus=PENDING taskDefinition=ecscompose-foodtrucks:2
INFO[0000] Describe ECS container status                 container=845e2368-170d-44a7-bf9f-84c7fcd9ae29/es desiredStatus=RUNNING lastStatus=PENDING taskDefinition=ecscompose-foodtrucks:2
INFO[0036] Describe ECS container status                 container=845e2368-170d-44a7-bf9f-84c7fcd9ae29/es desiredStatus=RUNNING lastStatus=PENDING taskDefinition=ecscompose-foodtrucks:2
INFO[0048] Describe ECS container status                 container=845e2368-170d-44a7-bf9f-84c7fcd9ae29/web desiredStatus=RUNNING lastStatus=PENDING taskDefinition=ecscompose-foodtrucks:2
INFO[0048] Describe ECS container status                 container=845e2368-170d-44a7-bf9f-84c7fcd9ae29/es desiredStatus=RUNNING lastStatus=PENDING taskDefinition=ecscompose-foodtrucks:2
INFO[0060] Started container...                          container=845e2368-170d-44a7-bf9f-84c7fcd9ae29/web desiredStatus=RUNNING lastStatus=RUNNING taskDefinition=ecscompose-foodtrucks:2
INFO[0060] Started container...                          container=845e2368-170d-44a7-bf9f-84c7fcd9ae29/es desiredStatus=RUNNING lastStatus=RUNNING taskDefinition=ecscompose-foodtrucks:2
```

То, что вывод похож на вывод **Docker Compose** — не совпадение. Аргумент `--file` используется для переопределения файла по умолчанию (`docker-compose.yml`). Если все прошло хорошо, то вы увидите строку `desiredStatus=RUNNING lastStatus=RUNNING` в самом конце.

Круто! Теперь приложение запущено. Как к нему обратиться?

```
ecs-cli ps
Name                                      State    Ports                     TaskDefinition
845e2368-170d-44a7-bf9f-84c7fcd9ae29/web  RUNNING  54.86.14.14:80->5000/tcp  ecscompose-foodtrucks:2
845e2368-170d-44a7-bf9f-84c7fcd9ae29/es   RUNNING                            ecscompose-foodtrucks:2
```

Откройте [http://54.86.14.14](http://54.86.14.14) в браузере, и увидите Food Trucks во всей своей желто-черной красе! Заодно, давайте взглянем на консоль [AWS ECS](https://console.aws.amazon.com/ecs/home?region=us-east-1#/clusters).


<img src="images/cluster.png" alt="ECS cluster" />
<img src="images/tasks.png" alt="ECS cluster" />


Видно, что был создан ECS-кластер 'foodtrucks', и в нем выполняется одна задача с двумя инстансами. Советую поковыряться в этой консоли и изучить разные ее части и опции.

Вот и все. Всего несколько команд — и приложение работает на AWS!

___________

<a href="#table-of-contents" class="top" id="preface">Top</a>
<a id="wrap-up"></a>

## 4. Заключение

Мы подошли к концу. После длинного, изматывающего, но интересного пособия вы готовы захватить мир контейнеров! Если вы следовали пособию до самого конца, то можете заслуженно гордиться собой. Вы научились устанавливать Docker, запускать свои контейнеры, запускать статические и динамические веб-сайты и, самое главное, получили опыт развёртывания приложения в облако.

Надеюсь, прохождение этого руководства помогло вам стать увереннее в своих способностях управляться с серверами. Когда у вас появится новая идея для сайта или приложения, можете быть уверены, что сможете показать его людям с минимальными усилиями.

<a id="next-steps"></a>

### 4.1 Следующие шаги

Ваше путешествие в мир контейнеров только началось. Моей целью в этом руководстве было нагулять ваш аппетит и показать мощь Docker. В мире современных технологий иногда бывает сложно разобраться самостоятельно, и руководства вроде этого призваны помогать вам. Это такое пособие, которое мне хотелось бы иметь, когда я только знакомился с Docker сам. Надеюсь, ему удалось заинтересовать вас, так что теперь вы сможете следить за прогрессом в этом области не со стороны, а с позиции знающего человека.

Ниже — список дополнительных полезных ресурсов. Советую использовать Docker в вашем следующем проекте. И не забывайте — практика приводит к совершенству.


**Дополнительные ресурсы**

- [Awesome Docker](https://github.com/veggiemonk/awesome-docker)
- [Hello Docker Workshop](http://docker.atbaker.me/)
- [Building a microservice with Node.js and Docker](https://www.youtube.com/watch?v=PJ95WY2DqXo)
- [Why Docker](https://blog.codeship.com/why-docker/)
- [Docker Weekly](https://www.docker.com/newsletter-subscription) and [archives](https://blog.docker.com/docker-weekly-archives/)
- [Codeship Blog](https://blog.codeship.com/)

Удачи!

<a id="feedback"></a>

### 4.2 Обратная связь

Now that the tutorial is over, it's my turn to ask questions. How did you like the tutorial? Did you find the tutorial to be a complete mess or did you have fun and learn something?

Send in your thoughts directly to [me](mailto:prakhar@prakhar.me) or just [create an issue](https://github.com/prakhar1989/docker-curriculum/issues/new). I'm on [Twitter](https://twitter.com/prakharsriv9), too, so if that's your deal, feel free to holler there!

I would totally love to hear about your experience with this tutorial. Give suggestions on how to make this better or let me know about my mistakes. I want this tutorial to be one of the best introductory tutorials on the web and I can't do it without your help.

Теперь моя очередь задавать вопросы. Вам понравилось пособие? Оно показалось вам запутанным, или вам удалось научиться чему-то?

Напишите [мне](mailto:prakhar@prakhar.me) (автору оригинального пособия, — прим. пер.) напрямую на prakhar@prakhar.me или просто создайте issue. Я есть в Твиттере, так что если хотите, то можете писать туда.

(Автор оригинального пособия говорит по-английски, — прим. пер.).

Буду рад услышать ваши отзывы. Не стесняйтесь предлагать улучшения или указывать на мои ошибки. Я хочу, чтобы это пособие стало одним из лучших стартовых руководств в интернете. У меня не получится это без вашей помощи.


___________

<a href="#table-of-contents" class="top" id="preface">Top</a>
<a id="references"></a>

## Источники
- [What containers can do for you](http://radar.oreilly.com/2015/01/what-containers-can-do-for-you.html)
- [What is Docker](https://www.docker.com/what-docker)
- [A beginner's guide to deploying production web apps](https://medium.com/@j_mcnally/a-beginner-s-guide-to-deploying-production-web-apps-to-docker-9458409c6180?_tmc=WrhaI1ejJlMmTpUmHOhTFZsYaUSPUP1yvyq19dsRQ5A#.bl50ga0uz)
- [Running Web Application in Linked Docker Containers Environment](https://aggarwalarpit.wordpress.com/2015/12/06/running-web-application-in-linked-docker-containers-environment/)
