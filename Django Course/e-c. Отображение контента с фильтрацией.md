Ключевая идея заключается в том, чтобы передавать в представление не все посты - `Post.published.all()`, а отфильтрованные в зависимости от категории, в которой мы находимся - `Post.published.filter(cat_id=cat_slug)`

**Пошагово**:

1. Изменяем url-маршрут для категории, добавляем слаги.
    
    ```Python
    # urls.py
    
    urlpatterns = [
        ...,
        path('category/<slug:cat_slug>/', views.show_cats, name='category'),
    ]
    ```
    
2. Изменяем представление для отображения категорий:
    
    - указываем, что функция принимает `cat_slug`;
    - получаем объект категории, используя функцию `get_object_or_404()`;
    - формируем коллекцию постов, используя фильтр по категории `filter(cat_id=category.pk)`;
    - изменяем заголовок - `‘title’: category.name`;
    - передаём в контекст коллекцию постов, вместо data_db;
    - передаём cat_id, в качестве текущей (выбранной) категории.
    
    ```Python
    # views.py
    
    def show_cats(request, cat_slug):
        category = get_object_or_404(Category, slug=cat_slug)
    
    		# я вынесла получение коллекции постов в глобальную переменную для удобства
        cat_posts = POSTS.filter(cat_id=category.pk)
    
        data = {
            'title': category.name,
            'posts': cat_posts,
            'cat_id': category.pk,
        }
    
        return render(request, 'basefunc/index.html', data)
    ```
    
3. Настраиваем пользовательский тег, чтобы он получал категории из БД - `cats = Category.objects.all()`.
    
    ```Python
    # custom_tags.py
    
    @register.inclusion_tag('basefunc/cats_menu.html', takes_context=True)
    def cats_menu(context, cat_id):
        cats = Category.objects.all()
        return {'cats': cats, 'cat_id': cat_id, 'title': context['title']}
    ```
    
4. Правим шаблон для отображения меню категорий, добавляя в него метод `get_absolute_url()`.
    
    ```HTML
    # cats_menu.html
    
    {% if title == 'Главная страница' %}
        <li class="selected">Все категории</li>
    {% else %}
        <li><a href="{% url 'home' %}">Все категории</a></li>
    {% endif %}
    
    {% for cat in cats %}
        {% if cat.id == cat_id %}
            <li class="selected">{{ cat.name }}</li>
        {% else %}
            <li><a href="{{ cat.get_absolute_url }}">{{ cat.name }}</a></li>
        {% endif %}
    {% endfor %}
    
    ```
    
    P.S. У меня в этот шаблон также включена ссылка на все категории, потому что думается мне, что она должна быть именно здесь, а не в базовом шаблоне)
    
5. В модель добавляем метод для формирования url.
    
    ```Python
    # models.py
    
    class Category(models.Model):
    
        name = models.CharField(max_length=100, db_index=True)
        slug = models.SlugField(max_length=100, unique=True)
    
        objects = models.Manager()
    
        def __str__(self):
            return self.name
    
        def get_absolute_url(self):
            return reverse('category', kwargs={'cat_slug': self.slug})
    ```
    

Done! Well done!)

  

**Добавление в отображение постов информации о категориях и дате обновления** задаётся в несколько строк в шаблоне `index.html`.

```HTML
{% extends 'base.html' %}

{% block content %}
<ul class="list-articles">
	{% for p in posts %}
			<li>
			<div class="article-panel">
         <p class="first">Категория: {{p.cat}}</p>
         <p class="last">Дата: {{p.time_update|date:"d-m-Y H:i:s"}}</p>
			</div>
			<h2>{{p.title}}</h2>
				{% autoescape off %}
				{{p.content|linebreaks|truncatechars:300}}
				{% endautoescape %}
				<div class="clear"></div>
			</li>
	{% endfor %}
</ul>
{% endblock %}
```

<div class="page-break" style="page-break-before: always;"></div>
