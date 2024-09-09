**Django-Taggit** – это приложение Django, которое позволяет легко добавлять теги к моделям. Работает с Django 4.1+ и Python 3.8+.

**Установка:**

1. Установите приложение с помощью `pip install django-taggit`.
2. Добавьте `taggit` в список `INSTALLED_APPS` вашего проекта Django.
3. Запустите миграции - `python manage.py migrate`.

> [!important]  
> Если вы хотите, чтобы django-taggit был не чувствительным к регистру при поиске существующих тегов, вам нужно установить TAGGIT_CASE_INSENSITIVE (в settings.py или там, где у вас находятся настройки Django) в True (по умолчанию False):TAGGIT_CASE_INSENSITIVE = True  

**Использование:**

1. **Добавьте** `**TaggableManager**` **к любой модели, для которой вы хотите добавить теги:**
    
    ```Shell
    from django.db import models
    from taggit.managers import TaggableManager
    
    class MyModel(models.Model):
        tags = TaggableManager()
        # ...
    ```
    
2. **Добавьте теги к модели, используя методы add или set:**
    
    ```Shell
    model = MyModel.objects.create(...)
    model.tags.add("tag1", "tag2")
    ```
    
3. **Доступ к тегам:**
    
    ```Shell
    tags = model.tags.all()
    ```
    
4. **Дополнительные методы:**
    - **similar_objects()**
        - Возвращает список (не QuerySet) объектов, имеющих схожие теги с текущим объектом. Объекты упорядочены по степени сходства, наиболее схожие – первыми. Каждому объекту присваивается атрибут `similar_tags`, который указывает на количество общих тегов.
        - **Важные моменты:**
            - Если модель использует стандартный механизм тегирования Django, этот метод ищет похожие объекты среди всех классов.
            - Если модель привязана к собственной таблице тегов, то поиск будет ограничен экземплярами этой же модели.
    - **names()**
        - Возвращает `ValuesListQuerySet` (по сути, просто итерируемый набор данных) с именами всех тегов в виде строк.
        - **Пример:**
            
            ```Shell
            apple.tags.names()
            ["green and juicy", "red"]
            ```
            
    - **slugs()**
        - Аналогично `names()`, но возвращает `ValuesListQuerySet` со слагами (slug) каждого тега в виде строк.
        - **Пример:**
            
            ```Shell
            apple.tags.slugs()
            ["green-and-juicy", "red"]
            ```
            
5. Использование в фильтрации:
    
    - Используя менеджеры объектов в Django можно также фильтровать объекты по тегам. Чтобы избежать дублирования в выводе, не забудьте добавить - `distinct()`.
    
    ```Shell
    Food.objects.filter(tags__name__in=["delicious"])
    [<Food: apple>, <Food: pear>, <Food: plum>]
    ```
    

**Администрирование:**

Django-Taggit автоматически добавляет поля тегов в интерфейс администратора Django.

**Правила разбора тегов, введённых через админку:**

|   |   |   |
|---|---|---|
|Пример строки ввода|Результирующие теги|Заметки|
|apple ball cat|`["apple", "ball", "cat"]`|Нет запятых, поэтому пробел воспринимается, как разделитель|
|apple, ball cat|`["apple", "ball cat"]`|Запятая присутствует, поэтому она воспринимается как разделитель|
|“apple, ball” cat dog|`["apple, ball", "cat", "dog"]`|Все запятые находятся в кавычках, разделитель - пробел|
|“apple, ball”, cat dog|`["apple, ball", "cat dog"]`|Содержит запятые вне кавычек, разделитель - запятая|
|apple “ball cat” dog|`["apple", "ball cat", "dog"]`|Нет запятых, но есть кавычки, делит по пробелам за кавычками|
|“apple” “ball dog|`["apple", "ball", "dog"]`|Не закрытые кавычки будут проигнорированы, разделитель пробел|

Основная [документация](https://django-taggit.readthedocs.io/en/latest/index.html).

<div class="page-break" style="page-break-before: always;"></div>
