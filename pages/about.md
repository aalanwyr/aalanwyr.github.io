---
layout: page
title: About
description: 保持锻炼 早睡早起
keywords: aalanwyr
comments: true
menu: 关于
permalink: /about/
---

Intel Graphic Software Engineer

## Skill Keywords

{% for skill in site.data.skills %}
### {{ skill.name }}
<div class="btn-inline">
{% for keyword in skill.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
