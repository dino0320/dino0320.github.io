---
layout: my-page
title: Categories
permalink: /categories/
---

{% assign delimiter = ":" %}

{% assign super_categories = "" %}
{% for post in site.posts %}

{% assign super_category = post.categories[0] %}

{% assign compared_super_category = delimiter | append: super_category | append: delimiter %}
{% if super_categories contains compared_super_category %}
{% continue %}
{% endif %}

{% assign sub_categories = "" %}
{% for post in site.posts %}

{% if post.categories[0] == super_category %}
{% assign sub_categories = sub_categories | append: "," | append: post.categories[1] %}
{% endif %}

{% endfor %}

{% assign sub_categories = sub_categories | remove_first: "," %}
{% assign sub_categories = sub_categories | split: "," | uniq %}

## {{ super_category }}

{% for sub_category in sub_categories %}

### {{ sub_category }}

{% for post in site.posts %}

{% if post.categories[1] == sub_category %}
[{{ post.title }}]({{ post.url }})
{% endif %}

{% endfor %}

{% endfor %}

{% assign super_categories = super_categories | append: delimiter | append: super_category | append: delimiter %}

{% endfor %}