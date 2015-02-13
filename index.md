---
layout: default
---

<h2>About me</h2>
I'm Adam Liska, a graduate student at the [MRI
Lab](http://cncs.iit.it/research-labs/mri.html) of Center for Neuroscience and
Cognitive Systems (Istituto Italiano di Tecnologia) in 
Rovereto, Italy 
([map](https://www.google.com/maps/place/Roveret://www.google.com/maps/place/38068+Rovereto+TN,+It%C3%A1lie/@47.2603133,11.7074777,5z/data=!4m2!3m1!1s0x47820ec143127041:0x6a9664123aebfadf)).

<h2>My research</h2>
I'm currently involved in implementing structural and functional 
Magnetic Resonance Imaging ([MRI](http://en.wikipedia.org/wiki/Magnetic_resonance_imaging)) 
methods to study the mouse brain. While various
MRI methods have been extensively applied to investigate the human brain and its 
organization in healthy and disease states, extending this approach 
to animal models opens new research avenues, as it offers the possibility of 
finding functional and structural 
correlates of genetic, drug related and optogenetic manipulations. 

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