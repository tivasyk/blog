---
layout: post
title:  "openwrt: порівняння затримки з qos та без"
date:   2020-03-21 10:00
categories: blog
tags: 
  - щоденник
  - комп'ютери
  - openwrt
  - мережі
  - linux
  - підказка
---

субота, карантин; що робить сисадмін? грається домашнім рутером або сервером: колись я тут [нахвалявся]({% post_url 2018/2018-10-16-blog-post_16 %}) шейпінгом трафіку для гостьової мережі на маршрутизаторі з прошивкою openwrt, — наразі маю час перевірити, як воно насправді працює. обмеження швидкости для гостей працює — цікаво було би ще покращити [затримку](https://uk.wikipedia.org/wiki/%D0%A7%D0%B0%D1%81_%D0%B7%D0%B0%D1%82%D1%80%D0%B8%D0%BC%D0%BA%D0%B8_(RTT)) (rtt, пінг) для основної мережі вдома.

це не готовий допис: пишу «на колінці», посьорбуючи ранкову каву і забавляючись ліниво в терміналі.

#### тестовий стенд

перш ніж покращувати, треба ж побачити, що саме і куди «тягти»? `ping` на google.com міряє затримку, `speedtest-cli` навантажує мережу; щось таке дозволяє запускати їх паралельно і спостерігати результат наживо:

```
> (sleep 5s; echo "--- 8< ---"; speedtest-cli --simple; echo "--- >8 ---"; sleep 5s; killall ping) & ping -i 2 google.com
```

друга частина (після `&`) пінгує google що дві секунди, перша `(...) &` — запускає на тлі `speedtest-cli` з невеликою затримкою, тоді вбиває ping. 

#### перша випроба (без qos)

наразі близько десятої ранку суботи, домашні ще сплять, ніхто нічого не тягне з мережі. перший тест ― на сервері (див. зняток), який «висить» на кабелі з гігабітового порта рутера, котрий через кабельний модем отримує [інтернет 60/10 мбіт/с]({% post_url 2019/2019-11-13-acanac %}):

* нормальний пінг: 17-25 мс
* speedtest: 53-67 мс (приблизно x3)
* швидкість: 66/11 мбіт/с (оптимістично)

другий тест — на ноутбуці (wifi приблизно 58 мбіт/с макс.):

* нормальний пінг: 18-30 мс
* speedtest: 65-222 мс (приблизно x4..6)
* швидкість: 3/5 мбіт/с (це щось аномальне, поігнорую наразі)

![зображення](/assets/images/2020/2020-03-21-openwrt-qos_01.jpg)

#### sqm на openwrt

на рутері з openwrt потрібні пакунки `sqm-scripts` та (для графічного інтерфейсу) `luci-app-sqm` — в меню _network_ з'являється пункт _sqm qos_.

закладка _queues_: обираю зовнішній інтерфейс (`eth0.2` в моєму випадку), ліміти швидкості завантаження та відвантаження встановлюю 95% від максимально досяжних (57/9,5 мбіт/с). закладка _link layer adaptation_: тип підключення — _ethernet with overhead_, значення _per packet overhead_ залишаю 0. 

решту залишаю по замовчуванню. зберігаю налаштування; час для другої випроби…

![зображення](/assets/images/2020/2020-03-21-openwrt-qos_03.jpg)

#### друга випроба (qos для wan)

домашні попрокидалися і повмикали youtube — тож роблю тест в умовах, «наближених до реальних»:

сервер (кабель, див. зняток):

* нормальний пінг: 18-22 мс
* speedtest: 17-24 мс (перемога!)
* швидкість: 44/7 мбіт/с

ноутбук (бездріт):

* нормальний пінг: 18-33 мс
* speedtest: 20-78 мс (приблизно x2)
* швидкість: 24/8 мбіт/с

![зображення](/assets/images/2020/2020-03-21-openwrt-qos_04.jpg)

#### висновок

вочевидячки, навіть з таким найпростішим налаштуванням все значно покращилося для кабельного підключення. 

результати для wifi дуже «стрибають» (не видно на цім однім тесті, але я робив декілька), особливо коли підключено багато пристроїв (налічив 7-12) — треба далі розбиратися, що можна покращити.

далі буде.
