
{% for tag in site.tags %}
  <h3>{{ tag[0] }}</h3>
  <ul>
    {% for post in tag[1] %}
      <li><a href="{{ site.baseurl }}{{ post.url }}">{{ post.date | date: "%B %Y" }} - {{ post.title }}</a></li>
    {% endfor %}
  </ul>
{% endfor %}

<!-- 1. **Project write-up:** _<ins><a href="https://reveng007.github.io/blog/reveng_rtkit_Detailed_README.md" target="_blank">How did I approach making reveng_rtkit?</a></ins>_ -->
