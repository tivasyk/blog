---
layout: post
title:  "перше правило"
date:   2019-08-02 21:50
categories: blog
tags: [docker, jekyll, підказка, поновлено]
---

постулат: щоби виправити щось, треба розуміти, що не так і як треба! можна, звісно, навмання тицяти в проблему пальцем, цікавий процес гарантовано, але результат — ні. отже, якщо контейнер з jekyll'ом перезавантажується в циклі й ані не генерує сайт, ані не віддає його по http/https — перш ніж колупати _config.yml чи gemfile, редагувати тему і в підсумку знести її к бісу, подивись же, дурню, журнал?!

```
docker logs --tail 50 jekyll
```

виявиться, що jekyll казиться просто через те, що в даті допису (!) помилка: 2019-08-025. мда, зайвий нулик. нормально написаний двигунець просто поігнорував би файл? але не jekyll, ніт…

**поновлено** (2019-08-03, 01:02). ну, але нема злого, щоб на добре не вийшло — через ту проблему поміняв стандартну тему minima на подібну, але зрілішу minima-reboot і трохи налаштував. changelog:

- [x] зробив просту [публікацію через owncloud]({% post_url 2019/2019-08-02-test-failed %});
- [x] поремонтував розмір зображень;
- [x] налаштував розбивання головного переліку на сторінки;
- [x] прикрутив коментарі disqus;
- [x] архів (повний перелік дописів по роках);
- [x] мітки (теми) і сторінка з переліком міток;
- [ ] пошук в щоденнику;
- [x] форматування підвалу, додати іконки соціалок;
- [ ] форматування заголовка, іконка rss замість тексту;
- [ ] далі буде…
