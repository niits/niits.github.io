---
layout: default
title: categories
permalink: /categories
---

<ul>
    {% for category in site.categories %}
        <li id="{{ category[0] }}">
            <b>{{ category[0] }}</b>
        </li>
        {% for post in category[1] %}
            {{ post.date | date:"%Y-%m-%d" }} &raquo; <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
            <br>
        {% endfor %}
        <br>
    {% endfor %}
</ul>