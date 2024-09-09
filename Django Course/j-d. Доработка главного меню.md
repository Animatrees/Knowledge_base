Про контекстные процессоры уже писала ранее:

Доработала главное меню иначе, чем было показано в уроке, чтобы не смешивать логику отображения пунктов меню и их формирования.

```Python
def get_context_menu(request):
    menu_items = [{'title': "О сайте", 'url_name': 'about'},
                  {'title': "Добавить статью", 'url_name': 'add_page'},
                  {'title': "Обратная связь", 'url_name': 'contact'},
                  {'title': "Войти", 'url_name': 'users:login', 'title2': "Регистрация", 'url_name2': 'home'},
                  ]
    if request.user.is_authenticated:
        menu_items[-1] = {'title': request.user.username, 'title2': "Выйти", 'url_name2': 'users:logout'}

    return {'menu': menu_items}
```

```HTML
{% for m in menu %}
	{% if not forloop.last %}
		<li><a href="{% url m.url_name %}">{{ m.title }}</a></li>
	{% else %}
		<li class="last">
			{% if m.url_name %}<a href="{% url m.url_name %}">{{ m.title }}</a>
			{% else %}
			{{ m.title }}
			{% endif %} |
			<a href="{% url m.url_name2 %}">{{ m.title2 }}</a></li>
	{% endif %}
{% endfor %}
```

Как было реализовано в уроке:

1. Удаляем последний пункт из меню (там где логин).
2. Изменяем шаблон
    
    ```HTML
    {% for m in mainmenu %}
        <li><a href="{% url m.url_name %}">{{m.title}}</a></li>
    {% endfor %}
        {% if user.is_authenticated %}
    			<li class="last"> {{user.username}} | <a href="{% url 'users:logout' %}">Выйти</a></li>
    		{% else %}
    			<li class="last"><a href="{% url 'users:login' %}">Войти</a> | <a href="#">Регистрация</a></li>
    		{% endif %}       
    {% endblock mainmenu %}
    ```

<div class="page-break" style="page-break-before: always;"></div>
