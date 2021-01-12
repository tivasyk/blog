---
layout: post
title:  "dwarf fortress в контейнері з веб-інтерфейсом?"
date:   2019-08-26 14:10
categories: blog
tags: 
  - комп’ютери
  - linux
  - docker
  - dwarf fortress
  - підказка
---

в кого є dwarf fortress в контейнері docker на домашньому сервері, з веб-інтерфейсом — той я! і так, тепер можна грати просто у веб-оглядачі, хоч на «розумному» телевізорі (ймовірно, не пробував). не знаю, чи є сенс так навантажувати сервер, просто хотів спробувати запустити, скориставшись [підказкою в тенетах](https://github.com/bencawkwell/dockerfile-dwarffortress).

![dwarf fortress в контейнері з веб-інтерфейсом](/assets/images/2019/2019-08-26-dwarf-fortress-container-http.jpg)

отже, як?

#### підготовка

найперше — підключаюся до свого сервера з debian, доступного через адресу на duckdns:

    ssh user@myserver.duckdns.org

роблюся root’ом і прямую до теки з контейнерами (в моєму випадку /opt):

    su
    cd /opt

встановлюю unzip (мінімальний debian не має його «з коробки»):

    apt-get update ; apt-get install unzip

завантажую і витягаю з архіва скрипт і *dockerfile* від бена коквела (ben cawkwell) для побудови образу dwarf fortress:

    TEMP=$(mktemp) ; wget -O $TEMP https://github.com/bencawkwell/dockerfile-dwarffortress/archive/master.zip ; unzip $TEMP ; rm $TEMP

мені не подобається довага назва теки в архіві, тож перейменовую на коротеньке *df*:

    mv dockerfile-dwarffortress-master df ; cd df

#### образ

перш ніж будувати образ, варто налаштувати дещо: мені не потрібен dfhack, тому відкриваю *dockerfile* в щойно створеній теці `/opt/df` і закоментовую один рядок:

    # RUN cd /ansible && ansible-playbook dfhack.yml --connection=local

тепер можна запустити побудову образу, це займе деякий час (в мене — близько п’яти хвилин):

    docker build -t dwarffortress
    
коли процес закінчиться, перевіряю, чи з’явився образ dwarffortress в переліку образів docker:

    docker image ls

#### контейнер

я використовую traefik для доступу до різних контейнерів за іменами, що адресують один айпішник (ip) через сервіс duckdns.org (звучить срашно, але вельми просто по суті), тож замість запускати контейнер з dockerfile’ом, створюю файл `docker-compose.yml`, де можна вказати одразу параметри для traefik’а:

    version: '3'

    services:
      dwarffortress:
        image: dwarffortress
        container_name: dwarffortress
        restart: unless-stopped
        command: dwarffortress --skip-intro
        ports:
          - "6080:6080"
        volumes:
          - "./save/:/df_linux/data/save"
        labels:
          - traefik.enable=true
          - traefik.frontend.rule=Host:df.myserver.duckdns.org
          - traefik.port=6080
          - traefik.docker.network=internet
        networks:
          - internet

    networks:
      internet:
        external: true

директива *volumes* монтує теку `/opt/save` як `/df_linux/data/save` усередині контейнера, інакше збережена гра видалятиметься щоразу, коли контейнер перезапускати.

і, в принципі, це все. запускаю контейнер:

    docker-compose up -d

#### веб-інтерфейс

відкриваю забавку веб-оглядачем: `http://df.myserver.duckdns.org/vnc.html`, клацаю `connect` — і «насолоджуюсь». насправді не дуже, бо є відчутні недоліки (з того, що вже встиг помітити):

* другу сесію (в сусідньому вікні веб-оглядача, чи з іншого комп’ютера) не відкриває (можливо, десь заблоковано в налаштуваннях novnc);
* інтерфейс dwarf fortress не масштабується на все вікно (можливо, я ще щось не налаштував);
* все відкрито: і тека з vnc.html, і безпарольний доступ до synthatqce novnc;
* dwarf fortress, xorg і novnc разом досить сильно завантажують процесор;
* ще й fps досить невисокий — ймовірно, через слабкий процесор на сервері й веб-інтерфейс.

ну, але спробувати було цікаво.

![вікно підключення novnc](/assets/images/2019/2019-08-26-dwarf-fortress-container-http-02.jpg)