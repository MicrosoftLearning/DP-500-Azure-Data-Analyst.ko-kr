---
title: 온라인 호스팅 지침
permalink: index.html
layout: home
---

# <a name="data-analyst-exercises"></a>데이터 분석가 연습

DP-500: Microsoft Azure 및 Microsoft Power BI를 통한 기업 규모의 분석 솔루션 디자인 및 구현

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
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
