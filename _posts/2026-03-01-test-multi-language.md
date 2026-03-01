---
title: Multi language
date: 2026-03-01 00:00:00
categories: []
tags: []
---

{% assign s = site.data[site.active_lang].strings %}

{{ s.about.title }}


{% assign s1 = site.data[site.active_lang].strings.about %}

{{ s1.intro }}