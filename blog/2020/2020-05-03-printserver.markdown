---
layout: post
title:  "сервер бездрайверного друку на debian?"
date:   2020-05-03 23:49
categories: blog
tags: 
  - щоденник
  - комп'ютери
  - linux
  - підказка
  - позитив
---

маю принтер (лазерний brother hl-2040); він [був підключений по usb до рутера]({% post_url 2018/2018-10-16-blog-post_16 %}) (d-link dir-835) з прошивкою openwrt і «прозорим» драйвером p910nd; достатньо було на будь-якому комп'ютері в домашній мережі встановити драйвер принтера, вказати ip рутера як адресу мережевого принтера — і можна друкувати. 

so far so good, але ось я [замінив рутер на linksys ea8300]({% post_url 2020/2020-04-26-linksys-ae8300 %}), і поки що граюся з фабричною прошивкою, котра призначена для «нормальних користувачів» — тих, хто тицяє галочки веб-інтерфейсу і не полізе по ssh встановлювати щось таке, чого не передбачив виробник. я майже чую, як служба підтримки не приховує зневаги: «што? принтер на usb?  йди купи собі нормальний принтер з wifi, голожопенко!»

але я маю сервер з debian, і тут відкривається вибір:

* той самий p910nd на debian, або…
* …сервер друку по-дорослому?

