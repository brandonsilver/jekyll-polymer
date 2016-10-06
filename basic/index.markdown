---
layout: default
title: Home
tags:
- brandon silver
- blog
---
## Contact Info ##
Email: <brandon@brandonsilver.com> (PGP Key ID: [8D5F182D](https://www.brandonsilver.com/gpg-key.asc))   
Twitter: [@alinuxuser](https://twitter.com/alinuxuser)  

## Blog ##
{% for post in site.posts %}
<!--iterate through each post's excerpt-->
<span class="post_date"> {{ post.date | date: "%d %B %Y" }} </span>
<h3><a href=" {{ post.url | replace_first: '/', '' }} "> {{ post.title }} </a></h3>
{{ post.content | postmorefilter: post.url, "&raquo;Read the rest of this post" }}

{% if forloop.index != forloop.length %}
<hr>
{% endif %}
{% endfor %}
