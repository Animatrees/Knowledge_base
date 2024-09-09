**Процедура добавления тегов.**

1. Для начала отобразим наши теги на главной странице. Для этого, добавим включающий тег (по аналогии, как мы делали для категорий). Обратите внимание, что у меня модель с тегами называется - `Tags`, а не `TagPost`.
    
    ```Python
    # custom_tags.py
    from django import template
    from basefunc.models import Tags
    
    
    register = template.Library()
    
    @register.inclusion_tag('basefunc/tags_menu.html')
    def tags_menu():
        tags = Tags.objects.all()
        return {'tags': tags}
    ```
    
2. Затем создадим шаблон для отображения этого меню в html виде. Обратите внимание, что для названия тега я использую атрибут - `title`, а не tag. Также я добавила проверку для тега, чтобы он выводился в меню только если за ним кто-то закреплён - `{% if t.women.all %}`. Здесь важно, что `related_name` для tag - `women` и, соответственно, используя это название, я получаю список всех женщин, которые прикреплены к выбранному тегу.
    
    ```HTML
    <!-- basefunc/tags_menu.html -->
    
    {% if tags %}
        Теги:</p>
        <ul class="tags-list">
            {% for t in tags %}
            {% if t.women.all %}
            <li><a href="{{t.get_absolute_url}}">{{t.title}}</a></li>
            {% endif %}
            {% endfor %}
        </ul>
    {% endif %}
    ```
    
3. Так как у нашей модели ещё нет метода `get_absolute_url`, а мы используем его в шаблоне меню для формирования ссылки на страницу тега, нужно его добавить в модель Tags.
    
    ```Python
    # models.py
    
    class Tags(models.Model):
        title = models.CharField(max_length=100, db_index=True)
        slug = models.SlugField(max_length=100, unique=True)
    
        objects = models.Manager()
    
        def __str__(self):
            return self.title
    
        def get_absolute_url(self):
            return reverse('tag', kwargs={'tag_slug': self.slug})
    ```
    
4. После этого можно добавить в базовый шаблон отображение меню тегов.
    
    ```HTML
    <!-- base.html -->
    
    <ul id="leftchapters">
    		{% cats_menu cat_id %}
    		<li>{% tags_menu %}</li>
    	</ul>
    ```
    
5. Далее необходимо добавить сам маршрут для отображения страницы с отфильтрованному по тегу контентом.
    
    ```Python
    # urls.py
    
    urlpatterns = 
    	[...
    	path('tag/<slug:tag_slug>/', views.show_tags, name='tag'),
    	...]
    ```
    
6. И определить для него функцию представления.
    
    1. Получаем объект текущего тега - get_object_or_404()
    2. Фильтруем объекты по теги. Я здесь использую фильтрацию через модель, так как у меня уже есть сохранённый в переменную `POSTS` список всех опубликованных постов о женщинах.
    3. Формируем словарь с контекстом для шаблона, который мы будем использовать в текущем представлении.
    4. Передаём контекст в функцию рендера.
    
    ```Python
    # views.py
    
    POSTS = Post.published.all()
    
    ...
    
    def show_tags(request, tag_slug):
        tag = get_object_or_404(Tags, slug=tag_slug)
        filtered_posts = POSTS.filter(tags=tag)
        data = {
            'title': f'Тег: {tag.title}',
            'posts': filtered_posts,
        }
    
        return render(request, 'basefunc/index.html', data)
    ```
    

  

**Добавление отображения тегов в саму статью.**

1. Модифицируем шаблон для постов - post.html, добавляя в него следующий html-код.

```HTML
<!-- post.html -->

{% extends 'base.html' %}

{% block breadcrumbs %}
<!-- Теги -->
    {% with post.tags.all as tags %}
        {% if tags %}
            <ul class="tags-list">
                <li>Теги:</li>
                {% for t in tags %}
                <li><a href="{{t.get_absolute_url}}">{{t.title}}</a></li>
                {% endfor %}
            </ul>
        {% endif %}
    {% endwith %}
{% endblock %}

{% block content %}
...
```

Тег `with` используется для сохранения результата вычисления выражения во временную переменную, которая затем может быть использована внутри блока шаблона.

Синтаксис тега:

```HTML
{% with <variable_name> = <expression> %}
    <!-- Тело блока -->
{% endwith %}
```

Ключевое слово `as` используется здесь, чтобы задать алиас для вычисленного значения. Хотя по сути строка может быть переписана в `{% with tags=post.tags.all %}` без потери функционала)

<div class="page-break" style="page-break-before: always;"></div>
