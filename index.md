---
title: 온라인 호스팅 지침
permalink: index.html
layout: home
---

# 데이터 분석가 연습

이 연습은 Microsoft 과정 [DP-500: Designing and Implementing Enterprise-Scale Analytics Solutions Using Microsoft Azure and Microsoft Power BI](https://docs.microsoft.com/training/courses/dp-500t00)을 지원합니다.

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/labs'" %}
| ILT 모듈 | 랩 |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

<!--

## Demos

{% assign demos = site.pages | where_exp:"page", "page.url contains '/Instructions/Demos'" %}
| Module | Demo |
| --- | --- | 
{% for activity in demos  %}| {{ activity.demo.module }} | [{{ activity.demo.title }}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}
 
-->
