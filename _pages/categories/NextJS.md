---
title: 'Next js'
layout: archive
permalink: categories/nextjs
author_profile: true
sidebar_main: true
---

NextJS에 대해 공부하고 정리한 글을 올립니다.

{% assign posts = site.categories.nextjs %}
{% for post in posts %}
{% include archive-single.html type=page.entries_layout %}
{% endfor %}
