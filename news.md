---
layout: page
title: News
permalink: /news
---

<ul>
    {% assign news_reversed = site.news | reverse %}
    {% for newsitem in news_reversed %}
        <li>
            {{ newsitem.date | date: "%b %Y" }}
            {{ newsitem.content }}
        </li>
    {% endfor %}
</ul>
