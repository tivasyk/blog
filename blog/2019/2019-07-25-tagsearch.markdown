---
layout: post
title:  "пошук міток у рядку"
date:   2019-07-25 23:15
categories: blog
tags: [bash, linux, комп'ютери]
---

[простий код](https://pastebin.com/LExq2vFx) (bash 3+, якщо не помиляюся) для пошуку міток виду <TAG_1> в текстовому рядку; знадобиться для спрощення коду розбору шаблонів:

```
#!/usr/bin/env bash
# tivasyk <tivasyk@gmail.com>
# Launch to test: ./test "<TAG_1>Test<TAG_2>"
 
TEST="$@"
printf "Received line: «${TEST}»\n"
 
while [[ ${TEST} =~ "<"[A-Z_0-9]*">" ]]; do
    TAG="<${TEST#*<}"
    TAG="${TAG%%>*}>"
    printf "Found a tag: ${TAG}, deleting...\n"
    TEST="${TEST/${TAG}/}"
done
 
printf "Result without tags: «${TEST}»\n"
```

навіщо це? чорновий код просто перебирає всі відомі йому мітки, і для кожної перевіряє, чи є вона в шаблоні: якщо є — заміняє відповідним значенням, якщо нема — перепрошую, але час на пошук згаяно.

це неоптимально. як треба: витягати першу мітку з шаблона, шукати для неї значення й підставляти, повторити поки ще є мітки в шаблоні.