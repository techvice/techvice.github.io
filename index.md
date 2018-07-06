## Blog Posts

{% for post in site.posts %}
  <article class="{% if forloop.first %}first{% elsif forloop.last %}last{% else %}middle{% endif %}">
		<div class="article-head">
			<h3 class="title"><a href="{{ post.url }}" class="js-pjax">{{ post.title }}</a></h3>
			<p class="date">{{ post.date | date: "%b %d, %Y" }}</p>
		</div><!--/.article-head-->
		<div class="article-content">
		{{ post.long_description }}
		<a href="{{ post.url }}" class="full-post-link js-pjax">Read more</a>	
		</div><!--/.article-content-->
	</article>
	{% if forloop.last %}{% else %} <hr> {% endif %}
{% endfor %}