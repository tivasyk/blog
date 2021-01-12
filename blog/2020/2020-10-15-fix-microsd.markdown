---
layout: post
title:  "спроба ремонту «поламаної» картки micro sd"
date:   2020-10-15 09:00
categories: blog
tags: 
  - щоденник
  - комп'ютери
  - linux
  - підказки
---
дано: карта пам'яті micro sdhc (sundisk ultra 16 гб), що використовується як системний диск для «малинки», з якої малинка більше не завантажується через помилку. задача — щоби завантажувалося! тобто: або «поремонтувати» проблемні розділи на картці, або замінити з мінімумом проблем. отже, картка в карт-рідері на «дорослому» комп'ютері (з linux, звісно) — і пробую розбиратися…

куди підключилося?

    # lsblk
    NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    ...
    sdh      8:112  1  14,8G  0 disk 
    ├─sdh1   8:113  1  43,1M  0 part 
    └─sdh2   8:114  1  14,8G  0 part
    
перш за все, роблю локальну копію картки, «на всяк випадок» (картка 16 гб підключена до usb 2.0, процес займає кілька хвилин):

    # dd if=/dev/sdh of=~/sdh.img status=progress
    
дивлюся, як розподілено і відформатовано картку («малинку» запускав не я, документації немає):

    fdisk -l /dev/sdh
    Disque /dev/sdh : 14,84 GiB, 15931539456 octets, 31116288 secteurs
    Modèle de disque : USB3.0 CRW   -SD
    Unités : secteur de 1 × 512 = 512 octets
    Taille de secteur (logique / physique) : 512 octets / 512 octets
    taille d'E/S (minimale / optimale) : 512 octets / 512 octets
    Type d'étiquette de disque : dos
    Identifiant de disque : 0x72047edf

    Périphérique Amorçage Début      Fin Secteurs Taille Id Type
    /dev/sdh1              8192    96453    88262  43,1M  c W95 FAT32 (LBA)
    /dev/sdh2             98304 31116287 31017984  14,8G 83 Linux

файлова система на кожному з розділів:

    # blkid /dev/sdh1
    /dev/sdh1: LABEL_FATBOOT="boot" LABEL="boot" UUID="D332-A79C" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="72047edf-01"
    # blkid /dev/sdh2
    /dev/sdh2: LABEL="rootfs" UUID="31688870-9e02-4182-8cda-25195c5a4739" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="72047edf-02"

отже, [два розділи](https://en.wikipedia.org/wiki/Partition_type): vfat (43 мб) та ext4 (14,8 гб). обов'язково відмонтувати/перевірити:

    # umount /dev/sdh1
    # umount /dev/sdh2

перевірка розділу з vfat перечіпляється через першу проблему:

    # fsck.fat /dev/sdh1
    fsck.vfat /dev/sdh1
    fsck.fat 4.1 (2017-01-24)
    0x41: Dirty bit is set. Fs was not properly unmounted and some data may be corrupt.
    1) Remove dirty bit
    2) No action
    ? 
    
якщо вибрати першу опцію і записати зміни на картку — повторна перевірка дає той самий результат, як наче запис нічого не міняє… це перший індикатор того, що все може бути погано: коли картка надто зношується, контролер блокує запис, щоби принаймні запобігти втраті наявної інформації.
    
перевірка розділу з ext4:

    # fsck.ext4 /dev/sdh2
    e2fsck 1.45.6 (20-Mar-2020)
    rootfs : récupération du journal
    le drapeau needs_recovery n'est pas activé, mais le journal contient des données.
    Exécuter quand même le journal<o>? non
    Effacer le journal<o>? oui
    fsck.ext4: impossible d'initialiser les drapeaux du superbloc sur rootfs
    rootfs : **ATTENTION : le système de fichiers contient encore des erreurs**

інше формулювання, але схожа причина: fsck жаліється, що не може встановити «прапорець» суперблоку і аварійно зупиняється. жодні «вмовляння» чи знайдені в тенетах підказки з перезапису суперблоків з резервних копій (на тому ж носії) не допомагають.

повертаюся до образу (`sdh.img`). роблю собі резервну копію (можна `cp`, але мені [подобається]({% post_url 2020/2020-08-26-fresh-from-bashtan %}) `rsync`):

    rsync -ah --info=progress2 sdh.img sdh.img.backup
    
