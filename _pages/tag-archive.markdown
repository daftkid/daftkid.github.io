---
title: "Posts by Tag"
permalink: /tags/
---

All posts, grouped by tags.

{% assign tags = site.tags | sort %}

{% for tag in tags %}
<h3>#{{ tag[0] }}</h3>

{% for post in tag[1] %}
- [{{ post.title }}]({{ post.url | relative_url }})
  {% endfor %}
  {% endfor %}

{% assign tags =  site.prs | map: 'tags' | uniq %}
{% for tag in tags %}
<h3>#{{ tag }}</h3>
  {% for pr in site.prs %}
    {% if pr.tags contains tag %}
- [{{ pr.title }}]({{ site.baseurl }}{{ pr.url }})
    {% endif %}
  {% endfor %}
{% endfor %}