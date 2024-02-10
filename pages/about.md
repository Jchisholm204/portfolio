---
layout: page
title: About
permalink: /about/
weight: 3
---

# **About Me**

Hi I am **{{ site.author.name }}** :wave:,<br>
Hey there! Welcome to my portfolio. My name is Jacob Chisholm. I graduated from Rockridge Secondary School in 2022 and I'm currently attending Queen's University for Computer and Electrical Engineering in the ECE innovation program (ECEi).

<div class="row">
{% include about/skills.html title="Programming Skills" source=site.data.programming-skills %}
{% include about/skills.html title="Other Skills" source=site.data.other-skills %}
</div>

<div class="row">
{% include about/timeline.html %}
</div>