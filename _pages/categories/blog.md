---
title: '블로그'
layout: archive
permalink: categories/blog
author_profile: true
sidebar_main: true
---

블로그를 운영하면서 겪은 시행착오를 정리한 글을 올립니다.

{% assign posts = site.categories.blog %}
{% for post in posts %}
{% include archive-single.html type=page.entries_layout %}
{% endfor %}