чому драйвер p910nd «прозорий»? тому що це не сервер друку: немає черги чи якогось адміністрування завдань — клієнтський драйвер неначе напряму спілкується з принтером. навіть не знаю, що буде, якщо декілька користувачів захочуть щось надрукувати одночасно. коротше кажучи, захотілося налаштувати **правильний сервер друку** «в горнятках» ([cups](https://uk.wikipedia.org/wiki/CUPS)), а в ідеалі — [без драйверів](https://wiki.debian.org/CUPSDriverlessPrinting) на клієнтських пристроях.

зразу зізнаюся, що «з наскоку» не вийшло, тож цей допис — спроба задокументувати, щоби розібратися, що пішло не так і як зробити правильно.


#### підключення принтера

тут все просто: принтер в порт usb сервера, `lsusb` повинен показати пристрій:

    > lsusb
    ...
    Bus 004 Device 003: ID 04f9:0028 Brother Industries, Ltd Printer
    ...
    
не знаю, чи це потрібно, але `lsusb -t` показує трохи точніше, до якого порту підключено принтер (хоча відповідність нумерації зовнішнім портам — під величезним питанням):

    > lsusb -t
    ...
    /:  Bus 04.Port 1: Dev 1, Class=root_hub, Driver=uhci_hcd/2p, 12M
    |__ Port 2: Dev 3, If 0, Class=Printer, Driver=, 12M
    ...

на цьому етапі система повинна була би побачити принтер на порті usb і задіяти драйвер `usblp`… але з тексту вище (*driver=*) видно, що в мене щось пішло не так. ядро звітує про якусь помилку:

    # dmesg | grep usblp
    ...
    [6058722.788649] usblp 4-2:1.0: usblp0: USB Bidirectional printer dev 3 if 0 alt 0 proto 2 vid 0x04F9 pid 0x0028
    [6058727.794263] usblp0: removed
    [6058728.803658] usblp: can't set desired altsetting 0 on interface 0

можливо, після моєї першої спроби налаштувати cups. «пересмикнув» кабель — принтер підключився: тепер `lsusb` звітує, що драйвер задіяно:

    > lsusb -t
    ...
    /:  Bus 04.Port 1: Dev 1, Class=root_hub, Driver=uhci_hcd/2p, 12M
    |__ Port 2: Dev 4, If 0, Class=Printer, Driver=usblp, 12M

`dmesg` теж заспокоюює:

    # dmesg | grep usblp

після цього в системі з'являється файл пристрою `/dev/usb/lp0`:

    # ls /dev/usb/*
    /dev/usb/lp0
    
    
#### встановлення cups

далі я керуюся [офіційною підказкою debian](https://wiki.debian.org/SystemPrinting#Software_Installation). по-перше, встановив cups (але не повний набір `task-print-server` — це проблема?):

    # apt-get update && apt-get install cups cups-client 
    
я також підглянув, який [драйвер рекомендовано для мого принтера](https://www.openprinting.org/printer/Brother/Brother-HL-2040), завантажив відповідний файл ppd та закинув до `/etc/cups/ppd`.

    > ls -l /etc/cups/ppd
    ...
    -rw-r--r-- 1 root root 13505 May  6 13:48 Brother-HL-2040-hl1250.ppd

перевірка, чи запущено сервіс cups:

    > systemctl status cups
    ● cups.service - CUPS Scheduler
      Loaded: loaded (/lib/systemd/system/cups.service; enabled; vendor preset: enabled)
      Active: active (running) since Wed 2020-05-06 13:53:28 EDT; 5h 6min ago
        Docs: man:cupsd(8)
    Main PID: 9802 (cupsd)
       Tasks: 1 (limit: 4915)
      Memory: 1.4M
         CPU: 140ms
      CGroup: /system.slice/cups.service
              └─9802 /usr/sbin/cupsd -l


#### веб-інтерфейс cups

на цьому етапі [інструкція зі встановлення cups](https://wiki.debian.org/SystemPrinting) згадує веб-інтерфейс для адміністрування, доступний за адресою [localhost:631/admin](https://localhost:631/admin) — але «засідка» в тому, що мій сервер не просто «безголовий» (без монітора), ба навіть не має іксів (графічного сервера), і запустити якогось вогнелиса на самому сервері не можу… а віддалене адміністрування за замовчуванням відключене. замість порпатися в `/etc/cups/`, безпечніше, мабуть, скористатися `cupsctl`:

    # cupsctl --remote-admin --remote-any --share-printers 

…і після цього можна «постукати» з сусідньої машини в локальній мережі на порт 631.

![веб-інтерфейс cups](/assets/images/2020/2020-05-03-printserver_01.jpg)


#### налаштування сервера cups

маючи доступ до адмінконсолі cups по https, пробую додати свій принтер (сервер запитає логін/пароль) — so far so good, бачу brother hl-2040 в переліку локальних друкарок: додається і налаштовується… і навіть тестова сторінка друкується.

![налаштування сервера друку cups](/assets/images/2020/2020-05-03-printserver_02.jpg)


#### налаштування друку на клієнтській машині (linux)

зненацька, мій manjaro побачив принтер в мережі без жодних танців з бубном — навіть адресу сервера не довелося вказувать — лише завантажити той самий драйвер (ppd) з openprinting.org, що і на сервері.

![налаштування друку на клієнті linux](/assets/images/2020/2020-05-03-printserver_03.jpg)

спроба друку з файлу pdf, втім, не проходить: драйвер на клієнті каже «sending data to printer», але згодом сервер відповідає «filter failed». журнал cups (`/var/log/cups/error_log`) на сервері:

    # less +G /var/log/cups/error_log
    ...
    D [06/May/2020:22:35:37 -0400] [Job 15] ================================================
    D [06/May/2020:22:35:37 -0400] [Job 15] File: /var/spool/cups/d00015-001
    D [06/May/2020:22:35:37 -0400] [Job 15] ================================================
    D [06/May/2020:22:35:37 -0400] [Job 15] Cannot process \"/var/spool/cups/d00015-001\": Unknown filetype.
    D [06/May/2020:22:35:37 -0400] [Job 15] Process is dying with \"Could not print file /var/spool/cups/d00015-001
    D [06/May/2020:22:35:37 -0400] [Job 15] \", exit stat 2
    D [06/May/2020:22:35:37 -0400] [Job 15] Cleaning up...
    D [06/May/2020:22:35:37 -0400] [Job 15] PID 14997 (/usr/lib/cups/filter/foomatic-rip) stopped with status 2.
    ...

вікі arch linux [натякає](https://wiki.archlinux.org/index.php/CUPS/Troubleshooting#CUPS:_%22Filter_failed%22), що проблема може виникати через  недовстановлені залежності для cups: ghostscript та foomatic… але я перевіряю — вони є:

    # dpkg -l | grep "ghostscript\|cups-filters"
    ii  cups-filters                1.11.6-3+deb9u1     amd64   OpenPrinting CUPS Filters - Main Package
    ii  cups-filters-core-drivers   1.11.6-3+deb9u1     amd64   OpenPrinting CUPS Filters - PPD-less printing
    ii  ghostscript                 9.26a~dfsg-0+deb9u6 amd64   interpreter for the PostScript language and for PDF


поліз до тенет, і за три години нагуглив та перечитав море згадок подібної проблеми («foomatic-rip stopped with status 2»), найчастіше з абсолютно різними пропозиціями щодо розв'язку… у відчаї, я перечепився через таке: *«everyone who sees this problem: you can avoid it by only performing job transformation at the server. do this by setting the client queues to „raw“»* ([тиць](https://bugzilla.redhat.com/show_bug.cgi?id=1160913#c11)).

…гхм? на клієнтській машинці в налаштуваннях принтера я замінив драйвер (такий, як на сервері, рекомендований для brother hl-2040) загальним — *generic > ipp everywhere printer* (використовується для бездрайверного друку), і…

…зненацька — все працює!

![налаштування друку на клієнті linux](/assets/images/2020/2020-05-03-printserver_04.jpg)


#### бонус!

перевірив, і таки так — тепер воно друкує навіть зі смартфонів у локальній мережі. отак випадково я собі налаштував отой «бездрайверний» друк вдома.
