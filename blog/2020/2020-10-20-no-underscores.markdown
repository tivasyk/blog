---
layout: post
title:  "жодних підкреслень?"
date:   2020-10-20 10:37
categories: blog
tags: 
  - щоденник
  - комп'ютери
  - мережі
---
гхм… соромно, але я лише щойно (!) дізнався, що  іменах вузлів (hostname) підкреслення не допускаються ([rfc 952](https://tools.ietf.org/html/rfc952)); допустимі символи: латинські літери (a-z), цифри (0-9) та мінус (-); крапка (.) — лише як розділовий знак між частинами доменного імені. підкреслення (_) використовується як префікс міток сервісів та протоколів, але не в іменах вузлів ([rfc 2782](https://www.ietf.org/rfc/rfc2782.txt)). виявив випадково, спробувавши перейменувати машинку за допомогою `hostnamectl` — утиліта поігнорувала підкреслення; пішов читати стандарти, і от… як тепер із цим жити?
