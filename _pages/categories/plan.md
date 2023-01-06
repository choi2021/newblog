---
title: '기획'
layout: archive
permalink: categories/plan
author_profile: true
sidebar_main: true
---

만들고 싶은 프로젝트들에 대해 정리합니다.

{% assign posts = site.categories.plan %}
{% for post in posts %}
{% include archive-single.html type=page.entries_layout %}
{% endfor %}
