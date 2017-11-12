---
title: "тимчасова тека"
date: 2017-11-12T15:40:34-05:00
draft: true

featuredImage: ""
categories: [комп'ютер]
tags: [комп'ютер,bash,linux]
author: "tivasyk"
---
помічний рядок для bash'ика:

    TEMP_DIR=$(mktemp -d) && <корисний вантаж> && rm -rf ${TEMP_DIR} && TEMP_DIR=""

банальщина, звісно, але часом забуваю деталі, тож щоби не перевинаходити ровер щоразу… 
