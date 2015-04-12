---
layout: page
title: Research blog
permalink: /research-blog/
inheader: true
---

Currently under reconstruction!

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
