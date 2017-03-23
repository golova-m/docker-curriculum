<a id="top"></a>
<img src="https://raw.githubusercontent.com/prakhar1989/docker-curriculum/master/images/logo.png" alt="docker logo">

*Научитесь легко создавать и разворачивать распределённые приложения в облаке с помощью Docker*

*По мотивам статьи https://prakhar.me/docker-curriculum/ и её перевода https://habrahabr.ru/post/310460/.*

<a href="#top" class="top" id="getting-started">Top</a>

## Базовые термины: FAQ

### Что такое Docker?
[Википедия](https://ru.wikipedia.org/wiki/Docker) даёт следующее определение [Docker](https://www.docker.com/):

> **программное обеспечение для автоматизации развёртывания и управления приложениями в среде виртуализации на уровне операционной системы. Позволяет «упаковать» приложение со всем его окружением и зависимостями в контейнер, который может быть перенесён на любую Linux-систему с поддержкой cgroups в ядре, а также предоставляет среду по управлению контейнерами.**

Ого, какая тирада! Простыми словами - Docker это инструмент, который позволяет разработчикам, системными администраторам и другим специалистам разворачивать приложения в песочнице ( контейнерах) для запуска на целевой операционной системе. Ключевое преимущество Docker заключается в том, что он позволяет пользователям упаковать приложение со всеми его зависимостями в стандартизированный модуль. В отличие от виртуальных машин, контейнеры не создают больших накладных расходов, что позволяет использовать систему и ресурсы эффективнее.

### Что такое контейнеры?


В индустрии по текущим стандартам в качестве среды исполнения приложений используются виртуальные машины. Виртуальные машины эмулируют аппаратное обеспечение на котором выполняется гостевая операционная система с целевым приложением.

Виртульные машины отлично подходят для изоляции приложеня - вариантов, когда проблема основной операцинной системы могла бы повлиять на выполнение приложения в гостевой ОС, не много. Но за такую изоляцию приходится платить большими накладными расходами на создание виртуальной среды. 

Контейнеры используют другой подход. Вместо эмуляции аппаратного обеспечения Docker эмулирует операционную систему, делая контейнеры более легковесными.

### Почему следует изучать контейнеры?

Взлёт Docker был по-настоящему молниеносным. Несмотря на то, что контейнеры сами по себе не новая технология, до Docker они не были так распространены и популярны. Docker изменил ситуацию, предоставив стандартный API, который существенно упростил создание и использование контейнеров, и позволил сообществу вместе разрабатывать библиотеки по работе с контейнерами. В статье, опубликованной в [The Register](http://www.theregister.co.uk/2014/05/23/google_containerization_two_billion/) в середине 2014 говорится, что Google более **двух миллиардов контейнеров в неделю**.

**Google Trends for 'Docker'**

<script type="text/javascript" src="https://ssl.gstatic.com/trends_nrtr/962_RC10/embed_loader.js">

</script>
<script type="text/javascript"> 

trends.embed.renderExploreWidget("TIMESERIES", {"comparisonItem":[{"keyword":"/m/0wkcjgj","geo":"","time":"today 5-y"}],"category":0,"property":""}, {"exploreQuery":"q=%2Fm%2F0wkcjgj","guestPath":"https://trends.google.ru:443/trends/embed/"});
</script> 

В дополнение к стабильному росту популярности, компания-разработчик Docker Inc. была оценена в два с лишним миллиарда долларов! Благодаря преимуществам в эффективности и портативности, Docker начал получать всё большую поддержку, и сейчас стоит во главе движения по контейнеризации (containerization). Как современные разработчики, мы должны понять этот тренд и выяснить, какую пользу мы можем из него извлечь.

### Чему научит меня эта лабораторная?

Задачей этой лабораторной является подробное знакомство с Docker. Кроме развеяния мифов о нём и его экосистеме, завершение лабораторной позволит получить небольшой опыт в сборке и развёртывании собственных веб-приложений в облаке. Мы будем использовать [Amazon Web Services](http://aws.amazon.com) для развёртывания статичного сайта. Два динамических веб-приложения мы развернём на [EC2](https://aws.amazon.com/ec2/) using [Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/) и [Elastic Container Service](https://aws.amazon.com/ecs/).

## Комментарии к данному документу

Данный документ состоит из нескольких разделов, каждый из которых посвящён определённому аспекту Docker. В каждом разделе мы будем вводить команды или писать код. Весь код доступен в [репозитории GitHub](http://github.com/prakhar1989/docker-curriculum).

<a href="#top" class="top" id="table-of-contents">Top</a>
## Оглавление

-	[Предисловие](#preface)
    -	[Предварительные требования](#prerequisites)
    -   [Настройка компьютера](#setup)
-   [1 Играем с Busybox](#busybox)
    -	[1.1 Docker Run](#dockerrun)
    -	[1.2 Терминология](#terminology)
-   [2 Веб-приложения с Docker](#webapps)
    -	[2.1 Статические сайты](#static-site)
    -	[2.2 Docker образы (image)](#docker-images)
    -	[2.3 Наш первый образ](#our-image)
    -	[2.4 Dockerfile](#dockerfiles)
    -	[2.5 Docker на AWS](#docker-aws)
-   [3 Многоконтейнерные окружения](#multi-container)
    -	[3.1 SF Food Trucks](#foodtrucks)
    -	[3.2 Docker сеть](#docker-network)
    -	[3.3 Docker Compose](#docker-compose)
    -	[3.4 AWS Elastic Container Service](#aws-ecs)
-   [4 Подводим итоги](#wrap-up)
    -	[4.1 Что дальше?](#next-steps)
    -	[4.2 Обратная связь](#feedback)
-   [Ссылки](#references)


------------------------------
<a href="#table-of-contents" class="top" id="preface">Top</a>
## Предисловие

> Внимание: Данная лабораторная предполагает использование Docker версии  **1.12.0-rc2**. Если какая-либо часть будет несовместима с более новыми версиями, откройте [issue](https://github.com/prakhar1989/docker-curriculum/issues). Спасибо!

<a id="prerequisites"></a>
### Предварительные требования

Для прохождения данной лабораторной не потребуется специальной подготовки, кроме уверенного использования командной строки и текстового редактора. Прошлый опыт разработки веб-приложений не требуется, но будет полезен. По мере прохождения обучения мы будем использовать облачные сервисы. Вам понадобится создать аккаунт на следующих сайтах:

- [Amazon Web Services](http://aws.amazon.com/),
- [Docker Hub](https://hub.docker.com/).

<a id="setup"></a>
### Настраиваем компьютер

Установка и настройка всех необходимых инструментов может быть утомительной задачей, но, к счастью, Docker стал довольно стабильным, и установка и запуск его на любой ОС стала простым. Итак, установим Docker.

Еще несколько релизов назад запуск Докера на OS X и Windows был проблемным. Но команда разработчиков проделала огромную работу, и сегодня весь процесс — проще некуда. На официальном сайте вы можете найти подробные инструкции по установке на [Mac](https://www.docker.com/products/docker#/mac), [Linux](https://www.docker.com/products/docker#/linux) and [Windows](https://www.docker.com/products/docker#/windows).

Проверим, все ли установлено корректно:

```
$ docker run hello-world

Hello from Docker.
This message shows that your installation appears to be working correctly.
...
```
___________

<a href="#table-of-contents" class="top" id="preface">Top</a>
<a id="busybox"></a>
## 1 Играем с BusyBox

Теперь, когда всё подготовлено, пора приняться за дело. В этом разделе нашей целью будет запуск контейнера [Busybox](https://en.wikipedia.org/wiki/BusyBox) и освоение команды `docker run`.

Для начала, запустите следующую команду:
```
$ docker pull busybox
```

> Внимание: В зависимости от того, как вы установили Docker в систему, вы можете увидеть сообщение `permission denied` (доступ запрещён) в ответ на вызов выше приведённой команды. Если вы на Mac, убедитесь, что Docker движок запущен. Если на Линукс, вам может потребоваться повысить права доступа с помощью команды `sudo`. В качестве альтернативного варианта вы можете создать [Docker группу](https://docs.docker.com/engine/installation/linux/linux-postinstall/) для решения этой проблемы.

Команда `pull` скачивает [образ busybox](https://hub.docker.com/_/busybox/) из [**Docker registry**](https://hub.docker.com/explore/) и сохраняет его в систему. Вы можете использовать команду `docker images` для вывода в консоль списка образов находящихся в вашей системе.

```
$ docker images
REPOSITORY              TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
busybox                 latest              c51f86c28340        4 weeks ago         1.109 MB
```

<a id="dockerrun"></a>
### 1.1 Запуск Docker

Великолепно! Теперь перейдём к запуску **контейнера** на основе этого образа. Для этого мы воспользуемся всемогущей командой `docker run`.

```
$ docker run busybox
$
```

Постойте, но ничего не произошло! Это баг? Ну, нет. Под капотом произошло много всего. Когда вы запустили команду `run`, клиент Docker нашёл образ (в нашем случае, busybox), загрузил контейнер и запустил команду внутри этого контейнера. Мы выполнили `docker run busybox`, но не указали никаких аргументов, так что контейнер загрузился, выполнил команду `sh` и процесс контейнер завершился. Ну, да, как-то обидно. Попробуем сделать что-нибудь поинтереснее.

```
$ docker run busybox echo "hello from busybox"
hello from busybox
```

Ура, наконец-то какой-то вывод. В данном случае Docker клиент добросовестно запустил команду `echo` внутри контейнера, а затем вышел из него. Вы, наверное, заметили, что всё произошло очень быстро. А теперь представьте себе процесс загрузки виртуальной машины, выполнения в ней команды и её выключения. Теперь ясно, почему говорят, что контейнеры быстрые! Чтобы узнать время выполнения попробуйте запустить последнюю команду со словом `time` в начале.

Теперь давайте взглянем на команду `docker ps`. Она выводит на жкран список всех запущенных контейнеров.

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

В силу того, что ни один контейнер не запушен, выводится пустая строка. Попробуем более информативный вариант `docker ps -a`:

```
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
305297d7a235        busybox             "uptime"            11 minutes ago      Exited (0) 11 minutes ago                       distracted_goldstine
ff0a5c3750b9        busybox             "sh"                12 minutes ago      Exited (0) 12 minutes ago                       elated_ramanujan
```

То, что мы видим в выдаче - список всех контейнеров, которые были запущены ранее. Обратите внимание, что колонка `STATUS` показывает, что эти контейнеры остановились несколько минут назад. 

Наверное вы удивляетесь, существует ли способ запустить более одной команды в контейнере. Давайте попробуем:

```
$ docker run -it busybox sh
/ # ls
bin   dev   etc   home  proc  root  sys   tmp   usr   var
/ # uptime
 05:45:21 up  5:58,  0 users,  load average: 0.00, 0.01, 0.04
```

Выполнение команды `run` с флагами `-it` подключает нас к интерактивному терминалу tty в контейнере. Теперь мы можем запустить столько команд, сколько захотим. Уделите немного времени запуску ваших любимых команд в этой консоли.

> **Опасная зона**: Если вы любите рисковать, вы можете попробовать выполнить команду `rm -rf bin`. Убедитесь, что выполняете команду в контейнере, а **не** в основной операционной системе. Удалив данной командой папку bin не даст возможности запускать команды как `ls`, `echo`. После того, как всё перестанет работать, вы можете выйти из контейнера (выполним команду `exit`), а затем запустить контейнер заново `docker run -it busybox sh`. Так как Docker каждый раз создаёт новый контейнер, папка bin и команды должны быть опять доступны.

> В Docker 1.3 запуск контейнера и создание слоя файловой системы разделены. Если вы используете эту версию, вы можете проделать аналогичный выше пример, но с командами `create`, `start`, `stop`. Запуск команды `docker create -it busybox sh` создаст слой файловой системы контейнера. Затем командами `docker start -ia <id_контейнера>` и `docker stop <id_контейнера>` вы можете запустить и остановить выполнение контейнера не уничтожая файловый слой. Проанализируйте, что будет в этом случае при повторном запуске. 

На этом захватывающий тур по возможностям команды docker run закончен. Скорее всего, вы будете использовать эту команду довольно часто. Так что важно, чтобы мы поняли как с ней обращаться. Чтобы узнать больше о run, используйте `docker run --help`, и увидите полный список поддерживаемых флагов. Скоро мы увидим еще несколько способов использования `docker run`.

Перед тем, как продолжать, давайте вкратце рассмотрим удаление контейнеров. Мы видели выше, что с помощью команды `docker ps -a` все еще можно увидеть остатки завершенных контейнеров. На протяжении этого пособия, вы будете запускать `docker run` несколько раз, и оставшиеся, бездомные контейнеры будут съедать дисковое пространство. Так что я взял за правило удалять контейнеры после завершения работы с ними. Для этого используется команда docker rm. Просто скопируйте ID (можно несколько) из вывода выше и передайте параметрами в команду.
```
$ docker rm 305297d7a235 ff0a5c3750b9
305297d7a235
ff0a5c3750b9
```

При удалении идентификаторы будут снова выведены на экран. Если нужно удалить много контейнеров, то вместо ручного копирования и вставления можно сделать так:

```
$ docker rm $(docker ps -a -q -f status=exited)
```

Эта команда удаляет все контейнеры, у которых статус `exited`. Флаг `-q` возвращает только численные ID, а флаг `-f` фильтрует вывод на основе предоставленных условий. Последняя полезная деталь — команде `docker run` можно передать флаг `--rm`, тогда контейнер будет автоматически удаляться при завершении. Это очень полезно для разовых запусков и экспериментов с Docker.

Также можно удалять ненужные образы командой `docker rmi`.

<a id="terminology"></a>

### 1.2 Терминология

В предыдущем разделе мы использовали много специфичного для Докера жаргона, и многих это может запутать. Перед тем, как продолжать, давайте разберем некоторые термины, которые часто используются в экосистеме Докера.

- *Images* (образы) - Схемы нашего приложения, которые являются основой контейнеров. В примере выше мы использовали команду `docker pull` чтобы скачать образ **busybox**.
- *Containers* (контейнеры) - Создаются на основе образа и запускают само приложение. Мы создали контейнер командой `docker run`, и использовали образ busybox, скачанный ранее. Список запущенных контейнеров можно увидеть с помощью команды docker ps.
- *Docker Daemon* (Docker демон) - Фоновый сервис, запущенный на хост-машине, который отвечает за создание, запуск и уничтожение Докер-контейнеров. Демон — это процесс, который запущен на операционной системе, с которой взаимодействует клиент.
- *Docker Client* (Docker клиент) - Утилита командной строки, которая позволяет пользователю взаимодействовать с демоном. Существуют другие формы клиента, например, [Kitematic](https://kitematic.com/), с графическим интерфейсом.
- *Docker Hub* - [Реестр Docker-образов](https://hub.docker.com/explore/). Грубо говоря, архив всех доступных образов. Если нужно, то можно содержать собственный реестр и использовать его для получения образов.


<a href="#table-of-contents" class="top" id="preface">Top</a>
<a id="webapps"></a>

## 2 Веб-приложения в Docker

Супер! Теперь мы научились работать с `docker run`, поиграли с несколькими контейнерами и разобрались в терминологии. Вооруженные этими знаниями, мы готовы переходить к реальным штукам: деплою веб-приложений с Docker!

<a id="static-site"></a>

### 2.1 Статичные сайты

Давайте начнем с малого. Вначале рассмотрим самый простой статический веб-сайт. Скачаем образ из Docker Hub, запустим контейнер и посмотрим, насколько легко будет запустить веб-сервер.

Поехали. Для одностраничного сайта нам понадобится образ, который я заранее создал для этого пособия и разместил в [реестре](https://hub.docker.com/r/prakhar1989/static-site/) - prakhar1989/static-site. Можно скачать образ напрямую командой `docker run`.

```
$ docker run prakhar1989/static-site
```
Так как образа не существует локально, клиент сначала скачает образ из реестра, а потом запустит его. Если всё без проблем, то вы увидите сообщение Nginx is running... в терминале. Теперь сервер запущен. Как увидеть сайт в действии? На каком порту работает сервер? И, что самое важное, как напрямую достучаться до контейнера из хост-контейнера?

В нашем случае клиент не открывает никакие порты, так что нужно будет перезапустить команду  `docker run` чтобы сделать порты публичными. Заодно давайте сделаем так, чтобы терминал не был прикреплен к запущенному контейнеру. В таком случае можно будет спокойно закрыть терминал, а контейнер продолжит работу. Этот режим называется **detached**.

```
$ docker run -d -P --name static-site prakhar1989/static-site
e61d12292d69556eabe2a44c16cbd54486b2527e2ce4f95438e504afb7b02810
```

Флаг `-d` открепит (detach) терминал, флаг `-P` сделает все открытые порты публичными и случайными, и, наконец, флаг `--name` это имя, которое мы хотим дать контейнеру. Теперь можно увидеть порты с помощью команды `docker port [CONTAINER]`.


```
$ docker port static-site
80/tcp -> 0.0.0.0:32769
443/tcp -> 0.0.0.0:32768
```

Вы можете открыть [http://localhost:32769](http://localhost:32769) в своём браузере. 

> Замечание. Если вы используете docker-toolbox, тогда вам потребуется узнать ip адрес машины, например, с помощью команды `docker-machine ip default`. В основной операционной системе также можно выполнить команду `ip addr`.

Вы также можете назначить свой порт, на который Docker клиент будет перенаправлять запросы на соединение к контейнеру.

```
$ docker run -p 8888:80 prakhar1989/static-site
Nginx is running...
```
Слева порт основной операционной системы, справа порт контейнера.

<img src="https://raw.githubusercontent.com/prakhar1989/docker-curriculum/master/images/static.png" title="static">

Чтобы остановить контейнер запустите docker stop и укажите идентификатор (ID) контейнера.

Согласитесь, все было очень просто. Чтобы развернуть это на реальный сервер, нужно просто установить Docker и запустить команду выше. Теперь, когда вы увидели, как запускать веб-сервер внутри образа, вам, наверное, интересно — а как создать свой Docker образ? Мы будем изучать эту тему в следующем разделе.


<a id="docker-images"></a>
### 2.2 Docker образы

Мы касались образов ранее, но в этом разделе мы заглянем глубже: что такое Docker образы и как создавать собственные образы. Наконец, мы используем собственный образ чтобы запустить приложение локально, а потом развернём его на [AWS](http://aws.amazon.com), чтобы показать друзьям. Круто? Круто! Давайте начнем.

Образы это основы для контейнеров. В прошлом примере мы скачали (**pull**) образ под названием Busybox из регистра, и попросили клиент Docker запустить контейнер, **основанный** на этом образе. Чтобы увидеть список доступных локально образов, используйте команду docker images.

```
$ docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
prakhar1989/catnip              latest              c7ffb5626a50        2 hours ago         697.9 MB
prakhar1989/static-site         latest              b270625a1631        21 hours ago        133.9 MB
python                          3-onbuild           cf4002b2c383        5 days ago          688.8 MB
martin/docker-cleanup-volumes   latest              b42990daaca2        7 weeks ago         22.14 MB
ubuntu                          latest              e9ae3c220b23        7 weeks ago         187.9 MB
busybox                         latest              c51f86c28340        9 weeks ago         1.109 MB
hello-world                     latest              0a6ba66e537a        11 weeks ago        960 B
```

Это список образов, которые я скачал из регистра, а также тех, что я сделал сам (скоро увидим, как это делать). `TAG` — это конкретный снимок или снэпшот (snapshot) образа, а `IMAGE ID` — это соответствующий уникальный идентификатор образа.

Для простоты, можно относиться к образу как к git-репозиторию. Образы можно [коммитить](https://docs.docker.com/engine/reference/commandline/commit/) с изменениями, и можно иметь несколько версий. Если не указывать конкретную версию, то клиент по умолчанию использует `latest`. Например, можно скачать определенную версию образа `ubuntu`:
```
$ docker pull ubuntu:12.04
```
Чтобы получить новый Docker образ, можно скачать его из реестра (такого, как Docker Hub) или создать собственный. На Docker Hub есть десятки тысяч образов. Можно искать напрямую из командной строки с помощью `docker search`.

Важно понимать разницу между базовыми и дочерними образами:

- **Base images** (базовые образы) — это образы, которые не имеют родительского образа. Обычно это образы с операционной системой, такие как ubuntu, busybox или debian.
- **Child images** (дочерние образы) — это образы, построенные на базовых образах и обладающие дополнительной функциональностью.

Существуют официальные и пользовательские образы, и любые из них могут быть базовыми и дочерними.

- **Официальные образы** — это образы, которые официально поддерживаются командой Docker. Обычно в их названии одно слово. В списке выше python, ubuntu, busybox и hello-world — базовые образы.
- **Пользовательские образы** — образы, созданные простыми пользователями вроде меня и вас. Они построены на базовых образах. Обычно, они называются по формату user/image-name.

<a id="our-image"></a>
### 2.3 Наш первый образ

Теперь, когда мы лучше понимаем, что такое образы и какие они бывают, самое время создать собственный образ. Цель этого раздела — создать образ с простым приложением на [Flask](http://flask.pocoo.org). Для этого пособия я сделал [маленькое приложение](https://github.com/prakhar1989/docker-curriculum/tree/master/flask-app), которое выводит случайную гифку с кошкой. Ну, потому что, кто не любит кошек? Склонируйте этот репозиторий к себе на локальную машину.

Следующим шагом является создание образа с данным веб-приложением. Как говорилось выше, все пользовательские образы базируются на базовых образах. Так как приложение написано на Python, базовый образ следует выбрать с предустановленным [Python 3](https://hub.docker.com/_/python/). Точнее мы собираемся использовать `python:3-onbuild` версию python образа.

Что ещё за `onbuild` версия вы можете спросить?

> Данные образы включают множество onbuild триггеров, которых должно хватить для запуска большинства приложений. При построении скопируется `requirements.txt` файл, запустится `pip install` на основе этого файла и затем скопируется текущая директория в `/usr/src/app`.

Другими словами `onbuild` версия образа содержит скрипты-помошники для автоматизации рутинных задач необходимых для запуска приложения. Вместо выполнения этих задач вручную (или написания для этого скриптов), эти образы делают эту работу за вас. Теперь у нас есть все ингредиенты для создания собственных образов - работающее веб-приложение и базовый образ. Как мы будем подходить к этой задаче? Ответ - **Dockerfile**.

<a id="dockerfiles"></a>
### 2.4 Dockerfile

[Dockerfile](https://docs.docker.com/engine/reference/builder/) — это простой текстовый файл, в котором содержится список команд Докер-клиента. Это простой способ автоматизировать процесс создания образа. Самое классное, что [команды](https://docs.docker.com/engine/reference/builder/#from) в Dockerfile почти идентичны своим аналогам в Linux. Это значит, что в принципе не нужно изучать никакой новый синтаксис, чтобы начать работать с докерфайлами.

В директории с приложением есть Dockerfile, но так как мы делаем все впервые, нам нужно создать его с нуля. Создайте новый пустой файл в любимом текстовом редакторе, и сохраните его в **той же** директории, где находится flask-приложение. Назовите файл `Dockerfile`.

Для начала укажем базовый образ. Для этого нужно использовать ключевое слово `FROM`.
```
FROM python:3-onbuild
```
Следующим шагом обычно указывают команды для копирования файлов и установки зависимостей. Но к счастью, `onbuild`-версия базового образа берет эти задачи на себя. Дальше нам нужно указать порт, который следует открыть. Наше приложение работает на порту `5000`, поэтому укажем его:
```
EXPOSE 5000
```
Последний шаг — указать команду для запуска приложения. Это просто `python ./app.py`. Для этого используем команду [CMD](https://docs.docker.com/engine/reference/builder/#cmd):
```
CMD ["python", "./app.py"]
```

Главное предназначение `CMD` — это сообщить контейнеру какие команды нужно выполнить при старте. Теперь наш `Dockerfile` готов. Вот как он выглядит:
```
# our base image
FROM python:3-onbuild

# specify the port number the container should expose
EXPOSE 5000

# run the application
CMD ["python", "./app.py"]
```

Теперь можно создать образ. Команда `docker build` занимается сложной задачей создания образа на основе `Dockerfile`.

Листинг ниже демонстрирует процесс. Перед тем, как запустите команду сами (не забудьте точку в конце), проверьте, чтобы там был ваш username вместо моего. Username должен соответствовать тому, что использовался при регистрации на [Docker hub](https://hub.docker.com). Если вы еще не регистрировались, то сделайте это до выполнения команды. Команда `docker build` довольно проста: она принимает опциональный тег с флагом `-t имя_пользователя/имя_образа` и путь до директории, в которой лежит `Dockerfile`.
```
$ docker build -t prakhar1989/catnip .
Sending build context to Docker daemon 8.704 kB
Step 1 : FROM python:3-onbuild
# Executing 3 build triggers...
Step 1 : COPY requirements.txt /usr/src/app/
 ---> Using cache
Step 1 : RUN pip install --no-cache-dir -r requirements.txt
 ---> Using cache
Step 1 : COPY . /usr/src/app
 ---> 1d61f639ef9e
Removing intermediate container 4de6ddf5528c
Step 2 : EXPOSE 5000
 ---> Running in 12cfcf6d67ee
 ---> f423c2f179d1
Removing intermediate container 12cfcf6d67ee
Step 3 : CMD python ./app.py
 ---> Running in f01401a5ace9
 ---> 13e87ed1fbc2
Removing intermediate container f01401a5ace9
Successfully built 13e87ed1fbc2
```

Если у вас нет образа `python:3-onbuild`, то клиент сначала скачает его, а потом возьмется за создание вашего образа. Так что, вывод на экран может отличаться от моего. Посмотрите внимательно, и найдете триггеры onbuild. Если все прошло хорошо, то образ готов! Запустите `docker images` и увидите свой образ в списке.

Последний шаг — запустить образ и проверить его работоспособность (замените username на свой):
```
$ docker run -p 8888:5000 prakhar1989/catnip
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
 ```

Зайдите на указанный URL и увидите приложение в работе.

<img src="https://raw.githubusercontent.com/prakhar1989/docker-curriculum/master/images/catgif.png" title="static">

Поздравляю! Вы успешно создали свой первый образ Докера!

<a id="docker-aws"></a>
### 2.5 Docker on AWS

Что хорошего в приложении, которое нельзя показать друзьям, правда? Так что в этом разделе мы научимся разворачивать наше офигенное приложение в облако. Будем использовать AWS [Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/), чтобы решить эту задачу за пару кликов. Мы увидим, как с помощью Beanstalk легко управлять и масштабировать наше приложение.


##### Docker push

Первое, что нужно сделать перед развёртыванием на AWS это опубликовать наш образ в реестре, для того, чтобы можно было скачивать его из AWS. Есть несколько [Docker реестров](https://aws.amazon.com/ecr/) (или можно создать собственный). Для начала, давайте используем [Docker Hub](https://hub.docker.com). Для публикации образа просто выполните:

```
$ docker push prakhar1989/catnip
```
Если это ваша первая публикация, то клиент попросит вас залогиниться. Введите те же данные, что используете для входа в [Docker Hub](https://hub.docker.com).

```
$ docker login
Username: prakhar1989
WARNING: login credentials saved in /Users/prakhar/.docker/config.json
Login Succeeded
```
Не забудьте заменить название образа на свое. Очень важно сохранить формат username/image_name, чтобы клиент понимал, куда публиковать образ.

После этого можете посмотреть на свой образ на Docker Hub. Например, вот [страница](https://hub.docker.com/r/prakhar1989/catnip/) моего образа.

> Замечание. Один важный момент, который стоит прояснить перед тем, как продолжить — **не обязательно** хранить образ в публичном реестре (или в любом другом реестре вообще), чтобы разворачивать на AWS. Если вы пишете код для следующего многомиллионного стартапа-единорога, то можно пропустить этот шаг. Мы публикуем свой образ, чтобы упростить развёртывание, пропустив несколько конфигурационных шагов.

Теперь наш образ онлайн, и любой докер-клиент может поиграться с ним с помощью простой команды:

```
$ docker run -p 8888:5000 prakhar1989/catnip
```

Если в прошлом вы мучались с установкой локального рабочего окружения и попытками поделиться своей конфигурацией с коллегами, то понимаете, как круто это звучит. Вот почему Docker — это сила!


##### Beanstalk

AWS Elastic Beanstalk (EB) это PaaS (Platform as a Service — платформа как сервис) от Amazon Web Services. Если вы использовали Heroku, Google App Engine и т.д., то все будет привычно. Как разработчик, вы сообщаете EB как запускать ваше приложение, а EB занимается всем остальным, в том числе масштабированием, мониторингом и даже обновлениями. В апреле 2014 в EB добавили возможность запускать Docker контейнеры, и мы будем использовать именно эту возможность для деплоя. У EB очень понятный [интерфейс командной строки](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3.html), но он требует небольшой конфигурации, поэтому для простоты давайте используем веб-интерфейс для запуска нашего приложения.

Чтобы продолжать, вам потребуется работающий аккаунт на [AWS](http://aws.amazon.com). Если у вас его нет, то создайте его. Для этого потребуется ввести данные кредитной карты. Но не волнуйтесь, эта услуга бесплатна, и все, что будет происходить в рамках этого пособия тоже бесплатно.

Давайте начнем:

- Войдите в свою [консоль](http://console.aws.amazon.com) AWS.
- Нажмите на Elastic Beanstalk. Ссылка находится в секции compute, в левом верхнем углу. Или просто перейдите [сюда](https://console.aws.amazon.com/elasticbeanstalk).

<img src="https://raw.githubusercontent.com/prakhar1989/docker-curriculum/master/images/eb-start.png" title="static">

- Нажмите на "Create New Application" в верхнем правом углу.
- Дайте своему приложению запоминающееся (но уникальное) имя и, если хотите, добавьте описание.
на экране **New Environment** выберите **Web Server Environment**.
- Следующий экран показан ниже. Выберите Docker из готовых вариантов конфигурации. Можно оставить *Environment type* как есть. Нажмите Next.


<img src="https://raw.githubusercontent.com/prakhar1989/docker-curriculum/master/images/eb-docker.png" title="static">

- Тут мы будем сообщать системе EB о нашем образе. Откройте файл `Dockerrun.aws.json` в директории flask-app и измените Name образа, чтобы оно соответствовало названию вашего образа. Не волнуйтесь, я опишу содержание файла попозже. Потом выберите вариант "upload your own" и выберите файл.
- Далее, выберите название окружения и URL. Этот URL как раз можно будет давать друзьям, так что постарайтесь придумать что-нибудь попроще.
- Пока не будем вносить никаких правок в секцию Additional Resources. Нажмите Next и переходите к Configuration Details.
- В этой секции вам нужно выбрать тип инстанса t1.micro. Это очень важно, потому что это бесплатный тип от AWS. Если хотите, можно выбрать пару ключей для входа. Если вы не знаете, что это значит, то не волнуйтесь и просто пропустите эту часть. Все остальное можно оставить по умолчанию и продолжать.
- Также не нужно указывать никакие Environment Tags and Permissions, так что просто жмите Next два раза подряд. В конце будет экран Review. Если все выглядит нормально, то нажимайте кнопку Launch.
- На последнем экране будет несколько спиннеров. Это поднимается и настраивается ваше окружение. Обычно, нужно около пяти минут для первой настройки.

Пока ждем, давайте быстренько взглянем на файл `Dockerrun.aws.json`. Это файл для AWS, в котором находится информация о приложении конфигурации Докера. EB получает информацию из этого файла.

```
{
  "AWSEBDockerrunVersion": "1",
  "Image": {
    "Name": "prakhar1989/catnip",
    "Update": "true"
  },
  "Ports": [
    {
      "ContainerPort": "5000"
    }
  ],
  "Logging": "/var/log/nginx"
}
```

Файл довольно понятный, но всегда можно обратиться к [официальной документации](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_image. Мы указываем название образа, и EB будет использовать его заодно с портом.

К этому моменту инстанс уже должен быть готов. Зайдите на страницу EB и увидите зеленый индикатор успешного запуска приложения.

<img src="https://raw.githubusercontent.com/prakhar1989/docker-curriculum/master/images/eb-deploy.png" title="static">

Зайдите на указанный URL в браузере и увидите приложение во все красе. Пошлите адрес своим друзьям, чтобы все могли насладиться гифками с кошками.

Поздравляю! Вы задеплоили свое первое Докер-приложение! Может показаться, что было очень много шагов, но с командной утилитой EB можно имитировать функциональность Хероку несколькими нажатиями клавиш. Надеюсь, вы согласитесь, что Докер сильно упрощает процесс и минимизирует болезненные моменты деплоя в облако. Я советую вам почитать [документацию](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/docker-singlecontainer-deploy.html) AWS про single-container Docker environment чтобы понимать, какие существуют возможности в EB.

В следующей, последней части пособия, мы пойдем немного дальше и развернём приложение, приближенное к реальному миру. В нем будет постоянное бэкэнд-хранилище. Поехали!


<a href="#table-of-contents" class="top" id="preface">Top</a>
<a id="multi-container"></a>
## 3.0 Multi-container Environments
In the last section, we saw how easy and fun it is to run applications with Docker. We started with a simple static website and then tried a Flask app. Both of which we could run locally and in the cloud with just a few commands. One thing both these apps had in common was that they were running in a **single container**.

Those of you who have experience running services in production know that usually apps nowadays are not that simple. There's almost always a database (or any other kind of persistent storage) involved. Systems such as [Redis](http://redis.io/) and [Memcached](http://memcached.org/) have become *de riguer* of most web application architectures. Hence, in this section we are going to spend some time learning how to Dockerize applications which rely on different services to run.

In particular, we are going to see how we can run and manage **multi-container** docker environments. Why multi-container you might ask? Well, one of the key points of Docker is the way it provides isolation. The idea of bundling a process with its dependencies in a sandbox (called containers) is what makes this so powerful.

Just like it's a good strategy to decouple your application tiers, it is wise to keep containers for each of the **services** separate. Each tier is likely to have different resource needs and those needs might grow at different rates. By separating the tiers into different containers, we can compose each tier using the most appropriate instance type based on different resource needs. This also plays in very well with the whole [microservices](http://martinfowler.com/articles/microservices.html) movement which is one of the main reasons why Docker (or any other container technology) is at the [forefront](https://medium.com/aws-activate-startup-blog/using-containers-to-build-a-microservices-architecture-6e1b8bacb7d1#.xl3wryr5z) of modern microservices architectures.

<a id="foodtrucks"></a>
### 3.1 SF Food Trucks

The app that we're going to Dockerize is called [SF Food Trucks](http://sf-foodtrucks.xyz). My goal in building this app was to have something that is useful (in that it resembles a real-world application), relies on at least one service, but is not too complex for the purpose of this tutorial. This is what I came up with.

<img src="https://raw.githubusercontent.com/prakhar1989/FoodTrucks/master/shot.png" alt="sf food trucks">

The app's backend is written in Python (Flask) and for search it uses [Elasticsearch](https://www.elastic.co/products/elasticsearch). Like everything else in this tutorial, the entire source is available on [Github](http://github.com/prakhar1989/FoodTrucks). We'll use this as our candidate application for learning out how to build, run and deploy a multi-container environment.

Now that you're excited (hopefully), let's think of how we can Dockerize the app. We can see that the application consists of a Flask backend server and an Elasticsearch service. A natural way to split this app would be to have two containers - one running the Flask process and another running the Elasticsearch (ES) process. That way if our app becomes popular, we can scale it by adding more containers depending on where the bottleneck lies.

Great, so we need two containers. That shouldn't be hard right? We've already built our own Flask container in the previous section. And for Elasticsearch, let's see if we can find something on the hub.

```
$ docker search elasticsearch
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
elasticsearch                     Elasticsearch is a powerful open source se...   697       [OK]
itzg/elasticsearch                Provides an easily configurable Elasticsea...   17                   [OK]
tutum/elasticsearch               Elasticsearch image - listens in port 9200.     15                   [OK]
barnybug/elasticsearch            Latest Elasticsearch 1.7.2 and previous re...   15                   [OK]
digitalwonderland/elasticsearch   Latest Elasticsearch with Marvel & Kibana       12                   [OK]
monsantoco/elasticsearch          ElasticSearch Docker image                      9                    [OK]
```

Quite unsurprisingly, there exists an officially supported [image](https://hub.docker.com/_/elasticsearch/) for Elasticsearch. To get ES running, we can simply use `docker run` and have a single-node ES container running locally within no time.
```
$ docker run -dp 9200:9200 elasticsearch
d582e031a005f41eea704cdc6b21e62e7a8a42021297ce7ce123b945ae3d3763

$ curl 0.0.0.0:9200
{
  "name" : "Ultra-Marine",
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
```

Note that if you are using Elasticsearch version 5 or higher you need to do the following to make it work.

1. Increase the memory size of your docker-machine instance to at least 2gb.
2. Do an SSH in your docker-machine instance by `docker-machine ssh` and run this command `sysctl -w vm.max_map_count=262144`. This will fix the max virtual memory areas error.

While we are at it, let's get our Flask container running too. But before we get to that, we need a `Dockerfile`. In the last section, we used `python:3-onbuild` image as our base image. This time, however, apart from installing Python dependencies via `pip`, we want our application to also generate our [minified Javascript file](http://sf-foodtrucks.xyz/static/build/main.js) for production. For this, we'll require Nodejs. Since we need a custom build step, we'll start from the `ubuntu` base image to build our `Dockerfile` from scratch.

> Note: if you find that an existing image doesn't cater to your needs, feel free to start from another base image and tweak it yourself. For most of the images on Docker Hub, you should be able to find the corresponding `Dockerfile` on Github. Reading through existing Dockerfiles is one of the best ways to learn how to roll your own.


Our [Dockerfile](https://github.com/prakhar1989/FoodTrucks/blob/master/Dockerfile) for the flask app looks like below -
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
Quite a few new things here so let's quickly go over this file. We start off with the [Ubuntu LTS](https://wiki.ubuntu.com/LTS) base image and use the package manager `apt-get` to install the dependencies namely - Python and Node. The `yqq` flag is used to suppress output and assumes "Yes" to all prompt. We also create a symbolic link for the node binary to deal with backward compatibility issues.

We then use the `ADD` command to copy our application into a new volume in the container - `/opt/flask-app`. This is where our code will reside. We also set this as our working directory, so that the following commands will be run in the context of this location. Now that our system-wide dependencies are installed, we get around to install app-specific ones. First off we tackle Node by installing the packages from npm and running the build command as defined in our `package.json` [file](https://github.com/prakhar1989/FoodTrucks/blob/master/flask-app/package.json#L7-L9). We finish the file off by installing the Python packages, exposing the port and defining the `CMD` to run as we did in the last section.

Finally, we can go ahead, build the image and run the container (replace `prakhar1989` with your username below).
```
$ docker build -t prakhar1989/foodtrucks-web .
```
In the first run, this will take some time as the Docker client will download the ubuntu image, run all the commands and prepare your image. Re-running `docker build` after any subsequent changes you make to the application code will almost be instantaneous. Now let's try running our app.
```
$ docker run -P prakhar1989/foodtrucks-web
Unable to connect to ES. Retying in 5 secs...
Unable to connect to ES. Retying in 5 secs...
Unable to connect to ES. Retying in 5 secs...
Out of retries. Bailing out...
```
Oops! Our flask app was unable to run since it was unable to connect to Elasticsearch. How do we tell one container about the other container and get them to talk to each other? The answer lies in the next section.

<a id="docker-network"></a>
### 3.2 Docker Network
Before we talk about the features Docker provides especially to deal with such scenarios, let's see if we can figure out a way to get around the problem. Hopefully this should give you an appreciation for the specific feature that we are going to study.

Okay, so let's run `docker ps` and see what we have.
```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                              NAMES
e931ab24dedc        elasticsearch       "/docker-entrypoint.s"   2 seconds ago       Up 2 seconds        0.0.0.0:9200->9200/tcp, 9300/tcp   cocky_spence
```
So we have one ES container running on `0.0.0.0:9200` port which we can directly access. If we can tell our Flask app to connect to this URL, it should be able to connect and talk to ES, right? Let's dig into our [Python code](https://github.com/prakhar1989/FoodTrucks/blob/master/flask-app/app.py#L7) and see how the connection details are defined.
```
es = Elasticsearch(host='es')
```
To make this work, we need to tell the Flask container that the ES container is running on `0.0.0.0` host (the port by default is `9200`) and that should make it work, right? Unfortunately that is not correct since the IP `0.0.0.0` is the IP to access ES container from the **host machine** i.e. from my Mac. Another container will not be able to access this on the same IP address. Okay if not that IP, then which IP address should the ES container be accessible by? I'm glad you asked this question.

Now is a good time to start our exploration of networking in Docker. When docker is installed, it creates three networks automatically.

```
$ docker network ls
NETWORK ID          NAME                DRIVER
075b9f628ccc        none                null
be0f7178486c        host                host
8022115322ec        bridge              bridge
```
The **bridge** network is the network in which containers are run by default. So that means that when I ran the ES container, it was running in this bridge network. To validate this, let's inspect the network

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

You can see that our container `e931ab24dedc` is listed under the `Containers` section in the output. What we also see is the IP address this container has been allotted - `172.17.0.2`. Is this the IP address that we're looking for? Let's find out by running our flask container and trying to access this IP.

```javascript
$ docker run -it --rm prakhar1989/foodtrucks-web bash
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
This should be fairly straightforward to you by now. We start the container in the interactive mode with the `bash` process. The `--rm` is a convenient flag for running one off commands since the container gets cleaned up when it's work is done. We try a `curl` but we need to install it first. Once we do that, we see that we can indeed talk to ES on `172.17.0.2:9200`. Awesome!

Although we have figured out a way to make the containers talk to each other, there are still two problems with this approach -

1. We would need to a add an entry into the `/etc/hosts` file of the Flask container so that it knows that `es` hostname stands for `172.17.0.2`. If the IP keeps changing, manually editing this entry would be quite tedious.

2. Since the *bridge* network is shared by every container by default, this method is **not secure**.

The good news that Docker has a great solution to this problem. It allows us to define our own networks while keeping them isolated. It also tackles the `/etc/hosts` problem and we'll quickly see how.

Let's first go ahead and create our own network.
```raw
$ docker network create foodtrucks
1a3386375797001999732cb4c4e97b88172d983b08cd0addfcb161eed0c18d89

$ docker network ls
NETWORK ID          NAME                DRIVER
1a3386375797        foodtrucks          bridge
8022115322ec        bridge              bridge
075b9f628ccc        none                null
be0f7178486c        host                host
```
The `network create` command creates a new *bridge* network, which is what we need at the moment. There are other kinds of networks that you can create, and you are encouraged to read about them in the official [docs](https://docs.docker.com/engine/userguide/networking/dockernetworks/).

Now that we have a network, we can launch our containers inside this network using the `--net` flag. Let's do that - but first, we will stop our ES container that is running in the bridge (default) network.

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
We've done the same thing as earlier but this time we gave our ES container a name `es`. Now before we try to run our flask container, let's inspect what happens when we launch in a network.

```javascript
$ docker run -it --rm --net foodtrucks prakhar1989/foodtrucks-web bash
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

Wohoo! That works! Magically Docker made the correct host file entry in `/etc/hosts` which means that `es:9200` correctly resolves to the IP address of the ES container. Great! Let's launch our Flask container for real now -
```javascript
$ docker run -d --net foodtrucks -p 5000:5000 --name foodtrucks-web prakhar1989/foodtrucks-web
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

Head over to [http://0.0.0.0:5000](http://0.0.0.0:5000) and see your glorious app live! Although that might have seemed like a lot of work, we actually just typed 4 commands to go from zero to running. I've collated the commands in a [bash script](https://github.com/prakhar1989/FoodTrucks/blob/master/setup-docker.sh).
```
#!/bin/bash

# build the flask container
docker build -t prakhar1989/foodtrucks-web .

# create the network
docker network create foodtrucks

# start the ES container
docker run -d --net foodtrucks -p 9200:9200 -p 9300:9300 --name es elasticsearch

# start the flask app container
docker run -d --net foodtrucks -p 5000:5000 --name foodtrucks-web prakhar1989/foodtrucks-web
```

Now imagine you are distributing your app to a friend, or running on a server that has docker installed. You can get a whole app running with just one command!

```javascript
$ git clone https://github.com/prakhar1989/FoodTrucks
$ cd FoodTrucks
$ ./setup-docker.sh
```
And that's it! If you ask me, I find this to be an extremely awesome, and a powerful way of sharing and running your applications!

<a id="docker-links"></a>
##### Docker Links

Before we leave this section though, I should mention that `docker network` is a relatively new feature - it was part of Docker 1.9 [release](https://blog.docker.com/2015/11/docker-1-9-production-ready-swarm-multi-host-networking/). Before `network` came along, links were the accepted way of getting containers to talk to each other. According to the official [docs](https://docs.docker.com/engine/userguide/networking/default_network/dockerlinks/), linking is expected to be deprecated in future releases. In case you stumble across tutorials or blog posts that use `link` to bridge containers, remember to use `network` instead.


<a id="docker-compose"></a>
### 3.3 Docker Compose

Till now we've spent all our time exploring the Docker client. In the Docker ecosystem, however, there are a bunch of other open-source tools which play very nicely with Docker. A few of them are -

1. [Docker Machine](https://docs.docker.com/machine/) - Create Docker hosts on your computer, on cloud providers, and inside your own data center
2. [Docker Compose](https://docs.docker.com/compose/) - A tool for defining and running multi-container Docker applications.
3. [Docker Swarm](https://docs.docker.com/swarm/) - A native clustering solution for Docker

In this section, we are going to look at one of these tools, Docker Compose, and see how it can make dealing with multi-container apps easier.

The background story of Docker Compose is quite interesting. Roughly two years ago, a company called OrchardUp launched a tool called Fig. The idea behind Fig was to make isolated development environments work with Docker. The project was very well received on [Hacker News](https://news.ycombinator.com/item?id=7132044) - I oddly remember reading about it but didn't quite get the hang of it.

The [first comment](https://news.ycombinator.com/item?id=7133449) on the forum actually does a good job of explaining what Fig is all about.

> So really at this point, that's what Docker is about: running processes. Now Docker offers a quite rich API to run the processes: shared volumes (directories) between containers (i.e. running images), forward port from the host to the container, display logs, and so on.  But that's it: Docker as of now, remains at the process level.

> While it provides options to orchestrate multiple containers to create a single "app", it doesn't address the managemement of such group of containers as a single entity.
> And that's where tools such as Fig come in: talking about a group of containers as a single entity. Think "run an app" (i.e. "run an orchestrated cluster of containers") instead of "run a container".

It turns out that a lot of people using docker agree with this sentiment. Slowly and steadily as Fig became popular, Docker Inc. took notice, acquired the company and re-branded Fig as Docker Compose.

So what is *Compose* used for? Compose is a tool that is used for defining and running multi-container Docker apps in an easy way. It provides a configuration file called `docker-compose.yml` that can be used to bring up an application and the suite of services it depends on with just one command.

Let's see if we can create a `docker-compose.yml` file for our SF-Foodtrucks app and evaluate whether Docker Compose lives up to its promise.

The first step, however, is to install Docker Compose. If you're running Windows or Mac, Docker Compose is already installed as it comes in the Docker Toolbox. Linux users can easily get their hands on Docker Compose by following the [instructions](https://docs.docker.com/compose/install/) on the docs. Since Compose is written in Python, you can also simply do `pip install docker-compose`. Test your installation with -
```
$ docker-compose version
docker-compose version 1.7.1, build 0a9ab35
docker-py version: 1.8.1
CPython version: 2.7.9
OpenSSL version: OpenSSL 1.0.1j 15 Oct 2014
```

Now that we have it installed, we can jump on the next step i.e. the Docker Compose file `docker-compose.yml`. The syntax for the `yml` is quite simple and the repo already contains the docker-compose [file](https://github.com/prakhar1989/FoodTrucks/blob/master/docker-compose.yml) that we'll be using.
```
version: "2"
services:
  es:
    image: elasticsearch
  web:
    image: prakhar1989/foodtrucks-web
    command: python app.py
    ports:
      - "5000:5000"
    volumes:
      - .:/code
```
Let me breakdown what the file above means. At the parent level, we define the names of our services - `es` and `web`. For each service, that Docker needs to run, we can add additional parameters out of which `image` is required. For `es`, we just refer to the `elasticsearch` image available on the Docker Hub. For our Flask app, we refer to the image that we built at the beginning of this section.

Via other parameters such as `command` and `ports` we provide more information about the container. The `volumes` parameter specifies a mount point in our `web` container where the code will reside. This is purely optional and is useful if you need access to logs etc. Refer to the [online reference](https://docs.docker.com/compose/compose-file) to learn more about the parameters this file supports.

> Note: You must be inside the directory with the `docker-compose.yml` file in order to execute most Compose commands.

Great! Now the file is ready, let's see `docker-compose` in action. But before we start, we need to make sure the ports are free. So if you have the Flask and ES containers running, lets turn them off.
```
$ docker stop $(docker ps -q)
39a2f5df14ef
2a1b77e066e6
```

Now we can run `docker-compose`. Navigate to the food trucks directory and run `docker-compose up`.

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

Head over to the IP to see your app live. That was amazing wasn't it? Just few lines of configuration and we have two Docker containers running successfully in unison. Let's stop the services and re-run in detached mode.

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

Unsurprisingly, we can see both the containers running successfully. Where do the names come from? Those were created automatically by Compose. But does *Compose* also create the network automatically? Good question! Let's find out.

First off, let us stop the services from running. We can always bring them back up in just one command.

```
$ docker-compose stop
Stopping foodtrucks_web_1 ... done
Stopping foodtrucks_es_1 ... done
```

While we're are at it, we'll also remove the `foodtrucks` network that we created last time. This should not be required since *Compose* would automatically manage this for us.

```
$ docker network rm foodtrucks
$ docker network ls
NETWORK ID          NAME                DRIVER
4eec273c054e        bridge              bridge
9347ae8783bd        none                null
54df57d7f493        host                host
```

Great! Now that we have a clean slate, let's re-run our services and see if *Compose* does it's magic.

```
$ docker-compose up -d
Recreating foodtrucks_es_1
Recreating foodtrucks_web_1
$ docker ps
CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                    NAMES
f50bb33a3242        prakhar1989/foodtrucks-web   "python app.py"          14 seconds ago      Up 13 seconds       0.0.0.0:5000->5000/tcp   foodtrucks_web_1
e299ceeb4caa        elasticsearch                "/docker-entrypoint.s"   14 seconds ago      Up 14 seconds       9200/tcp, 9300/tcp       foodtrucks_es_1
```
So far, so good. Time to see if any networks were created.

```
$ docker network ls
NETWORK ID          NAME                 DRIVER
0c8b474a9241        bridge               bridge              
293a141faac3        foodtrucks_default   bridge              
b44db703cd69        host                 host                
0474c9517805        none                 null  
```
You can see that compose went ahead and created a new network called `foodtrucks_default` and attached both the new services in that network so that each of these are discoverable to the other. Each container for a service joins the default network and is both reachable by other containers on that network, and discoverable by them at a hostname identical to the container name. Let's see if that information resides in `/etc/hosts`.

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

Whoops! It turns out that this file has no idea what the `es` network. So how is our app working? Let's see if can ping this hostname -

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

Voila! That works. So somehow, this container is magically able to ping `es` hostname.  It turns out that in Docker 1.10 a new networking system was added that does service discovery using a DNS server. If you're interested, you can read more about the [proposal](https://github.com/docker/libnetwork/issues/767) and [release notes](https://blog.docker.com/2016/02/docker-1-10/).

That concludes our tour of Docker Compose. With Docker Compose, you can also pause your services, run a one-off command on a container and even scale the number of containers. I also recommend you checkout a few other [use-cases](https://docs.docker.com/compose/overview/#common-use-cases) of Docker compose. Hopefully I was able to show you how easy it is to manage multi-container environments with Compose. In the final section, we are going to deploy our app to AWS!

<a id="aws-ecs"></a>
### 3.4 AWS Elastic Container Service
In the last section we used `docker-compose` to run our app locally with a single command: `docker-compose up`. Now that we have a functioning app we want to share this with the world, get some users, make tons of money and buy a big house in Miami. Executing the last three are beyond the scope of tutorial, so we'll spend our time instead on figuring out how we can deploy our multi-container apps on the cloud with AWS.


If you've read this far you are much pretty convinced that Docker is a pretty cool technology. And you are not alone. Seeing the meteoric rise of Docker, almost all Cloud vendors started working on adding support for deploying Docker apps on their platform. As of today, you can deploy Docker apps on AWS, [Azure](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-docker-vm-extension/), [Rackspace](http://blog.rackspace.com/docker-with-the-rackspace-open-cloud/), [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-use-the-digitalocean-docker-application) and many others. We already got a primer on deploying single container apps with Elastic Beanstalk and in this section we are going to look at [Elastic Container Service (or ECS)](https://aws.amazon.com/ecs/) by AWS.

AWS ECS is a scalable and super flexible container management service that supports Docker containers. It allows you to operate a Docker cluster on top of EC2 instances via an easy-to-use API. Where Beanstalk came with reasonable defaults, ECS allows you to completely tune your environment as per your needs. This makes ECS, in my opinion, quite complex to get started with.

Luckily for us, ECS has a friendly [CLI](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI.html) tool that understands Docker Compose files and automatically provisions the cluster on ECS! Since we already have a functioning `docker-compose.yml` it should not take a lot of effort in getting up and running on AWS. So let's get started!

The first step is to install the CLI. As of this writing, the CLI is not supported on Windows. Instructions to install the CLI on both Mac and Linux are explained very clearly in the [official docs](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_installation.html). Go ahead, install the CLI and when you are done, verify the install by running

```
$ ecs-cli --version
ecs-cli version 0.1.0 (*cbdc2d5)
```
The first step is to get a keypair which we'll be using to log into the instances. Head over to your [EC2 Console](https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#KeyPairs:sort=keyName) and create a new keypair. Download the keypair and store it in a safe location. Another thing to note before you move away from this screen is the region name. In my case, I have named my key - `ecs` and set my region as `us-east-1`. This is what I'll assume for the rest of this walkthrough.

<img src="images/keypair.png" alt="keypair.png" />

The next step is to configure the CLI.
```
$ ecs-cli configure --region us-east-1 --cluster foodtrucks
INFO[0000] Saved ECS CLI configuration for cluster (foodtrucks)
```
We provide the `configure` command with the region name we want our cluster to reside in and a cluster name. Make sure you provide the **same region name** that you used when creating the keypair. If you've not configured the [AWS CLI](https://aws.amazon.com/cli/) on your computer before, you can use the official [guide](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-set-up.html), which explains everything in great detail on how to get everything going.

The next step enables the CLI to create a [CloudFormation](https://aws.amazon.com/cloudformation/) template.
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
Here we provide the name of the keypair we downloaded initially (`ecs` in my case), the number of instances that we want to use (`--size`) and the type of instances that we want the containers to run on. The `--capability-iam` flag tells the CLI that we acknowledge that this command may create IAM resources.

The last and final step is where we'll use our `docker-compose.yml` file. We'll need to make a tiny change, so instead of modifying the original, let's make a copy of it and call it `aws-compose.yml`. The contents of [this file](https://github.com/prakhar1989/FoodTrucks/blob/master/aws-compose.yml) (after making the changes) look like (below) -
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

```
$ docker push prakhar1989/foodtrucks-web
```

Great! Now let's run the final command that will deploy our app on ECS!

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
It's not a coincidence that the invocation above looks similar to the one we used with **Docker Compose**. The `--file` argument is used to override the default file (`docker-compose.yml`) that the CLI will read. If everything went well, you should see a `desiredStatus=RUNNING lastStatus=RUNNING` as the last line.

Awesome! Our app is live, but how can we access it?
```
ecs-cli ps
Name                                      State    Ports                     TaskDefinition
845e2368-170d-44a7-bf9f-84c7fcd9ae29/web  RUNNING  54.86.14.14:80->5000/tcp  ecscompose-foodtrucks:2
845e2368-170d-44a7-bf9f-84c7fcd9ae29/es   RUNNING                            ecscompose-foodtrucks:2
```
Go ahead and open [http://54.86.14.14](http://54.86.14.14) in your browser and you should see the Food Trucks in all its black-yellow glory!
Since we're on the topic, let's see how our [AWS ECS](https://console.aws.amazon.com/ecs/home?region=us-east-1#/clusters) console looks.

<img src="images/cluster.png" alt="ECS cluster" />
<img src="images/tasks.png" alt="ECS cluster" />

We can see above that our ECS cluster called 'foodtrucks' was created and is now running 1 task with 2 container instances. Spend some time browsing this console to get a hang of all the options that are here.

So there you have it. With just a few commands we were able to deploy our awesome app on the AWS cloud!
___________

<a href="#table-of-contents" class="top" id="preface">Top</a>
<a id="wrap-up"></a>
## 4.0 Wrap Up
And that's a wrap! After a long, exhaustive but fun tutorial you are now ready to take the container world by storm! If you followed along till the very end then you should definitely be proud of yourself. You learnt how to setup Docker, run your own containers, play with static and dynamic websites and most importantly got hands on experience with deploying your applications to the cloud!

I hope that finishing this tutorial makes you more confident in your abilities to deal with servers. When you have an idea of building your next app, you can be sure that you'll be able to get it in front of people with minimal effort.

<a id="next-steps"></a>
### 4.1 Next Steps
Your journey into the container world has just started! My goal with this tutorial was to whet your appetite and show you the power of Docker. In the sea of new technology, it can be hard to navigate the waters alone and tutorials such as this one can provide a helping hand. This is the Docker tutorial I wish I had when I was starting out. Hopefully it served its purpose of getting you excited about containers so that you no longer have to watch the action from the sides.

Below are a few additional resources that will be beneficial. For your next project, I strongly encourage you to use Docker. Keep in mind - practice makes perfect!

**Additional Resources**

- [Awesome Docker](https://github.com/veggiemonk/awesome-docker)
- [Hello Docker Workshop](http://docker.atbaker.me/)
- [Building a microservice with Node.js and Docker](https://www.youtube.com/watch?v=PJ95WY2DqXo)
- [Why Docker](https://blog.codeship.com/why-docker/)
- [Docker Weekly](https://www.docker.com/newsletter-subscription) and [archives](https://blog.docker.com/docker-weekly-archives/)
- [Codeship Blog](https://blog.codeship.com/)

Off you go, young padawan!

<a id="feedback"></a>
### 4.2 Give Feedback
Now that the tutorial is over, it's my turn to ask questions. How did you like the tutorial? Did you find the tutorial to be a complete mess or did you have fun and learn something?

Send in your thoughts directly to [me](mailto:prakhar@prakhar.me) or just [create an issue](https://github.com/prakhar1989/docker-curriculum/issues/new). I'm on [Twitter](https://twitter.com/prakharsriv9), too, so if that's your deal, feel free to holler there!

I would totally love to hear about your experience with this tutorial. Give suggestions on how to make this better or let me know about my mistakes. I want this tutorial to be one of the best introductory tutorials on the web and I can't do it without your help.

___________

<a href="#table-of-contents" class="top" id="preface">Top</a>
<a id="references"></a>
## References
- [What containers can do for you](http://radar.oreilly.com/2015/01/what-containers-can-do-for-you.html)
- [What is Docker](https://www.docker.com/what-docker)
- [A beginner's guide to deploying production web apps](https://medium.com/@j_mcnally/a-beginner-s-guide-to-deploying-production-web-apps-to-docker-9458409c6180?_tmc=WrhaI1ejJlMmTpUmHOhTFZsYaUSPUP1yvyq19dsRQ5A#.bl50ga0uz)
- [Running Web Application in Linked Docker Containers Environment](https://aggarwalarpit.wordpress.com/2015/12/06/running-web-application-in-linked-docker-containers-environment/)
