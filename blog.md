---
layout: page
title: Blog
permalink: /blog/
inheader: true
---

<ul class="post-list">
    {% for post in site.posts %}
        <li>
            <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>
            <h2>
                {{ post.title }}
            </h2>
            {{ post.excerpt }}
        </li>
    {% endfor %}
</ul>
