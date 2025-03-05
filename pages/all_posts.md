---
layout: landing
title: All Articles
nav-menu: true
description: You may access all of my published content on Medium through this page.
image: ../assets/images/all-articles.jpg
author: null
show_tile: true
---

<style>
.spotlights {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
}

.spotlights section {
    width: 100%;
    display: flex;
    margin-bottom: 2em;
    height: 600px;  /* Reduced height */
}

.spotlights section .image {
    width: 40%;
    height: 600px;  /* Matching section height */
    position: relative;
    overflow: hidden;
}

.spotlights section .image img {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
}

.spotlights section .content {
    width: 60%;
    padding: 2em;
    overflow-y: auto; /* Add scrollbar if content overflows */
}
</style>

<!-- Main -->
<div id="main">

<!-- Two -->
<section id="two" class="spotlights">
    {% for post in site.posts %}
    <section>
        <a href="{{ post.url | relative_url }}" class="image">
            <img src="{{ post.image }}" alt="" data-position="center center" />
        </a>
        <div class="content">
            <div class="inner">
                <header class="major">
                    <h3>{{ post.title }}</h3>
                </header>
                <p>{{ post.excerpt | strip_html | truncatewords: 50 }}</p>
                <p><em>{{ post.date | date: "%B %d, %Y" }}</em></p>
                <ul class="actions">
                    <li><a href="{{ post.url | relative_url }}" class="button">Read the Article</a></li>
                </ul>
            </div>
        </div>
    </section>
    {% endfor %}
</section>

</div>