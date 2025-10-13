some text

<h1>{{ page.title }}</h1>

{% for post in site.posts %}
- <a href="{{ post.url | relative_url }}">{{ post.title }}</a> — <small>{{ post.date | date: "%Y-%m-%d" }}</small>
{% endfor %}
