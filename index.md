---
layout: default
---

<h2>About me</h2>
<img src="images/IMG_0095.JPG" style="float: right; width: 30%" alt="picture of me">
I'm a research engineer at Spotify in London.

Previously, I was a graduate student at the [Functional Neuroimaging
Lab](https://www.iit.it/research/lines/functional-neuroimaging) of
Istituto Italiano di Tecnologia in Rovereto, Italy, working with [Alessandro Gozzi](https://www.iit.it/people/alessandro-gozzi).
I did my BSc in Computer Science at Charles University in Prague working on
machine translation with [Ondřej Bojar](http://www1.cuni.cz/~obo/),
and my MSc in Computer and Cognitive Science at
The University of Melbourne and University of Trento
working on computer vision, distributional semantics and
machine learning with [Marco Baroni](http://clic.cimec.unitn.it/marco/)
and [Elia Bruni](https://eliabruni.github.io/).
All my theses are [here](/publications/#theses).

<h2>Recent posts</h2>
<ul class="post-list">
    {% for post in site.posts limit:3 %}
        <li>
            <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>
            <h2>
                <a class="post-link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
            </h2>
            {{ post.excerpt }}
        </li>
    {% endfor %}
</ul>