підглядаю зміщення двох розділів (vfat та ext4) у файлі образу:

    # parted sdh.img
    ...
    (parted) unit                                                             
    Unité ?  [compact]? B
    (parted) print                                                            
    Modèle :  (file)
    Disque /home/tivasyk/rpi_kpi/sdh.img : 15931539456B
    Taille des secteurs (logiques/physiques) : 512B/512B
    Table de partitions : msdos
    Drapeaux de disque : 
    Numéro  Début      Fin           Taille        Type     Système de fichiers  Drapeaux
    1      4194304B   49384447B     45190144B     primary  fat32                lba
    2      50331648B  15931539455B  15881207808B  primary  ext4
    (parted) quit

дивлюся, який перший віртуальний блоковий пристрій ([loop](https://en.wikipedia.org/wiki/Loop_device)) доступний:

    # losetup -f
    /dev/loop0
    
підключаю перший розділ (vfat, віступ 4194304 байтів) як віртуальний диск (хз, чи це правильне формулювання, але біс із ним):

    # losetup -o 4194304 /dev/loop0 sdh.img
    /dev/loop0: [2052]:4331003 (/home/tivasyk/rpi_kpi/sdh.img), index 4194304
    
пробую перевірити за допомогою fsck:

    # fsck -v /dev/loop0
    sudo fsck -v /dev/loop0
    fsck de util-linux 2.36
    fsck.fat 4.1 (2017-01-24)
    Checking we can access the last sector of the filesystem
    0x41: Dirty bit is set. Fs was not properly unmounted and some data may be corrupt.
    1) Remove dirty bit
    2) No action
    ? 
    
те саме питання (див. вище) — але цього разу після виправлення (`1`) та перезапису (`y`), повторна перевірка звітує про здорову файлову систему. монтую цей розділ на теку `test.image` і перевіряю візуально, що читається:

    # mkdir test.image
    # mount -o loop /dev/loop0 test.image
    # ls -l test.image/
    
перелік файлів виглядає ок. розмонтовую розділ, відключаю віртуальний диск:

    # umount /dev/loop0
    # losetup -d /dev/loop0
    
…і підключаю розділ ext4 з того ж файлу (відступ 50331648 байтів) як наступний доступний (`$(losetup -f)`) віртуальний диск, і перевіряю fsck'ом:

    # losetup-o 50331648 $(losetup -f) sdh.img
    /dev/loop2: [2052]:4331003 (/home/tivasyk/rpi_kpi/sdh.img), index 50331648
    # fsck -v /dev/loop2
    fsck de util-linux 2.36
    e2fsck 1.45.6 (20-Mar-2020)
    rootfs : récupération du journal
    Lors de l'effacement de l'i-noeud orphelin 259803 (uid=1000, gid=1000, mode=0100600, taille=8388608)
    Définition du compteur d'i-noeuds libres à 819882 (était 819894)
    Définition du compteur des blocs libres à 2527946 (était 2526121)
    rootfs : propre, 143542/963424 fichiers, 1349302/3877248 blocs
    
наче полікувалося? запускаю ще раз `fsck — каже, що все гаразд. монтую та дивлюся на перелік файлів; він нагадує корінь робочої системи (linux), тож схоже на правду:

    # mount -o loop /dev/loop2 test.image
    # ls -l test.image/
    
відмонтовую, відключаю:

    # umount /dev/loop2
    # losetup -d /dev/loop2

файл з образом картки sd — «здоровий». зайвий раз перевіряю, чи не перепідключилася картка деінде, і що вона не змонтувалася, поки я грався з файлами:

    # lsblk
    ...
    sdh      8:112  1  14,8G  0 disk 
    ├─sdh1   8:113  1  43,1M  0 part 
    └─sdh2   8:114  1  14,8G  0 part 
    
пробую залити «відремонтований» образ (`shd.img`) на флешку (`/dev/sdh`); цей процес значно повільніший за читання, тож іду пити каву:

    # dd if=sdh.img of=/dev/sdh status=progress
    
перевірка наживо, з «малинкою» — дзуськи, не завантажується, та сама помилка (`kernel panic - not syncing: vfs: unable to mount root fs on unknown-block(179,2)`). схоже, я від початку неправильно діагностував проблему? 

[думаю далі…]({% post_url 2020/2020-10-21-reconfigure-raspberry %})
