---
layout: post
title:  "manjaro на хромбоксі"
date:   2019-08-16 22:45
categories: blog
tags: [комп'ютери, linux, chromebox, підказка]
---

най би його шляк трафив, той google! бо ж я таки добряче наморочився, поки запустив manjaro на хромбоксі… але цей допис друкую з manjaro lxqt, що крутиться наживо з флешки на [asus chromebox cn60]({% post_url 2019/2019-08-16-chromebox %}) після перепрошивки того, що тут заміняє bios. поновлення щодо конфігурації: таки тут 4 гб оперативної пам'яті. 

![manjaro lxqt наживо на asus chromebox](/assets/images/2019/2019-08-16-manjaro-on-chromebox.jpg)

насправді, спершу я все поламав: якось одразу зі старту я примудрився ввімкнути режим розробника, і комп'ютер відмовився завантажуватися, мовляв, немає системи. відновив — за допомогою [скрипта](https://dl.google.com/dl/edgedl/chromeos/recovery/linux_recovery.sh), картки sd та лайки. далі виявилося, що на хромбоксі встановлено корпоративну систему для конференцій, а не звичайний користувацький chrome os, і вона вимагає реєстрації… я довгенько експериментував і гуглив, намагаючись обійти цю реєстрацію, але в підсумку зрозумів, що діла не буде. тож я так і не покористувався рідним chrome os'ом.

в загальних рисах, ось що довелося зробити, щоби запустити linux з флешки:

* [x] відкрити корпус і [викрутити один гвинтик](https://dareneiri.github.io/Asus-Chromebox-With-Full-Linux-Install) (!), котрий блокує запис в rom (див. ілюстрацію);
* [x] [увімкнути режим розробника](https://kodi.wiki/view/Chromebox#Put_in_Developer_Mode);
* [x] перезавантажитися і в графічному режимі підключитися до (бездротової) мережі;
* [x] отримати доступ до консолі (ctrl+alt+f2);
* [x] завантажити скрипт [kodi e-z setup](https://mrchromebox.tech/#kodi) і за його допомогою встановити прошивку із загрузчиком uefi (coreboot).

після цього chromebox вже більше не chromebox, а нормальний комп'ютер, здатний завантажуватися з напопичувачів usb. в мене під рукою якраз була флешка з manjaro lxqt. тепер я міркую, чи встановлювати linux на штатний накопичувач 16 гб — а чи замінити одразу на більший ssd? здається, я навіть маю зайвий ssd на 120 гб з інтерфейсом m.2 sata…

**поновлення** (2019-08-18, 10:45). невелике [продовження]({% post_url 2019/2019-08-17-manjaro-on-chromebox-continued %}).

![таємний гвинтик](/assets/images/2019/2019-08-16-manjaro-on-chromebox-02.jpg)
