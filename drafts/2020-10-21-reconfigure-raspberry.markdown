---
layout: post
title:  "перeналаштування малинки"
date:   2020-10-21 09:30
categories: blog
tags: 
  - щоденник
  - комп'ютери
  - linux
  - raspberry pi
  - підказки
---
гаразд, кнедлик'у (knedlyk), ти мав рацію: [картці гаплик]({% post_url 2020/2020-10-15-fix-microsd %}). довелося замінити; за'dd'ив образ `sdh.img` на нову картку, і «малинка» завелася з пів-оберта. постає питання: **як налаштувати «малинку» щоби збільшити час життя карток пам'яті**? бо не хочеться більше такою дурнею займатися на роботі.

з того, що спадає на думку одразу:

* `/var/logs`, `/tmp` (і, може, ще щось?) — до оперативки<sup>†</sup> ([підказка](https://ideaheap.com/2013/07/stopping-sd-card-corruption-on-a-raspberry-pi/));
* відключити підкачку<sup>†</sup> (swap) ([підказка](https://ideaheap.com/2013/07/stopping-sd-card-corruption-on-a-raspberry-pi/))?
* відключити журналювання ext4 ([підказка](https://www.raspberrypi.org/forums/viewtopic.php?p=1056442&sid=3666083a8d732fe9c94d2f0b8bb9866b#p1056442));
* відключити автоматичне поновлення chromium ([підказка](https://stackoverflow.com/a/60651207));
* замінити chromium (в режимі кіоска) чимось легшим?

† в нас модель 4b із 1 гб оперативки — повинно вистачити.


#### монтування в оперативку

два додаткових рядки (плюс коментар) до файлу /etc/fstab (попередньо зробивши резевну копію):

    > sudo nano /etc/fstab
    ...
    # монтування на оперативної пам'яті
    none    /var/log    tmpfs   defaults,noatime,size=1M    0   0
    none    /tmp        tmpfs   defaults,noatime,size=8M    0   0

перезавантаження, перевірка:

    sudo mount -l | grep tmpfs


#### відключення підкачки

сучасні версії raspbian не використовують розділ підкачки, — замість цього сервіс `dphys-swapfile` динамічно керує файлом підкачки. відключається так:

    > sudo dphys-swapfile swapoff
    > sudo dphys-swapfile uninstall

перевірка (рядок swap має показувати нулі:

    > free -h
    
але це лише до перезавантаження. щоби було персистентно, треба виключити сервіс із завантаження:

    > sudo systemctl disable dphys-swapfile
    

#### відключення журналювання

перевірка імені пристроя, який монтується в корінь (не /boot) — має бути щось подібне до `mmcblk0p2`:

    > sudo lsblk

переконатися, що нічого важливого не крутиться (навіть chromium), тоді перемкнути в read-only, відключити журналювання, і не забути скасувати read-only:

    > sudo -i
    # echo u > /proc/sysrq-trigger
    # echo s > /proc/sysrq-trigger
    # tune2fs -O ^has_journal /dev/mmcblk0p2
    # echo s > /proc/sysrq-trigger
    # echo b > /proc/sysrq-trigger
    # exit
    
перевірка (рядок не має містити «has_journal»):

    > sudo tune2fs -l /dev/mmcblk0p2 | grep "features"
    

#### відключення автоматичного поновлення chromium

на жаль, chromium чи не єдиний веб-оглядач з притомним автономним режимом (kiosk mode), ще й встановлений «з коробки» в raspbian. чому на жаль? бо важкий, постійно «ґвалтує» диск, і на додачу ходить шукати якісь поновлення в обхід нормального системного менеджера пакунків. це перетворюється на проблему, тому що chromium при цьому показує ідіотські вигульки, і принаймні один раз допоновлювався так, що довелося ремонтувати малинку.

тимчасове рішення — відключити перевірку поновлень на рік (копі-паста):

    > sudo touch /etc/chromium-browser/customizations/01-disable-update-check; echo CHROMIUM_FLAGS=\"\$\{CHROMIUM_FLAGS\} --check-for-update-interval=31536000\" | sudo tee /etc/chromium-browser/customizations/01-disable-update-check


#### замінити chromium чимось легшим?

спробував, не підходить з різних причин — головне через брак підтримки javascript чи притомного режиму кіоска:

* dillo
* hv3

добре виглядає `surf`, — працює навіть на тестовій малинці b+ (512 мб), але принаймні на одній зі старших малинок видає дивну помилку на 4b (1 гб) і не рендерить сторінку… треба гуглити, бо наразі це [найкращий претендент](http://troubleshooters.com/linux/surf.htm) для того, що мені треба від  «малинок» в конторі — неінтерактивна цифрова афіша з одним мікро-сайтом на весь екран.

далі буде?..
