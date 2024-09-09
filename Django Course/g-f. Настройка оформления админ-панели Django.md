Для того, чтобы настроить оформление админ-панели под стиль своего веб-сайта, необходимо изменить стили, используемые шаблонами администратора Django. Это можно сделать, создав новые шаблоны в своем проекте и переопределив стандартные шаблоны Django.

1. **Определение используемых шаблонов**
    
    С помощью Debug Toolbar можно определить, какие шаблоны используются для формирования страниц админ-панели. В данном случае, основными шаблонами являются:
    
    - `admin/index.html`
    - `admin/base_site.html`
    - `admin/base.html`
    - `admin/app_list.html`
    
    Базовым шаблоном для страниц админ-панели является `base_site.html`.
    
2. **Переопределение базового шаблона**
    
    Вместо изменения стандартных шаблонов Django, рекомендуется создать новый шаблон `base_site.html` в каталоге `sitewomen/templates/admin/` вашего проекта. Не забываем, что путь до коневой папки, где будут храниться шаблоны должен быть прописан в Templates → DIRS.
    
3. **Структура базового шаблона**
    
    Шаблон `base_site.html` содержит следующие блоки:
    
    - `{% block title %}` - отвечает за отображение заголовка во вкладке браузера.
    - `{% block branding %}` - отвечает за отображение ссылки "Администрирование Django" в верхней панели.
    - `{% block nav-global %}` - глобальный блок навигации.
    
    ```HTML
    {% extends "admin/base.html" %}
    
    {% block title %}{% if subtitle %}{{ subtitle }} | {% endif %}{{ title }} | {{ site_title|default:_('Django site admin') }}{% endblock %}
    
    {% block branding %}
    <h1 id="site-name"><a href="{% url 'admin:index' %}">{{ site_header|default:_('Django administration') }}</a></h1>
    {% if user.is_anonymous %}
      {% include "admin/color_theme_toggle.html" %}
    {% endif %}
    {% endblock %}
    
    {% block nav-global %}{% endblock %}
    ```
    
4. **Добавление дополнительных стилей**
    
    Для добавления дополнительных стилей оформления, необходимо использовать блок `{% block extrastyle %}` в шаблоне `base.html`. В вашем `base_site.html` добавьте этот блок со ссылкой на файл стилей
    
    ```HTML
    {% load static %}
    {% block extrastyle %}
    <link rel="stylesheet" href="{% static 'css/admin/admin.css' %}">
    {% endblock %}
    ```
    
5. **Создание файла стилей**
    
    Создайте файл `admin.css` в каталоге `static/css/admin/` вашего проекта. В этом файле можно определить новые стили для элементов админ-панели.
    
6. **Указание пути к статическим файлам**
    
    Для того, чтобы Django мог найти путь к папке `static/`, добавьте следующую настройку в `settings.py`
    
    ```Python
    STATICFILES_DIRS = [
        BASE_DIR / 'static',
    ]
    ```
    
7. **Изменение стилей**
    
    Для изменения стилей элементов админ-панели, используйте CSS-селекторы, соответствующие HTML-структуре этих элементов. Например, для изменения цвета фона шапки и заголовков блоков, можно использовать следующие стили:
    
    ```CSS
    \#header, .module caption {
        background: \#3F4137;
    }
    
    div.breadcrumbs {
        background: \#6A6E5D;
    }
    
    .module h2, .module caption, .inline-group h2 {
        background: \#6A6E5D;
    }
    ```
    
    Подсказкой для определения нужных селекторов может служить инспектирование HTML-кода страницы админ-панели в браузере.

<div class="page-break" style="page-break-before: always;"></div>
