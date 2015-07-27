---
layout: page
title: All Posts
tagline:
---
{% include JB/setup %}
<!-- Posts list -->
<section class="posts">
  {% for post in site.posts %}
    <article>
      <header>
        <ul class="categories">
          {% for category in post.categories %}
          <li><a href="{{ BASE_PATH }}{{ site.JB.categories_path }}#{{ category }}-ref">{{ category }}</a></li>
          {% endfor %}
        </ul>
        <h1><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></h1>
        <span class="date">{{ post.date | date_to_string }}</span>
      </header>
      <section class="excerpt">
        {{ post.content | strip_html | truncatewords:50}}
      </section>
      <section class="tags">
        <h2>tags</h2>
        <ul>
          {% for tag in post.tags %}
            <li><a href="{{ BASE_PATH }}{{ site.JB.tags_path }}#{{ tag }}-ref">{{ tag }}</a></li>
          {% endfor %}
        </ul>
      </section>
    </article>
  {% endfor %}
</section>

<!-- Pagination links -->
{% if paginator.total_pages > 1 %}
<div class="pagination">
  {% if paginator.previous_page %}
    <a rel="prev" href="{{ paginator.previous_page_path | prepend: site.baseurl | replace: '//', '/' }}">&laquo; Prev</a>
  {% else %}
    <span>&laquo; Prev</span>
  {% endif %}

  {% for page in (1..paginator.total_pages) %}
    {% if page == paginator.page %}
      <em>{{ page }}</em>
    {% elsif page == 1 %}
      <a href="{{ '/archives' | prepend: site.baseurl | replace: '//', '/' }}">{{ page }}</a>
    {% else %}
      <a href="{{ site.paginate_path | prepend: site.baseurl | replace: '//', '/' | replace: ':num', page }}">{{ page }}</a>
    {% endif %}
  {% endfor %}

  {% if paginator.next_page %}
    <a rel="next" href="{{ paginator.next_page_path | prepend: site.baseurl | replace: '//', '/' }}">Next &raquo;</a>
  {% else %}
    <span>Next &raquo;</span>
  {% endif %}
</div>
{% endif %}